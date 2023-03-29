<template>
  <form class="el-form" :class="[labelPosition ? 'el-form--label-' + labelPosition : '', { 'el-form--inline': inline }]">
    <slot></slot>
  </form>
</template>
<script>
import objectAssign from 'element-ui/src/utils/merge'

export default {
  name: 'ElForm',

  componentName: 'ElForm',

  provide() {
    return {
      elForm: this
    }
  },

  props: {
    model: Object, // form 表单值的接受兑现
    rules: Object, // form表单值的创建规则
    labelPosition: String, // 确定label 定位的位置，左对齐，右对齐，顶部对齐
    labelWidth: String, // label 的宽度
    // label的名称后面 增加的后缀
    labelSuffix: {
      type: String,
      default: ''
    },
    inline: Boolean, // inline是否是行内展示
    inlineMessage: Boolean, // 报错信息展示形式
    statusIcon: Boolean, // 是否展示校验状态图标
    // 展示校验错误信息
    showMessage: {
      type: Boolean,
      default: true
    },
    size: String, // form表单尺寸
    disabled: Boolean, // 禁用表单全部组件
    // 	是否在 rules 属性改变后立即触发一次验证
    validateOnRuleChange: {
      type: Boolean,
      default: true
    },
    // 是否隐藏必填字段的标签旁边的红色星号
    hideRequiredAsterisk: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    rules() {
      // remove then add event listeners on form-item after form rules change
      this.fields.forEach(field => {
        field.removeValidateEvents()
        field.addValidateEvents()
      })

      if (this.validateOnRuleChange) {
        this.validate(() => {})
      }
    }
  },
  computed: {
    // 自动计算lable宽度
    autoLabelWidth() {
      if (!this.potentialLabelWidthArr.length) return 0
      const max = Math.max(...this.potentialLabelWidthArr)
      return max ? `${max}px` : ''
    }
  },
  data() {
    return {
      fields: [], // 需要检验的props
      potentialLabelWidthArr: [] // use this array to calculate auto width
    }
  },
  created() {
    this.$on('el.form.addField', field => {
      if (field) {
        this.fields.push(field)
      }
    })
    /* istanbul ignore next */
    this.$on('el.form.removeField', field => {
      if (field.prop) {
        this.fields.splice(this.fields.indexOf(field), 1)
      }
    })
  },
  methods: {
    resetFields() {
      if (!this.model) {
        console.warn('[Element Warn][Form]model is required for resetFields to work.')
        return
      }
      this.fields.forEach(field => {
        field.resetField()
      })
    },
    clearValidate(props = []) {
      const fields = props.length ? (typeof props === 'string' ? this.fields.filter(field => props === field.prop) : this.fields.filter(field => props.indexOf(field.prop) > -1)) : this.fields
      fields.forEach(field => {
        field.clearValidate()
      })
    },
    validate(callback) {
      if (!this.model) {
        console.warn('[Element Warn][Form]model is required for validate to work!')
        return
      }

      let promise
      // if no callback, return promise
      if (typeof callback !== 'function' && window.Promise) {
        promise = new window.Promise((resolve, reject) => {
          callback = function(valid) {
            valid ? resolve(valid) : reject(valid)
          }
        })
      }

      let valid = true
      let count = 0
      // 如果需要验证的fields为空，调用验证时立刻返回callback
      if (this.fields.length === 0 && callback) {
        callback(true)
      }
      let invalidFields = {}
      this.fields.forEach(field => {
        field.validate('', (message, field) => {
          if (message) {
            valid = false
          }
          invalidFields = objectAssign({}, invalidFields, field)
          if (typeof callback === 'function' && ++count === this.fields.length) {
            callback(valid, invalidFields)
          }
        })
      })

      if (promise) {
        return promise
      }
    },
    validateField(props, cb) {
      props = [].concat(props)
      const fields = this.fields.filter(field => props.indexOf(field.prop) !== -1)
      if (!fields.length) {
        console.warn('[Element Warn]please pass correct props!')
        return
      }

      fields.forEach(field => {
        field.validate('', cb)
      })
    },
    getLabelWidthIndex(width) {
      const index = this.potentialLabelWidthArr.indexOf(width)
      // it's impossible
      if (index === -1) {
        throw new Error('[ElementForm]unpected width ', width)
      }
      return index
    },
    registerLabelWidth(val, oldVal) {
      if (val && oldVal) {
        const index = this.getLabelWidthIndex(oldVal)
        this.potentialLabelWidthArr.splice(index, 1, val)
      } else if (val) {
        this.potentialLabelWidthArr.push(val)
      }
    },
    deregisterLabelWidth(val) {
      const index = this.getLabelWidthIndex(val)
      this.potentialLabelWidthArr.splice(index, 1)
    }
  }
}
</script>
