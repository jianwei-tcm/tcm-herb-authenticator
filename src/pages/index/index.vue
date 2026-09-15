<template>
  <view class="home-page">
    <view class="home-page__content">
      <view class="top-panel">
        <text class="home-page__title">真假药材</text>

        <view class="search-bar">
          <image class="search-bar__icon" :src="assets.search" mode="aspectFit" />
          <input
            v-model="searchKeyword"
            class="search-bar__input"
            confirm-type="search"
            type="text"
            placeholder="搜索药材、功效、妙招等"
            placeholder-class="search-bar__placeholder"
            @confirm="handleSearchConfirm"
          />
        </view>
      </view>

      <scroll-view class="recommend-scroll" scroll-x :show-scrollbar="false">
        <view class="hero-card">
          <view class="hero-card__media">
            <image class="hero-card__image" :src="featuredHerb.image" mode="aspectFill" />
          </view>
          <view class="hero-card__content">
            <text class="hero-card__eyebrow">今日推荐药材</text>
            <text class="hero-card__title">{{ featuredHerb.name }}</text>
            <text class="hero-card__subtitle">{{ featuredHerb.description }}</text>
            <view class="hero-card__cta" @tap.stop="handleHerbTap">
              <text>查看详情</text>
            </view>
          </view>
        </view>
      </scroll-view>

      <view class="entry-grid">
        <view
          v-for="entry in featureEntries"
          :key="entry.id"
          class="entry-card"
          :style="{ backgroundColor: entry.background }"
          @tap="handleEntryTap(entry)"
        >
          <view class="entry-card__title">
            <text v-for="line in entry.titleLines" :key="line" class="entry-card__line">
              {{ line }}
            </text>
          </view>
          <view class="entry-card__subtitle">
            <text v-for="line in entry.subtitleLines" :key="line" class="entry-card__subtitle-line">
              {{ line }}
            </text>
          </view>
          <view class="entry-card__badge" :style="{ backgroundColor: entry.iconBackground }">
            <image class="entry-card__icon" :src="entry.icon" mode="aspectFit" />
          </view>
        </view>
      </view>

      <view class="section-head">
        <text class="section-head__title">热门话题</text>
        <view class="section-head__more" @tap="handleMoreTopics">
          <text>更多</text>
          <image class="section-head__more-icon" :src="assets.more" mode="aspectFit" />
        </view>
      </view>

      <view class="topic-list">
        <view
          v-for="topic in hotTopics"
          :key="topic.id"
          class="topic-card"
          :style="{ backgroundColor: topic.background }"
          @tap="handleTopicTap(topic)"
        ></view>
      </view>
    </view>

    <view class="bottom-nav">
      <view
        v-for="item in bottomNavItems"
        :key="item.key"
        class="bottom-nav__item"
        :class="{
          'bottom-nav__item--active': item.active,
          'bottom-nav__item--center': item.center,
        }"
        @tap="handleNavTap(item.key)"
      >
        <image v-if="item.center" class="bottom-nav__add-icon" :src="assets.add" mode="aspectFit" />
        <image v-else class="bottom-nav__icon" :src="item.icon" mode="aspectFit" />
        <text v-if="item.label" class="bottom-nav__label">{{ item.label }}</text>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'

type FeatureAction = 'identify' | 'library' | 'community' | 'challenge'
type NavKey = 'home' | 'community' | 'identify' | 'library' | 'mine'

interface FeatureEntry {
  id: string
  titleLines: string[]
  subtitleLines: string[]
  background: string
  iconBackground: string
  icon: string
  action: FeatureAction
}

interface TopicCard {
  id: string
  background: string
}

interface NavItem {
  key: NavKey
  label: string
  icon: string
  active?: boolean
  center?: boolean
}

const assets = {
  search: '/static/home/search.svg',
  home: '/static/home/home-active.svg',
  community: '/static/home/community.svg',
  library: '/static/home/library.svg',
  mine: '/static/home/mine.svg',
  more: '/static/home/chevron-right.svg',
  camera: '/static/home/camera.svg',
  book: '/static/home/book.svg',
  chat: '/static/home/chat.svg',
  feather: '/static/home/feather.svg',
  herb: '/static/home/huangqi.jpeg',
  add: '/static/home/add.svg',
} as const

const searchKeyword = ref('')

const featuredHerb = {
  id: 'huang-qi',
  name: '黄芪',
  description: '补气升阳、固表止汗',
  image: assets.herb,
}

