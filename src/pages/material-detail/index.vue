<template>
  <view class="material-detail-page">
    <view class="material-detail-page__hero" />
    <view class="material-detail-page__body" />

    <view class="detail-header">
      <button class="detail-header__button" hover-class="detail-header__button--hover" @tap="goBack">
        <image class="detail-header__back" :src="assets.back" mode="aspectFit" />
      </button>
      <button class="detail-header__button" hover-class="detail-header__button--hover" @tap="handleHeaderShare">
        <image class="detail-header__share" :src="assets.share" mode="aspectFit" />
      </button>
    </view>

    <view class="summary-card">
      <image class="summary-card__image" :src="material.image" mode="aspectFill" />
      <view class="summary-card__content">
        <text class="summary-card__name">{{ material.name }}</text>
        <text v-if="material.latinName" class="summary-card__latin">{{ material.latinName }}</text>
      </view>
    </view>

    <view class="info-card">
      <view class="info-section">
        <text class="info-section__title">药材功效</text>
        <text class="info-section__text">{{ material.efficacy }}</text>
      </view>

      <view class="info-section">
        <text class="info-section__title">注意事项</text>
        <text class="info-section__text">{{ material.precautions }}</text>
      </view>

      <view class="info-section">
        <text class="info-section__title">储存条件</text>
        <text class="info-section__text">{{ material.storage }}</text>
      </view>
    </view>

    <view class="detail-actions">
      <view
        class="detail-action detail-action--favorite"
        :class="{ 'detail-action--favorite-active': isFavorite }"
        hover-class="detail-action--hover"
        @tap="toggleFavorite"
      >
        <text class="detail-action__star">{{ isFavorite ? '★' : '☆' }}</text>
        <text class="detail-action__text">{{ isFavorite ? '已收藏' : '收藏' }}</text>
      </view>

      <view
        class="detail-action detail-action--share"
        hover-class="detail-action--hover"
        @tap="handleShareCommunity"
      >
        <text class="detail-action__text">分享社区</text>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

interface MaterialDetail {
  id: string
  name: string
  latinName?: string
  image: string
  efficacy: string
  precautions: string
  storage: string
}

interface FallbackQuery {
  name?: string
  efficacy?: string
}

const assets = {
  share: 'https://www.figma.com/api/mcp/asset/9cc9fe67-b4ee-42f0-9339-3dadd2279e96.svg',
  herb: 'https://www.figma.com/api/mcp/asset/22082bf4-155b-4674-92b5-a206685be786.png',
  back: 'https://www.figma.com/api/mcp/asset/08e592cf-3bdd-4079-9d56-ab65632f1971.svg',
} as const

const mockMaterialDetails: Record<string, MaterialDetail> = {
  huangqi: {
    id: 'huangqi',
    name: '黄芪',
    latinName: 'Astragalus membranaceus',
    image: assets.herb,
    efficacy: '补气升阳、益卫固表、利水消肿、脱毒生肌。',
    precautions: '阴虚火旺者慎用，表实邪盛者禁服。',
    storage: '置阴凉干燥处，防潮，防蛀。',
  },
}

const materialIdAliases: Record<string, string> = {
  'huang-qi': 'huangqi',
}

const selectedMaterialId = ref('huangqi')
const fallbackQuery = ref<FallbackQuery>({})
const isFavorite = ref(false)

const material = computed(() => {
  const normalizedId = normalizeMaterialId(selectedMaterialId.value)
  const matchedMaterial = mockMaterialDetails[normalizedId]

  if (matchedMaterial) {
    return matchedMaterial
  }

  return createFallbackMaterial(normalizedId, fallbackQuery.value)
})

onLoad((query?: Record<string, string | undefined>) => {
  selectedMaterialId.value = decodeQueryValue(query?.id) || 'huangqi'
  fallbackQuery.value = {
    name: decodeQueryValue(query?.name),
    efficacy: decodeQueryValue(query?.efficacy),
  }
  isFavorite.value = false
})

function normalizeMaterialId(id: string) {
  const trimmedId = id.trim()
  return materialIdAliases[trimmedId] ?? trimmedId
}

function decodeQueryValue(value?: string) {
  if (!value) {
    return ''
  }

  try {
    return decodeURIComponent(value)
  } catch {
    return value
  }
}

function createFallbackMaterial(id: string, query: FallbackQuery): MaterialDetail {
  return {
    id,
    name: query.name || '药材详情',
    image: assets.herb,
    efficacy: query.efficacy || '药材功效信息待补充。',
    precautions: '请结合个人体质，在专业人士指导下使用。',
    storage: '置阴凉干燥处，防潮，防蛀。',
  }
}

