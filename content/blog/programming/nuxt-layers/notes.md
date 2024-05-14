---
description: Everything you ever wanted to know about a layers migration but were too afraid to ask
date: 2024-05-01
---

# Layers

### Rationale

The work of migrating a site to layers can depend on the size and complication of the app, but assuming you do the work, the payoff is much-reduced cognitive overhead of working with a smaller, finite set of files within each layer or domain. 

### Points

Bugs:

- layer config watching is buggy (intermittent at best)
- If one layer causes an error, it can be hard to track

Layer points:

- nested layers are supported
- if a layer path is not found, it will be ignored

## Paths primer

### Imports

Imports do not seem to be supported in layers.

The workaround is to keep them in the root

### Layer folders vs config settings

As part of your layers refactor, you're going to be moving folders and files to new locations.

Whilst Layers support some level of [automatic integration](https://nuxt.com/docs/guide/going-further/layers) into your app, there are also a number of [config settings](https://nuxt.com/docs/api/nuxt-config) which pertain to folder locations, and it's important to understand the crossover and differences between the two, as well as the few inconsistencies and one or two bugs.

Essentially, if you're going to move some or all of your app's core folders, you're going to need to get down and dirty with the config, and know what to reconfigure, and how.

Note that:

- some layer folders are processed as you would expect, some are not
- some config settings require additional work to work the same as if they were in the root
- there can only be **one** of certain kinds of folders

The following table gives you an overview of these factors, along with some notes:

| One | Config Setting                                                       | Layer Folder     | Setting Description                                                                                                                                                   | Layer Description                                |
|:---:|----------------------------------------------------------------------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
|  x  | [`dir.assets`](https://nuxt.com/docs/api/nuxt-config#assets)         |                  | The assets directory (aliased as `~assets` in your build)                                                                                                             |                                                  |
|     | [`dir.layouts`](https://nuxt.com/docs/api/nuxt-config#layouts)       | `layouts/*`      | The layouts directory, each file of which will be auto-registered as a Nuxt layout.                                                                                   | Extend the default layouts                       |
|     | [`dir.middleware`](https://nuxt.com/docs/api/nuxt-config#middleware) |                  | The middleware directory, each file of which will be auto-registered as a Nuxt middleware.                                                                            |                                                  |
|  x  | [`dir.modules`](https://nuxt.com/docs/api/nuxt-config#modules)       |                  | The modules directory, each file in which will be auto-registered as a Nuxt module.                                                                                   |                                                  |
|     | [`dir.pages`](https://nuxt.com/docs/api/nuxt-config#pages)           | `pages/*`        | The directory which will be processed to auto-generate your application page routes.                                                                                  | Extend the default pages                         |
|     | [`dir.plugins`](https://nuxt.com/docs/api/nuxt-config#plugins)       | `plugins/*`      | The plugins directory, each file of which will be auto-registered as a Nuxt plugin.                                                                                   | Extend the default plugins                       |
|  x  | [`dir.public`](https://nuxt.com/docs/api/nuxt-config#public)         |                  | The directory containing your static files, which will be directly accessible via the Nuxt server and copied across into your dist folder when your app is generated. |                                                  |
|     | [`components`](https://nuxt.com/docs/api/nuxt-config#components)     | `components/*`   | Configure Nuxt component auto-registration.                                                                                                                           | Extend the default components                    |
|     | [`imports.dirs`](https://nuxt.com/docs/api/nuxt-config#dirs)         | `composables/*`  | An array of custom directories that will be auto-imported. Note that this option will not override the default directories (`~/composables`, `~/utils`).              | Extend the default composables                   |
|     | [`plugins`](https://nuxt.com/docs/api/nuxt-config#plugins-1)         |                  | An array of nuxt app plugins. Each plugin can be a string (which can be an absolute or relative path to a file).                                                      |                                                  |
|     | [`serverDir`](https://nuxt.com/docs/api/nuxt-config#serverdir)       | `server/*`       | Define the server directory of your Nuxt application, where Nitro routes, middleware and plugins are kept.                                                            | Extend the default server endpoints & middleware |
|     |                                                                      | `utils/*`        |                                                                                                                                                                       | Extend the default utils                         |
|     |                                                                      | `nuxt.config.ts` |                                                                                                                                                                       | Extend the default nuxt config                   |
|     |                                                                      | `app.config.ts`  |                                                                                                                                                                       | Extend the default app config                    |

One other thing to bear in mind is that when you move files to layers, aliases may (or may not!) becomeinvalid:

- If Nuxt creates an alias, then you're good
- If not, you may need to recreate it yourself in `alias` and possibly `vote.resolve.alias`

If the – effectively global – file you were accessing via an alias is no longer accessible, you could just access it locally:

```ts
// layers/site/pages/Welcome.vue
import { greet } from '../utils'
```

Questions:

- Does `serverDir` have any effect in layer config?

### Path format

If you've used Nuxt for any length of time, you have probably tripped up configuration path formats.

They vary significantly by configuration setting, with all of the following on offer:

| Type           | Code                               | Notes                               |
| -------------- | ---------------------------------- | ----------------------------------- |
| Absolute       | `__dirname + '/layers/some-layer'` | Remember to add the leading slash!  |
| Alias          | `~/layers/some-layer`              | Expands internally to absolute path |
| Root-relative  | `layers/some-layer`                |                                     |
| Layer-relative | `some-layer`                       |                                     |
| Glob           | `some-layer/**/*.vue`              | Expands to an array of paths        |

There is also the factor of whether the configuration option is recursive by default or not.

The following table outlines the differences between the various config options:

| Name                                                             | Abs | Alias | Root | Layer | Glob | Rec. | Notes                                            |
|------------------------------------------------------------------|:---:|:-----:|:----:|:-----:|:----:|:----:|--------------------------------------------------|
| [`extends`](https://nuxt.com/docs/api/nuxt-config#extends)       |     |       |  ●   |   ●   |      |      | Layers can be nested (mainly useful for modules) |
| [`modules`](https://nuxt.com/docs/api/nuxt-config#modules)       |  ●  |       |      |       |      |      |                                                  |
| [`dir.*`](https://nuxt.com/docs/api/nuxt-config#dir)             |  ●  |       |  ●   |       |      |      |                                                  |
| [`dir.public`](https://nuxt.com/docs/api/nuxt-config#public)     |  ●  |       |  ●   |       |      |      | First public folder found wins                   |
| [`imports.dirs`](https://nuxt.com/docs/api/nuxt-config#dirs)     |     |       |  ●   |       |      |      | Only supported in root config                    |
| [`components`](https://nuxt.com/docs/api/nuxt-config#components) |     |   ●   |      |       |      |      |                                                  |
| [`plugins`](https://nuxt.com/docs/api/nuxt-config#plugins-1)     |  ●  |   ●   |  ●   |       |      |      |                                                  |

Some things to bear in mind about paths in Nuxt / Node:

Note that:

- glob patterns such as `core/**/*.vue` are not supported by any of the config options above
- the pattern `"/<rootDir>"` in the docs does **not** expand internally to an absolute path
- usage of `__dirname` always requires a slash after, i.e. `__dirname + '/folder'`
  - you may also need to add eslint rules`/* eslint-disable prefer-template,node/no-path-concat */`

Some things to bear in mind about Nuxt's config docs:

- they are auto-generated from [source code](https://github.com/nuxt/nuxt/tree/main/packages/schema/src/config) doc-comments
- the comments seem to be more designed for source code hints rather than public documentation
- expect missing comments, lack of context, or incomplete code snippets
  - for example the [alias](https://nuxt.com/docs/api/nuxt-config#alias) example:
    - frustratingly [does not](https://github.com/nuxt/nuxt/blob/2c39b3ce610b946ad9397911c7ac099021651b02/packages/schema/src/config/common.ts#L334-L343) include `import { fileURLToPath } from 'node:url'`
    - does not render the full example; there should be [an additional](https://github.com/nuxt/nuxt/blob/2c39b3ce610b946ad9397911c7ac099021651b02/packages/schema/src/config/common.ts#L345-L363) `html` snippet below

### Public assets

Files update as you edit them

First found folder wins:

```
 ERROR  Pre-transform error: Failed to resolve import "/app/arrow-right-button.svg" from "components/app/AppSupport.vue". Does the file exist?

```

Absolute path works in dev, but not in build

### Aliases

Create the aliases first, then move the files

If your IDE has path completion, it should pick up the paths from the regenerated `./nuxt/nuxt.config.ts`:

```json
{
	"compilerOptions": {
		"paths": {
			"~/utils/*": ["../core/utils/*"],
			"~/utils": ["../core/utils/index.ts"]
		}
	}
}
```

You will also need to update Vite's aliases:

```ts
const aliasConfig = {
  '~/composables': __dirname + '/composables',
  '~/utils': __dirname + '/utils',
}

const alias = defineNuxtConfig({
  // these are added to .nuxt/tsconfig.json
  // https://nuxt.com/docs/api/nuxt-config#alias
  alias: aliasConfig,

  // it seems that moving framework aliases to core requires they be added to the bunder too
  // https://nuxt.com/docs/api/nuxt-config#resolve
  vite: {
    resolve: {
      alias: Object
        .entries(aliasConfig)
        .map(([find, replacement]) => ({ find, replacement })),
    },
  },
})
```

Also, the use of `~/` is maybe why `#hash` is used as an alias, to prevent collisions.

Note, if this combines with `imports` you need to update those at the same time (see note below)

A note regarding default aliases, such as `~/server`:

Note a few restrictions with aliases:

- An alias can only point at one folder
- If you move a core folder with a default alias, you will need to update the alias
- Although TypeScript supports multiple paths for aliases, Vite does not, so your alias can only point at a single location 

Where to place imports and aliases?

- The root config. It just makes it easier.

#### Local layer aliases

See:

- https://nuxt.com/docs/api/nuxt-config#locallayeraliases



### Loading

In dev, routes will be rendered as they are hit

### Tailwind

Be sure to update Tailwind's `content` setting:

```js
export default {
  content: [
    './core/components/**/*.vue',
    './core/plugins/**/*.{js,ts}',
    './layers/**/pages/**/*.vue',
    './layers/**/components/**/*.vue',
    ...
	],
  ...
}
```



## Strategy

#### Imports

Use an IDE like Webstorm which rewrites your paths

#### Where

Rather than deciding "what is site?" or "what is core?" instead:

- move all folders to the `core` or  `site` layers (IDE should update links, etc)
- move what doesn't belong to their own layers
- whatever is left is either `core` or `site`

## Content

