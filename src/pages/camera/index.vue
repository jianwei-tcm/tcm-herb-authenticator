<template>
  <view class="camera-page">
    <view class="camera-top">
      <button class="camera-back" hover-class="camera-back--hover" @tap="handleBack">
        <image class="camera-back__icon" src="/static/camera/arrow-filled.svg" mode="aspectFit" />
      </button>
      <text class="camera-title">识别药材</text>
      <view class="camera-status">
        <image class="camera-status__dot" src="/static/camera/ai-dot.svg" mode="aspectFit" />
        <text class="camera-status__text">AI就绪</text>
      </view>
    </view>

    <view class="camera-viewfinder">
      <image class="camera-viewfinder__frame" src="/static/camera/focus-frame.svg" mode="scaleToFill" />
      <image class="camera-viewfinder__plus" src="/static/camera/focus-plus.svg" mode="aspectFit" />
    </view>

    <text class="camera-tip">请拍摄清晰、完整的药材主体，确保光线充足</text>

    <view class="camera-actions">
      <button class="camera-action camera-action--album" hover-class="camera-action--hover" @tap="selectFromAlbum">
        <image class="camera-action__bg" src="/static/camera/side-button-bg.svg" mode="scaleToFill" />
        <image class="camera-action__album" src="/static/camera/album-filled.svg" mode="aspectFit" />
      </button>

      <button class="camera-action camera-action--shoot" hover-class="camera-action--shoot-hover" @tap="capturePhoto">
        <image class="camera-action__shutter" src="/static/camera/shutter-outer.svg" mode="scaleToFill" />
        <image class="camera-action__lightning" src="/static/camera/shutter-lightning.svg" mode="aspectFit" />
      </button>

      <button class="camera-action camera-action--retake" hover-class="camera-action--hover" @tap="retakePhoto">
        <image class="camera-action__bg" src="/static/camera/side-button-bg.svg" mode="scaleToFill" />
        <image class="camera-action__retake" src="/static/camera/retake.svg" mode="aspectFit" />
      </button>
    </view>
  </view>
</template>

<script setup lang="ts">
type ImageSourceType = 'album' | 'camera'

const selectedImageStorageKey = 'identification-selected-image'

function handleBack() {
  const pages = getCurrentPages()

  if (pages.length > 1) {
    uni.navigateBack({ delta: 1 })
    return
  }

  uni.reLaunch({
    url: '/pages/index/index',
  })
}

function selectFromAlbum() {
  chooseImage('album')
}

function capturePhoto() {
  chooseImage('camera')
}

function retakePhoto() {
  chooseImage('camera')
}

function chooseImage(sourceType: ImageSourceType) {
  uni.chooseImage({
    count: 1,
    sizeType: ['compressed'],
    sourceType: [sourceType],
    success: (res) => {
      const filePath = res.tempFilePaths[0]

      if (filePath) {
        finishWithImage(filePath)
      }
    },
    fail: () => {
      uni.showToast({
        title: '未获取照片',
        icon: 'none',
      })
    },
  })
}

function finishWithImage(filePath: string) {
  uni.setStorageSync(selectedImageStorageKey, filePath)

  const pages = getCurrentPages()
  const previousPage = pages[pages.length - 2] as { route?: string } | undefined

  if (previousPage?.route === 'pages/identification/index') {
    uni.navigateBack({ delta: 1 })
    return
  }

  uni.redirectTo({
    url: '/pages/identification/index',
  })
}
</script>

<style>
page {
  background: #f6f6f6;
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

.camera-page {
  position: relative;
  width: 750rpx;
  height: 1628rpx;
  min-height: 100vh;
  overflow: hidden;
  background: #f6f6f6;
  color: #000000;
  box-sizing: border-box;
  font-family: "Noto Sans SC", "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.camera-page view,
.camera-page text,
.camera-page image,
.camera-page button {
  box-sizing: border-box;
}

.camera-top {
  position: absolute;
  top: 0;
  left: 0;
  width: 750rpx;
  height: 185rpx;
  background: rgba(55, 111, 70, 0.8);
}

.camera-back {
  position: absolute;
  top: 61rpx;
  left: 38rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 33rpx;
  height: 65rpx;
}

.camera-back--hover,
.camera-action--hover {
  opacity: 0.72;
}

.camera-action--shoot-hover {
  opacity: 0.88;
}

.camera-back__icon {
  display: block;
  width: 33rpx;
  height: 65rpx;
  transform: rotate(180deg);
}

.camera-title {
  position: absolute;
  top: 63rpx;
  left: 107rpx;
  color: #000000;
  font-size: 46rpx;
  font-weight: 700;
  line-height: 65rpx;
  white-space: nowrap;
}

.camera-status {
  position: absolute;
  top: 63rpx;
  left: 567rpx;
  display: flex;
  align-items: center;
  width: 153rpx;
  height: 57rpx;
  padding-left: 15rpx;
  border-radius: 57rpx;
  background: #47604d;
}

.camera-status__dot {
  display: block;
  width: 32rpx;
  height: 32rpx;
  flex: 0 0 auto;
}

.camera-status__text {
  display: block;
  margin-left: 3rpx;
  color: #91e9a9;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 43rpx;
  white-space: nowrap;
}

.camera-viewfinder {
  position: absolute;
  top: 389rpx;
  left: 82rpx;
  width: 586rpx;
  height: 586rpx;
}

.camera-viewfinder__frame {
  position: absolute;
  top: 0;
  left: 0;
  display: block;
  width: 586rpx;
  height: 586rpx;
}

.camera-viewfinder__plus {
  position: absolute;
  top: 251rpx;
  left: 276rpx;
  display: block;
  width: 46rpx;
  height: 46rpx;
}

.camera-tip {
  position: absolute;
  top: 1000rpx;
  left: 111rpx;
  color: #706e69;
  font-size: 27rpx;
  font-weight: 400;
  line-height: 38rpx;
  white-space: nowrap;
}

.camera-actions {
  position: absolute;
  bottom: 0;
  left: 0;
  display: flex;
  align-items: flex-start;
  width: 750rpx;
  height: 305rpx;
  background: #ffffff;
}

.camera-action {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
}

.camera-action--album {
  top: 94rpx;
  left: 92rpx;
  width: 116rpx;
  height: 116rpx;
}

.camera-action--shoot {
  top: 61rpx;
  left: 279rpx;
  width: 176rpx;
  height: 176rpx;
}

.camera-action--retake {
  top: 92rpx;
  right: 82rpx;
  width: 116rpx;
  height: 116rpx;
}

.camera-action__bg,
.camera-action__shutter {
  position: absolute;
  top: 0;
  left: 0;
  display: block;
  width: 100%;
  height: 100%;
}

.camera-action__album {
  position: relative;
  z-index: 1;
  display: block;
  width: 61rpx;
  height: 61rpx;
}

.camera-action__lightning {
  position: relative;
  z-index: 1;
  display: block;
  width: 94rpx;
  height: 94rpx;
}

.camera-action__retake {
  position: relative;
  z-index: 1;
  display: block;
  width: 46rpx;
  height: 46rpx;
}
</style>
