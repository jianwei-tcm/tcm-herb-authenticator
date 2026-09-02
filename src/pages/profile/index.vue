<template>
  <view class="profile-page">
    <view class="profile-hero">
      <button class="settings-entry" hover-class="settings-entry--hover" @tap="openSettings">
        <image class="settings-entry__icon" src="/static/profile/settings-white.svg" mode="aspectFit" />
      </button>

      <view class="profile-user">
        <image class="profile-user__avatar" :src="user.avatar" mode="aspectFill" />
        <view class="profile-user__info">
          <view class="profile-user__name-row">
            <text class="profile-user__name">{{ user.nickname }}</text>
            <view class="level-badge">
              <text class="level-badge__text">LV{{ user.level }}</text>
            </view>
          </view>
          <text class="profile-user__id">ID：{{ user.id }}</text>
        </view>
      </view>
    </view>

    <view class="profile-body" />

    <view class="stats-card">
      <view
        v-for="stat in userStats"
        :key="stat.key"
        class="stats-card__item"
        hover-class="stats-card__item--hover"
        @tap="handleStatTap(stat.key)"
      >
        <text class="stats-card__value">{{ stat.value }}</text>
        <text class="stats-card__label">{{ stat.label }}</text>
      </view>
    </view>

    <view class="menu-card">
      <view
        v-for="(item, index) in menuItems"
        :key="item.key"
        class="menu-item"
        :class="{ 'menu-item--last': index === menuItems.length - 1 }"
        hover-class="menu-item--hover"
        @tap="handleMenuTap(item)"
      >
        <image class="menu-item__icon" :src="item.icon" mode="aspectFit" />
        <text class="menu-item__label">{{ item.label }}</text>
        <image class="menu-item__arrow" src="/static/profile/chevron-right.svg" mode="aspectFit" />
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
        <view v-if="item.center" class="bottom-nav__add-icon">
          <image class="bottom-nav__add-icon-outer" src="/static/materials/add-outer.svg" mode="aspectFit" />
          <image class="bottom-nav__add-icon-inner" src="/static/materials/add-inner.svg" mode="aspectFit" />
        </view>
        <image v-else class="bottom-nav__icon" :src="item.icon" mode="aspectFit" />
        <text v-if="item.label" class="bottom-nav__label">{{ item.label }}</text>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
type StatKey = 'favorites' | 'following' | 'fans' | 'likes'
type MenuKey = 'records' | 'posts' | 'favorites' | 'following' | 'feedback' | 'settings'
type NavKey = 'home' | 'community' | 'identify' | 'library' | 'mine'

interface UserProfile {
  nickname: string
  id: string
  level: number
  avatar: string
}

interface UserStat {
  key: StatKey
  label: string
  value: number
}

interface MenuItem {
  key: MenuKey
  label: string
  icon: string
  url: string
}

interface NavItem {
  key: NavKey
  label: string
  icon: string
  active?: boolean
  center?: boolean
}

const user: UserProfile = {
  nickname: '草本小仙女',
  id: '12345678',
  level: 3,
  avatar: '/static/profile/avatar.svg',
}

const userStats: UserStat[] = [
  { key: 'favorites', label: '收藏', value: 26 },
  { key: 'following', label: '关注', value: 18 },
  { key: 'fans', label: '粉丝', value: 36 },
  { key: 'likes', label: '获赞', value: 128 },
]

const menuItems: MenuItem[] = [
  { key: 'records', label: '检测记录', icon: '/static/profile/menu-record.svg', url: '/pages/records/index' },
  { key: 'posts', label: '我发布的', icon: '/static/profile/menu-publish.svg', url: '/pages/posts/index' },
  { key: 'favorites', label: '我的收藏', icon: '/static/profile/menu-favorite.svg', url: '/pages/favorites/index' },
  { key: 'following', label: '我的关注', icon: '/static/profile/menu-follow.svg', url: '/pages/following/index' },
  { key: 'feedback', label: '投诉与反馈', icon: '/static/profile/menu-feedback.svg', url: '/pages/feedback/index' },
  { key: 'settings', label: '设置', icon: '/static/profile/settings-black.svg', url: '/pages/settings/index' },
]

const bottomNavItems: NavItem[] = [
  { key: 'home', label: '首页', icon: '/static/materials/home.svg' },
  { key: 'community', label: '社区', icon: '/static/materials/community.svg' },
  { key: 'identify', label: '', icon: '', center: true },
  { key: 'library', label: '药材库', icon: '/static/profile/library.svg' },
  { key: 'mine', label: '我的', icon: '/static/profile/mine-active.svg', active: true },
]

function navigateTo(url: string) {
  uni.navigateTo({ url })
}

function openSettings() {
  navigateTo('/pages/settings/index')
}

function handleMenuTap(item: MenuItem) {
  navigateTo(item.url)
}

function handleStatTap(key: StatKey) {
  console.info('profile stat tapped:', key)
}

function handleNavTap(key: NavKey) {
  if (key === 'mine') {
    uni.pageScrollTo({
      scrollTop: 0,
      duration: 200,
    })
    return
  }

  if (key === 'home') {
    uni.reLaunch({
      url: '/pages/index/index',
    })
    return
  }

  if (key === 'identify') {
    navigateTo('/pages/camera/index')
    return
  }

  if (key === 'library') {
    navigateTo('/pages/materials/index')
    return
  }

  navigateTo('/pages/community/index')
}
</script>

