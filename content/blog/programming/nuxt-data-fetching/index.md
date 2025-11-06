---
description: Nuxt data fetching explained within the context of Nuxt's SSR lifecycle
tags:
  - nuxt
  - vue
  - data
date: 2025-11-05
media:
  opengraph: ./featured.png
  featured: ./featured.png
  thumbnail: ./thumb.png
---

# Data fetching in Nuxt

## Intro

One of the most common questions when building Nuxt applications is "how do I fetch data correctly?". In isomorphic frameworks like Nuxt, code runs on both server and client, making asking "where" and "when" equally as important.

Before diving into the tools, we'll unpack Nuxt's render lifecycle to understand how state gets serialized, transferred, and rehydrated across the server-client boundary. With a full understanding of isomorphic rendering, we'll then dig into the individual [`$fetch`](#fetch-a-better-http-client), [`useAsyncData`](#useasyncdata-adding-ssr-awareness) and [`useFetch`](#usefetch-the-convenience-wrapper) helpers before covering general usage, reference and examples.

> **Disclaimer**: as with most of my [Nuxt articles](/?s=nuxt) I wrote this to clarify my own understanding of Nuxt's JS voodoo!

<NavToc type="tree" exclude="intro" level="2,3" />

## Problem space

### Traditional vs Isomorphic rendering

#### Traditional applications

In traditional web applications, data fetching is straightforward because it happens in one place.

Traditional server-side applications (such as PHP or Ruby on Rails) fetch data on the server, render the HTML, then send it to the browser. The browser simply displays the HTML. This approach is simple and provides fast initial loads, but offers limited interactivity since new content requires requesting and rendering new pages from scratch.

Traditional client-rendered applications (single-page apps) work differently. The browser loads skeleton HTML and JavaScript, then the application starts, fetches data, and renders the application. This provides rich interactivity and smooth navigation between pages, but results in slower initial loads and poor SEO since search engines see empty pages.

#### Isomorphic applications

