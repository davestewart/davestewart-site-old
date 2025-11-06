---
title: Nuxt Data Fetching Cookbook
shortTitle: Cookbook
description: A cookbook of Nuxt data fetching patterns and best practices
date: 2025-11-05
permalink: /blog/nuxt-data-fetching/cookbook/
---

# Cookbook

This article is a companion section for the [Data Fetching in Nuxt](/blog/nuxt-data-fetching/) article, which provides extended examples beyond the primary `$fetch`, `useAsyncData` and `useFetch` illustrations.

>  All code is taken from the [Nuxt 4 docs](https://nuxt.com/docs/4.x/getting-started/data-fetching); just reorganised to make it simpler to browse and consume.

<NavToc type="tree" exclude="cookbook" level="2,3" prompt="Contents" />

## Data Fetching Patterns

### Basic fetching

`useFetch` and `useAsyncData` are the core composables for data fetching in Nuxt:

```vue
<script setup>
// Simple fetch with useFetch
const { data: users } = await useFetch('/api/users')

// More control with useAsyncData
const { data: posts } = await useAsyncData('posts', () => $fetch('/api/posts'))
</script>
```

**For `useFetch`:** The URL is the key (or you can provide one explicitly)

**For `useAsyncData`:** You must provide the key as the first argument

### Client-only fetching

To fetch only on the client (skipping SSR), set `server: false`:

```vue
<script setup>
// this won't fetch during SSR
const { data: comments } = await useFetch('/api/comments', {
  server: false
})
</script>
```

**Use cases:**

- Data that's not needed for SEO
- User-specific data that shouldn't be in HTML
- Third-party widgets or ads

**Important:** With `server: false`, data won't exist until after hydration. You must handle the loading state:

```vue
<script setup>
const { status, data } = await useFetch('/api/comments', {
  server: false,
  lazy: true  // often combined with server: false
})
</script>

<template>
  <div v-if="status === 'pending'">Loading...</div>
  <div v-else>{{ data }}</div>
</template>
```

**Critical:** If you have not fetched data on the server (with `server: false`), the data will **not** be fetched until hydration completes. This means even if you `await useFetch` on client-side, `data` will remain null within `<script setup>` until the component mounts.

### Delayed fetching

To wait for user interaction before fetching, use `immediate: false`:

```vue
<script setup>
const { status, data, execute } = await useFetch('/api/data', {
  immediate: false
})
</script>

<template>
  <div v-if="status === 'idle'">
    <button @click="execute">Load Data</button>
  </div>
  <div v-else-if="status === 'pending'">
    Loading...
  </div>
  <div v-else>
    {{ data }}
  </div>
</template>
```

### Dependent queries

When one query depends on another, await the first before the second:

```vue
<script setup>
// first, get the current user
const { data: user } = await useFetch('/api/current-user')

// then fetch their profile using their ID
const { data: profile } = await useFetch(`/api/users/${user.value.id}`)
</script>
```

For queries that should refetch when dependencies change, use computed URLs or the `watch` option (covered in the [Dynamic and Reactive Fetching](#dynamic-and-reactive-fetching) section).

### Parallel requests

When multiple requests don't depend on each other, fetch them in parallel for better performance:

```vue
<script setup>
const { data } = await useAsyncData('dashboard', async () => {
  return await Promise.all([
    $fetch('/api/user'),
    $fetch('/api/notifications'),
    $fetch('/api/stats')
  ])
})

const user = computed(() => data.value?.[0])
const notifications = computed(() => data.value?.[1])
const stats = computed(() => data.value?.[2])
</script>
```

Alternatively, use multiple composables (they'll fetch in parallel automatically):

```vue
<script setup>
// these three requests happen in parallel
const { data: user } = useFetch('/api/user')
const { data: notifications } = useFetch('/api/notifications')
const { data: stats } = useFetch('/api/stats')

// navigation waits for all three to complete
</script>
```

### Side effects

Don't use `useAsyncData` for triggering side effects, for example calling Pinia actions or mutations, as this can cause unintended repeated executions. For one-time side effects, use the `callOnce` utility instead:

```vue
<script setup>
// ❌ Don't do this
await useAsyncData(() => offersStore.getOffer(route.params.slug))

// ✅ Do this instead
await callOnce(() => offersStore.getOffer(route.params.slug))
</script>
```

## Dynamic and Reactive Fetching

### Computed and reactive URLs

When URLs depend on reactive values (like route params), you have two options:

#### Reactive query parameters

Pass reactive values as query parameters. Nuxt automatically watches them:

```vue
<script setup>
const userId = ref(1)

const { data: user } = await useFetch('/api/user', {
  query: {
    id: userId  // automatically watched
  }
})

// changing userId will trigger a refetch
userId.value = 2
</script>
```

#### Computed URL functions

Use a function that returns the URL. Nuxt watches the reactive dependencies:

```vue
<script setup>
const route = useRoute()

// function is reactive - refetches when route.params.id changes
const { data: post } = await useFetch(() => `/api/posts/${route.params.id}`)
</script>
```

Combine with `immediate: false` to wait for reactive values to be set:

```vue
<script setup>
const searchQuery = ref('')

const { status, data: results } = await useFetch(
  () => `/api/search?q=${searchQuery.value}`,
  { immediate: false }
)
</script>

<template>
  <input v-model="searchQuery" @input="execute">
  <div v-if="status === 'pending'">Searching...</div>
  <div v-else-if="data">{{ results }}</div>
</template>
```

### Watching specific values

To refetch when specific reactive values change, use the `watch` option:

```vue
<script setup>
const page = ref(1)
const sortBy = ref('name')

const { data: items } = await useFetch('/api/items', {
  query: {
    page,
    sort: sortBy
  },
  watch: [page]  // only refetch when page changes, not sortBy
})
</script>
```

**Important:** Watching doesn't change the URL. If you need to change the URL based on reactive values, use a computed URL (Computed URL functions above).

To disable automatic watching of query parameters, set `watch: false`:

```vue
<script setup>
const id = ref(1)

const { data, execute } = await useFetch('/api/user', {
  query: { id },
  watch: false  // won't automatically refetch when id changes
})

// manually trigger refetch when needed
function loadUser() {
  execute()
}
</script>
```

## Cache Management

### Keys and caching

Every `useFetch` and `useAsyncData` call has a key. This key is used to:

- Cache the result across your application
- Deduplicate identical requests
- Share data between components

**For `useFetch`:** The URL is the key (or you can provide one explicitly)

**For `useAsyncData`:** You must provide the key as the first argument

Multiple components using the same key will share the same `data`, `error`, and `status` refs:

```vue
<!-- Component A -->
<script setup>
const { data: users } = await useFetch('/api/users')
</script>

<!-- Component B -->
<script setup>
// same key ('/api/users'), so shares the same data ref
const { data: users } = await useFetch('/api/users')
</script>
```

You can access cached data anywhere using `useNuxtData`:

```vue
<script setup>
// get previously fetched data without refetching
const users = useNuxtData('/api/users')
</script>
```

### Manual refetching

Use the `refresh()` or `execute()` functions to manually refetch data:

```vue
<script setup>
const { data, refresh } = await useFetch('/api/posts')

function reloadPosts() {
  // manually trigger a refetch
  refresh()
}
</script>

<template>
  <button @click="reloadPosts">Reload</button>
</template>
```

**Note:** By default, Nuxt waits until a `refresh` completes before it can be executed again. Concurrent refresh calls are prevented to avoid race conditions.

### Clearing data

Use `clear()` to reset data to undefined:

```vue
<script setup>
const { data, clear } = await useFetch('/api/posts')

const route = useRoute()
watch(() => route.path, (path) => {
  if (path === '/') {
    clear()
  }
})
</script>
```

### Global cache invalidation

To invalidate cached data across your app:

```vue
<script setup>
// clear specific data
clearNuxtData('/api/posts')

// refresh specific data
refreshNuxtData('/api/posts')

// clear all cached data
clearNuxtData()
</script>
```

## Request and Response Control

### Headers and cookies

#### Automatic header forwarding

When you call `useFetch` from a component during SSR, Nuxt automatically forwards relevant headers from the original client request to your API route:

```vue
<script setup>
// cookies and relevant headers are automatically forwarded
const { data } = await useFetch('/api/user')
</script>
```

Behind the scenes, `useFetch` uses `useRequestFetch` to automatically proxy headers like `cookie`, `authorization`, etc. Headers that shouldn't be forwarded (like `host`) are excluded. This only works with relative URLs (starting with `/`) - external URLs don't receive forwarded headers.

#### Manual header access

If you need to manually access headers (for example, when using `$fetch` instead of `useFetch`), use `useRequestHeaders`:

```vue
<script setup>
// get specific headers from the incoming request
const headers = useRequestHeaders(['cookie'])

async function fetchUser() {
  // manually pass headers to $fetch
  return await $fetch('/api/user', { headers })
}

const { data } = await useAsyncData('user', fetchUser)
</script>
```

Or use `useRequestFetch` to automatically proxy headers:

```vue
<script setup>
const requestFetch = useRequestFetch()

const { data } = await useAsyncData('user', () => {
  return requestFetch('/api/user')
})
</script>
```

#### Passing cookies back to client

If your API route receives cookies from an external service and you want to pass them to the client:

```ts
// composables/fetch.ts
import { appendResponseHeader, H3Event } from 'h3'

export async function fetchWithCookie(event: H3Event, url: string) {
  const res = await $fetch.raw(url)
  const cookies = res.headers.getSetCookie()
  
  // forward each cookie to the client
  for (const cookie of cookies) {
    appendResponseHeader(event, 'set-cookie', cookie)
  }
  
  return res._data
}
```

```vue
<script setup>
const event = useRequestEvent()
const { data } = await useAsyncData(() => fetchWithCookie(event!, '/api/auth'))
</script>
```

#### Security considerations

Be careful when forwarding headers to external APIs. Only forward headers you explicitly need. Some headers should never be forwarded:

- `host`, `accept`
- `content-length`, `content-md5`, `content-type`
- `x-forwarded-host`, `x-forwarded-port`, `x-forwarded-proto`
- `cf-connecting-ip`, `cf-ray`

### Minimizing payload size

The `pick` option reduces payload size by selecting only the fields you need:

```vue
<script setup>
const { data: mountain } = await useFetch('/api/mountains/everest', {
  pick: ['title', 'description']  // only these fields in the payload
})
</script>
```

For more control, use `transform` to process the data before it's serialized:

```vue
<script setup>
const { data: mountains } = await useFetch('/api/mountains', {
  transform: (mountains) => {
    return mountains.map(m => ({
      title: m.title,
      description: m.description
    }))
  }
})
</script>
```

**Note:** These options only affect the payload transferred from server to client. The initial API call still fetches all data.

### Request deduplication

The `dedupe` option controls how duplicate requests are handled:

```vue
<script setup>
// 'defer' (default) - reuse pending requests
// 'cancel' - cancel previous requests when a new one starts
const { data } = await useFetch('/api/data', { dedupe: 'cancel' })
</script>
```

### Calling external APIs

You can choose where to call external APIs from.

**From API handlers** when:

- You need to hide API keys (via [runtime config](https://nuxt.com/docs/4.x/guide/going-further/runtime-config))
- You want to transform/combine data before sending to client
- The external API doesn't support CORS

**Directly from components** when:

- The API is public and supports CORS
- You want client-side only fetching
- You're ok exposing the API call to the client

```vue
<script setup>
// calling external API directly from component
const { data } = await useFetch('https://api.publicdata.com/items')
</script>
```

### Shared server logic

For shared server logic, use `server/utils/`:

```js
// server/utils/db.js
export function getPosts() {
  return db.select().from('posts')
}
```

```js
// server/api/posts.js
export default defineEventHandler(async (event) => {
  return await getPosts()
})
```

## Advanced Configuration

### Advanced options

#### Deep reactivity

The `deep` option controls whether the `data` ref is deeply reactive (using `ref()`) or shallowly reactive (using `shallowRef()`):

```vue
<script setup>
// deeply reactive - watches nested properties
const { data } = await useFetch('/api/user', { deep: true })

// shallowly reactive - only watches top-level changes (better performance)
const { data } = await useFetch('/api/user', { deep: false })
</script>
```

#### Custom cache retrieval

The `getCachedData` option lets you provide custom logic for retrieving cached data:

```vue
<script setup>
const nuxtApp = useNuxtApp()

const { data } = await useFetch('/api/posts', {
  getCachedData: (key) => {
    // custom cache lookup logic
    return nuxtApp.payload.data[key] ?? nuxtApp.static.data[key]
  }
})
</script>
```

### Shared state and option consistency

When multiple components use the same key with `useFetch` or `useAsyncData`, they share the same reactive refs. This requires certain options to be consistent across all uses:

**Must be consistent:**

- Handler function
- `deep` option
- `transform` function
- `pick` array
- `getCachedData` function
- `default` value

```ts
// ❌ this will cause a warning
const { data: users1 } = useAsyncData('users', () => $fetch('/api/users'), { deep: false })
const { data: users2 } = useAsyncData('users', () => $fetch('/api/users'), { deep: true })
```

**Can differ safely:**

- `server`
- `lazy`
- `immediate`
- `dedupe`
- `watch`

```ts
// ✅ this is fine
const { data: users1 } = useAsyncData('users', () => $fetch('/api/users'), { immediate: true })
const { data: users2 } = useAsyncData('users', () => $fetch('/api/users'), { immediate: false })
```

If you need independent instances, use different keys:

```ts
const { data: users1 } = useAsyncData('users-1', () => $fetch('/api/users'))
const { data: users2 } = useAsyncData('users-2', () => $fetch('/api/users'))
```

### Reactive keys

Keys can be reactive, enabling dynamic fetching:

```ts
const userId = ref('123')

const { data: user } = await useAsyncData(
  computed(() => `user-${userId.value}`),
  () => fetchUser(userId.value)
)

// changing userId automatically refetches with new key
userId.value = '456'
```

## Data Serialization

### Understanding serialization

Data flows from server to client in two ways in Nuxt:

**1. Via `useAsyncData` / `useFetch` (through Nuxt payload):**

- Uses `devalue` serializer
- Supports advanced types: `Date`, `Map`, `Set`, `RegExp`, `ref`, `reactive`, `Error`, etc.
- Data is embedded in the HTML payload

**2. Via API routes (through `$fetch`):**

- Uses `JSON.stringify`
- Limited to JSON-compatible types
- Data is fetched via HTTP

### API route serialization

When fetching from `server/api`, responses are serialized with `JSON.stringify`. This means `Date` objects become strings, `Map` becomes `{}`, etc.

```ts
// server/api/post.ts
export default defineEventHandler(() => {
  return {
    title: 'My Post',
    createdAt: new Date()  // becomes a string
  }
})
```

```vue
<script setup>
// createdAt is a string, not a Date object
const { data: post } = await useFetch('/api/post')

console.log(post.value.createdAt instanceof Date)  // false
console.log(typeof post.value.createdAt)  // 'string'
</script>
```

### Custom serialization

**Solution 1: Custom `toJSON` method**

Define a `toJSON` method on your return object:

```ts
// server/api/post.ts
export default defineEventHandler(() => {
  return {
    title: 'My Post',
    createdAt: new Date(),
    
    toJSON() {
      return {
        title: this.title,
        createdAt: this.createdAt.toISOString()
      }
    }
  }
})
```

**Solution 2: Transform on the client**

Use the `transform` option to process the response:

```vue
<script setup>
const { data: post } = await useFetch('/api/post', {
  transform: (data) => ({
    ...data,
    createdAt: new Date(data.createdAt)
  })
})

// now createdAt is a Date object
console.log(post.value.createdAt instanceof Date)  // true
</script>
```

**Solution 3: Custom serializer (superjson, etc.)**

For complex serialization needs:

```ts
// server/api/data.ts
import superjson from 'superjson'

export default defineEventHandler(() => {
  const data = {
    date: new Date(),
    map: new Map([['key', 'value']]),
    
    toJSON() {
      return this
    }
  }
  
  return superjson.stringify(data) as unknown as typeof data
})
```

```vue
<script setup>
import superjson from 'superjson'

const { data } = await useFetch('/api/data', {
  transform: (value) => superjson.parse(value as unknown as string)
})

// complex types are preserved
console.log(data.value.date instanceof Date)  // true
console.log(data.value.map instanceof Map)    // true
</script>
```

## Special Use Cases

### Server-sent events

For SSE via GET, use the browser's `EventSource` or VueUse's `useEventSource`.

For SSE via POST, handle the stream manually:

```ts
// make POST request to SSE endpoint
const response = await $fetch<ReadableStream>('/api/chat', {
  method: 'POST',
  body: { query: 'Hello' },
  responseType: 'stream'
})

// read the stream
const reader = response.pipeThrough(new TextDecoderStream()).getReader()

while (true) {
  const { value, done } = await reader.read()
  if (done) break
  
  console.log('Received:', value)
}
```

### Options API support

For Options API components, wrap your definition in `defineNuxtComponent` and use the `asyncData` option:

```vue
<script>
export default defineNuxtComponent({
  fetchKey: 'posts',
  
  async asyncData() {
    return {
      posts: await $fetch('/api/posts')
    }
  }
})
</script>
```

**Note:** Composition API with `<script setup>` is the recommended approach in Nuxt.
