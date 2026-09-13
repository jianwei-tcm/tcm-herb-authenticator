<template>
  <view class="ai-page">
    <view class="top-bar">
      <button class="nav-back" hover-class="nav-back--hover" @tap="handleBack">
        <image class="nav-back__icon" src="/static/identification/arrow-filled.svg" mode="aspectFit" />
      </button>
      <text class="top-bar__title">AI药材鉴别</text>
      <button class="history-entry" hover-class="history-entry--hover" @tap="handleHistory">
        历史记录
      </button>
    </view>

    <view class="photo-card" @tap="openCameraPage">
      <image
        v-if="selectedImage"
        class="photo-preview"
        :src="selectedImage"
        mode="aspectFill"
      />
      <image
        v-else
        class="photo-bg"
        src="/static/identification/photo-bg.png"
        mode="aspectFill"
      />
      <view v-if="!selectedImage" class="photo-empty">
        <image class="photo-empty__icon" src="/static/identification/camera-fill.svg" mode="aspectFit" />
        <text class="photo-empty__text">点击拍照</text>
      </view>
    </view>

    

    <view class="feature-title">
      <text class="feature-title__main">请补充药材特征(可多选)</text>
     
    </view>

    <view
      v-for="row in featureRows"
      :key="row.key"
      class="feature-row"
      :class="`feature-row--${row.key}`"
    >
      <text class="feature-row__label">{{ row.label }}</text>
      <view
        v-for="option in row.options"
        :key="option.id"
        class="feature-chip"
        :class="{ 'feature-chip--selected': option.selected }"
        @tap="toggleFeature(row.key, option.id)"
      >
        <text class="feature-chip__text">{{ option.label }}</text>
      </view>
    </view>

    <view class="description-label">
      <text>其他描述</text>
      <text class="description-label__sub">（选填）</text>
    </view>

    <view class="description-box">
      <textarea
        v-model="description"
        class="description-input"
        :maxlength="descriptionMaxLength"
        placeholder="请补充更多特征信息，如形状等..."
        placeholder-class="description-input__placeholder"
        :show-confirm-bar="false"
        :auto-height="false"
      />
      <text class="description-count">{{ description.length }}/{{ descriptionMaxLength }}</text>
    </view>

    <button class="next-button" hover-class="next-button--hover" @tap="handleNext">
      下一步
    </button>
  </view>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { onShow } from '@dcloudio/uni-app'

type FeatureRowKey = 'texture' | 'size' | 'smell'

interface FeatureOption {
  id: string
  label: string
  selected: boolean
}

interface FeatureRow {
  key: FeatureRowKey
  label: string
  options: FeatureOption[]
}

const descriptionMaxLength = 2000
const selectedImageStorageKey = 'identification-selected-image'
const pendingImageStorageKey = 'identification-pending-image'

const selectedImage = ref('')
const description = ref('')

const featureRows = reactive<FeatureRow[]>([
  {
    key: 'texture',
    label: '纹理',
    options: createFeatureOptions('texture', ['粗糙', '光滑', '有纹理', '其他']),
  },
  {
    key: 'size',
    label: '大小',
    options: createFeatureOptions('size', ['小', '中', '大', '其他']),
  },
  {
    key: 'smell',
    label: '气味',
    options: createFeatureOptions('smell', ['无味', '清香', '特殊气味', '其他']),
  },
])

onShow(() => {
  const pendingImage = uni.getStorageSync(selectedImageStorageKey)

  if (typeof pendingImage === 'string' && pendingImage) {
    selectedImage.value = pendingImage
    uni.removeStorageSync(selectedImageStorageKey)
  }
})

function createFeatureOptions(prefix: FeatureRowKey, labels: string[]): FeatureOption[] {
  return labels.map((label, index) => ({
    id: `${prefix}-${index + 1}`,
    label,
    selected: false,
  }))
}

function handleBack() {
  const pages = getCurrentPages()

  if (pages.length > 1) {
    uni.navigateBack({ delta: 1 })
    return
  }

  uni.showToast({
    title: '已在当前页面',
    icon: 'none',
  })
}

function handleHistory() {
  uni.showToast({
    title: '历史记录暂未接入',
    icon: 'none',
  })
}

function openCameraPage() {
  uni.navigateTo({
    url: '/pages/camera/index',
  })
}

function toggleFeature(rowKey: FeatureRowKey, optionId: string) {
  const row = featureRows.find((item) => item.key === rowKey)
  const option = row?.options.find((item) => item.id === optionId)

  if (option) {
    option.selected = !option.selected
  }
}

function handleNext() {
  if (!selectedImage.value) {
    uni.showToast({
      title: '请先拍照或选择图片',
      icon: 'none',
    })
    return
  }

  if (description.value.length > descriptionMaxLength) {
    uni.showToast({
      title: `描述不能超过${descriptionMaxLength}字`,
      icon: 'none',
    })
    return
  }

  uni.setStorageSync(pendingImageStorageKey, selectedImage.value)

  uni.navigateTo({
    url: '/pages/loading/index',
  })
}
</script>

<style>
page {
  background: #ffffff;
}

button {
  background: transparent;
  border: 0;
  border-radius: 0;
  color: inherit;
  font: inherit;
  line-height: normal;
  margin: 0;
  padding: 0;
}

button::after {
  border: 0;
}

