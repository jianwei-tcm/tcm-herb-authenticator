<template>
  <view class="identification-detail-page">
    <view class="detail-header">
      <view class="detail-header__back" hover-class="detail-header__back--hover" @tap="goBack">
        <image src="/static/result/back.svg" mode="aspectFit" />
      </view>
      <text class="detail-header__title">鉴别详情</text>
    </view>

    <view class="detail-section detail-section--basic">
      <text class="detail-section__title">基本信息</text>
      <view class="detail-card detail-card--basic">
        <view class="detail-row"><text class="detail-row__label">鉴别编号</text><text class="detail-row__value">ID202508210001</text></view>
        <view class="detail-row"><text class="detail-row__label">提交信息</text><text class="detail-row__value">2026-08-21  14：30：22</text></view>
        <view class="detail-row"><text class="detail-row__label">完成时间</text><text class="detail-row__value">2026-08-21  14：31：05</text></view>
        <view class="detail-row detail-row--description"><text class="detail-row__label">补充描述</text><text class="detail-row__value">颜色偏黄，气味较淡，质地较硬</text></view>
      </view>
    </view>

    <view class="detail-section detail-section--result">
      <text class="detail-section__title">鉴别结果</text>
      <view class="detail-card detail-card--result">
        <view class="detail-row"><text class="detail-row__label">药材名称</text><text class="detail-row__value">黄芪</text></view>
        <view class="detail-row"><text class="detail-row__label">识别置信度</text><view class="confidence"><view class="confidence__track"><view class="confidence__fill" :style="{ width: confidenceWidth }" /></view><text>{{ confidence }}%</text></view></view>
        <view class="detail-row"><text class="detail-row__label">真伪结论</text><text class="detail-row__value detail-row__value--accent">{{ conclusion }}</text></view>
        <view class="detail-row"><text class="detail-row__label">真伪可信度</text><view class="confidence"><view class="confidence__track"><view class="confidence__fill" :style="{ width: authenticityWidth }" /></view><text>{{ authenticity }}%</text></view></view>
      </view>
    </view>

    <view class="detail-section detail-section--model">
      <text class="detail-section__title">模型信息</text>
      <view class="detail-card detail-card--model">
        <view class="detail-row"><text class="detail-row__label">模型名称</text><text class="detail-row__value">中药材鉴别模型</text></view>
        <view class="detail-row"><text class="detail-row__label">模型版本</text><text class="detail-row__value">v2.1.0</text></view>
      </view>
    </view>
    <image class="detail-herb" src="/static/result/authentic-herb.png" mode="aspectFill" />
  </view>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

type Variant = 'authentic' | 'counterfeit' | 'uncertain'
const variant = ref<Variant>('authentic')
const confidence = computed(() => variant.value === 'counterfeit' ? 94 : variant.value === 'uncertain' ? 58 : 96)
const authenticity = computed(() => variant.value === 'counterfeit' ? 88 : variant.value === 'uncertain' ? 58 : 91)
const conclusion = computed(() => variant.value === 'counterfeit' ? '疑似假品' : variant.value === 'uncertain' ? '无法确定' : '疑似真品')
const confidenceWidth = computed(() => `${confidence.value}%`)
const authenticityWidth = computed(() => `${authenticity.value}%`)

onLoad((query?: Record<string, string | undefined>) => {
  if (query?.variant === 'counterfeit' || query?.variant === 'uncertain' || query?.variant === 'authentic') {
    variant.value = query.variant
  }
})

function goBack() {
  uni.redirectTo({
    url: '/pages/result/index',
  })
}
</script>

<style>
page { background: #fff; }
.identification-detail-page { position: relative; width: 750rpx; min-height: 1500rpx; padding-top: 1rpx; background: #fff; color: #000; font-family: "Microsoft YaHei", "Noto Sans SC", sans-serif; }
.detail-header { position: relative; height: 178rpx; background: #fff; }
.detail-header__back { position: absolute; top: 64rpx; left: 40rpx; z-index: 10; display: flex; align-items: center; justify-content: center; width: 52rpx; height: 52rpx; background: transparent; }
.detail-header__back image { display: block; width: 52rpx; height: 52rpx; }
.detail-header__back--hover { opacity: .6; }
.detail-header__title { position: absolute; top: 57rpx; left: 0; width: 100%; text-align: center; font-size: 46rpx; line-height: 58rpx; font-weight: 400; }
.detail-section { position: relative; margin: 0 26rpx; }
.detail-section__title { display: block; margin-left: 46rpx; font-size: 30rpx; line-height: 40rpx; font-weight: 700; }
.detail-card { position: relative; margin-top: 18rpx; border-radius: 30rpx; background: rgba(217,217,217,.25); }
.detail-card::before { content: ''; position: absolute; top: 12rpx; right: 12rpx; bottom: 12rpx; left: 12rpx; border-radius: 30rpx; background: #fff; }
.detail-card--basic { height: 500rpx; }
.detail-card--result { height: 360rpx; }
.detail-card--model { height: 232rpx; }
.detail-row { position: relative; z-index: 1; display: flex; align-items: center; height: 60rpx; padding: 0 24rpx; }
.detail-row__label { color: rgba(0,0,0,.7); font-size: 24rpx; line-height: 32rpx; }
.detail-row__value { margin-left: auto; max-width: 420rpx; color: #000; font-size: 24rpx; line-height: 32rpx; text-align: right; white-space: nowrap; }
.detail-row--description { margin-top: 240rpx; }
.detail-section--result { margin-top: 24rpx; }
.detail-section--model { margin-top: 24rpx; }
.detail-card--result .detail-row { height: 60rpx; }
.detail-card--model .detail-row { height: 62rpx; }
.detail-row__value--accent { color: #268f4c; }
.confidence { display: flex; align-items: center; gap: 14rpx; margin-left: auto; color: #000; font-size: 24rpx; }
.confidence__track { width: 255rpx; height: 15rpx; border-radius: 10rpx; background: rgba(217,217,217,.8); overflow: hidden; }
.confidence__fill { height: 100%; border-radius: 10rpx; background: #268f4c; }
.detail-herb { position: absolute; top: 450rpx; right: 72rpx; width: 192rpx; height: 192rpx; border-radius: 20rpx; opacity: .96; }
</style>
