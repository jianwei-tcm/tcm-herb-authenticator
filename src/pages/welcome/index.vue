<template>
  <view class="welcome-page">
    <image class="welcome-page__top-ink" :src="assets.topInk" mode="aspectFill" />
    <image class="welcome-page__brand-title" :src="assets.brandTitle" mode="aspectFill" />
    <text class="welcome-page__subtitle">AI识别 · 辨真假 · 学药材</text>
    <image class="welcome-page__herb-scene" :src="assets.herbScene" mode="aspectFill" />

    <view class="welcome-page__actions">
      <view class="welcome-page__button welcome-page__button--login" @tap="openLogin">
        <text>登录</text>
      </view>
      <view class="welcome-page__button welcome-page__button--register" @tap="openRegister">
        <text>注册新账号</text>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
const assets = {
  topInk: '/static/welcome/top-ink.png',
  brandTitle: '/static/welcome/brand-title.png',
  herbScene: '/static/welcome/herb-scene.png',
} as const

function openLogin() {
  uni.navigateTo({
    url: '/pages/login/index',
  })
}

function openRegister() {
  uni.navigateTo({
    url: '/pages/register/index',
  })
}
</script>

<style>
page {
  background: #fff3e6;
}

.welcome-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  overflow: hidden;
  background: #fff3e6;
  color: #4a4a4a;
  box-sizing: border-box;
  font-family: "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.welcome-page view,
.welcome-page text,
.welcome-page image {
  box-sizing: border-box;
}

.welcome-page__top-ink {
  position: absolute;
  top: 0;
  left: -34rpx;
  display: block;
  width: 817rpx;
  height: 361rpx;
}

.welcome-page__brand-title {
  position: absolute;
  top: 374rpx;
  left: -21rpx;
  display: block;
  width: 790rpx;
  height: 187rpx;
}

.welcome-page__subtitle {
  position: absolute;
  top: 574rpx;
  left: 210rpx;
  display: block;
  width: 330rpx;
  height: 38rpx;
  color: #4a4a4a;
  font-size: 31rpx;
  font-weight: 500;
  line-height: 38rpx;
  text-align: center;
  white-space: nowrap;
}

.welcome-page__herb-scene {
  position: absolute;
  top: 678rpx;
  left: 0;
  display: block;
  width: 750rpx;
  height: 510rpx;
}

.welcome-page__actions {
  position: absolute;
  top: 1248rpx;
  left: 95rpx;
  display: flex;
  flex-direction: column;
  width: 557rpx;
}

.welcome-page__button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 557rpx;
  height: 109rpx;
  border-radius: 19rpx;
}

.welcome-page__button text {
  display: block;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 38rpx;
  white-space: nowrap;
}

.welcome-page__button--login {
  background: #1b4925;
}

.welcome-page__button--login text {
  color: #ffffff;
  font-family: "Source Serif Pro", "Source Han Serif SC", "Songti SC", serif;
}

.welcome-page__button--register {
  margin-top: 36rpx;
  border: 2rpx solid #0e6621;
  background: transparent;
}

.welcome-page__button--register text {
  color: #0e6621;
}
</style>