function goBack() {
  const pages = getCurrentPages()

  if (pages.length > 1) {
    uni.navigateBack({
      delta: 1,
    })
    return
  }

  uni.reLaunch({
    url: '/pages/index/index',
  })
}

function toggleFavorite() {
  isFavorite.value = !isFavorite.value
}

function handleHeaderShare() {
  uni.showToast({
    title: '分享功能待接入',
    icon: 'none',
  })
}

function handleShareCommunity() {
  console.info('pending community share target:', '/pages/community/index', material.value.id)
  uni.showToast({
    title: '社区页面待开发',
    icon: 'none',
  })
}
</script>

<style>
page {
  background: #ffffff;
}

.material-detail-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  overflow-x: hidden;
  background: #ffffff;
  color: #000000;
  box-sizing: border-box;
  font-family: "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.material-detail-page view,
.material-detail-page text,
.material-detail-page image {
  box-sizing: border-box;
}

.material-detail-page button {
  margin: 0;
  padding: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  color: inherit;
  font: inherit;
  line-height: normal;
}

.material-detail-page button::after {
  border: 0;
}

.material-detail-page__hero {
  position: absolute;
  top: 0;
  left: 0;
  width: 750rpx;
  height: 568rpx;
  background: #376f46;
}

.material-detail-page__body {
  position: absolute;
  top: 568rpx;
  right: 0;
  bottom: 0;
  left: 0;
  background: #ffffff;
}

.detail-header {
  position: relative;
  z-index: 4;
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 750rpx;
  height: calc(108rpx + env(safe-area-inset-top));
  padding: calc(20rpx + env(safe-area-inset-top)) 28rpx 20rpx;
}

.detail-header__button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 64rpx;
  height: 64rpx;
}

.detail-header__button--hover,
.detail-action--hover {
  opacity: 0.72;
}

.detail-header__back {
  display: block;
  width: 63rpx;
  height: 63rpx;
}

.detail-header__share {
  display: block;
  width: 46rpx;
  height: 46rpx;
}

.summary-card {
  position: relative;
  z-index: 3;
  display: flex;
  align-items: center;
  width: 658rpx;
  min-height: 372rpx;
  margin: 14rpx auto 0;
  padding: 42rpx 42rpx 40rpx 58rpx;
  border-radius: 29rpx;
  background: #fffaf3;
}

.summary-card__image {
  flex: 0 0 auto;
  display: block;
  width: 238rpx;
  height: 238rpx;
  overflow: hidden;
  border-radius: 29rpx;
}

.summary-card__content {
  flex: 0 0 308rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-width: 0;
  margin-left: 26rpx;
}

.summary-card__name {
  display: block;
  overflow: hidden;
  color: #000000;
  font-size: 69rpx;
  font-weight: 700;
  line-height: 82rpx;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.summary-card__latin {
  display: block;
  width: 308rpx;
  margin-top: 18rpx;
  color: rgba(0, 0, 0, 0.24);
  font-size: 23rpx;
  font-weight: 700;
  line-height: 30rpx;
  text-align: center;
  white-space: nowrap;
}

.info-card {
  position: relative;
  z-index: 2;
  width: 658rpx;
  min-height: 730rpx;
  margin: 24rpx auto 236rpx;
  padding: 58rpx 26rpx 66rpx;
  border-radius: 29rpx;
  background: #fffaf3;
}

.info-section + .info-section {
  margin-top: 96rpx;
}

.info-section__title {
  display: block;
  color: #000000;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 48rpx;
  white-space: nowrap;
}

.info-section__text {
  display: block;
  margin-top: 29rpx;
  color: #000000;
  font-size: 31rpx;
  line-height: 46rpx;
}

.detail-actions {
  position: fixed;
  bottom: 0;
  left: 50%;
  z-index: 20;
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 750rpx;
  padding: 28rpx 46rpx calc(48rpx + env(safe-area-inset-bottom));
  box-sizing: border-box;
  background: #ffffff;
  transform: translateX(-50%);
}

.detail-action {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 305rpx;
  height: 96rpx;
  border-radius: 29rpx;
  box-sizing: border-box;
  overflow: hidden;
}

.detail-action--favorite {
  border: 2rpx solid #014722;
  background: #fef9f0;
  color: #014722;
}

.detail-action--favorite-active {
  background: #e7f1e9;
}

.detail-action--share {
  background: #014722;
  color: #ffffff;
}

.detail-action__star {
  display: block;
  margin-right: 10rpx;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 48rpx;
  white-space: nowrap;
}

.detail-action__text {
  display: block;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 48rpx;
  white-space: nowrap;
}
</style>
