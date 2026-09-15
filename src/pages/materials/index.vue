<template>
  <view class="materials-page">
    <view class="materials-top">
      <text class="materials-title"></text>

      <view class="materials-search">
        <image class="materials-search__icon" src="/static/materials/search.svg" mode="aspectFit" />
        <input
          v-model="searchKeyword"
          class="materials-search__input"
          confirm-type="search"
          type="text"
          placeholder="搜索药材名称、功效、别名等"
          placeholder-class="materials-search__placeholder"
        />
      </view>

      <view class="category-tabs">
        <button
          v-for="category in categoryOptions"
          :key="category.key"
          class="category-tabs__item"
          :class="{ 'category-tabs__item--active': activeCategory === category.key }"
          hover-class="category-tabs__item--hover"
          @tap="handleCategoryTap(category.key)"
        >
          {{ category.label }}
        </button>
      </view>
    </view>

    <view class="materials-body">
      <scroll-view
        class="material-scroll"
        scroll-y
        scroll-with-animation
        :scroll-into-view="scrollIntoViewId"
        :scroll-top="listScrollTop"
        @scroll="handleListScroll"
      >
        <view class="material-scroll__inner">
          <view
            v-for="group in groupedMaterials"
            :id="getGroupId(group.letter)"
            :key="group.letter"
            class="material-group"
          >
            <view class="material-group__header">
              <text class="material-group__letter">{{ group.letter }}</text>
            </view>

            <view
              v-for="item in group.items"
              :key="item.id"
              class="material-card"
              @tap="handleMaterialTap(item)"
            >
              <view class="material-card__main">
                <text class="material-card__name">{{ item.name }}</text>
                <text class="material-card__alias">别名：{{ item.aliases.join('、') }}</text>
              </view>
              <text class="material-card__effect">{{ item.effects.join('、') }}</text>
            </view>
          </view>

          <view v-if="groupedMaterials.length === 0" class="material-empty">
            <text class="material-empty__title">暂无匹配药材</text>
            <text class="material-empty__text">请更换药材名称、别名或功效关键词</text>
          </view>
        </view>
      </scroll-view>

      <view class="alphabet-index">
        <view
          v-for="letter in alphabet"
          :key="letter"
          class="alphabet-index__item"
          :class="{
            'alphabet-index__item--active': activeLetter === letter && hasMaterials(letter),
            'alphabet-index__item--available': activeLetter !== letter && hasMaterials(letter),
            'alphabet-index__item--empty': !hasMaterials(letter),
          }"
          @tap="handleAlphabetTap(letter)"
        >
          <text class="alphabet-index__text">{{ letter }}</text>
        </view>
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
import { computed, nextTick, onMounted, ref, watch } from 'vue'

type CategoryKey = 'plant' | 'animal' | 'mineral'
type NavKey = 'home' | 'community' | 'identify' | 'library' | 'mine'

interface CategoryOption {
  key: CategoryKey
  label: string
}

interface MaterialItem {
  id: string
  category: CategoryKey
  initial: string
  name: string
  aliases: string[]
  effects: string[]
}

interface MaterialGroup {
  letter: string
  items: MaterialItem[]
}

interface NavItem {
  key: NavKey
  label: string
  icon: string
  active?: boolean
  center?: boolean
}

interface ScrollEvent {
  detail?: {
    scrollTop?: number
  }
}

interface NodeRect {
  top?: number
}

const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('')
const activeCategory = ref<CategoryKey>('plant')
const searchKeyword = ref('')
const activeLetter = ref('A')
const scrollIntoViewId = ref('')
const listScrollTop = ref(0)
const groupOffsets = ref<Record<string, number>>({})

let measureTimer: ReturnType<typeof setTimeout> | undefined
let currentScrollTop = 0
let activeLetterFrame: number | null = null

const categoryOptions: CategoryOption[] = [
  { key: 'plant', label: '植物药' },
  { key: 'animal', label: '动物药' },
  { key: 'mineral', label: '矿物药' },
]

