---
description: Nuxt data fetching explained in terms "when", "where" and "how"
date: 2025-11-05
media:
  opengraph: ./featured.png
  featured: ./featured.png
  thumbnail: ./thumb.png
---

# Data fetching in Nuxt

## Intro

One of the most common questions when building Nuxt applications is "how do I fetch data correctly?".

Due to how isomorphic frameworks like Nuxt render on both the server and client, "where" and "when" become equally important questions, so before diving into the tools themselves, we'll unpack Nuxt's render lifecycle to understand the particulars of how state must be serialized, transferred, and rehydrated across the server-client boundary.

Once we've covered that, we'll dive into [`$fetch`](#fetch-your-http-client), [`useAsyncData`](#useasyncdata-adding-ssr-awareness), and [`useFetch`](#usefetch-the-convenience-wrapper) (which will make much more sense as you'll understand how each mitigates isomorphic concerns) before linking to some common patterns and recipes.

> **Disclaimer**: as with most of my [Nuxt articles](/?s=nuxt) I wrote this to clarify my own understanding of Nuxt's JS voodoo!

<NavToc type="tree" exclude="intro" level="2,3" />

## Problem space

### Traditional vs Isomorphic rendering

#### Traditional applications

In traditional web applications, data fetching is straightforward because it happens in one place.
 with PHP or Ruby on Rails) fetch data on the server, render the HTML, then send it to the browser. The browser simply displays the HTML. This approach is simple and provides fast initial loads, but offers limited interactivity since new content requires rendering pages from scratch.

Traditional client-rendered applications (single-page apps) work differently. The browser loads skeleton HTML and JavaScript, then the application starts, fetches data, and renders the application. This provides rich interactivity and smooth navigation between pages, but results in slower initial loads and poor SEO since search engines see empty pages.

#### Isomorphic applications

