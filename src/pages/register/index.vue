<template>
  <view class="register-page">
    <image class="register-page__background" src="/static/login/background.png" mode="aspectFill" />

    <button class="nav-back" hover-class="register-page__tap-hover" @tap="handleBack">
      <image class="nav-back__icon" src="/static/login/back.svg" mode="aspectFit" />
    </button>

    <view class="register-hero">
      <text class="register-hero__title">欢迎加入</text>
      <text class="register-hero__subtitle">开启真假药材鉴别之旅</text>
    </view>

    <view class="register-card">
      <text class="register-card__title">注册</text>
      <view class="register-card__divider" />

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
          hover-class="register-page__tap-hover"
          @tap="handleGetCode"
        >
          {{ codeButtonText }}
        </button>
      </view>

      <view class="form-field form-field--password">
        <input
          v-model="password"
          class="form-field__input form-field__input--password"
          type="text"
          :password="!isPasswordVisible"
          maxlength="20"
          placeholder="设置登录密码"
          placeholder-class="form-field__placeholder"
        />
        <button class="password-button" hover-class="register-page__tap-hover" @tap="togglePasswordVisible">
          <image
            class="password-button__icon"
            :src="isPasswordVisible ? '/static/register/eye-open.svg' : '/static/register/eye-off.svg'"
            mode="aspectFit"
          />
        </button>
      </view>

      <view class="form-field form-field--confirm">
        <input
          v-model="confirmPassword"
          class="form-field__input form-field__input--password"
          type="text"
          :password="!isConfirmPasswordVisible"
          maxlength="20"
          placeholder="再次输入密码"
          placeholder-class="form-field__placeholder"
        />
        <button class="password-button" hover-class="register-page__tap-hover" @tap="toggleConfirmPasswordVisible">
          <image
            class="password-button__icon"
            :src="isConfirmPasswordVisible ? '/static/register/eye-open.svg' : '/static/register/eye-off.svg'"
            mode="aspectFit"
          />
        </button>
      </view>

      <button
        class="register-button"
        :class="{ 'register-button--loading': isRegistering }"
        hover-class="register-page__tap-hover"
        @tap="handleRegister"
      >
        注册
      </button>

      <view class="login-entry">
        <text class="login-entry__text">已有账号？</text>
        <text class="login-entry__link" @tap="openLogin">去登录</text>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { computed, onUnmounted, ref } from 'vue'

const phone = ref('')
const verifyCode = ref('')
const password = ref('')
const confirmPassword = ref('')
const countdown = ref(0)
const isPasswordVisible = ref(false)
const isConfirmPasswordVisible = ref(false)
const isRegistering = ref(false)

let countdownTimer: ReturnType<typeof setInterval> | undefined
let registerTimer: ReturnType<typeof setTimeout> | undefined

const isCountingDown = computed(() => countdown.value > 0)
const codeButtonText = computed(() => (isCountingDown.value ? `${countdown.value}s` : '获取验证码'))

function handleBack() {
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

function handleRegister() {
  if (isRegistering.value) {
    return
  }

  const phoneValue = phone.value.trim()
  const codeValue = verifyCode.value.trim()
  const passwordValue = password.value.trim()
  const confirmPasswordValue = confirmPassword.value.trim()

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

  if (!passwordValue) {
    uni.showToast({
      title: '请设置登录密码',
      icon: 'none',
    })
    return
  }

  if (!confirmPasswordValue) {
    uni.showToast({
      title: '请再次输入密码',
      icon: 'none',
    })
    return
  }

  if (passwordValue !== confirmPasswordValue) {
    uni.showToast({
      title: '两次密码不一致',
      icon: 'none',
    })
    return
  }

  isRegistering.value = true
  uni.showToast({
    title: '注册成功',
    icon: 'success',
    duration: 800,
  })

  registerTimer = setTimeout(() => {
    uni.reLaunch({
      url: '/pages/index/index',
    })
    isRegistering.value = false
  }, 650)
}

function togglePasswordVisible() {
  isPasswordVisible.value = !isPasswordVisible.value
}

function toggleConfirmPasswordVisible() {
  isConfirmPasswordVisible.value = !isConfirmPasswordVisible.value
}

function openLogin() {
  const pages = getCurrentPages()
  const previousPage = pages[pages.length - 2]

  if (previousPage?.route === 'pages/login/index') {
    uni.navigateBack({ delta: 1 })
    return
  }

  uni.redirectTo({
    url: '/pages/login/index',
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

  if (registerTimer) {
    clearTimeout(registerTimer)
  }
})
</script>

<style>
page {
  background: #ffffff;
}

.register-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  overflow: hidden;
  background: #ffffff;
  color: #000000;
  box-sizing: border-box;
  font-family: "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.register-page view,
.register-page text,
.register-page image,
.register-page input,
.register-page button {
  box-sizing: border-box;
}

.register-page button {
  margin: 0;
  padding: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  color: inherit;
  font: inherit;
  line-height: normal;
}

.register-page button::after {
  border: 0;
}

.register-page__background {
  position: absolute;
  top: -2rpx;
  left: -31rpx;
  z-index: 0;
  display: block;
  width: 779rpx;
  height: 1626rpx;
}

.register-page__tap-hover {
  opacity: 0.68;
}

.register-page .nav-back {
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

.nav-back__icon {
  display: block;
  width: 63rpx;
  height: 63rpx;
}

.register-hero {
  position: absolute;
  top: 330rpx;
  left: 90rpx;
  z-index: 2;
  display: flex;
  flex-direction: column;
  width: 560rpx;
}

.register-hero__title {
  display: block;
  width: 284rpx;
  height: 80rpx;
  color: #000000;
  font-size: 69rpx;
  font-weight: 700;
  line-height: 80rpx;
  white-space: nowrap;
}

.register-hero__subtitle {
  display: block;
  width: 430rpx;
  height: 44rpx;
  margin-top: 20rpx;
  color: #000000;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 44rpx;
  white-space: nowrap;
}

.register-card {
  position: absolute;
  top: 471rpx;
  left: 90rpx;
  z-index: 3;
  width: 569rpx;
  height: 830rpx;
  overflow: hidden;
  border-radius: 29rpx;
  background: #ffffff;
}

.register-card__title {
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

.register-card__divider {
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

.form-field--password {
  top: 445rpx;
  padding-left: 26rpx;
}

.form-field--confirm {
  top: 565rpx;
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

.form-field__input--password {
  padding-right: 10rpx;
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

.register-page .code-button {
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

.register-page .code-button--counting {
  background: #668468;
}

.register-page .password-button {
  flex: 0 0 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 64rpx;
  height: 72rpx;
  margin-right: 5rpx;
}

.password-button__icon {
  display: block;
  width: 38rpx;
  height: 38rpx;
}

.register-page .register-button {
  position: absolute;
  top: 685rpx;
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

.register-page .register-button--loading {
  background: #376f46;
}

.login-entry {
  position: absolute;
  top: 773rpx;
  left: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 569rpx;
  height: 25rpx;
}

.login-entry__text,
.login-entry__link {
  display: block;
  font-size: 19rpx;
  font-weight: 400;
  line-height: 25rpx;
  white-space: nowrap;
}

.login-entry__text {
  color: #000000;
}

.login-entry__link {
  color: #1c840e;
}
</style>