const materialItems: MaterialItem[] = [
  { id: 'ai-ye', category: 'plant', initial: 'A', name: '艾叶', aliases: ['艾蒿', '医草'], effects: ['温经止血', '散寒止痛'] },
  { id: 'an-xi-xiang', category: 'plant', initial: 'A', name: '安息香', aliases: ['拙贝罗香'], effects: ['开窍辟秽', '行气活血'] },
  { id: 'bai-zhu', category: 'plant', initial: 'B', name: '白术', aliases: ['于术', '冬术'], effects: ['补气健脾', '燥湿利水'] },
  { id: 'bai-shao', category: 'plant', initial: 'B', name: '白芍', aliases: ['杭芍', '白芍药'], effects: ['养血调经', '柔肝止痛'] },
  { id: 'ban-lan-gen', category: 'plant', initial: 'B', name: '板蓝根', aliases: ['蓝靛根'], effects: ['清热解毒', '凉血利咽'] },
  { id: 'chen-pi', category: 'plant', initial: 'C', name: '陈皮', aliases: ['橘皮', '广陈皮'], effects: ['理气健脾', '燥湿化痰'] },
  { id: 'chuan-bei', category: 'plant', initial: 'C', name: '川贝', aliases: ['川贝母'], effects: ['清热润肺', '化痰止咳'] },
  { id: 'dan-shen', category: 'plant', initial: 'D', name: '丹参', aliases: ['赤参', '紫丹参'], effects: ['活血祛瘀', '通经止痛'] },
  { id: 'dang-gui', category: 'plant', initial: 'D', name: '当归', aliases: ['秦归', '云归'], effects: ['补血活血', '调经止痛'] },
  { id: 'fang-feng', category: 'plant', initial: 'F', name: '防风', aliases: ['关防风'], effects: ['祛风解表', '胜湿止痛'] },
  { id: 'fu-ling', category: 'plant', initial: 'F', name: '茯苓', aliases: ['云苓', '白茯苓'], effects: ['利水渗湿', '健脾宁心'] },
  { id: 'gan-cao', category: 'plant', initial: 'G', name: '甘草', aliases: ['甜草', '国老'], effects: ['补脾益气', '清热解毒'] },
  { id: 'huang-qi', category: 'plant', initial: 'H', name: '黄芪', aliases: ['北芪', '绵芪'], effects: ['补气升阳', '固表止汗'] },
  { id: 'huang-jing', category: 'plant', initial: 'H', name: '黄精', aliases: ['鸡头黄精'], effects: ['补气养阴', '健脾润肺'] },
  { id: 'jin-yin-hua', category: 'plant', initial: 'J', name: '金银花', aliases: ['忍冬花'], effects: ['清热解毒', '疏散风热'] },
  { id: 'jie-geng', category: 'plant', initial: 'J', name: '桔梗', aliases: ['包袱花'], effects: ['宣肺利咽', '祛痰排脓'] },
  { id: 'lian-qiao', category: 'plant', initial: 'L', name: '连翘', aliases: ['连壳'], effects: ['清热解毒', '消肿散结'] },
  { id: 'long-dan', category: 'plant', initial: 'L', name: '龙胆', aliases: ['胆草'], effects: ['清热燥湿', '泻肝胆火'] },
  { id: 'mai-dong', category: 'plant', initial: 'M', name: '麦冬', aliases: ['麦门冬'], effects: ['养阴生津', '润肺清心'] },
  { id: 'mu-dan-pi', category: 'plant', initial: 'M', name: '牡丹皮', aliases: ['丹皮'], effects: ['清热凉血', '活血化瘀'] },
  { id: 'qiang-huo', category: 'plant', initial: 'Q', name: '羌活', aliases: ['羌青'], effects: ['解表散寒', '祛风胜湿'] },
  { id: 'ren-shen', category: 'plant', initial: 'R', name: '人参', aliases: ['园参', '山参'], effects: ['大补元气', '补脾益肺'] },
  { id: 'san-qi', category: 'plant', initial: 'S', name: '三七', aliases: ['田七'], effects: ['散瘀止血', '消肿定痛'] },
  { id: 'tian-ma', category: 'plant', initial: 'T', name: '天麻', aliases: ['赤箭'], effects: ['息风止痉', '平抑肝阳'] },
  { id: 'wu-wei-zi', category: 'plant', initial: 'W', name: '五味子', aliases: ['辽五味'], effects: ['收敛固涩', '益气生津'] },
  { id: 'xuan-shen', category: 'plant', initial: 'X', name: '玄参', aliases: ['元参'], effects: ['清热凉血', '滋阴降火'] },
  { id: 'xin-yi', category: 'plant', initial: 'X', name: '辛夷', aliases: ['木笔花'], effects: ['散风寒', '通鼻窍'] },
  { id: 'yi-mu-cao', category: 'plant', initial: 'Y', name: '益母草', aliases: ['坤草'], effects: ['活血调经', '利尿消肿'] },
  { id: 'yan-hu-suo', category: 'plant', initial: 'Y', name: '延胡索', aliases: ['元胡'], effects: ['活血行气', '止痛'] },
  { id: 'a-jiao', category: 'animal', initial: 'A', name: '阿胶', aliases: ['驴皮胶'], effects: ['补血滋阴', '润燥止血'] },
  { id: 'bie-jia', category: 'animal', initial: 'B', name: '鳖甲', aliases: ['团鱼甲'], effects: ['滋阴潜阳', '软坚散结'] },
  { id: 'chan-tui', category: 'animal', initial: 'C', name: '蝉蜕', aliases: ['蝉衣'], effects: ['疏散风热', '利咽透疹'] },
  { id: 'di-long', category: 'animal', initial: 'D', name: '地龙', aliases: ['蚯蚓'], effects: ['清热息风', '通络平喘'] },
  { id: 'hai-piao-xiao', category: 'animal', initial: 'H', name: '海螵蛸', aliases: ['乌贼骨'], effects: ['收敛止血', '制酸止痛'] },
  { id: 'ji-nei-jin', category: 'animal', initial: 'J', name: '鸡内金', aliases: ['鸡肫皮'], effects: ['健胃消食', '涩精止遗'] },
  { id: 'lu-rong', category: 'animal', initial: 'L', name: '鹿茸', aliases: ['斑龙珠'], effects: ['补肾阳', '益精血'] },
  { id: 'she-tui', category: 'animal', initial: 'S', name: '蛇蜕', aliases: ['蛇皮'], effects: ['祛风定惊', '退翳'] },
  { id: 'shui-zhi', category: 'animal', initial: 'S', name: '水蛭', aliases: ['蚂蟥'], effects: ['破血通经', '逐瘀消癥'] },
  { id: 'tu-bie-chong', category: 'animal', initial: 'T', name: '土鳖虫', aliases: ['土元'], effects: ['破血逐瘀', '续筋接骨'] },
  { id: 'wu-shao-she', category: 'animal', initial: 'W', name: '乌梢蛇', aliases: ['乌蛇'], effects: ['祛风通络', '止痉'] },
  { id: 'xiong-dan-fen', category: 'animal', initial: 'X', name: '熊胆粉', aliases: ['熊胆'], effects: ['清热解毒', '明目'] },
  { id: 'chi-shi-zhi', category: 'mineral', initial: 'C', name: '赤石脂', aliases: ['红高岭'], effects: ['涩肠止泻', '收敛止血'] },
  { id: 'ci-shi', category: 'mineral', initial: 'C', name: '磁石', aliases: ['吸铁石'], effects: ['镇惊安神', '平肝潜阳'] },
  { id: 'dai-zhe-shi', category: 'mineral', initial: 'D', name: '代赭石', aliases: ['赭石'], effects: ['平肝潜阳', '降逆止呕'] },
  { id: 'hua-shi', category: 'mineral', initial: 'H', name: '滑石', aliases: ['液石'], effects: ['利尿通淋', '清热解暑'] },
  { id: 'long-gu', category: 'mineral', initial: 'L', name: '龙骨', aliases: ['五花龙骨'], effects: ['镇惊安神', '收敛固涩'] },
  { id: 'lu-gan-shi', category: 'mineral', initial: 'L', name: '炉甘石', aliases: ['甘石'], effects: ['解毒明目', '收湿止痒'] },
  { id: 'mang-xiao', category: 'mineral', initial: 'M', name: '芒硝', aliases: ['朴硝'], effects: ['泻下通便', '润燥软坚'] },
  { id: 'qing-meng-shi', category: 'mineral', initial: 'Q', name: '青礞石', aliases: ['礞石'], effects: ['坠痰下气', '平肝镇惊'] },
  { id: 'shi-gao', category: 'mineral', initial: 'S', name: '石膏', aliases: ['细理石'], effects: ['清热泻火', '除烦止渴'] },
  { id: 'xuan-ming-fen', category: 'mineral', initial: 'X', name: '玄明粉', aliases: ['风化硝'], effects: ['泻热通便', '软坚散结'] },
  { id: 'yang-qi-shi', category: 'mineral', initial: 'Y', name: '阳起石', aliases: ['羊起石'], effects: ['温肾壮阳'] },
]