const featureEntries: FeatureEntry[] = [
  {
    id: 'identify',
    titleLines: ['AI在线', '鉴别'],
    subtitleLines: ['拍照识别药材', '真伪'],
    background: '#e8ebe0',
    iconBackground: '#7f9a77',
    icon: assets.camera,
    action: 'identify',
  },
  {
    id: 'library',
    titleLines: ['药材库'],
    subtitleLines: ['记录药材知识'],
    background: '#f0e4dc',
    iconBackground: '#8e5b47',
    icon: assets.book,
    action: 'library',
  },
  {
    id: 'community',
    titleLines: ['社区话题'],
    subtitleLines: ['分享交流养生'],
    background: '#fcefdf',
    iconBackground: '#f0852f',
    icon: assets.chat,
    action: 'community',
  },
  {
    id: 'challenge',
    titleLines: ['药材', '挑战赛'],
    subtitleLines: ['答题比拼识药', '本领'],
    background: '#f8eee2',
    iconBackground: '#aca25f',
    icon: assets.feather,
    action: 'challenge',
  },
]

const hotTopics: TopicCard[] = [
  {
    id: 'topic-1',
    background: '#668468',
  },
  {
    id: 'topic-2',
    background: '#e8ebe0',
  },
  {
    id: 'topic-3',
    background: '#faeee0',
  },
]

const bottomNavItems: NavItem[] = [
  {
    key: 'home',
    label: '首页',
    icon: assets.home,
    active: true,
  },
  {
    key: 'community',
    label: '社区',
    icon: assets.community,
  },
  {
    key: 'identify',
    label: '',
    center: true,
    icon: '',
  },
  {
    key: 'library',
    label: '药材库',
    icon: assets.library,
  },
  {
    key: 'mine',
    label: '我的',
    icon: assets.mine,
  },
]

function openIdentification() {
  uni.navigateTo({
    url: '/pages/identification/index',
  })
}

function openCameraPage() {
  uni.navigateTo({
    url: '/pages/camera/index',
  })
}

function openMaterials() {
  uni.navigateTo({
    url: '/pages/materials/index',
  })
}

function openProfile() {
  uni.navigateTo({
    url: '/pages/profile/index',
  })
}

function handleSearchConfirm() {
  console.info('home search keyword:', searchKeyword.value)
}

function handleHerbTap() {
  uni.navigateTo({
    url: `/pages/material-detail/index?id=${featuredHerb.id}`,
  })
}

function handleEntryTap(entry: FeatureEntry) {
  if (entry.action === 'identify') {
    openIdentification()
    return
  }

  if (entry.action === 'library') {
    openMaterials()
    return
  }

  uni.showToast({
    title: `${entry.titleLines.join('')}页面待开发`,
    icon: 'none',
  })
}

function handleTopicTap(topic: TopicCard) {
  console.info('TODO: navigate to topic detail', topic.id)
  uni.showToast({
    title: '热门话题内容待补充',
    icon: 'none',
  })
}

function handleMoreTopics() {
  uni.showToast({
    title: '更多内容待补充',
    icon: 'none',
  })
}

function handleNavTap(key: NavKey) {
  if (key === 'home') {
    uni.pageScrollTo({
      scrollTop: 0,
      duration: 200,
    })
    return
  }

  if (key === 'identify') {
    openCameraPage()
    return
  }

  if (key === 'library') {
    openMaterials()
    return
  }

  if (key === 'mine') {
    openProfile()
    return
  }

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

.home-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  background: #ffffff;
  color: #000000;
  overflow-x: hidden;
  box-sizing: border-box;
  font-family: "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.home-page view,
.home-page text,
.home-page image,
.home-page scroll-view {
  box-sizing: border-box;
}

.home-page__content {
  padding-bottom: calc(180rpx + env(safe-area-inset-bottom));
}

.top-panel {
  width: 750rpx;
  height: 244rpx;
  padding: 42rpx 34rpx 0;
  background: rgba(255, 243, 230, 0.45);
}

.home-page__title {
  display: block;
  width: 100%;
  color: #000000;
  font-size: 46rpx;
  font-weight: 700;
  line-height: 52rpx;
  text-align: center;
  white-space: nowrap;
}

.search-bar {
  display: flex;
  align-items: center;
  width: 681rpx;
  height: 71rpx;
  margin-top: 47rpx;
  padding: 0 21rpx;
  border-radius: 19rpx;
  background: #ffffff;
}

.search-bar__icon {
  flex: 0 0 auto;
  width: 46rpx;
  height: 46rpx;
  margin-right: 11rpx;
}

.search-bar__input {
  flex: 1;
  width: 0;
  height: 100%;
  padding: 0;
  border: 0;
  background: transparent;
  font-size: 31rpx;
  line-height: 71rpx;
  color: #222222;
}

.search-bar__placeholder {
  color: #c2c1bf;
  font-size: 31rpx;
}

.recommend-scroll {
  width: 750rpx;
  height: 326rpx;
  margin-top: 37rpx;
  padding-left: 32rpx;
  white-space: nowrap;
}

.hero-card {
  position: relative;
  display: inline-flex;
  overflow: hidden;
  width: 681rpx;
  height: 326rpx;
  border-radius: 29rpx;
  background: #faeee0;
  vertical-align: top;
}

.hero-card__media {
  position: relative;
  flex: 0 0 317rpx;
  width: 317rpx;
  overflow: hidden;
}

.hero-card__image {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  display: block;
  width: 100%;
  height: 100%;
  border-radius: 29rpx 0 0 29rpx;
}

.hero-card__content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
  padding: 69rpx 22rpx 27rpx 48rpx;
}