.ai-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  overflow-x: hidden;
  background: #ffffff;
  color: #000000;
  font-family: "Noto Sans SC", "Source Sans Pro", Arial, sans-serif;
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
  top: 59rpx;
  left: 55rpx;
  width: 31rpx;
  height: 61rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-back--hover,
.history-entry--hover {
  opacity: 0.65;
}

.nav-back__icon {
  width: 31rpx;
  height: 61rpx;
  transform: rotate(180deg);
}

.top-bar__title {
  position: absolute;
  top: 79rpx;
  left: 0;
  width: 750rpx;
  color: #000000;
  font-size: 46rpx;
  font-weight: 700;
  line-height: 58rpx;
  text-align: center;
  white-space: nowrap;
}

.history-entry {
  position: absolute;
  top: 90rpx;
  right: 17rpx;
  width: 124rpx;
  height: 43rpx;
  color: #000000;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 38rpx;
  text-align: left;
  white-space: nowrap;
}

.photo-card {
  position: absolute;
  top: 246rpx;
  left: 59rpx;
  width: 632rpx;
  height: 405rpx;
  overflow: hidden;
  border-radius: 38rpx;
}

.photo-bg,
.photo-preview {
  position: absolute;
  top: 0;
  left: 0;
  width: 632rpx;
  height: 405rpx;
  border-radius: 38rpx;
}

.photo-bg {
  opacity: 0.6;
}

.photo-empty {
  position: absolute;
  top: 95rpx;
  left: 0;
  width: 632rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.photo-empty__icon {
  width: 160rpx;
  height: 120rpx;
}

.photo-empty__text {
  margin-top: 40rpx;
  color: #000000;
  font-size: 42rpx;
  font-weight: 700;
  line-height: 53rpx;
  white-space: nowrap;
}

.action-row {
  position: absolute;
  top: 584rpx;
  left: 61rpx;
  display: flex;
  gap: 42rpx;
}

.action-button {
  width: 294rpx;
  height: 78rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 13rpx;
  border: 2rpx solid #706e69;
  border-radius: 19rpx;
  background: #fefaf8;
}

.action-button--hover {
  background: #f6efe6;
}

.action-button__camera {
  width: 44rpx;
  height: 44rpx;
}

.action-button__album {
  width: 50rpx;
  height: 50rpx;
}

.action-button__text {
  color: #000000;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 38rpx;
  white-space: nowrap;
}

.feature-title {
  position: absolute;
  top: 704rpx;
  left: 61rpx;
  width: 344rpx;
  height: 38rpx;
  white-space: nowrap;
}

.feature-title__main {
  color: #000000;
  font-size: 29rpx;
  font-weight: 700;
  line-height: 36rpx;
}

.feature-title__sub {
  color: #706e69;
  font-size: 29rpx;
  font-weight: 700;
  line-height: 36rpx;
}

.feature-row {
  position: absolute;
  left: 61rpx;
  display: flex;
  align-items: center;
  height: 46rpx;
}

.feature-row--texture {
  top: 784rpx;
}

.feature-row--size {
  top: 857rpx;
}

.feature-row--smell {
  top: 929rpx;
}

.feature-row__label {
  width: 97rpx;
  color: #000000;
  font-size: 29rpx;
  font-weight: 400;
  line-height: 36rpx;
  white-space: nowrap;
}

.feature-chip {
  width: 116rpx;
  height: 46rpx;
  margin-right: 23rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  padding: 0 4rpx;
  overflow: hidden;
  border: 2rpx solid transparent;
  border-radius: 10rpx;
  background: #d9d9d9;
}

.feature-chip:last-child {
  margin-right: 0;
}

.feature-chip--selected {
  border-color: #6f9b6f;
  background: #c8dec6;
}

.feature-chip__text {
  display: block;
  width: 100%;
  height: 46rpx;
  color: #000000;
  font-size: 24rpx;
  font-weight: 400;
  line-height: 46rpx;
  text-align: center;
  word-break: keep-all;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: clip;
}

.feature-chip--selected .feature-chip__text {
  color: #1b4925;
}

.description-label {
  position: absolute;
  top: 1008rpx;
  left: 61rpx;
  color: #000000;
  font-size: 29rpx;
  font-weight: 400;
  line-height: 36rpx;
  white-space: nowrap;
}

.description-label__sub {
  color: #706e69;
}

.description-box {
  position: absolute;
  top: 1070rpx;
  left: 61rpx;
  width: 630rpx;
  height: 326rpx;
  border-radius: 38rpx;
  background: #fefaf8;
}

.description-input {
  position: absolute;
  top: 21rpx;
  left: 29rpx;
  width: 572rpx;
  height: 245rpx;
  color: #000000;
  font-size: 29rpx;
  font-weight: 400;
  line-height: 42rpx;
}

.description-input__placeholder {
  color: #c2c1bf;
  font-size: 29rpx;
  font-weight: 400;
}

.description-count {
  position: absolute;
  right: 23rpx;
  bottom: 22rpx;
  color: #c2c1bf;
  font-size: 29rpx;
  font-weight: 400;
  line-height: 36rpx;
}

.next-button {
  position: absolute;
  top: 1427rpx;
  left: 61rpx;
  width: 632rpx;
  height: 92rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 29rpx;
  background: #1b4925;
  color: #ffffff;
  font-size: 38rpx;
  font-weight: 400;
  line-height: 48rpx;
}

.next-button--hover {
  background: #14381d;
}
</style>
