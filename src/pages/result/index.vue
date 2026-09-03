<template>
  <view class="result-page">
    <view class="top-bar">
      <button class="nav-back" hover-class="nav-back--hover" @tap="handleBack">
        <image class="nav-back__icon" src="/static/result/back.svg" mode="aspectFit" />
      </button>
      <text class="top-bar__title">鉴别结果</text>
      <image class="top-bar__action" src="/static/result/share-outline.svg" mode="aspectFit" />
    </view>

    <template v-if="unrecognizedResult">
      <image class="result-illustration" :src="unrecognizedResult.illustration" mode="scaleToFill" />

      <text class="result-title">{{ unrecognizedResult.title }}</text>
      <text class="result-message">{{ unrecognizedResult.message }}</text>

      <view class="advice-card">
        <text class="advice-card__title">{{ unrecognizedResult.adviceTitle }}</text>
        <text class="advice-card__body">{{ adviceText }}</text>
      </view>

      <view class="action-row">
        <button class="action-button action-button--outline" hover-class="action-button--hover" @tap="handleReupload">
          <image class="action-button__icon" src="/static/result/camera-photo-outline.svg" mode="aspectFit" />
          <text class="action-button__text action-button__text--outline">{{ unrecognizedResult.secondaryActionLabel }}</text>
        </button>

        <button class="action-button action-button--solid" hover-class="action-button--hover-solid" @tap="handleSupplement">
          <text class="action-button__text action-button__text--solid">{{ unrecognizedResult.primaryActionLabel }}</text>
        </button>
      </view>
    </template>

    <template v-else-if="structuredResult">
      <view class="structured-result" :class="`structured-result--${structuredResult.theme}`">
        <view class="structured-summary">
          <image class="structured-summary__icon" :src="structuredResult.summaryIcon" mode="aspectFit" />

          <view class="structured-summary__copy">
            <text class="structured-summary__eyebrow">鉴别结果</text>
            <text class="structured-summary__headline">{{ structuredResult.headline }}</text>
          </view>

          <view class="structured-summary__badge">
            <text class="structured-summary__badge-text">{{ structuredResult.summaryBadgeText }}</text>
          </view>
        </view>

        <view class="structured-card">
          <image class="structured-card__image" :src="structuredResult.herbImage" mode="aspectFill" />

          <view class="structured-card__meta">
            <text class="structured-card__label">药材名称</text>
            <text class="structured-card__name">{{ structuredResult.herbName }}</text>
            <text class="structured-card__latin">{{ structuredResult.herbLatin }}</text>
            <text class="structured-card__desc">{{ structuredResult.herbDescription }}</text>
          </view>

          <image class="structured-card__chevron" src="/static/result/chevron-right.svg" mode="aspectFit" />

          <view class="structured-card__divider" />

          <view class="structured-card__analysis">
            <text class="structured-card__section-label">真伪判断</text>

            <view class="structured-card__result-chip">
              <text class="structured-card__result-label">{{ structuredResult.assessmentLabel }}</text>
              <image
                v-if="structuredResult.assessmentIcon"
                class="structured-card__result-icon"
                :src="structuredResult.assessmentIcon"
                mode="aspectFit"
              />
            </view>

            <text class="structured-card__confidence-value">{{ structuredResult.confidenceValue }}</text>

            <view class="structured-card__confidence-row">
              <text class="structured-card__confidence-label">真伪判断可信度</text>
              <view class="structured-card__track">
                <view class="structured-card__fill" :style="{ width: structuredResult.progressFillWidth }" />
              </view>
            </view>

            <text class="structured-card__scale">{{ structuredResult.confidenceScale }}</text>

            <view class="structured-card__note">
              <text class="structured-card__note-text">{{ structuredResult.noteText }}</text>
            </view>

            <view class="structured-card__basis">
              <text class="structured-card__basis-title">鉴别依据</text>
              <view v-for="line in structuredResult.basisLines" :key="line" class="structured-card__basis-line">
                <text class="structured-card__basis-text">• {{ line }}</text>
              </view>
            </view>
          </view>
        </view>

        <view class="structured-actions">
          <button class="structured-action structured-action--outline" hover-class="structured-action--hover" @tap="handleViewDetail">
            <text class="structured-action__text">{{ structuredResult.leftActionLabel }}</text>
          </button>

          <button class="structured-action structured-action--solid" hover-class="structured-action--hover-solid" @tap="handleSaveRecord">
            <text class="structured-action__text">{{ structuredResult.rightActionLabel }}</text>
          </button>
        </view>
      </view>
    </template>
  </view>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { onShow } from '@dcloudio/uni-app'