.hero-card__eyebrow {
  display: block;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 38rpx;
  color: #504638;
  white-space: nowrap;
}

.hero-card__title {
  display: block;
  margin-top: 9rpx;
  font-size: 46rpx;
  font-weight: 700;
  line-height: 55rpx;
  color: #504638;
  white-space: nowrap;
}

.hero-card__subtitle {
  display: block;
  margin-top: 8rpx;
  font-size: 23rpx;
  font-weight: 400;
  line-height: 29rpx;
  color: #b9b09f;
  white-space: nowrap;
}

.hero-card__cta {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  align-self: flex-start;
  width: 160rpx;
  height: 46rpx;
  margin-top: 17rpx;
  border-radius: 19rpx;
  background: #968050;
  color: #ffffff;
  font-size: 25rpx;
  line-height: 1;
}

.entry-grid {
  display: grid;
  grid-template-columns: 321rpx 321rpx;
  column-gap: 40rpx;
  row-gap: 36rpx;
  margin: 44rpx 34rpx 0;
}

.entry-card {
  position: relative;
  width: 321rpx;
  height: 202rpx;
  overflow: hidden;
  border-radius: 19rpx;
}

.entry-card__title {
  position: absolute;
  top: 36rpx;
  left: 36rpx;
  display: flex;
  flex-direction: column;
  max-width: 142rpx;
}

.entry-card:nth-child(2) .entry-card__title {
  top: 59rpx;
}

.entry-card:nth-child(3) .entry-card__title {
  top: 57rpx;
  left: 23rpx;
  max-width: 166rpx;
}

.entry-card:nth-child(4) .entry-card__title {
  top: 34rpx;
  max-width: 142rpx;
}

.entry-card__line + .entry-card__line {
  margin-top: 0;
}

.entry-card__line {
  display: block;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 41rpx;
  color: #000000;
}

.entry-card__subtitle {
  position: absolute;
  top: 130rpx;
  left: 34rpx;
  display: flex;
  flex-direction: column;
  max-width: 216rpx;
}

.entry-card:nth-child(2) .entry-card__subtitle {
  top: 122rpx;
}

.entry-card:nth-child(3) .entry-card__subtitle {
  top: 109rpx;
  left: 34rpx;
}

.entry-card:nth-child(4) .entry-card__subtitle {
  top: 128rpx;
  left: 36rpx;
}

.entry-card__subtitle-line {
  display: block;
  color: rgba(0, 0, 0, 0.56);
  font-size: 23rpx;
  font-weight: 400;
  line-height: 29rpx;
  white-space: nowrap;
}

.entry-card__badge {
  position: absolute;
  top: 57rpx;
  right: 44rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 86rpx;
  height: 86rpx;
  border-radius: 50%;
}

.entry-card__icon {
  width: 55rpx;
  height: 55rpx;
}

.entry-card:nth-child(2) .entry-card__icon,
.entry-card:nth-child(4) .entry-card__icon {
  width: 59rpx;
  height: 59rpx;
}

.section-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 46rpx;
  margin: 67rpx 34rpx 0 57rpx;
}

.section-head__title {
  font-family: "Source Han Serif SC", "Noto Serif SC", "Songti SC", serif;
  font-size: 31rpx;
  font-weight: 700;
  line-height: 46rpx;
  color: #000000;
}

.section-head__more {
  display: flex;
  align-items: center;
  font-size: 23rpx;
  line-height: 1;
  color: #c2c1bf;
}

.section-head__more-icon {
  width: 13rpx;
  height: 23rpx;
  margin-left: 3rpx;
}

.topic-list {
  display: flex;
  flex-direction: column;
  margin: 33rpx 34rpx 0;
}

.topic-card + .topic-card {
  margin-top: 42rpx;
}

.topic-card {
  overflow: hidden;
  width: 100%;
  height: 158rpx;
  border-radius: 29rpx;
}

.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 50%;
  z-index: 20;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  width: 750rpx;
  min-height: 155rpx;
  transform: translateX(-50%);
  padding: 29rpx 0 calc(16rpx + env(safe-area-inset-bottom));
  box-sizing: border-box;
  border-top: 1rpx solid #ece4d2;

  background: #fdf7e9;
}

.bottom-nav__item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-height: 94rpx;
  color: #000000;
}

.bottom-nav__item--active {
  color: #1b4925;
}

.bottom-nav__item--center {
  flex: 1;
}

.bottom-nav__icon {
  display: block;
  width: 46rpx;
  height: 46rpx;
}

.bottom-nav__add-icon {
  display: block;
  width: 57rpx;
  height: 57rpx;
  margin-top: 5rpx;
}

.bottom-nav__label {
  display: block;
  margin-top: 7rpx;
  color: #000000;
  font-size: 23rpx;
  line-height: 29rpx;
  white-space: nowrap;
}
</style>
