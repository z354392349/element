<script>
export default {
  props: {
    isAutoWidth: Boolean,
    updateAll: Boolean
  },

  inject: ['elForm', 'elFormItem'],

  render() {
    const slots = this.$slots.default
    if (!slots) return null
    if (this.isAutoWidth) {
      const autoLabelWidth = this.elForm.autoLabelWidth
      const style = {}
      if (autoLabelWidth && autoLabelWidth !== 'auto') {
        const marginLeft = parseInt(autoLabelWidth, 10) - this.computedWidth
        if (marginLeft) {
          style.marginLeft = marginLeft + 'px'
        }
      }
      return (
        <div class="el-form-item__label-wrap" style={style}>
          {slots}
        </div>
      )
    } else {
      return slots[0]
    }
  },

  methods: {
    // 获取 el-form-item__label-wrap 的宽度。
    getLabelWidth() {
      if (this.$el && this.$el.firstElementChild) {
        const computedWidth = window.getComputedStyle(this.$el.firstElementChild).width
        return Math.ceil(parseFloat(computedWidth))
      } else {
        return 0
      }
    },

    // 只有在 isAutoWidth 为 true 时才会调用，也就是el-from的 lable-width 为 auto 时才会调用.
    // firstElementChild指的是 el-form-item__label-wrap，如果 isAUTOWidth 为false，el-form-item__label-wrap 也不会不存在
    updateLabelWidth(action = 'update') {
      if (this.$slots.default && this.isAutoWidth && this.$el.firstElementChild) {
        if (action === 'update') {
          this.computedWidth = this.getLabelWidth()
        } else if (action === 'remove') {
          this.elForm.deregisterLabelWidth(this.computedWidth)
        }
      }
    }
  },

  watch: {
    computedWidth(val, oldVal) {
      if (this.updateAll) {
        this.elForm.registerLabelWidth(val, oldVal)
        this.elFormItem.updateComputedLabelWidth(val)
      }
    }
  },

  data() {
    return {
      computedWidth: 0
    }
  },

  mounted() {
    this.updateLabelWidth('update')
  },

  updated() {
    this.updateLabelWidth('update')
  },

  beforeDestroy() {
    this.updateLabelWidth('remove')
  }
}
</script>