const bottomNavItems: NavItem[] = [
  { key: 'home', label: '首页', icon: '/static/materials/home.svg' },
  { key: 'community', label: '社区', icon: '/static/materials/community.svg' },
  { key: 'identify', label: '', icon: '', center: true },
  { key: 'library', label: '药材库', icon: '/static/materials/library-active.svg', active: true },
  { key: 'mine', label: '我的', icon: '/static/materials/mine.svg' },
]

const filteredMaterials = computed(() => {
  const keyword = searchKeyword.value.trim().toLowerCase()

  return materialItems.filter((item) => {
    if (item.category !== activeCategory.value) {
      return false
    }

    if (!keyword) {
      return true
    }

    return [item.name, item.initial, ...item.aliases, ...item.effects]
      .join(' ')
      .toLowerCase()
      .includes(keyword)
  })
})

const groupedMaterials = computed<MaterialGroup[]>(() => {
  return alphabet
    .map((letter) => ({
      letter,
      items: filteredMaterials.value.filter((item) => item.initial === letter),
    }))
    .filter((group) => group.items.length > 0)
})

const availableLetters = computed(() => new Set(groupedMaterials.value.map((group) => group.letter)))
const firstAvailableLetter = computed(() => groupedMaterials.value[0]?.letter ?? '')