<style>
page {
  background: #fdf7e9;
}

.profile-page {
  position: relative;
  width: 750rpx;
  min-height: 1626rpx;
  overflow-x: hidden;
  background: #ffffff;
  color: #000000;
  box-sizing: border-box;
  font-family: "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.profile-page view,
.profile-page text,
.profile-page image {
  box-sizing: border-box;
}

.profile-page button {
  margin: 0;
  padding: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  color: inherit;
  font: inherit;
  line-height: normal;
}

.profile-page button::after {
  border: 0;
}

.profile-hero {
  position: absolute;
  top: 0;
  left: 0;
  width: 750rpx;
  height: 426rpx;
  background: #376f46;
}

.settings-entry {
  position: absolute;
  top: calc(env(safe-area-inset-top) + 32rpx);
  right: 44rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 46rpx;
  height: 46rpx;
}

.settings-entry--hover,
.menu-item--hover,
.stats-card__item--hover {
  opacity: 0.68;
}

.settings-entry__icon {
  display: block;
  width: 46rpx;
  height: 46rpx;
}

.profile-user {
  position: absolute;
  top: 126rpx;
  left: 111rpx;
  display: flex;
  align-items: center;
}

.profile-user__avatar {
  display: block;
  width: 172rpx;
  height: 172rpx;
  border-radius: 50%;
}

.profile-user__info {
  min-width: 0;
  margin-left: 30rpx;
  padding-top: 4rpx;
}

.profile-user__name-row {
  display: flex;
  align-items: center;
  min-width: 0;
}

.profile-user__name {
  display: block;
  color: #ffffff;
  font-size: 46rpx;
  font-weight: 400;
  line-height: 58rpx;
  white-space: nowrap;
}

.level-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 92rpx;
  height: 31rpx;
  margin-left: 16rpx;
  border-radius: 100rpx;
  background: #d9d9d9;
}

.level-badge__text {
  display: block;
  color: #ffffff;
  font-size: 23rpx;
  font-weight: 700;
  line-height: 31rpx;
  white-space: nowrap;
}

.profile-user__id {
  display: block;
  margin-top: 2rpx;
  color: rgba(255, 255, 255, 0.73);
  font-size: 29rpx;
  font-weight: 400;
  line-height: 42rpx;
  white-space: nowrap;
}

.profile-body {
  position: absolute;
  top: 426rpx;
  right: 0;
  bottom: 0;
  left: 0;
  background: #ffffff;
}

.stats-card {
  position: absolute;
  top: 349rpx;
  left: 44rpx;
  z-index: 4;
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 658rpx;
  height: 160rpx;
  overflow: hidden;
  border-radius: 29rpx;
  background: #fefaf7;
  box-shadow: 0 18rpx 38rpx rgba(45, 38, 22, 0.08);
}

.stats-card__item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 160rpx;
}

.stats-card__value {
  display: block;
  color: #000000;
  font-size: 38rpx;
  font-weight: 700;
  line-height: 46rpx;
  white-space: nowrap;
}

.stats-card__label {
  display: block;
  margin-top: 2rpx;
  color: #000000;
  font-size: 25rpx;
  font-weight: 400;
  line-height: 31rpx;
  white-space: nowrap;
}

.menu-card {
  position: absolute;
  top: 555rpx;
  left: 42rpx;
  z-index: 2;
  width: 658rpx;
  overflow: hidden;
  background: rgba(254, 250, 248, 0.42);
}

.menu-item {
  display: flex;
  align-items: center;
  width: 658rpx;
  height: 137rpx;
  padding: 0 72rpx 0 69rpx;
  border-bottom: 1rpx solid #d9d9d9;
  background: rgba(254, 250, 248, 0.42);
}

.menu-item--last {
  border-bottom: 0;
}

.menu-item__icon {
  flex: 0 0 auto;
  display: block;
  width: 46rpx;
  height: 46rpx;
}

.menu-item__label {
  display: block;
  margin-left: 31rpx;
  color: #000000;
  font-size: 31rpx;
  font-weight: 400;
  line-height: 46rpx;
  white-space: nowrap;
}

.menu-item__arrow {
  flex: 0 0 auto;
  display: block;
  width: 46rpx;
  height: 46rpx;
  margin-left: auto;
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
  transform: translateX(-50%);
  padding: 19rpx 23rpx calc(19rpx + env(safe-area-inset-bottom));
  box-sizing: border-box;
  border-top: 1rpx solid #000000;
  background: #fdf7e9;
}

.bottom-nav__item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 115rpx;
  color: #000000;
}

.bottom-nav__item--active {
  color: #1b4925;
}

.bottom-nav__item--center {
  flex: 0 0 122rpx;
}

.bottom-nav__icon {
  display: block;
  width: 46rpx;
  height: 46rpx;
}

.bottom-nav__add-icon {
  position: relative;
  width: 57rpx;
  height: 57rpx;
}

.bottom-nav__add-icon-outer,
.bottom-nav__add-icon-inner {
  position: absolute;
  top: 0;
  left: 0;
  display: block;
  width: 57rpx;
  height: 57rpx;
}

.bottom-nav__label {
  display: block;
  margin-top: 4rpx;
  color: inherit;
  font-size: 23rpx;
  line-height: 27rpx;
  white-space: nowrap;
}
</style>