type ResultVariant = 'unrecognized' | 'authentic' | 'counterfeit' | 'uncertain'
type StructuredTheme = 'green' | 'red' | 'orange'

interface StoredResultPayload {
  variant?: unknown
  image?: unknown
}

interface UnrecognizedResultState {
  layout: 'unrecognized'
  variant: 'unrecognized'
  title: string
  message: string
  adviceTitle: string
  adviceLines: string[]
  illustration: string
  primaryActionLabel: string
  secondaryActionLabel: string
}

interface StructuredResultState {
  layout: 'structured'
  variant: Exclude<ResultVariant, 'unrecognized'>
  theme: StructuredTheme
  summaryIcon: string
  summaryBadgeText: string
  headline: string
  herbImage: string
  herbName: string
  herbLatin: string
  herbDescription: string
  assessmentLabel: string
  assessmentIcon?: string
  confidenceValue: string
  confidenceScale: string
  progressFillWidth: string
  noteText: string
  basisLines: string[]
  leftActionLabel: string
  rightActionLabel: string
}

type ResultState = UnrecognizedResultState | StructuredResultState

const resultStorageKey = 'identification-result-mock'
const savedResultStorageKey = 'identification-result-saved'
const mockResultVariants: ResultVariant[] = ['unrecognized', 'authentic', 'counterfeit', 'uncertain']

const unrecognizedBase: Omit<UnrecognizedResultState, 'layout' | 'variant'> = {
  title: '无法识别药材',
  message: '系统未能识别出该药的种类，\n请尝试重新上传清晰的药材图片。',
  adviceTitle: '建议您：',
  adviceLines: ['拍摄清晰、完整的药材主体', '确保光线充足、背景干净', '尝试从不同角度拍摄'],
  illustration: '/static/result/unrecognized-illustration.png',
  primaryActionLabel: '补充信息',
  secondaryActionLabel: '重新上传',
}

const structuredBase = {
  confidenceScale: '0%                                 50%                        100%',
  leftActionLabel: '查看详情',
  rightActionLabel: '保存记录',
  herbName: '黄芪',
  herbLatin: 'Astragalus membranaceus',
  herbDescription: '豆科植物黄芪或膜荚黄芪的干燥根',
  noteText: '该结果基于当前模型分析，仅供参考，不能替代专业检验或医疗建议。',
} as const

const structuredResultMap: Record<Exclude<ResultVariant, 'unrecognized'>, StructuredResultState> = {
  authentic: {
    layout: 'structured',
    variant: 'authentic',
    theme: 'green',
    summaryIcon: '/static/result/shield-check.svg',
    summaryBadgeText: '可信度较高',
    headline: '疑似真品',
    herbImage: '/static/result/authentic-herb.png',
    herbName: structuredBase.herbName,
    herbLatin: structuredBase.herbLatin,
    herbDescription: structuredBase.herbDescription,
    assessmentLabel: '疑似真品',
    assessmentIcon: '/static/result/judgement-ok.svg',
    confidenceValue: '91%',
    confidenceScale: structuredBase.confidenceScale,
    progressFillWidth: '280rpx',
    noteText: structuredBase.noteText,
    basisLines: ['纹理特征与正品相似', '颜色、气味等特征符合正品特征', '形态结构匹配度较高'],
    leftActionLabel: structuredBase.leftActionLabel,
    rightActionLabel: structuredBase.rightActionLabel,
  },
  counterfeit: {
    layout: 'structured',
    variant: 'counterfeit',
    theme: 'red',
    summaryIcon: '/static/result/shield-cross.svg',
    summaryBadgeText: '可信度较高',
    headline: '疑似假品',
    herbImage: '/static/result/counterfeit-herb.png',
    herbName: structuredBase.herbName,
    herbLatin: structuredBase.herbLatin,
    herbDescription: structuredBase.herbDescription,
    assessmentLabel: '疑似假品',
    assessmentIcon: '/static/result/judgement-error.svg',
    confidenceValue: '94%',
    confidenceScale: structuredBase.confidenceScale,
    progressFillWidth: '296rpx',
    noteText: structuredBase.noteText,
    basisLines: ['纹理特征与正品差异较大', '颜色异常，存在染色可能', '断面结构不符合正品特征'],
    leftActionLabel: structuredBase.leftActionLabel,
    rightActionLabel: structuredBase.rightActionLabel,
  },
  uncertain: {
    layout: 'structured',
    variant: 'uncertain',
    theme: 'orange',
    summaryIcon: '/static/result/shield-question.svg',
    summaryBadgeText: '可信度不足',
    headline: '无法确定',
    herbImage: '/static/result/uncertain-herb.png',
    herbName: structuredBase.herbName,
    herbLatin: structuredBase.herbLatin,
    herbDescription: structuredBase.herbDescription,
    assessmentLabel: '无法确定',
    confidenceValue: '58%',
    confidenceScale: structuredBase.confidenceScale,
    progressFillWidth: '280rpx',
    noteText: structuredBase.noteText,
    basisLines: ['纹理特征与正品相似', '颜色、气味等特征符合正品特征', '形态结构匹配度较高'],
    leftActionLabel: structuredBase.leftActionLabel,
    rightActionLabel: structuredBase.rightActionLabel,
  },
}