watch(
  [activeCategory, searchKeyword],
  () => {
    if (activeLetter.value !== firstAvailableLetter.value) {
      activeLetter.value = firstAvailableLetter.value
    }
    resetListScroll()
    nextTick(scheduleGroupMeasure)
  },
  { flush: 'post' },
)

onMounted(() => {
  if (activeLetter.value !== firstAvailableLetter.value) {
    activeLetter.value = firstAvailableLetter.value
  }
  scheduleGroupMeasure()
})

function getGroupId(letter: string) {
  return `material-group-${letter}`
}

function hasMaterials(letter: string) {
  return availableLetters.value.has(letter)
}

function handleCategoryTap(key: CategoryKey) {
  if (activeCategory.value === key) {
    return
  }

  activeCategory.value = key
  searchKeyword.value = ''
}

function handleAlphabetTap(letter: string) {
  if (!hasMaterials(letter)) {
    return
  }

  if (activeLetter.value !== letter) {
    activeLetter.value = letter
  }
  scrollIntoViewId.value = ''
  nextTick(() => {
    scrollIntoViewId.value = getGroupId(letter)
    scheduleGroupMeasure()
  })
}

function handleListScroll(event: ScrollEvent) {
  currentScrollTop = event.detail?.scrollTop ?? 0

  if (activeLetterFrame !== null) {
    return
  }

  activeLetterFrame = requestAnimationFrame(() => {
    activeLetterFrame = null
    updateActiveLetterByScroll(currentScrollTop)
  })
}

function updateActiveLetterByScroll(scrollTop: number) {
  const groups = groupedMaterials.value

  if (groups.length === 0) {
    if (activeLetter.value !== '') {
      activeLetter.value = ''
    }
    return
  }

  let current = groups[0].letter

  for (const group of groups) {
    const offset = groupOffsets.value[group.letter]

    if (typeof offset === 'number' && scrollTop + 12 >= offset) {
      current = group.letter
    }
  }

  if (activeLetter.value !== current) {
    activeLetter.value = current
  }
}

