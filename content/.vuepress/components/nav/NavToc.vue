<template>
  <div v-if="items.length > 1"
       class="navToc"
       :data-depth="depth"
       v-html="html"
  ></div>
</template>

<script>
function split (value) {
  if (Array.isArray(value)) {
    return value
  }
  if (typeof value === 'string') {
    return value.split(',').map(value => value.trim())
  }
  return [value]
}

export default {
  props: {
    // optional page headers to render (defaults to the containing page)
    headers: {
      type: Array,
      default: () => [],
    },

    // change the text which precedes the links
    prompt: {
      type: String,
      default: 'Jump to',
    },

    // takes a number, or a comma-delimited string of levels, i.e. 2,3,4
    level: {
      type: [Number, String],
      default: 2,
    },

    // takes a string or array of strings of slugs to exclude
    exclude: {
      type: [String, Array],
      default: '',
    },

    // render links of a named section only (by slug)
    section: {
      type: String,
      default: '',
    },

    // begin rendering from a specific header (by slug)
    from: {
      type: String,
      default: '',
    },

    // end rendering at a specific header (by slug)
    to: {
      type: String,
      default: '',
    },

    // the type of structure to render; defaults to auto, which depends on the number of levels
    type: {
      type: String,
      validator: value => ['list', 'tree', 'auto'].includes(value),
      default: 'auto',
    },
  },

  data () {
    return {
      items: this.headers,
      depth: 1,
    }
  },

  computed: {
    options () {
      // variables
      let items = [...this.items]

      // props
      const levels = split(this.level).map(level => parseInt(level))
      const excludes = this.exclude ? split(this.exclude) : []
      const fromIndex = items.findIndex(item => item.slug === this.from)
      const toIndex = items.findIndex(item => item.slug === this.to)

      // section
      if (this.section) {
        let found = false
        let level = null
        let done = false
        items = items.reduce((items, item) => {
          if (!done) {
            if (!found) {
              if (item.slug === this.section) {
                found = true
                level = item.level
              }
            }
            else {
              if (item.level > level) {
                items.push(item)
              }
              else {
                done = true
              }
            }
          }
          return items
        }, [])
      }

      // slice if from or to and included
      if (toIndex > -1) {
        items = items.slice(0, toIndex + 1)
      }
      if (fromIndex > -1) {
        items = items.slice(fromIndex)
      }

      // filter the items
      items = items
        .filter(item => levels.includes(item.level))
        .filter(item => !excludes.includes(item.slug))

      // add tips to top level items
      const tips = Object.keys(this.$attrs).reduce((output, name) => {
        if (name.startsWith('tip-')) {
          output[name.substr(4)] = this.$attrs[name]
        }
        return output
      }, {})

      // return
      return {
        levels,
        excludes,
        items,
        tips,
      }
    },

    html () {
      // helpers
      const makeLink = (item, tip) => {
        const title = this.hasHierarchy
          ? item.title
          : item.title.replace(/^\W|\W$/g, '')
        let html = `<a href="#${item.slug}">${title}</a>`
        if (tip) {
          html += `<br><small>${tip}</small>`
        }
        return html
      }

      // variables
      const prompt = this.prompt
      const { levels, items, tips } = this.options

      // check for items
      if (!items.length) {
        const filteredProps = Object.fromEntries(Object.entries(this.$props).filter(entry => !!entry[1] || (Array.isArray(entry[1]) && entry[1].length > 0)))
        console.warn(`NavToc: no items resolved for props`, JSON.stringify(filteredProps, null,  '  '))
        return ''
      }

      // generate html
      if (levels.length) {
        // set depth
        if (this.type === 'tree') {
          this.depth = levels[levels.length - 1] - 1
        }

        // list
        if (this.hasHierarchy) {
          let prev = items[0]
          let html = '<ul>'
          for (const item of items) {
            if (item.level > prev.level) {
              html += '<ul>'
            }
            else if (item.level < prev.level) {
              html += '</ul>'.repeat(prev.level - item.level)
            }
            html += `<li>${makeLink(item, tips[item.slug])}</li>`
            prev = item
          }
          return prompt
            ? `<p>${prompt}:</p>${html}`
            : html
        }

        // paragraph
        else {
          const links = items.map(item => makeLink(item))
          const last = links.pop()
          return `<p>${prompt}: ${links.join(', ')} or ${last}.</p>`
        }
      }
    },

    hasHierarchy () {
      return this.options.levels.length > 1 || this.type === 'list'
    }
  },

  mounted () {
    if (!this.items.length) {
      this.items = this.$parent?.$page?.headers || []
    }
  },
}
</script>

<style lang="scss">
@import "../../styles/variables";

.navToc {
  li {
    margin-bottom: 0;
  }

  &:not([data-depth="1"]) > ul > li {
    margin-top: .75em;

    small {
      color: $grey;
    }

    a {
      color: $textColor;
      font-weight: 600;
    }

    small {
      display: block;
      margin-top: -.15em;
    }
  }

  > ul > ul > li {
    font-size: .85rem;
  }
}
</style>