const resultState = ref<ResultState>(createUnrecognizedState())

const unrecognizedResult = computed(() => (resultState.value.layout === 'unrecognized' ? resultState.value : null))
const structuredResult = computed(() => (resultState.value.layout === 'structured' ? resultState.value : null))
const adviceText = computed(() => unrecognizedResult.value?.adviceLines.join('\n') ?? '')

onShow(() => {
  const cachedPayload = uni.getStorageSync(resultStorageKey)
  const variant = resolveVariant(cachedPayload)

  resultState.value = variant === 'unrecognized' ? createUnrecognizedState() : structuredResultMap[variant]
})

function createUnrecognizedState(): UnrecognizedResultState {
  return {
    layout: 'unrecognized',
    variant: 'unrecognized',
    ...unrecognizedBase,
    adviceLines: [...unrecognizedBase.adviceLines],
  }
}

function resolveVariant(payload: unknown): ResultVariant {
  if (!payload || typeof payload !== 'object') {
    return 'unrecognized'
  }

  const candidate = (payload as StoredResultPayload).variant

  if (isResultVariant(candidate)) {
    return candidate
  }

  return 'unrecognized'
}

function isResultVariant(value: unknown): value is ResultVariant {
  return typeof value === 'string' && mockResultVariants.includes(value as ResultVariant)
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

function handleViewDetail() {
  uni.showToast({
    title: '查看详情暂未接入',
    icon: 'none',
  })
}

function handleSaveRecord() {
  if (structuredResult.value) {
    uni.setStorageSync(savedResultStorageKey, {
      variant: structuredResult.value.variant,
      savedAt: Date.now(),
    })
  }

  uni.showToast({
    title: '已保存记录',
    icon: 'none',
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

.structured-result {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
}

.structured-result--green {
  --summary-bg: rgba(143, 212, 162, 0.28);
  --badge-bg: #376f46;
  --badge-text: #ffffff;
  --result-accent: #1c840e;
  --confidence-color: #37804b;
  --progress-fill: #376f46;
  --note-bg: rgba(143, 212, 162, 0.28);
  --button-border: #0e6621;
  --button-bg: #0e6621;
  --button-text: #ffffff;
  --button-outline-text: #000000;
  --card-shadow: 76px 103px 36px 0px rgba(63, 120, 78, 0), 49px 66px 33px 0px rgba(63, 120, 78, 0.01), 27px 37px 28px 0px rgba(63, 120, 78, 0.05), 12px 16px 20px 0px rgba(63, 120, 78, 0.09), 3px 4px 11px 0px rgba(63, 120, 78, 0.1);
  --confidence-shadow: -1px 1px 3px rgba(162, 193, 159, 0.83), -5px 3px 5px rgba(162, 193, 159, 0.72), -10px 6px 7px rgba(162, 193, 159, 0.43), -19px 11px 9px rgba(162, 193, 159, 0.13), -29px 18px 10px rgba(162, 193, 159, 0.02);
}

.structured-result--red {
  --summary-bg: #ffe2e2;
  --badge-bg: #db1b1e;
  --badge-text: #ffffff;
  --result-accent: #db1b1e;
  --confidence-color: #db1b1e;
  --progress-fill: #db1b1e;
  --note-bg: #ffe2e2;
  --button-border: #d01010;
  --button-bg: #d01010;
  --button-text: #ffffff;
  --button-outline-text: #000000;
  --card-shadow: 76px 103px 36px 0px rgba(248, 93, 93, 0), 49px 66px 33px 0px rgba(248, 93, 93, 0.01), 27px 37px 28px 0px rgba(248, 93, 93, 0.05), 12px 16px 20px 0px rgba(248, 93, 93, 0.09), 3px 4px 11px 0px rgba(248, 93, 93, 0.1);
  --confidence-shadow: -1px 1px 3px rgba(212, 145, 145, 0.83), -5px 3px 5px rgba(212, 145, 145, 0.72), -10px 6px 7px rgba(212, 145, 145, 0.43), -19px 11px 9px rgba(212, 145, 145, 0.13), -29px 18px 10px rgba(212, 145, 145, 0.02);
}

.structured-result--orange {
  --summary-bg: #fef1da;
  --badge-bg: #ffc44a;
  --badge-text: #ffffff;
  --result-accent: #f9a901;
  --confidence-color: #f9a901;
  --progress-fill: #f9a901;
  --note-bg: #fef1da;
  --button-border: #f9a901;
  --button-bg: #f9a901;
  --button-text: #ffffff;
  --button-outline-text: #000000;
  --card-shadow: 76px 103px 36px 0px rgba(233, 170, 63, 0), 49px 66px 33px 0px rgba(233, 170, 63, 0.01), 27px 37px 28px 0px rgba(233, 170, 63, 0.05), 12px 16px 20px 0px rgba(233, 170, 63, 0.09), 3px 4px 11px 0px rgba(233, 170, 63, 0.1);
  --confidence-shadow: -1px 1px 3px rgba(247, 204, 130, 0.83), -5px 3px 5px rgba(247, 204, 130, 0.72), -10px 6px 7px rgba(247, 204, 130, 0.43), -19px 11px 9px rgba(247, 204, 130, 0.13), -29px 18px 10px rgba(247, 204, 130, 0.02);
}

.structured-summary {
  position: absolute;
  top: 241rpx;
  left: 57rpx;
  width: 639rpx;
  height: 222rpx;
  border-radius: 29rpx;
  background: var(--summary-bg);
}

.structured-summary__icon {
  position: absolute;
  top: 44rpx;
  left: 27rpx;
  width: 134rpx;
  height: 134rpx;
  display: block;
}

.structured-summary__copy {
  position: absolute;
  top: 42rpx;
  left: 160rpx;
  width: 256rpx;
}

.structured-summary__eyebrow {
  display: block;
  color: #000000;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 46rpx;
  white-space: nowrap;
}

.structured-summary__headline {
  display: block;
  color: #000000;
  font-size: 53rpx;
  font-weight: 700;
  line-height: 60rpx;
  white-space: nowrap;
}

.structured-summary__badge {
  position: absolute;
  top: 27rpx;
  left: 191rpx;
  width: 193rpx;
  height: 55rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 38rpx;
  background: var(--badge-bg);
}

.structured-summary__badge-text {
  color: var(--badge-text);
  font-size: 30rpx;
  font-weight: 400;
  line-height: 1;
  white-space: nowrap;
}

.structured-card {
  position: absolute;
  top: 503rpx;
  left: 57rpx;
  width: 639rpx;
  height: 886rpx;
  overflow: hidden;
  border-radius: 29rpx;
  background: #ffffff;
  box-shadow: var(--card-shadow);
}

.structured-card__image {
  position: absolute;
  top: 34rpx;
  left: 29rpx;
  width: 172rpx;
  height: 172rpx;
  display: block;
  border-radius: 29rpx;
}

.structured-card__meta {
  position: absolute;
  top: 25rpx;
  left: 231rpx;
  width: 312rpx;
}

.structured-card__label {
  display: block;
  color: #000000;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 46rpx;
  white-space: nowrap;
}

.structured-card__name {
  display: block;
  color: #000000;
  font-size: 57rpx;
  font-weight: 700;
  line-height: 64rpx;
  white-space: nowrap;
}

.structured-card__latin {
  display: block;
  color: rgba(0, 0, 0, 0.24);
  font-size: 23rpx;
  font-weight: 700;
  line-height: 28rpx;
  white-space: nowrap;
}

.structured-card__desc {
  display: block;
  color: #000000;
  font-size: 23rpx;
  font-weight: 700;
  line-height: 28rpx;
  white-space: nowrap;
}

.structured-card__chevron {
  position: absolute;
  top: 27rpx;
  left: 611rpx;
  width: 29rpx;
  height: 29rpx;
  display: block;
}

.structured-card__divider {
  position: absolute;
  top: 244rpx;
  left: 0;
  width: 639rpx;
  height: 1rpx;
  background: #d9d9d9;
  transform: rotate(0.17deg);
  transform-origin: center;
}

.structured-card__analysis {
  position: absolute;
  inset: 0;
}

.structured-card__section-label {
  position: absolute;
  top: 284rpx;
  left: 27rpx;
  color: #000000;
  font-size: 23rpx;
  font-weight: 400;
  line-height: 28rpx;
  white-space: nowrap;
}

.structured-card__result-chip {
  position: absolute;
  top: 330rpx;
  left: 27rpx;
  display: flex;
  align-items: center;
  gap: 12rpx;
}

.structured-card__result-label {
  color: var(--result-accent);
  font-size: 38rpx;
  font-weight: 700;
  line-height: 46rpx;
  white-space: nowrap;
}

.structured-card__result-icon {
  width: 46rpx;
  height: 46rpx;
  display: block;
}

.structured-card__confidence-value {
  position: absolute;
  top: 298rpx;
  left: 427rpx;
  color: var(--confidence-color);
  font-size: 76rpx;
  font-weight: 700;
  line-height: 1;
  white-space: nowrap;
  text-shadow: var(--confidence-shadow);
}

.structured-card__confidence-row {
  position: absolute;
  top: 414rpx;
  left: 27rpx;
  width: 575rpx;
  display: flex;
  align-items: center;
  gap: 18rpx;
}

.structured-card__confidence-label {
  color: #000000;
  font-size: 25rpx;
  font-weight: 400;
  line-height: 30rpx;
  white-space: nowrap;
}

.structured-card__track {
  width: 355rpx;
  height: 17rpx;
  overflow: hidden;
  border-radius: 10rpx;
  background: rgba(217, 217, 217, 0.53);
}

.structured-card__fill {
  height: 100%;
  border-radius: 10rpx;
  background: var(--progress-fill);
}

.structured-card__scale {
  position: absolute;
  top: 448rpx;
  left: 241rpx;
  color: #878378;
  font-size: 19rpx;
  font-weight: 400;
  line-height: 1;
  white-space: pre;
}

.structured-card__note {
  position: absolute;
  top: 513rpx;
  left: 27rpx;
  width: 575rpx;
  height: 99rpx;
  overflow: hidden;
  border-radius: 29rpx;
  background: var(--note-bg);
}

.structured-card__note-text {
  position: absolute;
  top: 21rpx;
  left: 23rpx;
  width: 530rpx;
  color: #000000;
  font-size: 25rpx;
  font-weight: 400;
  line-height: 36rpx;
  white-space: pre-wrap;
}

.structured-card__basis {
  position: absolute;
  top: 660rpx;
  left: 27rpx;
  width: 575rpx;
}

.structured-card__basis-title {
  display: block;
  color: #000000;
  font-size: 28rpx;
  font-weight: 400;
  line-height: 36rpx;
  white-space: nowrap;
}

.structured-card__basis-line {
  color: #000000;
  font-size: 28rpx;
  font-weight: 400;
  line-height: 34rpx;
  white-space: nowrap;
}

.structured-card__basis-text {
  display: block;
}

.structured-actions {
  position: absolute;
  top: 1447rpx;
  left: 53rpx;
  width: 644rpx;
  display: flex;
  justify-content: space-between;
}

.structured-action {
  width: 292rpx;
  height: 120rpx;
  border-radius: 29rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.structured-action--outline {
  box-sizing: border-box;
  background: #ffffff;
  border: 2rpx solid var(--button-border);
}

.structured-action--solid {
  background: var(--button-bg);
  border: 2rpx solid var(--button-bg);
}

.structured-action--hover {
  opacity: 0.78;
}

.structured-action--hover-solid {
  opacity: 0.9;
}

.structured-action__text {
  color: var(--button-outline-text);
  font-size: 42rpx;
  font-weight: 400;
  line-height: 1;
  white-space: nowrap;
}

.structured-action--solid .structured-action__text {
  color: var(--button-text);
}
</style>
