---
description: Build sites that scale by structuring code by domain, not concern
xdate: 2024-03-08
preview: true
media:
  opengraph: ./featured.png
  featured: ./featured.png
  thumbnail: ./thumbnail.png
---

# Modular site architecture with Nuxt layers

> Everything you wanted to know about Nuxt Layers but were too afraid to ask
## Intro

Nuxt 3 introduces a new paradigm called "layers" that is described in the docs as "a powerful system that allows you to extend the default files, configs, and much more".

Whilst this explanation may be _technically_ accurate, I feel the emphasis on "extending" misses a simpler and more everyday use case; that of "siloing" application functionality as self-contained domain-driven units.

In the following article I'll outline building domain-driven sites with Nuxt layers:

- I'll start by reviewing traditional [site structure](#a-review-of-site-structure) 
- Then I'll introduce [Nuxt layers](#nuxt-layers-intro) to separate concerns by domain
- Finally, we'll explore [real world techniques](#real-world-layers-development) to get up to speed

Plus, a bonus section:

- Other options for [site modularity](#other-options-for-modularity)

## A review of site structure

Let's take a look at two main ways to structure sites and apps; by [concern](#by-concern) and by [domain](#by-domain).

### By concern

Most projects are born from starter templates which – for simplicity's sake – tend to silo files by concern:

```
+- src
    +- components
    |   +- blog
    |   +- home
    +- content
    |   +- blog
    +- pages
    |   +- blog.vue
    |   +- home.vue
    +- ...
```

Because it's easy to organise smaller sites, it's quick to add folders and files, and reasonably easy to keep track of them.

However, as sites grow in size it becomes increasingly difficult to grok the relations and functionality striped across the application's many top-level concerns, not to mention an increasingly-sprawling root-level config file.

### By domain

At a certain size of site (and actually, not that big!) it becomes more intuitive to silo files by domain:

```
+- src
    +- blog
    |   +- components
    |   +- content
    |   +- pages
    +- home
    |   +- components
    |   +- pages
    +- ...
```

Transposing "domains" for "concerns" has myriad benefits...

File management:

- domains are self-contained thanks to the natural siloing
- the code you need will generally be located in a sibling folder
- less open folders / scrolling / jumping in your IDE

Configuration and settings:

- domain config is discrete from from global config
- simpler, smaller, domain entry points, rather than one huge config file
- minimal mixing of global and local concerns

Developer experience:

- PRs are simpler as most files will exist downstream from a common folder
- you can more easily develop new features or site sections
- you can more easily turn complete features on / off

The paradigm shift from concerns to domains may feel familiar to you if you moved from Vue's Options API to the Composition API; rather than related functionality being striped across a huge options-based file, concerns are declared in the same place and can be treated as a single, self-contained unit.

> Note that the choice of words "domain" and "concern" could easily be "feature" and "responsibility"; feel free to use whatever terms make the most sense to you.

##  Nuxt layers intro

### Demo repo

For the purpose of the article, I've prepared a Nuxt 3 repository which demonstrates the concepts and techniques covered in the rest of the article:

- [github.com/davestewart/nuxt-layers-demo](https://github.com/davestewart/nuxt-layers-demo)

### Preface

To avoid parroting what has already been written, it might be a good idea to skim the docs before continuing:

- [Get Started » Layers](https://nuxt.com/docs/getting-started/layers)
- [Guide » Authoring Nuxt Layers](https://nuxt.com/docs/guide/going-further/layers)

Note that the docs primarily discuss  _extending_ (from a theme or such like) whereas this article is more about modularising (to silo complex functionality).

However, I have [linked](#resources) to resources at the end of the article which discuss extending in more detail.

### Layers 101

Nuxt layers can be thought of as kind of "mini applications" which are stitched together to form the complete app.

Each layer:

- must contain a `nuxt.config.ts` file
- may contain `pages`, `components`, `server` folders, etc

For example, a small personal site might be structured as follows:

```
+- src
    +- base                          <-- global, shared or one-off functionality
    |   +- ...
    +- blog                          <-- nuxt content configuration, markdown articles
    |   +- ...
    +- home                          <-- one-off components, animation plugin and configuration
    |   +- components
    |   |   +- Hero.vue
    |   |   +- Services.vue
    |   |   +- Testimonials.vue
    |   |   +- ...
    |   +- pages
    |   |   +- index.vue
    |   +- plugins
    |   |   +- ...
    |   +- nuxt.config.ts
    +- ...
    +- nuxt.config.ts 
```

The top level layers or "domains" would each have their own roles:

- `base` would silo shared concerns such as `assets`, `layouts`, etc.
- `blog` would configure Nuxt Content and contain a tree of articles
- `home` might contain animation `plugins`, `config` and `components` only used for a landing page

Subfolders within each layer or domain work exactly the same way as a full Nuxt app.

Finally, the root-level `nuxt.config.ts` instructs Nuxt to merge all layers via `extends`:

```ts
export default defineNuxtConfig({
  extends: [
    './base',
    './blog',
    './home',
  ]
})
```

Note that the `extends` functionality is courtesy of the [unjs/c12](https://github.com/unjs/c12?tab=readme-ov-file#extending-configuration) configuration library.

## Real world layers development

Now that you've seen the basics of a modular layered site, let's address the sorts of issues you might trip over in your first few hours of migrating an existing or building a new site:

- [Folder structure](#folder-structure)
- [Global concerns](#global-concerns)
- [Framework folders](#framework-folders)
- [Imports and exports](#imports-and-exports)
- [Config](#config)
- [Routing](#routing)
- [Nuxt Content](#nuxt-content)
- [Feature flags](#feature-flags)

### Folder structure

My currently-preferred structure for layers is a flat hierarchy of top-level folders. 

But layers are completely flexible, so if you preferred, you could layer only specific concerns:

<table>
<thead>
<tr>
<th style="text-align:center">Flat</th>
<th style="text-align:center">Split</th>
<th style="text-align:center">Partial</th>
</tr>
</thead>
<tbody>
<tr>
<td>
<pre class="language-text"><code>+- src
    +- blog
    |   +- ...
    +- core
    |   +- ...
    +- home
    |   +- ...
    |
    +- nuxt.config.ts
</code></pre>
</td>
<td>
<pre class="language-text"><code>+- src
    +- core
    |   +- ...
    |
    +- layers
    |   +- blog
    |   |   +- ...
    |   +- home
    |       +- ...
    |
    +- nuxt.config.ts
</code></pre>
</td>
<td>
<pre class="language-text"><code>+- src
    +- assets
    |
    +- layers
    |   +- blog
    |   |   +- ...
    |   +- home
    |       +- ...
    |
    +- layouts
    +- plugins
    +- components
    +- nuxt.config.ts
</code></pre>
</td>
</tr>
</tbody>
</table>

```
+- src
    +- assets
    +- layers
    |   +- blog
    |   |   +- ...
    |   +- home
    |       +- ...
    +- layouts
    +- plugins
    +- components
    +- nuxt.config.ts
```

```ts
// src/nuxt.config.ts
export default defineNuxtConfig({
  extends: [
    './layers/blog',
    './layers/home',
  ]
})
```

Note that layers can [reference](https://nuxt.com/docs/getting-started/layers#usage) NPM packages and GitHub repos – but for the purpose of this article I'm intentionally sticking with a flat file structure and local folder references.

### Global concerns

As soon as you begin shuffling folders around you're left with concerns that don't seem to have an obvious home.

For folders I consider to have more of a supporting role or are only occasionally updated, I move to `base`:

```
+- src
    +- base
    |   +- middleware
    |   +- modules
    |   +- public
    |   +- styles
    |   +- utils
    +- ... 
```

As `base` starts with "b" it generally sits at the top of the tree so I can ignore it, or know just where to look if I need it.

If a concern spans multiple domains, or isn't specific enough to get its own domain, `site` feels like a nice bucket:

```
+- src
    +- ... 
    +- site
        +- components
        |   +- Footer.vue
        |   +- Header.vue
        +- pages
        |   +- about.vue
        |   +- contact.vue
        +- ...
```

It generally sits at the bottom of the tree, so I can leave it open if I need to dip in and out.

However, where and how to draw your domain boundaries is [an entirely subjective question](https://github.com/nuxt/nuxt/issues/26444) – but the bottom line is _do what works for your project_ – and don't be afraid to iterate; what feels right today might feel outdated next week.

### Framework folders

Core framework folders [within layers](https://nuxt.com/docs/guide/going-further/layers) are auto-scanned and work as expected:

```
+- src 
    +- some-layer
        +- components
        +- composables
        +- layouts
        +- pages
        +- plugins
        +- server
        +- utils
```

For example:

- `pages` creates navigable [routes](https://nuxt.com/docs/guide/directory-structure/pages)
- `server/api` creates callable [endpoints](https://nuxt.com/docs/guide/directory-structure/server#server-routes)
- `components` are [auto-imported](https://nuxt.com/docs/guide/concepts/auto-imports)

This means you can break out concerns across domains **as you see fit** – and Nuxt will stitch them back together again.

### Imports and exports

Given that layers are generally self-contained, importing becomes much simper:

```ts
// src/dashboard/components/User.vue
import { queryUser } from '../services'
```

If you want to import from another layer – and you opted for a flat layer structure – you essentially get aliases for free:

```ts
// src/profile/components/User.ts
import { queryUser } from '~/dashboard/services'
```

If you want to expose only certain dependencies from a layer, consider an index file:

```ts
// src/dashboard/index.ts
export * from './services/foo'
export * from './utils/bar'
```
```ts
// src/profile/components/User.ts
import { queryUser } from '~/dashboard'
```

Though, remember auto-imports work with layers, so often you may not need to import anything at all.

### Config

One of my favourite features of layers is localised config.

It makes it so much easier to declare and isolate domain specific plugins, configuration and settings, without having to structure and navigate a sprawling, ever-expanding global config.

For global or one-off settings and plugins I prefer a `base` layer as noted above:

```ts
// src/base/nuxt.config.ts
export default defineNuxtConfig({
  alias: { ... },
  runtimeConfig: { ... },
  nitro: { ... },
  ...
})
```

(Though you could just as easily add this to the root nuxt config).

For domain-specific configuration, you can add it directly to a layer:

```ts
// src/blog/nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    ['fooify', {
      paths: [
        `~/blog/foo/*`,
      ]
    }]
  ]
})
```

For complex configuration that may differ only _slightly_ across layers, consider abstracting to helpers:

```ts
// src/base/utils/layers.ts
export function defineFooConfig (layer: string) {
  return {
    modules: [
      ['fooify', {
        paths: [
          `~/${layer}/foo/*`
        ]
      }]
    ]
  }
}
```

And within layers:

```ts
// src/blog/nuxt.config.ts
import { defineFooConfig } from '../base/utils/layers'

export default {
  ...defineFooConfig('blog'),
}
```
```ts
// src/home/nuxt.config.ts
import { defineFooConfig } from '../base/utils/layers'

export default {
  ...defineFooConfig('home'),
}
```

> Note that you cannot use path aliases such as `~` in config `import` statements – because the config file itself has not yet been compiled – so the `~` is not yet available in Vite .

Nuxt should intelligently merge configs (via [unjs/defu](https://github.com/unjs/defu)) rather than overwriting previously-declared layer configs:

```ts
// compiled nuxt config
{
  modules: [
    ['fooify', {
      paths: [
        `~/blog/foo/*`,
        `~/home/foo/*`,
      ],
    }],
  ]
}
```

### Routing

Note that layers can happily contain their own routes.

However, `pages` folders must contain **full folder paths** as the layer name is **not** automatically prepended:

```
+- src
    +- blog
    |   +- pages                <-- route starts here
    |       +- blog
    |           +- index.vue
    |           +- ...
    +- home
        +- pages                <-- route starts here
            +- index.vue
```

Another option might be [adding routes dynamically](https://nuxt.com/docs/api/nuxt-config#options) –  but it's almost certainly not worth the hassle.

It's possible to solve this using Nitro to rewrite the route before the app is loaded. 

`pages:extend`
`pages:routerOptions`

### Nuxt Content

Nuxt Layers also play nice with Nuxt Content.

This allows you to silo content along with its related `pages`, `components`, `services`, etc. rather than treat content as a global concern – which is great if your site has different-but-distinct content-driven sections such as Blog, Guide, etc.:

```
+- src
    +- blog
    |   +- ...
    +- guide
        +- components
        |   +- ...
        +- content
        |   +- index.md
        |   +- ...
        +- layouts
        |   +- ...
        +- pages
        |   +- ...
        +- nuxt.config.ts

```

Note that unlike [routes](#routing) you **can** configure content without re-nesting the folder.

To make it work you need to define each layer's `content` with a new `source`:

```ts
// src/blog/nuxt.config.ts
export default defineNuxtConfig ({
  content: {
    sources: {
      blog: { ... }
    }
  }
})
```

The `sources` boilerplate is a little bulky, so for multiple layers you can abstract to a helper (as [previously](#config) outlined):

```ts
// src/base/utils/layers.ts
export function defineContentConfig (name: string) {
  return {
    content: {
      sources: {
        [name]: {
          driver: 'fs',
          base: `./${name}/content`,
          prefix: `/${name}`,
        }
      },
    }
  }
}
```
```ts
// src/blog/nuxt.config.ts
import { defineContentConfig } from '../base/utils/layers'

export default defineNuxtConfig ({
  ...defineContentConfig('blog')
})
```

### Feature flags

Another great thing about layers is that because they silo all concerns under a common folder, it's really easy to toggle all that functionality on or off – at build time, at least.

Let's say you're developing a blog module, but only want it live in development; you can place the layer behind a switch:

```ts
export default defineNuxtConfig({
  extends: [
    './base',
    './home',
    isDev && './blog',
  ]
})
```

## Other options for modularity

It should be noted that Nuxt 3's layers are not exclusive to Nuxt 3; siloing sites this way can be accomplished in Nuxt 2, using namespaces, or at build time using Webpack or Vite.


### Nuxt 2

### Namespaces

### Webpack

### Vite

<!--
-->

## Last words

Hopefully this section gives you some solid ideas on how to modularise your site or app – and if I've skipped over anything – ideas on how to approach it.

Layers are generally quite logical and predicable, don't be afraid to jump in and refactor an existing site – which you can successfully do, layer-by-layer.

Kudos to the UnJS and Nuxt team for the work they've done here.


## Resources

Links to resources here

