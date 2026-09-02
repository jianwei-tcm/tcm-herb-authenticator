<template>
  <view class="login-page">
    <image class="login-page__background" src="/static/login/background.png" mode="aspectFill" />

    <button class="nav-back" hover-class="nav-back--hover" @tap="handleBack">
      <image class="nav-back__icon" src="/static/login/back.svg" mode="aspectFit" />
    </button>

    <view class="login-hero">
      <text class="login-hero__title">欢迎回来</text>
      <text class="login-hero__subtitle">登录后探索更多精彩内容</text>
    </view>

    <view class="login-card">
      <text class="login-card__title">登录</text>
      <view class="login-card__divider" />

      <view class="form-field form-field--phone">
        <input
          v-model="phone"
          class="form-field__input"
          type="number"
          maxlength="11"
          placeholder="请输入手机号"
          placeholder-class="form-field__placeholder"
        />
        <image class="form-field__phone-icon" src="/static/login/phone.svg" mode="aspectFit" />
      </view>

      <view class="form-field form-field--code">
        <input
          v-model="verifyCode"
          class="form-field__input form-field__input--code"
          type="number"
          maxlength="6"
          placeholder="请输入验证码"
          placeholder-class="form-field__placeholder"
        />
        <button
          class="code-button"
          :class="{ 'code-button--counting': isCountingDown }"
          hover-class="code-button--hover"
          @tap="handleGetCode"
        >
          {{ codeButtonText }}
        </button>
      </view>

      <button
        class="login-button"
        :class="{ 'login-button--loading': isLoggingIn }"
        hover-class="login-button--hover"
        @tap="handleLogin"
      >
        {{ isLoggingIn ? '登录中...' : '登录' }}
      </button>

      <view class="register-entry">
        <text class="register-entry__text">还没有账号？去</text>
        <text class="register-entry__link" @tap="openRegister">注册</text>
      </view>

      <view class="other-login">
        <view class="other-login__line" />
        <text class="other-login__title">其他登录方式</text>
        <view class="other-login__line" />
      </view>

      <view class="social-list">
        <button
          v-for="item in socialLoginItems"
          :key="item.key"
          class="social-item"
          :class="`social-item--${item.key}`"
          hover-class="social-item--hover"
          @tap="handleSocialLogin(item.label)"
        >
          <view class="social-item__icon-area">
            <image
              v-if="item.backgroundIcon"
              class="social-item__icon-bg"
              :src="item.backgroundIcon"
              mode="aspectFit"
            />
            <image
              class="social-item__icon"
              :class="`social-item__icon--${item.key}`"
              :src="item.icon"
              mode="aspectFit"
            />
          </view>
          <text class="social-item__label">{{ item.label }}</text>
        </button>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { computed, onUnmounted, ref } from 'vue'

type SocialLoginKey = 'qq' | 'account' | 'wechat'

interface SocialLoginItem {
  key: SocialLoginKey
  label: string
  icon: string
  backgroundIcon?: string
}

const phone = ref('')
const verifyCode = ref('')
const countdown = ref(0)
const isLoggingIn = ref(false)

let countdownTimer: ReturnType<typeof setInterval> | undefined
let loginTimer: ReturnType<typeof setTimeout> | undefined

const socialLoginItems: SocialLoginItem[] = [
  { key: 'qq', label: 'QQ登录', icon: '/static/login/qq.png' },
  {
    key: 'account',
    label: '账号登录',
    icon: '/static/login/account.svg',
    backgroundIcon: '/static/login/account-bg.svg',
  },
  { key: 'wechat', label: '微信登录', icon: '/static/login/wechat.png' },
]

const isCountingDown = computed(() => countdown.value > 0)
const codeButtonText = computed(() => (isCountingDown.value ? `${countdown.value}s` : '获取验证码'))

function handleBack() {
  const pages = getCurrentPages()

  if (pages.length > 1) {
    uni.navigateBack({ delta: 1 })
    return
  }

  uni.reLaunch({
    url: '/pages/welcome/index',
  })
}