Nuxt uses **universal rendering** (aka [isomorphic](https://stackoverflow.com/questions/48008502/what-is-an-isomorphic-application) rendering) by default, which combines the best of both approaches.

To do this, the same Vue files run on both the server and the client. On any given page load:

- the server fetches any data and renders the page, providing a fast initial HTML load and good SEO
- the client re-renders the same page, and "hydrates" the DOM tree, turning the static HTML into a live application

This creates a complex problem: **when and where should data fetching happen?** 

The same component that runs on the server during initial load also runs on the client during hydration _and_ subsequent navigation. Nuxt must determine in which environment and where in the lifecycle any executing code is, and whether or not any data requests have yet to be fulfilled, and how to coordinate this between server and client.

### The "double-fetch" problem

#### The vanilla (non-Nuxt) way

Let's walk through what happens when a user visits your blog's posts page for the first time.

Your site is using universal rendering, so whilst the following code looks reasonable, it actually runs _twice_:

```vue
<script setup>
// ⚠️ runs once on server, then again on the client
const data = await fetch('/api/posts').then(r => r.json())
</script>

<template>
  <div v-for="post in data" :key="post.id">
    {{ post.title }}
  </div>
</template>
```

**What actually happens:**

1. **On the server:** Your component code runs. It fetches `/api/posts`, gets the data, and renders the HTML with all your post titles.
2. **The browser receives** fully-rendered HTML with all the posts visible immediately. Great user experience so far!
3. **Hydration begins:** Vue takes over on the client to make the page interactive. To do this, it needs to run your component code again.
4. **The same fetch runs again:** Your component executes `await fetch('/api/posts')` a second time, even though the data is already rendered in the HTML the user is looking at.

You've now made two identical requests for the same data. This is known as the "double fetch problem."

**Why this is bad:**
- Wasted network requests and server resources
- Slower page interactivity (waiting for the second fetch to complete)
- Risk of hydration mismatches if the data changed between requests
- Doubled load on your API

#### The idiomatic (Nuxt) way

For our posts page, we can use [`useFetch`](#usefetch---the-convenience-wrapper) instead of plain `fetch`:

```vue
<script setup>
// ✅ This fetches once on server, transfers to client
const { data: posts } = await useFetch('/api/posts')
</script>

<template>
  <div v-for="post in posts" :key="post.id">
    {{ post.title }}
  </div>
</template>
```

On the initial page load, the data is:

- fetched once on the server
- transferred to the client via a **payload** embedded in the HTML
- hydration **skips the refetch** because the data is already there

On subsequent visits to the page, it fetches *client*-side, blocking navigation until the data is ready.

### Isometric permutations

Now you understand a little of the isometric orchestration Nuxt needs to coordinate, let's summarise some of the scenarios Nuxt's data fetching functionality transparently handles.

#### Initial page load (server rendering)

When a user visits `https://yoursite.com/posts` directly in their browser, the server needs to run your component code, fetch the data, and render complete HTML. The browser receives this fully-rendered HTML and displays it immediately. Then, Vue takes over on the client side to make the page interactive (this is called "hydration"). The challenge: the client needs access to the same data that was used to render the HTML on the server, but without making a duplicate request.

#### Transferring data from server to client

When data is fetched on the server during initial load, Nuxt must transfer that data to the client. This happens by embedding the data in the HTML as a JavaScript object (the "payload"). The data must be serialized (converted to a format that can be embedded) and then deserialized on the client. Complex JavaScript types like `Date`, `Map`, and `Set` need special handling because they don't serialize to JSON naturally.

You can inspect this payload using [Nuxt DevTools](https://devtools.nuxt.com/) (Payload tab) to see exactly what data is being transferred.

#### Subsequent navigation (client-side routing)

Once the site has loaded and hydrated, when the user clicks a link to `/posts/123`, the client is now in charge of rendering the page. The component runs entirely in the browser, fetches its data directly, and renders. This is much simpler than the initial load because everything happens in one place.

#### Other considerations and optimisations

**Direct API route calls on server:** When `useFetch` calls an internal API route during SSR, Nuxt bypasses the HTTP layer entirely and calls your API handler function directly. This eliminates network overhead and improves performance compared to making actual HTTP requests from server to server.

**Request deduplication:** If multiple components request the same data simultaneously (using the same key), Nuxt deduplicates these into a single request. The first request executes while subsequent ones wait for and share the result, preventing unnecessary load on your API.

**Cache invalidation coordination:** When you call `refresh()` on cached data, Nuxt intelligently invalidates related cache entries and coordinates updates across all components using that data, maintaining consistency without manual cache management.

**Supporting different rendering strategies:** Nuxt's data fetching tools must also work across different rendering strategies (universal, client-only, static generation, hybrid), each with varying complexity requirements. The beauty of the system is that you write the same code regardless of strategy - the composables detect the environment and handle the differences transparently.

## Nuxt's isomorphic-aware tooling

Nuxt offers three core tools to fetch data in the right place and at the right time.

Firstly, a core fetch wrapper to load and retrieve data:

- [`$fetch`](#fetch---your-http-client) is a basic HTTP client that works anywhere (browser, server, API routes) with no double-fetch protection

Next, related tools designed to work across the server/client boundary, and respecting route navigation:

- [`useAsyncData`](#useasyncdata---adding-ssr-awareness) wraps any async function to make it SSR-aware, coordinating state between server and client
- [`useFetch`](#usefetch---the-convenience-wrapper) wraps both `useAsyncData` and `$fetch` together for the common case of fetching from a URL

These three tools work together to provide a powerful and flexible data fetching system.

### `$fetch` - your HTTP client

`$fetch` is Nuxt's built-in HTTP client (powered by the `ofetch` library). It's a better `fetch()` with automatic JSON parsing, error handling, and base URL resolution.

**When to use it:**

- Client-side only operations (event handlers, user interactions)
- Inside API routes on the server
- When you explicitly don't need SSR behavior

**Where it works:**
- Anywhere: components, composables, API routes, middleware

**What it does:**
- Makes an HTTP request
- Returns a promise with the response
- That's it - no SSR magic, no state management

**Basic usage:**
```vue
<script setup>
// This runs only when the user clicks the button (client-side only)
async function handleFormSubmit() {
  const res = await $fetch('/api/submit', {
    method: 'POST',
    body: { /* form data */ }
  })
}
</script>
```

In the example above, `$fetch` is used inside an event handler that only runs on user interaction (client-side only), so the double-fetch problem doesn't apply.

### `useAsyncData` - adds SSR awareness

`useAsyncData` wraps **any async operation** and makes it SSR-aware. It solves the double-fetch problem by:

1. Executing your function on the server during SSR
2. Serializing the result and transferring it to the client (via the payload)
3. Skipping execution on the client during hydration (it already has the data)

**When to use it:**

- You need to wrap custom async logic (not just URL fetching)
- You're using a third-party library with its own query layer (like a CMS SDK)
- You want fine-grained control over caching and execution

**What it returns:**

Reactive state objects:

- `data`: the result (a Vue ref)
- `status`: loading state (`'idle'`, `'pending'`, `'success'`, `'error'`)
- `error`: error object if the fetch failed
- `refresh`: function to manually refetch
- `clear`: function to reset the data

**Basic usage:**

```vue
<script setup>
// The second argument is any async function
const { data, status, error } = await useAsyncData('my-data', async () => {
  return await $fetch('/api/posts')
})
</script>
```

**The key parameter:**

The first argument (`'my-data'`) is a unique key used to:
- Cache the result
- Deduplicate requests across components
- Enable manual cache invalidation

You can omit the key and it will auto-generate one, but explicit keys are recommended for consistency:

```vue
<script setup>
// Auto-generated key (based on file + line number)
const { data } = await useAsyncData(async () => {
  return await $fetch('/api/posts')
})
</script>
```

**Wrapping multiple operations:**

Since `useAsyncData` accepts any async function, you can combine multiple operations:

```vue
<script setup>
const { data: deals } = await useAsyncData('deals', async () => {
  const [coupons, offers] = await Promise.all([
    $fetch('/api/coupons'),
    $fetch('/api/offers')
  ])
  
  return { coupons, offers }
})

// Access via: deals.value.coupons, deals.value.offers
</script>
```

**Important:** `useAsyncData` is for fetching and caching data, not triggering side effects. Don't use it to call Pinia actions or mutations, as this can cause unintended repeated executions. For one-time side effects, use the `callOnce` utility instead:

```vue
<script setup>
// ❌ Don't do this
await useAsyncData(() => offersStore.getOffer(route.params.slug))

// ✅ Do this instead
await callOnce(() => offersStore.getOffer(route.params.slug))
</script>
```

### `useFetch` - the convenience wrapper

`useFetch` combines `useAsyncData` and `$fetch` for the most common use case: fetching from a URL. Under the hood, it's just:

```js
function useFetch(url, options) {
  return useAsyncData(url, () => $fetch(url, options))
}
```

**When to use it:**
- You're fetching from a URL (internal API or external)
- You want the simplest, most common pattern
- You need SSR behavior

**Most common usage:**

```vue
<script setup>
const { data: posts } = await useFetch('/api/posts')
</script>

<template>
  <div v-for="post in posts" :key="post.id">
    {{ post.title }}
  </div>
</template>
```

This is equivalent to:

```vue
<script setup>
const { data: posts } = await useAsyncData('/api/posts', () => {
  return $fetch('/api/posts')
})
</script>
```

**Important:** `useFetch` and `useAsyncData` must be called in the setup function or directly at the top level of lifecycle hooks. They cannot be called conditionally or inside regular functions. If you need to fetch data based on user interaction, use `$fetch` inside event handlers instead.


## General usage

### Where to fetch data

The tool you choose depends on where your code runs.

#### In components and pages

Components and pages execute on both server (SSR) and client (hydration). Use `useFetch` or `useAsyncData` to avoid double fetching.

For initial data that should be fetched during SSR:

```vue
<script setup>
// Fetches on server, transfers to client
const { data: user } = await useFetch('/api/user')
</script>
```

For user-triggered actions that only happen client-side, use `$fetch` in event handlers:

```vue
<script setup>
async function saveProfile() {
  // Only runs on user click (client-side)
  await $fetch('/api/profile', {
    method: 'PUT',
    body: { /* user data */ }
  })
}
</script>

<template>
  <button @click="saveProfile">Save</button>
</template>
```

#### In API routes

API routes only run on the server, never on the client. There's no SSR hydration happening here, so you don't need `useAsyncData` or `useFetch`.

Use `$fetch` to call external APIs, or just call your logic directly:

```js
// server/api/posts.js
export default defineEventHandler(async (event) => {
  // Direct database call
  const posts = await db.query('SELECT * FROM posts')
  
  // Or fetch from external API
  const external = await $fetch('https://api.example.com/data')
  
  return posts
})
```

**Don't use `useFetch` in API routes** - it's designed for components that run on both server and client.

### The data flow

**During SSR:**

1. Component calls `useFetch('/api/posts')`
2. Nuxt calls the API route handler internally
3. API route fetches from database/external API (using `$fetch`)
4. Data flows back to component
5. Component renders with data
6. HTML + serialized data sent to browser
7. Browser hydrates with existing data (no refetch)

**During client-side navigation:**

1. User clicks a link
2. Component calls `useFetch('/api/posts')`
3. HTTP request goes from browser to your API route
4. The navigation transition pauses (unless using lazy options) whilst the data loads
5. Data flows back and component renders / page loads


### Lazy variants

By default, `useFetch` and `useAsyncData` block navigation until the data loads - waiting for the fetch to complete on the server, or blocking route transitions during client-side navigation. Sometimes you want to show the page immediately and load data progressively.

The lazy variants (`useLazyFetch` and `useLazyAsyncData`) don't block in either context - on the server, they render immediately without waiting; during client navigation, they allow the route transition to complete. They begin the fetch in parallel and return with `status: 'pending'`, letting you handle the loading state manually.

When data is not critical for initial render (below-the-fold content, secondary data):

```vue
<script setup>
const { status, data: comments } = useLazyFetch('/api/comments')
</script>

<template>
  <div v-if="status === 'pending'">
    Loading comments...
  </div>
  <div v-else>
    <Comment v-for="comment in comments" :key="comment.id" :comment="comment" />
  </div>
</template>
```

These are just shortcuts for:

```vue
<script setup>
// These are equivalent
const { data } = useLazyFetch('/api/data')
const { data } = useFetch('/api/data', { lazy: true })
</script>
```


## Reference

### Common options

```ts
useFetch('/api/data', {
  // Execution control
  lazy: false,           // If true, don't block navigation
  server: true,          // If false, only fetch on client
  immediate: true,       // If false, don't fetch until execute() is called
  
  // Caching and refetching
  key: 'my-key',         // Custom cache key
  watch: [someRef],      // Refetch when these values change
  dedupe: 'defer',       // 'defer' (reuse pending) or 'cancel' (cancel previous)
  getCachedData: (key) => { /* custom cache logic */ },
  
  // Data transformation
  transform: (data) => data,  // Process data before caching
  pick: ['id', 'title'],      // Only include these fields
  default: () => [],          // Default value before data loads
  deep: true,                 // Deep reactivity (true) or shallow (false)
  
  // Request options (passed to $fetch)
  method: 'GET',
  query: { page: 1 },
  body: { /* data */ },
  headers: { /* headers */ }
})
```

### Return values and state

Both `useFetch` and `useAsyncData` return the same reactive state object:

```vue
<script setup>
const { 
  data,      // Ref containing the fetched data
  status,    // Ref: 'idle' | 'pending' | 'success' | 'error'
  error,     // Ref containing error if fetch failed
  refresh,   // Function to manually refetch
  execute,   // Alias for refresh (more semantic)
  clear      // Function to reset data to undefined
} = await useFetch('/api/posts')
</script>
```

**Important:** These are Vue refs, so access them with `.value` in JavaScript:

```vue
<script setup>
const { data: posts } = await useFetch('/api/posts')

// In script, use .value
console.log(posts.value)

// In template, .value is automatic
</script>

<template>
  <div>{{ posts }}</div>
</template>
```

**Status values:**
- `'idle'`: Fetch hasn't started (only with `immediate: false`)
- `'pending'`: Currently fetching
- `'success'`: Fetch completed successfully
- `'error'`: Fetch failed

## Conclusion

### Summary

General operation:

- Understand that Nuxt runs code on both server and client
- Let the composables handle SSR for you - they prevent double fetching
- Use keys strategically to control caching and data sharing
- Handle loading states when using `lazy` or `server: false`

Which tool to use:

- Use `useFetch` for most data fetching in pages and components
- Use `useAsyncData` when you need to wrap custom async logic
- Use `$fetch` in client-side event handlers and API routes

### Cookbook

For further reading I've reorganised Nuxt's own [data fetching docs](https://nuxt.com/docs/4.x/getting-started/data-fetching) examples into a cookbook:

- [Nuxt Data Fetching Cookbook](./cookbook/)
