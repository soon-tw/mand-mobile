<template>
  <div class="md-captcha" v-show="isView || value || visible">
    <template v-if="isView">
      <div class="md-captcha-content">
        <h2 class="md-captcha-title" v-if="title" v-text="title"></h2>
        <div>
          <slot></slot>
        </div>
        <md-button
          v-if="count"
          type="ghost"
          size="small"
          v-text="counterText"
          :disabled="this.isCounting"
          @click.native="$_onClickResend"
        ></md-button>
      </div>
      <md-codebox
        ref="codebox"
        v-model="code"
        :maxlength="maxlength"
        :system="system"
        :closable="false"
        :isView="isView"
        :mask="mask"
        :autofocus="false"
        @submit="$_onSubmit"
      />
    </template>
    <template v-else>
      <md-dialog
        :value="value"
        :closable="true"
        :appendTo="false"
        position="center"
        @input="$_onInput"
        @show="$_onShow"
        @hide="$_onHide"
      >
        <div class="md-captcha-content">
          <h2 class="md-captcha-title" v-if="title" v-text="title"></h2>
          <div>
            <slot></slot>
          </div>
          <md-button
            v-if="count"
            type="ghost"
            size="small"
            v-text="counterText"
            :disabled="this.isCounting"
            @click.native="$_onClickResend"
          ></md-button>
        </div>
        <md-codebox
          ref="codebox"
          v-model="code"
          :maxlength="maxlength"
          :system="system"
          :closable="false"
          :mask="mask"
          :autofocus="false"
          @submit="$_onSubmit"
        />
      </md-dialog>
    </template>
  </div>
</template>

<script>import Dialog from '../dialog'
import Codebox from '../codebox'
import Button from '../button'

export default {
  name: 'md-captcha',

  components: {
    [Dialog.name]: Dialog,
    [Codebox.name]: Codebox,
    [Button.name]: Button,
  },

  props: {
    title: {
      type: String,
    },
    value: {
      type: Boolean,
      default: false,
    },
    maxlength: {
      type: [Number, String],
      default: 4,
    },
    mask: {
      type: Boolean,
      default: false,
    },
    system: {
      type: Boolean,
      default: false,
    },
    appendTo: {
      default: () => document.body,
    },
    count: {
      type: Number,
      default: 60,
    },
    isView: {
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      code: '',
      visible: false,
      counterText: '发送验证码',
      isCounting: false,
      firstShown: false,
    }
  },

  watch: {
    value(val) {
      if (val) {
        this.code = ''
        if (!this.firstShown) {
          this.firstShown = true
          this.$emit('send', this.countdown)
        }
      }
    },
  },

  mounted() {
    if (this.appendTo && !this.isView) {
      this.appendTo.appendChild(this.$el)
    }
    if (this.value) {
      this.firstShown = true
      this.$emit('send', this.countdown)
    }
  },

  methods: {
    // MARK: events handler, 如 $_onButtonClick
    $_onInput(val) {
      this.$emit('input', val)
    },
    $_onShow() {
      this.visible = true
      this.$refs.codebox.focus()
      this.$emit('show')
    },
    $_onHide() {
      this.visible = false
      this.$refs.codebox.blur()
      this.$emit('hide')
    },
    $_onSubmit(code) {
      this.$emit('submit', code)
      this.resetcount()
    },
    $_onClickResend() {
      this.$emit('send', this.countdown)
    },
    // MARK: public methods
    countdown() {
      clearInterval(this.__counter__)
      let i = this.count - 1
      this.isCounting = true
      this.counterText = `${i}s后重发`
      this.__counter__ = setInterval(() => {
        if (i === 0) {
          this.resetcount()
        } else {
          i--
          this.counterText = `${i}s后重发`
        }
      }, 1000)
    },
    resetcount() {
      this.isCounting = false
      this.counterText = '发送验证码'
      clearInterval(this.__counter__)
    },
  },
}
</script>

<style lang="stylus">
  .md-captcha
    .md-dialog .md-dialog-content
      margin-bottom number-keyboard-height
    .md-captcha-content
      text-align center
      margin-bottom 20px
      font-size 24px
      .md-captcha-title
        color color-text-base
        font-size 32px
        margin-bottom 15px
      .md-button
        margin 0.34rem auto
</style>
