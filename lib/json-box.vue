<script>
import JsonString from './types/json-string'
import JsonUndefined from './types/json-undefined'
import JsonNumber from './types/json-number'
import JsonBoolean from './types/json-boolean'
import JsonObject from './types/json-object'
import JsonArray from './types/json-array'
import JsonFunction from './types/json-function'
import JsonDate from './types/json-date'
import Mark from 'mark.js'

export default {
  name: 'JsonBox',
  inject: ['parentProps'],
  props: {
    value: {
      type: [Object, Array, String, Number, Boolean, Function, Date],
      default: null
    },
    keyName: {
      type: String,
      default: ''
    },
    sort: Boolean,
    depth: {
      type: Number,
      default: 0
    },
    previewMode: Boolean,
  },
  data() {
    return {
      expand: true
    }
  },
  watch: {
    'parentProps.expandDepth' () {
      const newExpand = this.recomputeExpand()
      if (this.expand !== newExpand) {
        this.toggle()
      }
    },
    'parentProps.highlight' () {
      if (this.mark) {
        this.parentProps.searchCallback(this.mark)
      }
    },
    expand (v) {
      if (v) {
        this.$emit('expanded', this)
      }
    }
  },
  beforeMount () {
    },
  mounted() {
    this.mark = new Mark(this.$el)
    this.expand = this.recomputeExpand()
  },
  methods: {
    recomputeExpand () {
      return this.previewMode || (this.depth >= this.parentProps.expandDepth ? false : true)
    },
    toggle() {
      this.expand = !this.expand

      try {
        this.$el.dispatchEvent(new Event('resized'))
      } catch (e) {
        // handle IE not supporting Event constructor
        var evt = document.createEvent('Event')
        evt.initEvent('resized', true, false)
        this.$el.dispatchEvent(evt)
      }
    }
  },
  render (h) {
    let elements = []
    let dataType

    if (this.value === null || this.value === undefined) {
      dataType = JsonUndefined
    } else if (Array.isArray(this.value)) {
      dataType = JsonArray
    } else if (Object.prototype.toString.call(this.value) === '[object Date]') {
      dataType = JsonDate
    } else if (typeof this.value === 'object') {
      dataType = JsonObject
    } else if (typeof this.value === 'number') {
      dataType = JsonNumber
    } else if (typeof this.value === 'string') {
      dataType = JsonString
    } else if (typeof this.value === 'boolean') {
      dataType = JsonBoolean
    } else if (typeof this.value === 'function') {
      dataType = JsonFunction
    }
    const complex = this.keyName && (this.value && (Array.isArray(this.value) || (typeof this.value === 'object' && Object.prototype.toString.call(this.value) !== '[object Date]')))

    if (!this.previewMode && complex) {
      elements.push(h('span', {
        class: {
          'jv-toggle': true,
          open: !!this.expand
        },
        on: {
          click: this.toggle
        }
      }))
    }

    if (this.keyName) {
      elements.push(h('span', {
        class: {
          'jv-key': true
        },
        domProps: {
          innerText: `${this.keyName}:`
        },
        on: {
          click: () => {
            if (!this.previewMode && complex && window.getSelection().toString() === '') {
              this.toggle()
            }
          }
        }
      }))
    }

    elements.push(h(dataType, {
      class: {
        'jv-push': true
      },
      props: {
        jsonValue: this.value,
        keyName: this.keyName,
        sort: this.sort,
        depth: this.depth,
        expand: this.expand,
        previewMode: this.previewMode,
      },
      on: {
        'update:expand': value => {
          this.expand = value
        }
      }
    }))

    if (this.parentProps.searchCallback) {
      this.$nextTick().then(() => this.parentProps.searchCallback(this.mark))
    }

    return h('div', {
      class: {
        'jv-node': true,
        'jv-key-node': Boolean(this.keyName) && !complex,
        'toggle': !this.previewMode && complex
      }
    }, elements)
  }
}
</script>

<style lang="scss">
.jv-node {
  position: relative;

  &:after {
    content: ','
  }
  &:last-of-type {
    &:after {
      content: ''
    }
  }

  &.toggle {
    margin-left: 13px !important;
  }

  & .jv-node {
    margin-left: 25px;
  }
}
</style>