function handleGetCode() {
  if (isCountingDown.value) {
    return
  }

  if (!phone.value.trim()) {
    uni.showToast({
      title: '请输入手机号',
      icon: 'none',
    })
    return
  }

  countdown.value = 60
  uni.showToast({
    title: '验证码已发送',
    icon: 'none',
  })

  countdownTimer = setInterval(() => {
    countdown.value -= 1

    if (countdown.value <= 0) {
      clearCountdownTimer()
    }
  }, 1000)
}

function handleLogin() {
  if (isLoggingIn.value) {
    return
  }

  const phoneValue = phone.value.trim()
  const codeValue = verifyCode.value.trim()

  if (!phoneValue && !codeValue) {
    uni.showToast({
      title: '请填写手机号和验证码',
      icon: 'none',
    })
    return
  }

  if (!phoneValue) {
    uni.showToast({
      title: '请输入手机号',
      icon: 'none',
    })
    return
  }

  if (!codeValue) {
    uni.showToast({
      title: '请输入验证码',
      icon: 'none',
    })
    return
  }

  isLoggingIn.value = true
  uni.showToast({
    title: '登录成功',
    icon: 'success',
    duration: 800,
  })

  loginTimer = setTimeout(() => {
    uni.reLaunch({
      url: '/pages/index/index',
    })
    isLoggingIn.value = false
  }, 650)
}

function openRegister() {
  uni.navigateTo({
    url: '/pages/register/index',
  })
}

function handleSocialLogin(label: string) {
  uni.showToast({
    title: `${label}暂未接入`,
    icon: 'none',
  })
}

function clearCountdownTimer() {
  if (countdownTimer) {
    clearInterval(countdownTimer)
    countdownTimer = undefined
  }

  countdown.value = 0
}

onUnmounted(() => {
  clearCountdownTimer()

  if (loginTimer) {
    clearTimeout(loginTimer)
  }
})
</script>

<style>
page {
  background: #ffffff;
}

.login-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  overflow: hidden;
  background: #ffffff;
  color: #000000;
  box-sizing: border-box;
  font-family: "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.login-page view,
.login-page text,
.login-page image,
.login-page input,
.login-page button {
  box-sizing: border-box;
}

.login-page button {
  margin: 0;
  padding: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  color: inherit;
  font: inherit;
  line-height: normal;
}

.login-page button::after {
  border: 0;
}

.login-page__background {
  position: absolute;
  top: -2rpx;
  left: -31rpx;
  z-index: 0;
  display: block;
  width: 779rpx;
  height: 1626rpx;
}

.login-page .nav-back {
  position: absolute;
  top: 74rpx;
  left: 27rpx;
  z-index: 6;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 63rpx;
  height: 63rpx;
}

.nav-back--hover,
.code-button--hover,
.login-button--hover,
.social-item--hover {
  opacity: 0.68;
}

.nav-back__icon {
  display: block;
  width: 63rpx;
  height: 63rpx;
}

.login-hero {
  position: absolute;
  top: 330rpx;
  left: 90rpx;
  z-index: 2;
  display: flex;
  flex-direction: column;
  width: 520rpx;
}

.login-hero__title {
  display: block;
  width: 284rpx;
  height: 80rpx;
  color: #000000;
  font-size: 69rpx;
  font-weight: 700;
  line-height: 80rpx;
  white-space: nowrap;
}

.login-hero__subtitle {
  display: block;
  width: 414rpx;
  height: 44rpx;
  margin-top: 20rpx;
  color: #000000;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 44rpx;
  white-space: nowrap;
}

.login-card {
  position: absolute;
  top: 471rpx;
  left: 90rpx;
  z-index: 3;
  width: 569rpx;
  height: 937rpx;
  overflow: hidden;
  border-radius: 29rpx;
  background: #ffffff;
}

.login-card__title {
  position: absolute;
  top: 84rpx;
  left: 0;
  display: block;
  width: 569rpx;
  height: 48rpx;
  color: #000000;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 48rpx;
  text-align: center;
  white-space: nowrap;
}

