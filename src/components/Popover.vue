<template>
  <div ref="reference">
    <div
      ref="target"
      :class="['inline-block', $attrs.class]"
      @click="updatePosition"
      @focusin="updatePosition"
      @keydown="updatePosition"
      @mouseover="onMouseover"
      @mouseleave="onMouseleave"
    >
      <slot
        name="target"
        v-bind="{ togglePopover, updatePosition, open, close, isOpen }"
      />
    </div>
    <teleport to="#frappeui-popper-root">
      <div
        ref="popover"
        :class="popoverClass"
        class="relative z-[100] popover-container"
        :style="{ minWidth: targetWidth ? targetWidth + 'px' : null }"
        v-show="isOpen"
      >
        <transition v-bind="popupTransition">
          <div v-show="isOpen">
            <slot
              name="body"
              v-bind="{ togglePopover, updatePosition, open, close, isOpen }"
            >
              <div class="bg-white border border-gray-100 rounded-lg shadow-xl">
                <slot
                  name="body-main"
                  v-bind="{
                    togglePopover,
                    updatePosition,
                    open,
                    close,
                    isOpen,
                  }"
                />
              </div>
            </slot>
          </div>
        </transition>
      </div>
    </teleport>
  </div>
</template>

<script>
import { createPopper } from '@popperjs/core'

export default {
  name: 'Popover',
  inheritAttrs: false,
  props: {
    show: {
      default: undefined,
    },
    trigger: {
      type: String,
      default: 'click', // click, hover
    },
    hoverDelay: {
      type: Number,
      default: 0,
    },
    right: Boolean,
    placement: {
      type: String,
      default: 'bottom-start',
    },
    popoverClass: [String, Object, Array],
    transition: {
      default: null,
    },
    hideOnBlur: {
      default: true,
    },
  },
  emits: ['open', 'close', 'update:show'],
  expose: ['open', 'close'],
  data() {
    return {
      showPopup: false,
      targetWidth: null,
    }
  },
  watch: {
    show(val) {
      if (val) {
        this.open()
      } else {
        this.close()
      }
    },
  },
  created() {
    if (!document.getElementById('frappeui-popper-root')) {
      const root = document.createElement('div')
      root.id = 'frappeui-popper-root'
      document.body.appendChild(root)
    }
  },
  mounted() {
    this.listener = (e) => {
      let $els = [this.$refs.reference, this.$refs.popover]
      let insideClick = $els.some(
        ($el) => $el && (e.target === $el || $el.contains(e.target))
      )
      if (insideClick) {
        return
      }
      this.close()
    }
    if (this.hideOnBlur) {
      document.addEventListener('click', this.listener)
    }
    this.$nextTick(() => {
      this.targetWidth = this.$refs['target'].clientWidth
    })
  },
  beforeDestroy() {
    this.popper && this.popper.destroy()
    document.removeEventListener('click', this.listener)
  },
  computed: {
    showPropPassed() {
      return this.show != null
    },
    isOpen: {
      get() {
        if (this.showPropPassed) {
          return this.show
        }
        return this.showPopup
      },
      set(val) {
        val = Boolean(val)
        if (this.showPropPassed) {
          this.$emit('update:show', val)
        } else {
          this.showPopup = val
        }
        if (val === false) {
          this.$emit('close')
        } else if (val === true) {
          this.$emit('open')
        }
      },
    },
    popupTransition() {
      let templates = {
        default: {
          enterActiveClass: 'transition duration-200 ease-out',
          enterFromClass: 'translate-y-1 opacity-0',
          enterToClass: 'translate-y-0 opacity-100',
          leaveActiveClass: 'transition duration-150 ease-in',
          leaveFromClass: 'translate-y-0 opacity-100',
          leaveToClass: 'translate-y-1 opacity-0',
        },
      }
      if (typeof this.transition === 'string') {
        return templates[this.transition]
      }
      return this.transition
    },
  },
  methods: {
    setupPopper() {
      if (!this.popper) {
        this.popper = createPopper(this.$refs.reference, this.$refs.popover, {
          placement: this.placement,
        })
      } else {
        this.updatePosition()
      }
    },
    updatePosition() {
      this.popper && this.popper.update()
    },
    togglePopover(flag) {
      if (flag instanceof Event) {
        flag = null
      }
      if (flag == null) {
        flag = !this.isOpen
      }
      flag = Boolean(flag)
      if (flag) {
        this.open()
      } else {
        this.close()
      }
    },
    open() {
      if (this.isOpen) {
        return
      }
      this.isOpen = true
      this.$nextTick(() => this.setupPopper())
    },
    close() {
      if (!this.isOpen) {
        return
      }
      this.isOpen = false
    },
    onMouseover() {
      if (this.trigger === 'hover') {
        if (this.hoverDelay) {
          this.hoverTimer = setTimeout(() => {
            this.open()
          }, Number(this.hoverDelay) * 1000)
        } else {
          this.open()
        }
      }
    },
    onMouseleave() {
      if (this.hoverTimer) {
        clearTimeout(this.hoverTimer)
      }
      if (this.trigger === 'hover') {
        this.close()
      }
    },
  },
}
</script>
