---
description: Build sites that scale by structuring code by domain, not responsibility
xdate: 2024-03-08
media:
  opengraph: ./featured.png
  featured: ./featured.png
  thumbnail: ./thumbnail.png
---

# Modular site architecture using Nuxt layers

## Abstract

This post is about structuring Nuxt sites so they're easier to manage once they grow past a few 

## Intro

As small sites grow into big sites it becomes increasingly difficult to locate, understand, and reason about the files, functionality and responsibilities within. Usually, this is because starter templates group files by responsibility:

```
+- site
    +- components
    |   +- blog
    |   +- home
    +- pages
    |   +- blog
    |   +- home
    +- ...
```

However, past a certain point it becomes more intuitive to group files by _domain_ or _feature_:

```
+- site
    +- home
    |   +- components
    |   +- pages
    +- blog
    |   +- components
    |   +- pages
    +- ...
```

Grouping files this way has myriad benefits:

- the code you need will be located in a sibling folder
- less open folders / scrolling / jumping in your IDE
- silos are self-contained thanks to the domain-based nature
- domain code / config is separated from core code / config
- simpler, domain smaller entry points, rather than once huge config file
- less muddling of global and local concerns
- PRs are simpler as most files will exist downstream from a common folder
- you can more easily introduce new features or site sections
- you can more easily turn complete features on / off

In this article I'll share my [commercial experience](/work/) developing modular architectures, then share how I leveraged Nuxt 3's Layers to migrate an existing Nuxt 3 project to this new modular architecture.

## Modularity and Nuxt Layers

You're probably familiar with high-level modularization such as [Node modules](https://www.npmjs.com/), [Vue Plugins](https://vuejs.org/guide/reusability/plugins), or [Nuxt Modules](https://nuxt.com/docs/guide/concepts/modules).

These are all officially-architected ways to develop smaller parts of code in isolation from a larger system, with an agreed contract between module and framework so the code can be imported, spliced-in and called when needed.

Whilst this is great, it doesn't address the architectural issue mentioned above, and this is where [Nuxt Layers](https://nuxt.com/docs/getting-started/layers) comes in; each layer is effectively a mini Nuxt application (or even _part_ of an application) and Nuxt stitches each's `components`, `pages`, etc together to work as a larger whole.

The advantage of this is you can much more easily define the boundaries within your app, and work on much smaller, self-contained sets of files, such as `/common`, `/home`, `/products`, and so on.


## Modularity and JavaScript

I thought it would be interesting to cover my own experiences of developing more modular architectures in Vue, as it's something I've been thinking about and implementing for a number of years now.

***Feel free to skip this section if you're just here for the Nuxt content!***

### Intro

I was first exposed to very large frontend applications when I worked on backoffice systems for [Clear Bank](/work/clearbank/) in 2018.

Their system was huge; mirroring the bank's extensive API and business domains, yet the core architecture was the same `routes`, `components`, `store`, `views` folders spawned from  `vue create app` but with similar (yet subtly different!) sub-folder structures nested in each.

This meant a lot of folder traversal, access to code via a few top-level entry points, and potential fragility. Refactoring related concerns was risky as it wasn't immediatly clear what other dependencies might be affected, and onboarding was tough as you effectively had to say "jump in and explore, *but try not to break anything*".

### Vue / JS

I was fairly sure I could make inroads into clarifying the overal application structure, and the first step was breaking out the monolithic routes file into files matching the top-level views folders.

Next was attempting to make a lot more of the app local. Rather than having every `view` dependent on a global Vuex file, we started moving state local to the views. After this came config, and services, so each "module" of the app would import from a local `index` file, rather than having to reach all the way out to various global dependencies.

Whilst 

### Webpack

For [Asterisk](/work/asterisk/#modular-build) I got to test this out

For [Control Space](/products/control-space) I took this a step further

### Nuxt 2

For [FGH](/work/fgh) as we scaled from a small to medium application, I created [Nuxt Areas](/projects/open-source/nuxt-areas/) 
