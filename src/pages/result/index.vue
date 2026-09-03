<template>
  <view class="result-page">
    <view class="top-bar">
      <button class="nav-back" hover-class="nav-back--hover" @tap="handleBack">
        <image class="nav-back__icon" src="/static/result/back.svg" mode="aspectFit" />
      </button>
      <text class="top-bar__title">鉴别结果</text>
      <image class="top-bar__action" src="/static/result/share-outline.svg" mode="aspectFit" />
    </view>

    <image class="result-illustration" src="/static/result/unrecognized-illustration.png" mode="scaleToFill" />

    <text class="result-title">{{ resultData.title }}</text>
    <text class="result-message">{{ resultData.message }}</text>

    <view class="advice-card">
      <text class="advice-card__title">{{ resultData.adviceTitle }}</text>
      <text class="advice-card__body">{{ adviceText }}</text>
    </view>

    <view class="action-row">
      <button class="action-button action-button--outline" hover-class="action-button--hover" @tap="handleReupload">
        <image class="action-button__icon" src="/static/result/camera-photo-outline.svg" mode="aspectFit" />
        <text class="action-button__text action-button__text--outline">重新上传</text>
      </button>

      <button class="action-button action-button--solid" hover-class="action-button--hover-solid" @tap="handleSupplement">
        <text class="action-button__text action-button__text--solid">补充信息</text>
      </button>
    </view>
  </view>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { onShow } from '@dcloudio/uni-app'

interface ResultPayload {
  title: string
  message: string
  adviceTitle: string
  adviceLines: string[]
  image?: string
}

const resultStorageKey = 'identification-result-mock'

const defaultResultPayload: ResultPayload = {
  title: '无法识别药材',
  message: '系统未能识别出该药的种类，\n请尝试重新上传清晰的药材图片。',
  adviceTitle: '建议您：',
  adviceLines: ['拍摄清晰、完整的药材主体', '确保光线充足、背景干净', '尝试从不同角度拍摄'],
}

const resultData = ref<ResultPayload>({ ...defaultResultPayload })

const adviceText = computed(() => resultData.value.adviceLines.join('\n'))

onShow(() => {
  const cachedPayload = uni.getStorageSync(resultStorageKey)

  if (isResultPayload(cachedPayload)) {
    resultData.value = {
      ...defaultResultPayload,
      ...cachedPayload,
      adviceLines: normalizeAdviceLines(cachedPayload.adviceLines),
    }
    return
  }

  resultData.value = { ...defaultResultPayload }
})

function isResultPayload(payload: unknown): payload is Partial<ResultPayload> {
  return Boolean(payload) && typeof payload === 'object'
}

function normalizeAdviceLines(value: unknown): string[] {
  if (!Array.isArray(value)) {
    return defaultResultPayload.adviceLines
  }

  const lines = value.filter((item): item is string => typeof item === 'string' && item.length > 0)

  return lines.length > 0 ? lines : defaultResultPayload.adviceLines
}

function handleBack() {
  uni.reLaunch({
    url: '/pages/index/index',
  })
}

function handleSupplement() {
  uni.navigateTo({
    url: '/pages/identification/index',
  })
}

function handleReupload() {
  uni.navigateTo({
    url: '/pages/camera/index',
  })
}
</script>

<style>
page {
  background: #ffffff;
}

button {
  margin: 0;
  padding: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  color: inherit;
  font: inherit;
  line-height: normal;
}

button::after {
  border: 0;
}

.result-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  overflow-x: hidden;
  background: #ffffff;
  color: #000000;
  font-family: "Noto Sans SC", "Microsoft YaHei", Arial, sans-serif;
}

.top-bar {
  position: absolute;
  top: 0;
  left: 0;
  width: 750rpx;
  height: 191rpx;
  background: #fdf9f6;
}

.nav-back {
  position: absolute;
  top: 74rpx;
  left: 29rpx;
  width: 84rpx;
  height: 84rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-back--hover {
  opacity: 0.68;
}

.nav-back__icon {
  width: 50rpx;
  height: 61rpx;
  display: block;
}

.top-bar__title {
  position: absolute;
  top: 92rpx;
  left: 0;
  width: 750rpx;
  color: #000000;
  font-size: 46rpx;
  font-weight: 700;
  line-height: 55rpx;
  text-align: center;
  white-space: nowrap;
}

.top-bar__action {
  position: absolute;
  top: 86rpx;
  right: 54rpx;
  width: 61rpx;
  height: 61rpx;
  display: block;
}

.result-illustration {
  position: absolute;
  top: 265rpx;
  left: 86rpx;
  width: 578rpx;
  height: 466rpx;
  display: block;
}

.result-title {
  position: absolute;
  top: 761rpx;
  left: 244rpx;
  width: 382rpx;
  color: #000000;
  font-size: 46rpx;
  font-weight: 700;
  line-height: 52rpx;
  white-space: nowrap;
}

.result-message {
  position: absolute;
  top: 846rpx;
  left: 172rpx;
  width: 492rpx;
  color: #000000;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 42rpx;
  white-space: pre-line;
}

.advice-card {
  position: absolute;
  top: 983rpx;
  left: 71rpx;
  width: 638rpx;
  height: 320rpx;
  overflow: hidden;
  border-radius: 29rpx;
  background: #f2f8f4;
}

.advice-card__title {
  position: absolute;
  top: 52rpx;
  left: 44rpx;
  color: #000000;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 50rpx;
  white-space: nowrap;
}

.advice-card__body {
  position: absolute;
  top: 121rpx;
  left: 44rpx;
  width: 521rpx;
  color: #000000;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 42rpx;
  white-space: pre-line;
}

.action-row {
  position: absolute;
  top: 1400rpx;
  left: 57rpx;
  display: flex;
  gap: 31rpx;
}

.action-button {
  position: relative;
  width: 301rpx;
  height: 95rpx;
  overflow: hidden;
  border-radius: 29rpx;
}

.action-button--outline {
  box-sizing: border-box;
  background: #ffffff;
  border: 2rpx solid #249547;
}

.action-button--solid {
  background: #249547;
  border: 2rpx solid #249547;
}

.action-button--hover {
  opacity: 0.78;
}

.action-button--hover-solid {
  opacity: 0.9;
}

.action-button__icon {
  position: absolute;
  top: 13rpx;
  left: 34rpx;
  width: 65rpx;
  height: 65rpx;
  display: block;
}

.action-button__text {
  position: absolute;
  top: 19rpx;
  color: #000000;
  font-size: 42rpx;
  font-weight: 400;
  line-height: 50rpx;
  white-space: nowrap;
}

.action-button__text--outline {
  left: 100rpx;
  width: 191rpx;
  color: #249547;
}

.action-button__text--solid {
  left: 0;
  width: 100%;
  color: #ffffff;
  text-align: center;
}
</style>
