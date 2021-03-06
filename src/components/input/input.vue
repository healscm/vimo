<template>
    <div class="ion-input" :class="[modeClass,inputClass,{'clearInput':clearInput}]" @click="$_clickToFocus($event)">
        <div class="input-inner-wrap">
            <input ref="input"
                   :class="[textInputClass]"
                   :value="inputValue"
                   :type="typeValue"
                   :placeholder="placeholder"
                   :disabled="disabled"
                   :readonly="readonly"
                   :max="max"
                   :min="min"
                   :step="step"
                   :autofocus="autofocus"
                   @keyup="$_inputKeyUp($event)"
                   @blur="$_inputBlurred($event)"
                   @focus="$_inputFocused($event)"
                   @input="$_inputChanged($event)"
                   @keydown="$_inputKeyDown($event)">
        </div>
        <vm-button v-if="clearInput && hasValue"
                   clear
                   class="text-input-clear-icon"
                   @click="$_clearTextInput()"></vm-button>
    </div>
</template>
<style lang="scss" src="./style.scss"></style>
<script type="text/javascript">
  /**
   * @component Input
   * @description
   *
   * ## 表单组件 Input输入框
   *
   * ### 注意
   *
   * Input组件只能对以下类型的type作出相应 : `text`,`password`, `email`, `number`, `search`, `tel`, and `url`. 但是不适用一下类型: `checkbox`, `radio`, `toggle`, `range`, `select`, etc.
   *
   * ### 如何引入
   * ```
   * // 引入
   * import { Input } from 'vimo'
   * // 安装
   * Vue.component(Input.name, Input)
   * // 或者
   * export default{
   *   components: {
   *     Input
   *  }
   * }
   * ```
   *
   * ### 关于输入验证
   *
   * - 只在blur阶段才进行
   * - ```check```默认关闭, ```check```只是作为内部正误标示, 只有开启了检查, 才会发出```onValid```和```onInvalid```两个事件, 外部提交判断需要额外代码判断(内部```isValid```变量).
   * - 如果```check```开启, 但是regex无值, 则使用内置判断(如下).
   * - 如果```regex```有值, 则自动开启```check```, 支持的regex可以使正则, 也可以是返回Boolena的函数, 传入参数为传入的value
   * - 内部验证的typ如下
   *
   * ### 内置的验证type
   *
   * 名称    | 类型              |内部类型              | 说明
   * ------|-----------------|------------------------------------------------------------------------------
   * 整数    | integer         | number| 整数
   * 正整数   | positiveInteger |number| 正整数
   * 负整数   | negativeInteger |number| 负整数
   * 邮箱    | email           |email| 电子邮件
   * IP地址  | ip              |number| IP地址
   * 身份证   | idCard          | text| 严格验证
   * 密码    | password        | password|密码需6-18位，以字母开头可含数字
   * 国内电话  | tel             | tel|正确格式为：XXXX-XXXXXXX，XXXX- XXXXXXXX，XXX-XXXXXXX，XXX-XXXXXXXX，XXXXXXX，XXXXXXXX。
   * 国内手机号 | mobile          | tel|13/14/15/18/17开头
   * 验证汉字  | cn              |text|
   * 验证码   | securityCode    | number|至少4位数字
   * 昵称    | nickName        | text|可由中英文字母、数字、"-"、"_"组成。
   * QQ号码  | qq              | number|qq: 1-9开头，最少5位。
   * 网址URL | url             | url|网址URL, 必须以(https,http,ftp,rtsp,mms)开头
   *
   * @props {Boolean} [clearInput]              - 如果为true, 当输入值的时候一个清除按钮会在input右边出现, 点击按钮则清除输入.
   * @props {Boolean} [clearOnEdit]             - 如果为true, 当再次输入的时候会清空上次的输入, 如果type为password时默认为true, 其余情况默认为false, 默认值的变更, 需要js控制
   * @props {Boolean} [disabled]                - 如果为true, 用户无法输入
   * @props {Boolean} [showFocusHighlight]      - focus时底部是否 highlight 显示
   * @props {Boolean} [showValidHighlight]      - 验证成功是否显示 highlight 显示
   * @props {Boolean} [showInvalidHighlight]    - 验证失败是否显示 highlight 显示
   * @props {Number} [max]                      - 设置最大值, 1. type=number时限制输入数字的大小; 2. type=text时限制输入字符的长度
   * @props {Number} [min]                      - 设置最小值, 1. type=number时限制输入数字的大小; 2. type=text时限制输入字符的长度
   * @props {Number} [decimal=2]                - 设置数字的小数位, 默认为2
   * @props {Number} [step]                     - 设置数字变化的阶梯值, 只对type=number有效
   * @props {String} [mode='ios']               - 当前平台
   * @props {String} [placeholder]              - 占位文字
   * @props {Boolean} [readonly]                - 只读模式, 不能修改
   * @props {String} [type='text']              - 输入的类型: "text", "password", "email", "number", "search", "tel", or "url"
   * @props {*} [value]                         - 内容输入值
   * @props {Number} [debounce=0]               - 触发间隔
   * @props {RegExp} [regex]                    - 自定义正则
   * @props {Boolean} [check]                   - 是否check输入结果, 如果regex有值, 则开启, 否则关闭. 如果check开启, 但是regex无值, 则使用内置判断. 默认关闭check, check只是作为内部正误标示, 对外提交不起作用, 如果点击能知道各个input的状态, 需要在dom中search'ng-invalid'类名, 这样的话, 验证位置就会统一.
   *
   * @fires component:Input#onBlur
   * @fires component:Input#onFocus
   * @fires component:Input#onInput
   * @fires component:Input#onKeyup
   * @fires component:Input#onKeydown
   * @fires component:Input#onValid
   * @fires component:Input#onInvalid
   *
   * @demo #/input
   * @usage
   * <Input placeholder="Text Input">
   * <Input placeholder="Clear Input" clearInput></Input>
   * <Input placeholder="请输入手机号" type="mobile" check clearInput></Input>
   * <Input placeholder="请输入至少4位" type="securityCode" check clearInput></Input>
   * <Input placeholder="XX-XX-XXX格式" type="text" check :regex=/\d{2}-\d{2}-\d{3}/ clearInput></Input>
   * */
  import { hasFocus } from '../../util/util'
  import Button from '../button'
  import REGEXP from '../../util/regexp'
  import debounce from 'lodash.debounce'
  import { isBlank, isFunction, isObject, isPresent, isRegexp } from '../../util/type'

  export default {
    name: 'Input',
    inject: {
      itemComponent: {
        from: 'itemComponent',
        default: null
      }
    },
    components: {
      'vm-button': Button
    },
    props: {
      /**
       * focus时, 下划线是否高亮
       * */
      showFocusHighlight: Boolean,

      /**
       * 验证成功是否显示 highlight
       * */
      showValidHighlight: Boolean,

      /**
       * 验证失败是否显示 highlight
       * */
      showInvalidHighlight: Boolean,

      /**
       * 如果为true, 当输入值的时候一个清除按钮会在input右边出现, 点击按钮则清除输入
       * */
      clearInput: Boolean,

      /**
       * 如果为true, 当再次输入的时候会清空上次的输入, 如果type为password时默认为true, 其余情况默认为false
       * 默认值的变更, 需要js控制
       * */
      clearOnEdit: Boolean,

      /**
       * 如果为true, 用户无法输入
       * */
      disabled: Boolean,

      /**
       * 设置最大值, 用于text/number等类型的长度限制
       * */
      max: Number,

      /**
       * 设置最小值
       * */
      min: Number,

      /**
       * 设置type=number的小数位
       * */
      decimal: {
        type: Number,
        validator (val) {
          return val >= 0
        },
        default: 2
      },

      /**
       * 自动focus
       * */
      autofocus: Boolean,

      /**
       * 设置数字变化的阶梯值, 只对type=number有效
       * */
      step: Number,

      /**
       * 当前平台
       * */
      mode: {
        type: String,
        default () { return this.$config && this.$config.get('mode', 'ios') || 'ios' }
      },

      placeholder: String,

      /**
       * 只读模式, 不能修改
       * */
      readonly: Boolean,

      /**
       * 输入的类型: "text", "password", "email", "number", "search", "tel", or "url"
       * */
      type: {
        type: String,
        default: 'text'
      },

      /**
       * 内容输入值
       * */
      value: [String, Number, Object, Function],

      debounce: {
        type: Number,
        default: 0
      },

      // 自定义输入结果验证的正则表达式
      regex: RegExp,

      /**
       * 是否check输入结果, 如果regex有值, 则开启, 否自关闭.
       * 如果check开启, 但是regex无值, 则使用内置判断
       * 默认关闭check
       * check只是作为内部正误标示, 对外提交不起作用
       * 如果点击能知道各个input的状态, 需要在dom中search'ng-invalid'类名
       * 这样的话, 验证位置就会统一.
       * */
      check: Boolean
    },
    data () {
      return {
        oldInputValue: null,    // 内部value值
        inputValue: this.value, // 内部value值
        typeValue: this.type, // 内部type值
        checkValue: this.check || this.regex, // 内部check值, 判断是否需要验证结果

        isValid: false, // 验证结果

        executeEmit () {},

        clearOnEditValue: this.clearOnEdit, // 内部维护的clearOnEdit副本, 因为会修改的
        didBlurAfterEdit: false, // clearOnEdit状态唤起的标志
        shouldBlur: true, // 点击清楚按钮时使用
        inputClass: {
          'input-has-value': false,
          'input-has-focus': false
        }
      }
    },
    watch: {
      value (val) {
        this.inputValue = val
      },
      inputValue (newVal, oldVal) {

      }
    },
    computed: {
      modeClass () {
        return `input-${this.mode}`
      },
      textInputClass () {
        return `text-input text-input-${this.mode}`
      },
      inputElement () {
        return this.$refs.input
      },
      hasValue () {
        const inputValue = this.inputValue
        return (inputValue !== null && inputValue !== undefined && inputValue !== '')
      }
    },
    methods: {
      /**
       * 设置当前组件为focus状态
       * @public
       * */
      setFocus () {
        if (!hasFocus(this.inputElement)) {
          this.inputElement.focus()
        }
      },

      /**
       * 边界检查
       * @param {String|Number} val - 数值
       * @param {String} text - 对应的string
       * @private
       * */
      $_checkBoundary ($event) {
        let inputText = $event.target.value // text
        let resetValue = null

        // 数字边界限制
        // 这段代码已在很卡顿的安卓机上试验过了, 之所以不在watch阶段重置, 是因为在较慢的安卓机上有数字抖动的情况
        // 现在已能很好的处理
        if (this.typeValue === 'number') {
          resetValue = inputText
          if (isPresent(inputText)) {
            if (isPresent(this.max) && parseFloat(inputText) > this.max) {
              resetValue = this.oldInputValue
            }

            if (isPresent(this.min) && parseFloat(inputText) < this.min) {
              resetValue = this.min
            }

            // 小数点检查, 使用string的方式, number的方式会有奇怪的问题, 比如: 222.22 -> 222.19
            let int = resetValue.toString().split('.')[0]
            let decimals = resetValue.toString().split('.')[1]

            if (decimals && this.decimal > 0) {
              if (decimals.length > this.decimal) {
                decimals = decimals.substr(0, this.decimal)
                resetValue = `${int}.${decimals}`
              }
            }
          }

          if (resetValue !== inputText) {
            $event.target.value = resetValue
          }
        } else {
          resetValue = inputText
          // 非数字 且有 最大长度限制
          if (isPresent(this.max) && isPresent(inputText) && inputText.toString().length > this.max) {
            resetValue = this.oldInputValue
            // 重置 input 输入框
            $event.target.value = resetValue
          }
        }

        return resetValue
      },

      /**
       * @event component:Input#onKeyup
       * @description keyup事件
       * @private
       */
      $_inputKeyUp ($event) {
        this.$emit('onKeyup', $event)
      },

      /**
       * @event component:Input#onKeydown
       * @description keydown事件
       * @private
       */
      $_inputKeyDown ($event) {
        this.$emit('onKeydown', $event)
        if (this.clearOnEditValue) {
          this.$_checkClearOnEdit()
        }

        this.oldInputValue = this.inputValue
      },

      /**
       * 执行验证, 如果错误则设置ng-invalid, 正确则设置ng-valid
       * @private
       * */
      $_verification () {
        // 只有开启才检查
        if (!this.checkValue) return

        this.isValid = this.$_getVerifyResult(this.inputValue, this.type)
        if (this.isValid) {
          /**
           * @event  component:Input#onValid
           * @description 验证通过, 只在check开启或者有regex时判断
           * @property {*} value - 当前检查的value
           * @property {string} type - 当前检查的value的类型
           */
          this.$emit('onValid', this.inputValue, this.type)
          if (this.itemComponent) {
            this.itemComponent.inputClass['ng-valid'] = true
            this.itemComponent.inputClass['ng-invalid'] = false
          }
        } else {
          /**
           * @event  component:Input#onInvalid
           * @description 验证失败, 只在check开启或者有regex时判断
           * @property {*} value - 当前检查的value
           * @property {string} type - 当前检查的value的类型
           */
          this.$emit('onInvalid', this.inputValue, this.type)
          if (this.itemComponent) {
            this.itemComponent.inputClass['ng-valid'] = false
            this.itemComponent.inputClass['ng-invalid'] = true
          }
        }
      },

      /**
       * 获取验证结果
       * @param {*} value - 待验证的值
       * @param {String} type - 待验证的值的类型
       * @return Boolean
       * @private
       * */
      $_getVerifyResult (value, type = 'text') {
        if (!isPresent(value)) {
          // '当前没有值, 验证跳过, 返回 false!'
          return false
        }

        let _regex = this.regex
        if (isBlank(_regex)) {
          let regexpInfo = REGEXP[type]
          if (regexpInfo && (isRegexp(regexpInfo) || isFunction(regexpInfo))) {
            _regex = regexpInfo
          } else if (regexpInfo && (isRegexp(regexpInfo.regexp) || isFunction(regexpInfo.regexp))) {
            _regex = regexpInfo.regexp
          }
        }

        // 如果没有正则信息则返回true, 表示不验证
        if (!isPresent(_regex)) {
          // '未找到匹配type:' + type + '的regex, 验证跳过, 返回 false!'
          return false
        }

        // 如果是函数则执行判断
        if (isFunction(_regex)) {
          return _regex(value)
        }

        // 判断是不是正则
        if (isRegexp(_regex)) {
          return _regex.test(value)
        }

        // 'regex:' + _regex + '不是正则/函数, 验证跳过, 返回 false!'
        return false
      },

      /**
       * 当该组件被点击的时候触发, 扩大focus触发范围
       * @private
       */
      $_clickToFocus () {
        this.setFocus()
      },

      /**
       * 监听并发送blur事件
       * @private
       */
      $_inputBlurred () {
        // debug: clearInput会在onBlur之后,造成blur后点击clearInput失效, 故需要延迟blur
        window.setTimeout(() => {
          if (this.shouldBlur) {
            // 向父组件Item添加标记
            this.$_setItemHasFocusClass(false)

            /**
             * @event component:Input#onBlur
             * @description blur事件
             */
            this.$emit('onBlur')
            // 如果是clearOnEdit模式， blur时还有值的情况下，定一个flag
            if (this.clearOnEditValue && this.hasValue) {
              this.didBlurAfterEdit = true
            }

            // 验证输入结果
            this.$_verification()
          } else {
            this.shouldBlur = true
          }
        }, 16 * 2)
      },

      /**
       * 监听并发送focus事件
       * @private
       */
      $_inputFocused () {
        // 向父组件Item添加标记
        this.$_setItemHasFocusClass(true)
        this.setFocus()
        /**
         * @event  component:Input#onFocus
         * @description focus事件
         */
        this.$emit('onFocus')
        if (this.itemComponent) {
          this.itemComponent.inputClass['ng-touched'] = true
        }
      },

      /**
       * 监听input事件, 更新input的value(inputValue)
       * @param {Event} [$event] - 事件(可选)
       * @private
       */
      $_inputChanged ($event) {
        if ($event && $event.target) {
          // 输入限制检查
          this.inputValue = this.$_checkBoundary($event)
          // debounce
          this.executeEmit()
        } else {
          // clear的情况
          // 需要同步设置input元素的值
          this.inputElement.value = null
          this.inputValue = null
          // 立即发送变化, 不需要debounce
          this.$_emitChange()
        }

        this.$_setItemHasValueClass()
      },

      $_initDebounce () {
        if (this.debounce > 0) {
          return debounce(function () {
            this.$_emitChange()
          }, this.debounce)
        } else {
          return () => {
            this.$_emitChange()
          }
        }
      },

      $_emitChange () {
        /**
         * @event  component:Input#onInput
         * @description input事件
         * @property {*} value - 当前输入的值
         */
        this.$emit('onInput', this.inputValue)
        this.$emit('input', this.inputValue)
      },

      /**
       * @private
       * */
      $_checkClearOnEdit () {
        if (!this.clearOnEditValue) {
          return
        }

        // clearOnEdit模式激活,并且input有值
        if (this.didBlurAfterEdit && this.hasValue) {
          this.$_inputChanged()
        }

        // 重置标记
        this.didBlurAfterEdit = false
      },

      /**
       * 点击清除输入项
       * @private
       * */
      $_clearTextInput () {
        this.inputValue = ''
        this.$_inputChanged()
        this.shouldBlur = false

        this.setFocus()
        this.$_setItemHasFocusClass(true)
      },

      /**
       *  设置父组件Item被点中时的class
       *  @private
       */
      $_setItemHasFocusClass (isFocus) {
        if (this.itemComponent) {
          this.itemComponent.inputClass['input-has-focus'] = isFocus
        }
        this.inputClass['input-has-focus'] = isFocus
      },

      /**
       *  设置父组件Item有值时的class
       *  @private
       */
      $_setItemHasValueClass () {
        if (this.itemComponent) {
          this.itemComponent.inputClass['input-has-value'] = !!this.hasValue
        }
        this.inputClass['input-has-value'] = !!this.hasValue
      }
    },
    created () {
      // 默认情况下, 如果password有值, 则点击执行清空
      if (this.type === 'password') {
        this.clearOnEditValue = true
        this.typeValue = 'password'
      }

      // 根据 REGEXP 匹配 type 的真正规则
      if (isObject(REGEXP[this.type]) && isPresent(REGEXP[this.type].type)) {
        this.typeValue = REGEXP[this.type].type
      } else {
        this.typeValue = this.type
      }

      // 生成emit执行体
      this.executeEmit = this.$_initDebounce()
    },
    mounted () {
      // 找到外部item实例
      if (this.itemComponent) {
        this.itemComponent.inputClass['item-input'] = true
        this.itemComponent.inputClass['show-focus-highlight'] = this.showFocusHighlight
        this.itemComponent.inputClass['show-valid-highlight'] = this.showValidHighlight
        this.itemComponent.inputClass['show-invalid-highlight'] = this.showInvalidHighlight
      }

      // 初始化时,判断是否有value
      this.$_setItemHasValueClass()

      // 手动focus
      if (this.autofocus) {
        window.setTimeout(() => {
          this.setFocus()
        }, 16 * 3)
      }
    }
  }
</script>