Nuxt uses **universal rendering** (aka [isomorphic](https://stackoverflow.com/questions/48008502/what-is-an-isomorphic-application) rendering) by default, which combines the best of both approaches.

To do this, the same Vue component code runs in both server and the client environments.

On any given page load:

- The server fetches any data and renders the HTML, providing a fast initial load and good SEO
- The client re-renders the same page, and [hydrates](https://vuejs.org/guide/scaling-up/ssr#client-hydration) the DOM tree, turning the static page into a live application
- On subsequent page visits, the client fetches any data and re-renders the page, with no need for hydration

This creates a complex problem: **where and when should data fetching happen?** 

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
  <!-- html is rendered once on the server, then hydrated once on the client -->
  <div v-for="post in data" :key="post.id">
    {{ post.title }}
  </div>
</template>
```

On the server:

- The component code runs
- It makes an HTTP request to `/api/posts` to get the data
- It renders the HTML with all your post titles

In the browser:

- The fully-rendered HTML is received with all the posts visible immediately
- Vue begins the [hydration](https://nuxt.com/docs/4.x/guide/concepts/rendering#universal-rendering) process to make the page interactive (to do this, it runs the component code again)
- The component makes an HTTP request to `/api/posts` to get the data (even though the data is already rendered)

You've now made two identical requests for the same data. This is known as the "double fetch problem."

**Why this is bad:**

- Wasted network requests and server resources
- Slower page interactivity (waiting for the second fetch to complete)
- Risk of [hydration mismatches](https://vuejs.org/guide/scaling-up/ssr#hydration-mismatch) if the data changed between requests
- Doubled load on your API

#### The idiomatic (Nuxt) way

For our posts page, we can use [`useFetch`](#usefetch---the-convenience-wrapper) instead of plain `fetch`:

```vue
<script setup>
// ✅ This fetches once, either on server or on the client
const { data: posts } = await useFetch('/api/posts')
</script>

<template>
  <div v-for="post in posts" :key="post.id">
    {{ post.title }}
  </div>
</template>
```

On the server:

- The component code runs
- Internally it [directly calls the handler](https://nuxt.com/docs/4.x/guide/concepts/server-engine#direct-api-calls) for `/api/posts` (rather than making an HTTP request)
- It renders the HTML with all your post titles (as before)
- It embeds a copy of the data via a [payload](https://nuxt.com/docs/4.x/getting-started/data-fetching#serializing-data-from-server-to-client) in the HTML

In the browser:

- The rendered HTML is immediately visible and the [hydration](https://nuxt.com/docs/4.x/guide/concepts/rendering#universal-rendering) process re-renders the component (as before)
- The data is immediately available to the component via the globally-embedded payload
- Nuxt **skips the second fetch** because the data is already there

On subsequent visits to the page:

- The component runs client-side only
- It *does* make an HTTP request to `/api/posts`
- It blocks the navigation transition until the data is ready

As you can see, Nuxt is jumping through a lot of hoops on your behalf to mitigate the issues of server/client boundaries!

## Nuxt's isomorphic-aware tooling

Now we've explored isometric orchestration, let's review Nuxt's data-fetching helpers.

- [`$fetch`](#fetch---your-http-client) 
  - Nuxt's core HTTP client
  - works anywhere (browser, server, API routes)
  - no double-fetch protection
- [`useAsyncData`](#useasyncdata---adding-ssr-awareness) 
  - wraps any async function to make it SSR-aware
  - orchestrates state between server and client
- [`useFetch`](#usefetch---the-convenience-wrapper) 
  - combines `useAsyncData` and `$fetch` for SSR-aware URL fetching
  - convenience function for common case of fetching from a URL

> In practice, you'll likely use `useFetch` most often – but to understand *why* and *when* to use each tool, we'll build up from the foundational helpers `$fetch` and `useAsyncData`.

### `$fetch` - a better HTTP client

`$fetch` is Nuxt's built-in HTTP client (powered by the `ofetch` library).

It's a better `fetch()` with automatic JSON parsing, error handling, and base URL resolution.

**When to use it:**

- Inside API routes on the server
- Client-side only operations (event handlers, user interactions)
- Anywhere outside component setup (event handlers, middleware, API routes)
- When you explicitly don't need SSR behaviour (there is no double-fetch protection)

**Where it works:**

- Anywhere: pages, components, composables, API routes, middleware

**What it does:**

- Makes an HTTP request
- Returns a promise with the response
- That's it - no SSR magic, no state management

**Server usage:**

```js
export default defineEventHandler(async (event) => {
  return $fetch('https://api.example.com/data')
})
```

**Client usage:**

```vue
<script>
async function handleFormSubmit() {
  const res = await $fetch('/api/submit', {
    method: 'POST',
    body: { /* form data */ }
  })
}
</script>
```

In the example above, `$fetch` is used inside an event handler that only runs on a user interaction (client-side only), so the double-fetch problem doesn't apply.

### `useAsyncData` - adding SSR awareness

`useAsyncData` wraps any async operation to make it SSR-aware, orchestrating when data loads and preventing navigation until complete.

It solves the double-fetch problem by:

1. executing your function on the server during SSR
2. serializing the result and transferring it to the client (via the payload)
3. skipping execution on the client during hydration (it already has the data)

**When to use it:**

- You need to wrap custom async logic (not just URL fetching)
- You're using a third-party library with its own query layer (like a CMS SDK)
- You want fine-grained control over caching and execution

**Where it works:**

- Only: pages, components, composables

**What it returns:**

- Reactive state objects for `data`, `status` and `error`
- Functions to `refresh` and `clear` the data

**Basic usage:**

```vue
<script setup>
// Parallel fetch example (access via: deals.value.coupons, deals.value.offers)
const { data: deals, refresh } = await useAsyncData('my-deals', async () => {
  const [coupons, offers] = await Promise.all([
    $fetch('/api/coupons'),
    $fetch('/api/offers')
  ])
  return { coupons, offers }
})
</script>

<template>
  <pre>{{ deals }}</pre>
  <button @click="refresh">Show latest deals</button>
</template>
```

Note the first argument `'my-deals'`, a unique key used to identify and cache the result. See the [cookbook](./cookbook/#cache-management) for more info.

### `useFetch` - the convenience wrapper

`useFetch` wraps both `useAsyncData` and `$fetch`, providing SSR-aware URL fetching with minimal boilerplate.

**When to use it:**

- You're fetching from a URL (internal API or external)
- You want the simplest, most common pattern
- You need SSR behaviour

**Where it works:**

- Pages, components, composables

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

See the [cookbook](./cookbook/#data-fetching-patterns) for more examples.

## General usage

### Where to fetch data

The tool you choose depends on where your code runs.

#### In pages and components

Components and pages execute on both server (SSR) and client (hydration). Use `useFetch` or `useAsyncData` to coordinate data loading - they prevent double fetching and ensure users don't see loading states by blocking server rendering and client navigation until data is ready.

They must be called synchronously at the top level of your `<script setup>` (or `setup()` function) or from a composable that is also called synchronously within setup:

```vue
<script setup>
// Fetches on server, transfers to client
const { data: user } = await useFetch('/api/user')
</script>
```

They cannot be called conditionally, inside loops, regular functions, or lifecycle hooks like `onMounted`.

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

API routes only run on the server; there's no client or hydration so use `$fetch` or call your logic directly:

```js
// server/api/posts.js
export default defineEventHandler(async (event) => {
  // Fetch from external API
  const example = await $fetch('https://api.example.com/data')
  
  // Direct database call
  const posts = await db.query('SELECT * FROM posts')
  
  return posts
})
```


### Non-blocking navigation

By default, `useFetch` and `useAsyncData` block navigation until the data loads - waiting for the fetch to complete on the server, or blocking route transitions during client-side navigation. Sometimes you want to show the page immediately and load data progressively.

The lazy variants (`useLazyFetch` and `useLazyAsyncData`) don't block in either context - on the server, they render immediately without waiting; during client navigation, they allow the route transition to complete. They begin the fetch in parallel and return with `status: 'pending'`, letting you handle the loading state manually.

> Note that if the server finishes rendering before the fetch completes, the client will need to fetch again - trading double-fetch prevention for faster initial page display.

When data is not critical for initial render (below-the-fold content, secondary data):

```vue
<script setup>
// no "await" required
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

This is just a shortcut for:

```ts
const { data } = useFetch('/api/comments', { lazy: true }) 
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

> Interestingly, the return value from `useAsyncData` is actually an enhanced `Promise` object; you can `await` it or *not* `await` it and it will still destructure! See [this article](https://masteringnuxt.com/blog/async-and-sync-how-useasyncdata-does-it-all) from Michael Thiessen to see exactly how it's done.

Note that the returned properties are Vue refs, so access them with `.value` in JavaScript:

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

Which tool to use:

- Use `useFetch` in page and component setup for fetching URLs
- Use `useAsyncData` in page and component setup for custom async fetch logic 
- Use `$fetch` anywhere outside component setup, such as client-side event handlers and API routes

General operation:

- Understand that Nuxt runs code on both server and client
- Let the composables handle SSR for you - they prevent double fetching
- Use keys strategically to control caching and data sharing
- Handle loading states when using `lazy` or `server: false`

### Cookbook

For further reading I've reorganised Nuxt's own [data fetching docs](https://nuxt.com/docs/4.x/getting-started/data-fetching) examples into a cookbook:

- [Nuxt Data Fetching Cookbook](./cookbook/)
