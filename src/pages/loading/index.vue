<template>
  <view class="loading-page">
    <image class="loading-page__background" src="/static/loading-background.png" mode="scaleToFill" />

    <view class="loading-indicator">
      <text class="loading-indicator__text">Loading....</text>
      <view class="loading-indicator__track">
        <view class="loading-indicator__fill" :style="{ width: `${progress}%` }" />
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'

type ResultVariant = 'unrecognized' | 'authentic' | 'counterfeit' | 'uncertain'

const pendingImageStorageKey = 'identification-pending-image'
const resultStorageKey = 'identification-result-mock'

// 修改此值即可在开发测试时切换到结果页支持的四种状态。
const mockResultStatus: ResultVariant = 'authentic'

const progress = ref(0)
let progressTimer: ReturnType<typeof setInterval> | undefined
let redirectTimer: ReturnType<typeof setTimeout> | undefined

onMounted(() => {
  progressTimer = setInterval(() => {
    progress.value = Math.min(progress.value + 1, 100)

    if (progress.value >= 100) {
      stopProgressTimer()
      redirectTimer = setTimeout(goToResult, 240)
    }
  }, 60)
})

onBeforeUnmount(() => {
  stopProgressTimer()

  if (redirectTimer) {
    clearTimeout(redirectTimer)
    redirectTimer = undefined
  }
})

function stopProgressTimer() {
  if (progressTimer) {
    clearInterval(progressTimer)
    progressTimer = undefined
  }
}

function goToResult() {
  const image = uni.getStorageSync(pendingImageStorageKey)

  uni.setStorageSync(resultStorageKey, {
    variant: mockResultStatus,
    ...(typeof image === 'string' && image ? { image } : {}),
  })
  uni.removeStorageSync(pendingImageStorageKey)

  uni.redirectTo({
    url: '/pages/result/index',
  })
}
</script>

<style>
page {
  background: #fff9f3;
}

.loading-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  margin: 0 auto;
  overflow: hidden;
  background: #fff9f3;
  color: #000000;
  font-family: "Noto Sans SC", "Microsoft YaHei", Arial, sans-serif;
}

.loading-page__background {
  position: absolute;
  top: 0;
  left: 0;
  width: 750rpx;
  height: 1626rpx;
  display: block;
}

.loading-indicator {
  position: absolute;
  top: 601rpx;
  left: 80rpx;
  width: 590rpx;
  height: 128rpx;
}

.loading-indicator__text {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  color: #000000;
  font-size: 34rpx;
  font-weight: 700;
  line-height: 42rpx;
  text-align: center;
  white-space: nowrap;
}

.loading-indicator__track {
  position: absolute;
  top: 52rpx;
  left: 0;
  width: 100%;
  height: 76rpx;
  box-sizing: border-box;
  overflow: hidden;
  border: 2rpx solid #9cbfa4;
  border-radius: 38rpx;
  background: #ffffff;
}

.loading-indicator__fill {
  position: absolute;
  top: 6rpx;
  left: 6rpx;
  height: 64rpx;
  max-width: calc(100% - 12rpx);
  border-radius: 32rpx;
  background: #2e6f3c;
  transition: width 60ms linear;
}
</style>