function resetListScroll() {
  currentScrollTop = 0
  scrollIntoViewId.value = ''
  listScrollTop.value = 1
  nextTick(() => {
    listScrollTop.value = 0
  })
}

function scheduleGroupMeasure() {
  if (measureTimer) {
    clearTimeout(measureTimer)
  }

  measureTimer = setTimeout(measureGroupOffsets, 80)
}

function measureGroupOffsets() {
  if (groupedMaterials.value.length === 0) {
    groupOffsets.value = {}
    return
  }

  const query = uni.createSelectorQuery()
  query.select('.material-scroll').boundingClientRect()

  groupedMaterials.value.forEach((group) => {
    query.select(`#${getGroupId(group.letter)}`).boundingClientRect()
  })

  query.exec((result) => {
    const rects = result as Array<NodeRect | null>
    const containerRect = rects[0]
    const nextOffsets: Record<string, number> = {}

    if (!containerRect || typeof containerRect.top !== 'number') {
      return
    }

    const containerTop = containerRect.top

    groupedMaterials.value.forEach((group, index) => {
      const rect = rects[index + 1]

      if (rect && typeof rect.top === 'number') {
        nextOffsets[group.letter] = currentScrollTop + rect.top - containerTop
      }
    })

    groupOffsets.value = nextOffsets
    updateActiveLetterByScroll(currentScrollTop)
  })
}

function handleMaterialTap(item: MaterialItem) {
  uni.navigateTo({
    url: `/pages/material-detail/index?id=${encodeURIComponent(item.id)}&name=${encodeURIComponent(item.name)}&efficacy=${encodeURIComponent(item.effects.join('、'))}`,
  })
}

function handleNavTap(key: NavKey) {
  if (key === 'library') {
    resetListScroll()
    if (activeLetter.value !== firstAvailableLetter.value) {
      activeLetter.value = firstAvailableLetter.value
    }
    return
  }

  if (key === 'home') {
    uni.reLaunch({
      url: '/pages/index/index',
    })
    return
  }

  if (key === 'identify') {
    uni.navigateTo({
      url: '/pages/camera/index',
    })
    return
  }

  if (key === 'mine') {
    uni.navigateTo({
      url: '/pages/profile/index',
    })
    return
  }

  const labels: Record<Exclude<NavKey, 'home' | 'identify' | 'library' | 'mine'>, string> = {
    community: '社区',
  }

  uni.showToast({
    title: `${labels[key]}页面待开发`,
    icon: 'none',
  })
}
</script>

<style>
page {
  background: #fdf7e9;
}