.login-card__divider {
  position: absolute;
  top: 158rpx;
  left: 0;
  width: 569rpx;
  height: 1rpx;
  background: #000000;
}

.form-field {
  position: absolute;
  left: 57rpx;
  display: flex;
  align-items: center;
  width: 452rpx;
  height: 76rpx;
  border: 2rpx solid #000000;
  border-radius: 19rpx;
  background: #fdf9f6;
}

.form-field--phone {
  top: 204rpx;
  padding: 0 8rpx 0 26rpx;
}

.form-field--code {
  top: 324rpx;
  padding-left: 26rpx;
}

.form-field__input {
  flex: 1;
  width: 0;
  height: 72rpx;
  padding: 0;
  color: #000000;
  font-size: 27rpx;
  font-weight: 400;
  line-height: 72rpx;
}

.form-field__input--code {
  padding-right: 6rpx;
}

.form-field__placeholder {
  color: #c2c1bf;
  font-size: 27rpx;
}

.form-field__phone-icon {
  flex: 0 0 auto;
  display: block;
  width: 46rpx;
  height: 46rpx;
  margin-left: 8rpx;
}

.login-page .code-button {
  flex: 0 0 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 162rpx;
  height: 42rpx;
  margin-right: 13rpx;
  border-radius: 8rpx;
  background: #1b4925;
  color: #ffffff;
  font-size: 27rpx;
  font-weight: 400;
  line-height: 42rpx;
  white-space: nowrap;
}

.login-page .code-button--counting {
  background: #668468;
}

.login-page .login-button {
  position: absolute;
  top: 445rpx;
  left: 57rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 452rpx;
  height: 76rpx;
  border-radius: 19rpx;
  background: #1b4925;
  color: #ffffff;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 38rpx;
  white-space: nowrap;
}

.login-page .login-button--loading {
  background: #376f46;
}

.register-entry {
  position: absolute;
  top: 533rpx;
  left: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 569rpx;
  height: 25rpx;
}

.register-entry__text,
.register-entry__link {
  display: block;
  font-size: 19rpx;
  font-weight: 400;
  line-height: 25rpx;
  white-space: nowrap;
}

.register-entry__text {
  color: #000000;
}

.register-entry__link {
  color: #1c840e;
}

.other-login {
  position: absolute;
  top: 619rpx;
  left: 57rpx;
  display: flex;
  align-items: center;
  width: 452rpx;
  height: 32rpx;
}

.other-login__line {
  flex: 1;
  height: 1rpx;
  background: #000000;
}

.other-login__title {
  display: block;
  margin: 0 4rpx;
  color: #000000;
  font-size: 23rpx;
  font-weight: 400;
  line-height: 32rpx;
  white-space: nowrap;
}

.social-list {
  position: absolute;
  top: 683rpx;
  left: 57rpx;
  width: 452rpx;
  height: 178rpx;
}

.login-page .social-item {
  position: absolute;
  top: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 120rpx;
  height: 178rpx;
}

.login-page .social-item--qq {
  left: 0;
}

.login-page .social-item--account {
  left: 164rpx;
}

.login-page .social-item--wechat {
  left: 332rpx;
}

.social-item__icon-area {
  position: relative;
  width: 120rpx;
  height: 111rpx;
}

.social-item__icon-bg {
  position: absolute;
  top: 3rpx;
  left: 8rpx;
  display: block;
  width: 105rpx;
  height: 105rpx;
}

.social-item__icon {
  position: absolute;
  display: block;
}

.social-item__icon--qq,
.social-item__icon--wechat {
  top: 0;
  left: 0;
  width: 120rpx;
  height: 111rpx;
}

.social-item__icon--account {
  top: 18rpx;
  left: 25rpx;
  width: 69rpx;
  height: 71rpx;
}

.social-item__label {
  display: block;
  margin-top: 13rpx;
  color: #1b4925;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 38rpx;
  text-align: center;
  white-space: nowrap;
}

.social-item--account .social-item__label {
  color: #000000;
}
</style>