.materials-page {
  position: relative;
  width: 750rpx;
  height: 1626rpx;
  min-height: 100vh;
  overflow: hidden;
  background: #fdf7e9;
  color: #000000;
  box-sizing: border-box;
  font-family: "Source Sans Pro", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.materials-page button {
  margin: 0;
  padding: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  color: inherit;
  font: inherit;
  line-height: normal;
}

.materials-page button::after {
  border: 0;
}

.materials-top {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 8;
  width: 750rpx;
  height: 328rpx;
  background: #fdf7e9;
}

.materials-title {
  position: absolute;
  top: 36rpx;
  left: 0;
  display: block;
  width: 750rpx;
  color: #000000;
  font-size: 46rpx;
  font-weight: 700;
  line-height: 58rpx;
  text-align: center;
  white-space: nowrap;
}

.materials-search {
  position: absolute;
  top: 109rpx;
  left: 34rpx;
  display: flex;
  align-items: center;
  width: 682rpx;
  height: 71rpx;
  padding: 0 22rpx;
  box-sizing: border-box;
  border-radius: 19rpx;
  background: #ffffff;
}

.materials-search__icon {
  flex: 0 0 auto;
  width: 46rpx;
  height: 46rpx;
  margin-right: 10rpx;
}

.materials-search__input {
  flex: 1;
  width: 0;
  height: 71rpx;
  padding: 0;
  color: #1e1e1e;
  font-size: 31rpx;
  line-height: 71rpx;
}

.materials-search__placeholder {
  color: #c2c1bf;
  font-size: 31rpx;
}

.category-tabs {
  position: absolute;
  top: 214rpx;
  left: 83rpx;
  right: 83rpx;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.materials-page .category-tabs__item {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 138rpx;
  height: 63rpx;
  padding: 0 17rpx;
  box-sizing: border-box;
  border-radius: 25rpx;
  background: #ffffff;
  color: #706e69;
  font-size: 38rpx;
  font-weight: 400;
  line-height: 48rpx;
  white-space: nowrap;
}

.materials-page .category-tabs__item--active {
  background: #1b4925;
  color: #ffffff;
}

.materials-page .category-tabs__item--hover {
  opacity: 0.72;
}

.materials-body {
  position: absolute;
  top: 328rpx;
  right: 0;
  bottom: 154rpx;
  left: 0;
  overflow: hidden;
}

.material-scroll {
  width: 750rpx;
  height: 100%;
}

.material-scroll__inner {
  min-height: 100%;
  padding-bottom: 40rpx;
  box-sizing: border-box;
}

.material-group {
  position: relative;
}

.material-group__header {
  display: flex;
  align-items: center;
  height: 36rpx;
  padding-left: 32rpx;
  box-sizing: border-box;
  background: rgba(254, 250, 248, 0.82);
}

.material-group__letter {
  color: #000000;
  font-size: 34rpx;
  font-weight: 700;
  line-height: 36rpx;
}

.material-card {
  margin: 20rpx 58rpx 20rpx 0;
  padding: 18rpx 28rpx 18rpx 32rpx;
  box-sizing: border-box;
  border-radius: 14rpx;
  border: 1rpx solid rgba(27, 73, 37, 0.08);
  background: rgba(254, 250, 248, 0.86);
}

.material-card__main {
  display: flex;
  align-items: baseline;
  min-width: 0;
}

.material-card__name {
  flex: 0 0 auto;
  color: #1b4925;
  font-size: 34rpx;
  font-weight: 700;
  line-height: 42rpx;
  white-space: nowrap;
}

.material-card__alias {
  flex: 1;
  min-width: 0;
  margin-left: 18rpx;
  overflow: hidden;
  color: #706e69;
  font-size: 24rpx;
  line-height: 34rpx;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.material-card__effect {
  display: block;
  margin-top: 10rpx;
  color: #8d867d;
  font-size: 25rpx;
  line-height: 36rpx;
}

.material-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 520rpx;
  padding-right: 58rpx;
  box-sizing: border-box;
}

.material-empty__title {
  color: #1b4925;
  font-size: 34rpx;
  font-weight: 700;
  line-height: 44rpx;
}

.material-empty__text {
  margin-top: 14rpx;
  color: #8d867d;
  font-size: 26rpx;
  line-height: 36rpx;
}

.alphabet-index {
  position: absolute;
  top: 63rpx;
  right: 8rpx;
  z-index: 12;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 46rpx;
  padding: 8rpx 4rpx;
  box-sizing: border-box;
  border-radius: 23rpx;
  background: rgba(254, 250, 248, 0.82);
}

.alphabet-index__item {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 38rpx;
  height: 34rpx;
  border-radius: 17rpx;
  box-sizing: border-box;
}

.alphabet-index__text {
  color: inherit;
  font-size: 23rpx;
  line-height: 31rpx;
  text-align: center;
  white-space: nowrap;
}

.alphabet-index__item--active {
  background: rgba(27, 73, 37, 0.9);
  color: #ffffff;
  font-weight: 700;
}

.alphabet-index__item--available {
  color: rgba(27, 73, 37, 0.72);
  font-weight: 400;
}

.alphabet-index__item--empty {
  color: rgba(184, 184, 184, 0.78);
  font-weight: 400;
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
  color: #706e69;
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
  display: block;
  top: 0;
  left: 0;
  width: 57rpx;
  height: 57rpx;
}

.bottom-nav__label {
  display: block;
  margin-top: 4rpx;
  color: #000000;
  font-size: 23rpx;
  line-height: 27rpx;
  white-space: nowrap;
}
</style>
