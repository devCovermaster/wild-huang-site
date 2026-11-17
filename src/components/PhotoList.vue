<template>
  <div class="image-gallery">
    <!-- 首次进入页面的全屏手绘露营车 Loading 遮罩 -->
    <Transition name="initial-loader-fade">
      <div v-if="isInitialLoading" class="initial-loading-overlay">
        <VanLoader
          theme="light"
          :progress="initialProgress"
          :show-progress="true"
          text="ON THE ROAD · 正在装载荒野光影..."
        />
      </div>
    </Transition>

    <!-- 顶部控制栏：筛选 + 布局切换 + 随机漫游 -->
    <div
      ref="toolbarRef"
      class="gallery-toolbar-wrapper"
      :class="{ 'is-sticky': isToolbarSticky }"
    >
      <div class="gallery-toolbar">
        <!-- 左侧：画幅筛选标签 -->
        <div class="filter-tabs" role="tablist">
          <button
            v-for="tab in filterTabs"
            :key="tab.key"
            class="filter-tab"
            :class="{ active: activeFilter === tab.key }"
            @click="setFilter(tab.key)"
          >
            <span class="tab-label">{{ tab.label }}</span>
            <span class="tab-count">{{ tab.count }}</span>
          </button>
        </div>

        <!-- 右侧：交互操作（随机漫游 & 列数切换） -->
        <div class="toolbar-actions">
          <button
            class="action-pill"
            :class="{ 'is-shuffling': isShuffling }"
            @click="handleShuffle"
            title="随机打乱图片排列"
          >
            <svg class="action-icon shuffle-svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
              <polyline points="16 3 21 3 21 8"></polyline>
              <line x1="4" y1="20" x2="21" y2="3"></line>
              <polyline points="21 16 21 21 16 21"></polyline>
              <line x1="15" y1="15" x2="21" y2="21"></line>
              <line x1="4" y1="4" x2="9" y2="9"></line>
            </svg>
            <span>随机漫游</span>
          </button>

          <button
            v-if="isShuffled"
            class="action-pill ghost"
            @click="handleResetOrder"
            title="恢复默认编号顺序"
          >
            <span>重置顺序</span>
          </button>

          <!-- 桌面端列数切换器 -->
          <div class="density-switcher" title="切换画廊列数">
            <button
              v-for="col in [2, 3, 4]"
              :key="col"
              class="density-btn"
              :class="{ active: userColumns === col }"
              @click="setUserColumns(col)"
              :aria-label="`${col}列视图`"
            >
              {{ col }}列
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 瀑布流画廊主体 -->
    <div class="gallery-stage" :class="{ 'is-ready': !isInitialLoading }">
      <div
        ref="containerRef"
        class="waterfall-container"
        :style="{ height: `${containerHeight}px` }"
      >
        <div
          v-for="(image, index) in filteredImages"
          :key="image.id"
          class="waterfall-item"
          :style="getItemStyle(index)"
          @click="openLightbox(index)"
        >
          <div class="image-card" :class="{ 'is-loaded': loadedImages.has(image.id) }">
            <!-- 骨架屏占位 -->
            <div v-if="!loadedImages.has(image.id)" class="skeleton-loader">
              <div class="skeleton-shimmer"></div>
              <span class="skeleton-index">NO. {{ formatIndex(image.id) }}</span>
            </div>

            <!-- 实际图片 - 懒加载 -->
            <img
              :src="loadedImages.has(image.id) ? image.url : undefined"
              :data-src="image.url"
              :alt="image.alt || `Wild Huang Photography #${image.id}`"
              class="gallery-image"
              :class="{ loaded: loadedImages.has(image.id) }"
              @load="onImageLoad(image.id)"
              @error="onImageError(image.id)"
            />

            <!-- 悬浮精致遮罩层 -->
            <div class="image-overlay">
              <div class="overlay-top">
                <span class="frame-badge">NO. {{ formatIndex(image.id) }}</span>
                <span class="expand-trigger" aria-hidden="true">
                  <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <polyline points="15 3 21 3 21 9"></polyline>
                    <polyline points="9 21 3 21 3 15"></polyline>
                    <line x1="21" y1="3" x2="14" y2="10"></line>
                    <line x1="3" y1="21" x2="10" y2="14"></line>
                  </svg>
                </span>
              </div>

              <div class="overlay-bottom">
                <div class="meta-info">
                  <span class="meta-ratio">{{ getRatioLabel(image) }}</span>
                  <span class="meta-dot">·</span>
                  <span class="meta-dims">{{ image.width }} × {{ image.height }}</span>
                </div>
                <span class="view-hint">点击预览</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 底部结束标识 -->
      <div v-if="filteredImages.length > 0" class="gallery-footer-mark">
        <span class="mark-line"></span>
        <span class="mark-text">END OF ARCHIVE · {{ filteredImages.length }} FRAMES</span>
        <span class="mark-line"></span>
      </div>
    </div>

    <!-- 悬浮返回顶部按钮 -->
    <Transition name="fade-up">
      <button
        v-if="showBackToTop && !isInitialLoading"
        class="back-to-top-btn"
        @click="scrollToTop"
        aria-label="回到顶部"
      >
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M18 15l-6-6-6 6" stroke-linecap="round" stroke-linejoin="round" />
        </svg>
      </button>
    </Transition>

    <!-- 全屏沉浸式灯箱 (Lightbox) -->
    <Teleport to="body">
      <Transition name="lightbox-fade">
        <div
          v-if="lightboxVisible && currentLightboxImage"
          class="lightbox-backdrop"
          @click="closeLightbox"
          @wheel.prevent="handleLightboxWheel"
          @touchstart.passive="handleTouchStart"
          @touchend.passive="handleTouchEnd"
        >
          <!-- 灯箱顶部栏 -->
          <div class="lightbox-topbar" @click.stop>
            <div class="lightbox-counter">
              <span class="counter-current">{{ formatIndex(lightboxIndex + 1) }}</span>
              <span class="counter-sep">/</span>
              <span class="counter-total">{{ formatIndex(filteredImages.length) }}</span>
              <span class="counter-divider"></span>
              <span class="counter-meta">
                NO. {{ formatIndex(currentLightboxImage.id) }} · {{ getRatioLabel(currentLightboxImage) }} ({{ currentLightboxImage.width }}×{{ currentLightboxImage.height }})
              </span>
            </div>

            <div class="lightbox-tools">
              <!-- 自动放映按钮 -->
              <button
                class="tool-btn"
                :class="{ active: isSlideshowPlaying }"
                @click="toggleSlideshow"
                :title="isSlideshowPlaying ? '暂停自动放映 (Space)' : '开启自动放映 (Space)'"
              >
                <svg v-if="!isSlideshowPlaying" width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                  <polygon points="6 4 20 12 6 20 6 4"></polygon>
                </svg>
                <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                  <rect x="6" y="4" width="4" height="16"></rect>
                  <rect x="14" y="4" width="4" height="16"></rect>
                </svg>
                <span class="tool-label">{{ isSlideshowPlaying ? '放映中' : '放映' }}</span>
              </button>

              <!-- 缩放按钮 -->
              <button
                class="tool-btn"
                :class="{ active: zoomScale > 1 }"
                @click="toggleZoom"
                title="放大/还原 (双击图片)"
              >
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <circle cx="11" cy="11" r="8"></circle>
                  <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
                  <line v-if="zoomScale === 1" x1="11" y1="8" x2="11" y2="14"></line>
                  <line x1="8" y1="11" x2="14" y2="11"></line>
                </svg>
                <span class="tool-label">{{ zoomScale > 1 ? '还原' : '放大' }}</span>
              </button>

              <!-- 下载按钮 -->
              <button
                class="tool-btn"
                @click="downloadImage(currentLightboxImage)"
                title="下载原图"
              >
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
                  <polyline points="7 10 12 15 17 10"></polyline>
                  <line x1="12" y1="15" x2="12" y2="3"></line>
                </svg>
              </button>

              <!-- 关闭按钮 -->
              <button
                class="tool-btn close-tool"
                @click="closeLightbox"
                title="关闭 (ESC)"
              >
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <line x1="18" y1="6" x2="6" y2="18"></line>
                  <line x1="6" y1="6" x2="18" y2="18"></line>
                </svg>
              </button>
            </div>
          </div>

          <!-- 灯箱主舞台 -->
          <div class="lightbox-stage">
            <!-- 左切换按钮 -->
            <button
              class="stage-nav-btn prev"
              @click.stop="prevImage"
              aria-label="上一张 (←)"
              title="上一张 (←)"
            >
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
                <polyline points="15 18 9 12 15 6"></polyline>
              </svg>
            </button>

            <!-- 黑色背景上的手绘露营车 Loading 动画（图片未就绪时显示，杜绝黑屏） -->
            <Transition name="loader-fade">
              <div
                v-if="isLightboxImgLoading"
                class="lightbox-loader-overlay"
                @click.stop
              >
                <VanLoader
                  theme="dark"
                  compact
                  :text="`DEVELOPING NO. ${formatIndex(currentLightboxImage.id)}...`"
                />
              </div>
            </Transition>

            <!-- 图片展示容器 -->
            <div
              class="stage-image-wrapper"
              :class="{
                'is-zoomed': zoomScale > 1,
                'is-dragging': isDragging,
                'is-loading': isLightboxImgLoading
              }"
              @click.stop
              @dblclick="toggleZoom"
              @mousedown="startDrag"
              @mousemove="onDrag"
              @mouseup="endDrag"
              @mouseleave="endDrag"
            >
              <img
                :key="currentLightboxImage.id"
                :src="currentLightboxImage.url"
                :alt="currentLightboxImage.alt || `Photo #${currentLightboxImage.id}`"
                class="stage-image"
                :class="{ 'is-ready': !isLightboxImgLoading }"
                :style="{
                  transform: `translate3d(${panOffset.x}px, ${panOffset.y}px, 0) scale(${
                    isLightboxImgLoading ? 0.97 : zoomScale
                  })`
                }"
                draggable="false"
                @load="onLightboxImgLoad(activeLoadSeq)"
                @error="onLightboxImgLoad(activeLoadSeq)"
              />
            </div>

            <!-- 右切换按钮 -->
            <button
              class="stage-nav-btn next"
              @click.stop="nextImage"
              aria-label="下一张 (→)"
              title="下一张 (→)"
            >
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
                <polyline points="9 18 15 12 9 6"></polyline>
              </svg>
            </button>
          </div>

          <!-- 底部胶片缩略图导览栏 -->
          <div class="lightbox-filmstrip" @click.stop>
            <div class="filmstrip-track" ref="filmstripRef">
              <button
                v-for="(thumb, idx) in filteredImages"
                :key="thumb.id"
                class="filmstrip-item"
                :class="{ active: idx === lightboxIndex }"
                @click="goToLightboxIndex(idx)"
                :title="`NO. ${formatIndex(thumb.id)}`"
              >
                <img :src="thumb.url" :alt="`Thumb ${thumb.id}`" loading="lazy" />
              </button>
            </div>
            <div class="keyboard-hint">
              <span>← / → 切换图片</span>
              <span>·</span>
              <span>双击或滚轮缩放</span>
              <span>·</span>
              <span>Space 自动放映</span>
              <span>·</span>
              <span>ESC 关闭</span>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'
import VanLoader from '@/components/VanLoader.vue'

export interface ImageItem {
  id: string | number
  url: string
  alt?: string
  author?: string
  authorAvatar?: string
  likes?: number
  width: number
  height: number
}

export interface PaginationConfig {
  fetchFunction: (page: number, pageSize: number) => Promise<{
    data: ImageItem[]
    hasMore: boolean
  }>
  pageSize?: number
}

interface Props {
  images?: ImageItem[]
  columnCount?: number
  gap?: number
}

const props = withDefaults(defineProps<Props>(), {
  images: () => [],
  columnCount: 3,
  gap: 20
})

type FilterKey = 'all' | 'landscape' | 'portrait' | 'cinema'

// 状态管理
const toolbarRef = ref<HTMLElement | null>(null)
const containerRef = ref<HTMLElement | null>(null)
const filmstripRef = ref<HTMLElement | null>(null)
const workingImages = ref<ImageItem[]>([])
const loadedImages = ref(new Set<string | number>())
const activeFilter = ref<FilterKey>('all')
const userColumns = ref<number>(props.columnCount)
const viewportWidth = ref<number>(typeof window !== 'undefined' ? window.innerWidth : 1440)
const containerWidth = ref<number>(0)
const isShuffled = ref(false)
const isShuffling = ref(false)
const isToolbarSticky = ref(false)
const showBackToTop = ref(false)
const lazyObserver = ref<IntersectionObserver | null>(null)

// 首屏加载动画状态
const isInitialLoading = ref(true)
const initialProgress = ref(8)
const FIRST_SCREEN_COUNT = 9

// 初始化工作列表
watch(
  () => props.images,
  (newList) => {
    workingImages.value = [...newList]
    isShuffled.value = false
  },
  { immediate: true, deep: true }
)

// 预加载首屏图片，全部就绪后再揭幕页面，彻底消除首屏弹跳
const preloadFirstScreenImages = async () => {
  const startTime = performance.now()
  const targets = workingImages.value.slice(0, FIRST_SCREEN_COUNT)

  if (targets.length === 0) {
    isInitialLoading.value = false
    return
  }

  let finished = 0
  const total = targets.length

  const loadPromises = targets.map((item) => {
    return new Promise<void>((resolve) => {
      const img = new Image()
      const done = () => {
        finished++
        loadedImages.value.add(item.id)
        initialProgress.value = Math.min(100, Math.round((finished / total) * 100))
        resolve()
      }
      img.onload = done
      img.onerror = done
      img.src = item.url
    })
  })

  // 最长等待 3.5 秒保底，防止弱网无限卡住
  const timeoutPromise = new Promise<void>((resolve) => setTimeout(resolve, 3500))
  await Promise.race([Promise.all(loadPromises), timeoutPromise])

  initialProgress.value = 100

  // 保证最少展示 850ms 的露营车动画，让视觉过渡从容有趣
  const elapsed = performance.now() - startTime
  const minDisplayMs = 850
  if (elapsed < minDisplayMs) {
    await new Promise((resolve) => setTimeout(resolve, minDisplayMs - elapsed))
  }

  isInitialLoading.value = false
  document.body.style.overflow = ''
}

// 分类统计与筛选
const isPortrait = (img: ImageItem) => img.height > img.width
const isLandscape = (img: ImageItem) => img.width >= img.height
const isCinema = (img: ImageItem) => img.width / img.height >= 1.7

const filterTabs = computed(() => [
  { key: 'all' as FilterKey, label: '全部作品', count: workingImages.value.length },
  { key: 'landscape' as FilterKey, label: '横画幅', count: workingImages.value.filter(isLandscape).length },
  { key: 'portrait' as FilterKey, label: '竖画幅', count: workingImages.value.filter(isPortrait).length },
  { key: 'cinema' as FilterKey, label: '宽银幕', count: workingImages.value.filter(isCinema).length },
])

const filteredImages = computed<ImageItem[]>(() => {
  switch (activeFilter.value) {
    case 'landscape':
      return workingImages.value.filter(isLandscape)
    case 'portrait':
      return workingImages.value.filter(isPortrait)
    case 'cinema':
      return workingImages.value.filter(isCinema)
    default:
      return workingImages.value
  }
})

// 实际生效的列数（响应式）
const effectiveColumns = computed(() => {
  if (viewportWidth.value < 640) return 1
  if (viewportWidth.value < 1024) return Math.min(2, userColumns.value)
  return userColumns.value
})

// 动态间距
const effectiveGap = computed(() => {
  if (viewportWidth.value < 640) return 16
  if (effectiveColumns.value === 4) return 16
  if (effectiveColumns.value === 2) return 28
  return props.gap
})

// 单列宽度
const columnWidth = computed(() => {
  if (containerWidth.value <= 0) return 0
  const cols = effectiveColumns.value
  const gap = effectiveGap.value
  return (containerWidth.value - gap * (cols - 1)) / cols
})

// 同步预计算所有卡片的精确瀑布流坐标
const layoutResult = computed(() => {
  const cols = effectiveColumns.value
  const colW = columnWidth.value
  const gap = effectiveGap.value
  const positions = new Map<number, { left: number; top: number; width: number; height: number }>()

  if (colW <= 0 || cols <= 0) {
    return { positions, height: 600 }
  }

  const colHeights = new Array(cols).fill(0)

  filteredImages.value.forEach((img, index) => {
    let minCol = 0
    for (let c = 1; c < cols; c++) {
      if (colHeights[c] < colHeights[minCol]) {
        minCol = c
      }
    }

    const ratio = img.height && img.width ? img.height / img.width : 0.667
    const itemHeight = Math.round(colW * ratio)
    const left = Math.round(minCol * (colW + gap))
    const top = Math.round(colHeights[minCol])

    positions.set(index, {
      left,
      top,
      width: Math.round(colW),
      height: itemHeight
    })

    colHeights[minCol] += itemHeight + gap
  })

  const maxHeight = colHeights.length > 0 ? Math.max(...colHeights) : 0
  return {
    positions,
    height: Math.max(maxHeight, 200)
  }
})

const containerHeight = computed(() => layoutResult.value.height)

const getItemStyle = (index: number) => {
  const pos = layoutResult.value.positions.get(index)
  if (!pos) {
    return { opacity: 0 }
  }
  return {
    transform: `translate3d(${pos.left}px, ${pos.top}px, 0)`,
    width: `${pos.width}px`,
    height: `${pos.height}px`
  }
}

// 辅助格式化函数
const formatIndex = (val: string | number): string => {
  const num = Number(val)
  if (isNaN(num)) return String(val)
  return num < 10 ? `0${num}` : String(num)
}

const getRatioLabel = (img: ImageItem): string => {
  const r = img.width / img.height
  if (Math.abs(r - 1.5) < 0.06) return '3:2'
  if (Math.abs(r - 1.333) < 0.06) return '4:3'
  if (Math.abs(r - 1.778) < 0.06) return '16:9'
  if (r > 2.0) return '21:9'
  if (Math.abs(r - 0.667) < 0.06) return '2:3'
  if (Math.abs(r - 0.75) < 0.06) return '3:4'
  if (Math.abs(r - 1) < 0.15) return '1:1'
  return r > 1 ? '横幅' : '竖幅'
}

// 交互操作：切换筛选、随机漫游、列数切换
const setFilter = (key: FilterKey) => {
  if (activeFilter.value === key) return
  activeFilter.value = key
  nextTick(() => {
    setupLazyLoading()
  })
}

const handleShuffle = () => {
  isShuffling.value = true
  const copy = [...workingImages.value]
  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    const temp = copy[i]!
    copy[i] = copy[j]!
    copy[j] = temp
  }
  workingImages.value = copy
  isShuffled.value = true
  nextTick(() => {
    setupLazyLoading()
  })
  setTimeout(() => {
    isShuffling.value = false
  }, 450)
}

const handleResetOrder = () => {
  workingImages.value = [...props.images]
  isShuffled.value = false
  nextTick(() => {
    setupLazyLoading()
  })
}

const setUserColumns = (col: number) => {
  userColumns.value = col
  nextTick(() => {
    setupLazyLoading()
  })
}

// 容器尺寸测量
const updateDimensions = () => {
  viewportWidth.value = window.innerWidth
  if (containerRef.value) {
    containerWidth.value = containerRef.value.clientWidth
  }
}

// 懒加载逻辑
const setupLazyLoading = () => {
  if (!containerRef.value) return

  if (lazyObserver.value) {
    lazyObserver.value.disconnect()
  }

  lazyObserver.value = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const img = entry.target as HTMLImageElement
          const src = img.getAttribute('data-src')
          if (src && !img.getAttribute('src')) {
            img.src = src
          }
          lazyObserver.value?.unobserve(img)
        }
      })
    },
    {
      rootMargin: '320px 0px',
      threshold: 0.01
    }
  )

  const imgs = containerRef.value.querySelectorAll('img[data-src]')
  imgs.forEach((img) => {
    if (!img.getAttribute('src')) {
      lazyObserver.value?.observe(img)
    }
  })
}

const onImageLoad = (id: string | number) => {
  loadedImages.value.add(id)
}

const onImageError = (id: string | number) => {
  loadedImages.value.add(id)
}

// 滚动监听
const handleWindowScroll = () => {
  const y = window.scrollY
  if (toolbarRef.value) {
    const headerHeight = window.innerWidth <= 768 ? 50 : 58
    const rect = toolbarRef.value.getBoundingClientRect()
    isToolbarSticky.value = rect.top <= headerHeight + 1
  } else {
    isToolbarSticky.value = y > 260
  }
  showBackToTop.value = y > 600
}

const scrollToTop = () => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// ======================== 灯箱 (Lightbox) 逻辑 ========================
const lightboxVisible = ref(false)
const lightboxIndex = ref(0)
const zoomScale = ref(1)
const panOffset = ref({ x: 0, y: 0 })
const isDragging = ref(false)
const dragStart = ref({ x: 0, y: 0 })
const isSlideshowPlaying = ref(false)
let slideshowTimer: ReturnType<typeof setInterval> | null = null

// 灯箱大图切换 Loading 状态
const isLightboxImgLoading = ref(false)
const activeLoadSeq = ref(0)
let switchStartTime = 0
let lightboxLoadTimer: ReturnType<typeof setTimeout> | null = null

const currentLightboxImage = computed<ImageItem | null>(() => {
  return filteredImages.value[lightboxIndex.value] || null
})

const beginLightboxFrameLoad = () => {
  if (lightboxLoadTimer) {
    clearTimeout(lightboxLoadTimer)
    lightboxLoadTimer = null
  }
  activeLoadSeq.value++
  switchStartTime = performance.now()
  isLightboxImgLoading.value = true
}

const onLightboxImgLoad = (seq: number) => {
  if (seq !== activeLoadSeq.value) return
  const elapsed = performance.now() - switchStartTime
  // 留出 320ms 最短过渡时间，让手绘露营车 Loading 自然呈现而不发生单帧闪烁
  const remaining = Math.max(0, 320 - elapsed)
  lightboxLoadTimer = setTimeout(() => {
    if (seq === activeLoadSeq.value) {
      isLightboxImgLoading.value = false
    }
  }, remaining)
}

const resetZoom = () => {
  zoomScale.value = 1
  panOffset.value = { x: 0, y: 0 }
  isDragging.value = false
}

const openLightbox = (index: number) => {
  lightboxIndex.value = index
  resetZoom()
  beginLightboxFrameLoad()
  lightboxVisible.value = true
  document.body.style.overflow = 'hidden'
  scrollActiveThumbIntoView()
}

const closeLightbox = () => {
  stopSlideshow()
  if (lightboxLoadTimer) {
    clearTimeout(lightboxLoadTimer)
    lightboxLoadTimer = null
  }
  lightboxVisible.value = false
  isLightboxImgLoading.value = false
  resetZoom()
  document.body.style.overflow = ''
}

const prevImage = () => {
  if (filteredImages.value.length <= 1) return
  resetZoom()
  beginLightboxFrameLoad()
  lightboxIndex.value =
    (lightboxIndex.value - 1 + filteredImages.value.length) % filteredImages.value.length
  scrollActiveThumbIntoView()
}

const nextImage = () => {
  if (filteredImages.value.length <= 1) return
  resetZoom()
  beginLightboxFrameLoad()
  lightboxIndex.value = (lightboxIndex.value + 1) % filteredImages.value.length
  scrollActiveThumbIntoView()
}

const goToLightboxIndex = (index: number) => {
  if (index === lightboxIndex.value) return
  resetZoom()
  beginLightboxFrameLoad()
  lightboxIndex.value = index
  scrollActiveThumbIntoView()
}

// 预加载相邻前后两张大图
watch([lightboxVisible, lightboxIndex], ([visible, idx]) => {
  if (!visible || filteredImages.value.length === 0) return
  const total = filteredImages.value.length
  const nextIdx = (idx + 1) % total
  const prevIdx = (idx - 1 + total) % total
  ;[nextIdx, prevIdx].forEach((i) => {
    const target = filteredImages.value[i]
    if (target) {
      const preloadImg = new Image()
      preloadImg.src = target.url
    }
  })
})

const scrollActiveThumbIntoView = () => {
  nextTick(() => {
    if (!filmstripRef.value) return
    const activeEl = filmstripRef.value.children[lightboxIndex.value] as HTMLElement | undefined
    if (activeEl) {
      activeEl.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' })
    }
  })
}

// 缩放与拖拽平移
const toggleZoom = () => {
  if (isLightboxImgLoading.value) return
  if (zoomScale.value > 1) {
    resetZoom()
  } else {
    zoomScale.value = 2.2
    panOffset.value = { x: 0, y: 0 }
  }
}

const handleLightboxWheel = (e: WheelEvent) => {
  if (!lightboxVisible.value || isLightboxImgLoading.value) return
  if (Math.abs(e.deltaY) < 5) return
  if (e.deltaY < 0) {
    zoomScale.value = Math.min(3.5, +(zoomScale.value + 0.25).toFixed(2))
  } else {
    const next = Math.max(1, +(zoomScale.value - 0.25).toFixed(2))
    zoomScale.value = next
    if (next === 1) panOffset.value = { x: 0, y: 0 }
  }
}

const startDrag = (e: MouseEvent) => {
  if (zoomScale.value <= 1) return
  isDragging.value = true
  dragStart.value = {
    x: e.clientX - panOffset.value.x,
    y: e.clientY - panOffset.value.y
  }
}

const onDrag = (e: MouseEvent) => {
  if (!isDragging.value || zoomScale.value <= 1) return
  panOffset.value = {
    x: e.clientX - dragStart.value.x,
    y: e.clientY - dragStart.value.y
  }
}

const endDrag = () => {
  isDragging.value = false
}

// 移动端触摸左右滑动切换
let touchStartX = 0
let touchStartY = 0
const handleTouchStart = (e: TouchEvent) => {
  const firstTouch = e.touches[0]
  if (e.touches.length === 1 && firstTouch) {
    touchStartX = firstTouch.clientX
    touchStartY = firstTouch.clientY
  }
}

const handleTouchEnd = (e: TouchEvent) => {
  const changedTouch = e.changedTouches[0]
  if (zoomScale.value > 1 || !changedTouch) return
  const dx = changedTouch.clientX - touchStartX
  const dy = changedTouch.clientY - touchStartY
  if (Math.abs(dx) > 50 && Math.abs(dx) > Math.abs(dy)) {
    if (dx < 0) {
      nextImage()
    } else {
      prevImage()
    }
  }
}

// 自动放映
const startSlideshow = () => {
  stopSlideshow()
  isSlideshowPlaying.value = true
  slideshowTimer = setInterval(() => {
    nextImage()
  }, 3200)
}

const stopSlideshow = () => {
  isSlideshowPlaying.value = false
  if (slideshowTimer) {
    clearInterval(slideshowTimer)
    slideshowTimer = null
  }
}

const toggleSlideshow = () => {
  if (isSlideshowPlaying.value) {
    stopSlideshow()
  } else {
    startSlideshow()
  }
}

// 下载图片
const downloadImage = (image: ImageItem) => {
  const link = document.createElement('a')
  link.href = image.url
  link.download = `WildHuang-${formatIndex(image.id)}.jpg`
  link.click()
}

// 键盘快捷键监听
const handleKeydown = (e: KeyboardEvent) => {
  if (!lightboxVisible.value) return

  if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') {
    e.preventDefault()
    prevImage()
  } else if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') {
    e.preventDefault()
    nextImage()
  } else if (e.key === 'Escape') {
    e.preventDefault()
    closeLightbox()
  } else if (e.key === ' ') {
    e.preventDefault()
    toggleSlideshow()
  }
}

let resizeObserver: ResizeObserver | null = null

onMounted(async () => {
  document.body.style.overflow = 'hidden'
  updateDimensions()
  await nextTick()
  updateDimensions()
  await preloadFirstScreenImages()
  setupLazyLoading()

  if (containerRef.value && typeof ResizeObserver !== 'undefined') {
    resizeObserver = new ResizeObserver(() => {
      updateDimensions()
    })
    resizeObserver.observe(containerRef.value)
  }

  window.addEventListener('resize', updateDimensions)
  window.addEventListener('scroll', handleWindowScroll, { passive: true })
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  stopSlideshow()
  if (lightboxLoadTimer) {
    clearTimeout(lightboxLoadTimer)
  }
  document.body.style.overflow = ''
  resizeObserver?.disconnect()
  lazyObserver.value?.disconnect()
  window.removeEventListener('resize', updateDimensions)
  window.removeEventListener('scroll', handleWindowScroll)
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
.image-gallery {
  width: 100%;
  min-height: 100vh;
  background: #fafafa;
  color: #141414;
}

/* 首次进入页面的全屏手绘露营车 Loading 遮罩 */
.initial-loading-overlay {
  position: fixed;
  inset: 0;
  z-index: 1500;
  background: #fafafa;
  display: flex;
  align-items: center;
  justify-content: center;
}

.initial-loader-fade-enter-active,
.initial-loader-fade-leave-active {
  transition: opacity 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

.initial-loader-fade-enter-from,
.initial-loader-fade-leave-to {
  opacity: 0;
}

/* 顶部交互控制栏：与顶部 Header 零间隙无缝衔接 */
.gallery-toolbar-wrapper {
  position: sticky;
  top: 57px;
  z-index: 990;
  margin-top: -1px;
  padding: 12px 40px;
  background: rgba(250, 250, 250, 0.92);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
  transition:
    background 0.3s ease,
    box-shadow 0.3s ease,
    border-color 0.3s ease;
}

.gallery-toolbar-wrapper.is-sticky {
  background: rgba(250, 250, 250, 0.92);
  border-bottom-color: rgba(0, 0, 0, 0.07);
  box-shadow: 0 12px 30px -4px rgba(0, 0, 0, 0.05);
}

.gallery-toolbar {
  width: 100%;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
}

.filter-tabs {
  display: flex;
  align-items: center;
  gap: 6px;
  background: #efefef;
  padding: 4px;
  border-radius: 999px;
}

.filter-tab {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 7px 16px;
  border-radius: 999px;
  border: none;
  background: transparent;
  color: #555;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
  letter-spacing: 0.3px;
}

.filter-tab:hover {
  color: #111;
}

.filter-tab.active {
  background: #141414;
  color: #fff;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.14);
}

.tab-count {
  font-size: 11px;
  opacity: 0.65;
  font-variant-numeric: tabular-nums;
}

.toolbar-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.action-pill {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  padding: 7px 15px;
  border-radius: 999px;
  border: 1px solid rgba(0, 0, 0, 0.12);
  background: #fff;
  color: #1a1a1a;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.25s ease;
}

.action-pill:hover {
  border-color: #141414;
  background: #141414;
  color: #fff;
  transform: translateY(-1px);
}

.action-pill.ghost {
  background: transparent;
  color: #666;
  border-color: rgba(0, 0, 0, 0.1);
}

.action-pill.ghost:hover {
  color: #111;
  background: rgba(0, 0, 0, 0.05);
  border-color: rgba(0, 0, 0, 0.2);
}

.action-icon {
  width: 14px;
  height: 14px;
  transition: transform 0.45s cubic-bezier(0.22, 1, 0.36, 1);
}

.action-pill.is-shuffling .shuffle-svg {
  transform: rotate(180deg);
}

.density-switcher {
  display: flex;
  align-items: center;
  background: #efefef;
  padding: 3px;
  border-radius: 999px;
}

.density-btn {
  border: none;
  background: transparent;
  color: #666;
  font-size: 12px;
  font-weight: 500;
  padding: 5px 12px;
  border-radius: 999px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.density-btn:hover {
  color: #111;
}

.density-btn.active {
  background: #fff;
  color: #111;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
}

/* 瀑布流画廊主体 */
.gallery-stage {
  max-width: 1680px;
  margin: 0 auto;
  padding: 36px 40px 80px;
  opacity: 0;
  transform: translateY(10px);
  transition:
    opacity 0.55s ease,
    transform 0.55s cubic-bezier(0.22, 1, 0.36, 1);
}

.gallery-stage.is-ready {
  opacity: 1;
  transform: translateY(0);
}

.waterfall-container {
  position: relative;
  width: 100%;
  transition: height 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

.waterfall-item {
  position: absolute;
  top: 0;
  left: 0;
  will-change: transform, width, height;
  transition:
    transform 0.55s cubic-bezier(0.22, 1, 0.36, 1),
    width 0.4s cubic-bezier(0.22, 1, 0.36, 1),
    height 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

.image-card {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 6px;
  overflow: hidden;
  background: #ececec;
  cursor: zoom-in;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.04);
  transition:
    box-shadow 0.4s cubic-bezier(0.22, 1, 0.36, 1),
    transform 0.4s cubic-bezier(0.22, 1, 0.36, 1);
}

.image-card:hover {
  box-shadow: 0 18px 40px rgba(0, 0, 0, 0.14);
}

/* 骨架屏 */
.skeleton-loader {
  position: absolute;
  inset: 0;
  background: #ebebeb;
  display: flex;
  align-items: flex-end;
  padding: 14px;
  overflow: hidden;
}

.skeleton-shimmer {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    100deg,
    rgba(255, 255, 255, 0) 20%,
    rgba(255, 255, 255, 0.45) 50%,
    rgba(255, 255, 255, 0) 80%
  );
  background-size: 200% 100%;
  animation: shimmer 1.6s infinite;
}

.skeleton-index {
  position: relative;
  z-index: 1;
  font-size: 11px;
  letter-spacing: 1.5px;
  color: rgba(0, 0, 0, 0.22);
  font-family: 'SF Mono', 'Menlo', monospace;
}

@keyframes shimmer {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}

.gallery-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  opacity: 0;
  transform: scale(1.03);
  transition:
    opacity 0.55s ease,
    transform 0.7s cubic-bezier(0.22, 1, 0.36, 1);
}

.gallery-image.loaded {
  opacity: 1;
  transform: scale(1);
}

.image-card:hover .gallery-image.loaded {
  transform: scale(1.045);
}

/* 卡片悬浮遮罩层 */
.image-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    180deg,
    rgba(0, 0, 0, 0.38) 0%,
    rgba(0, 0, 0, 0) 35%,
    rgba(0, 0, 0, 0) 60%,
    rgba(0, 0, 0, 0.65) 100%
  );
  opacity: 0;
  transition: opacity 0.32s ease;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 16px 18px;
  pointer-events: none;
}

.image-card:hover .image-overlay {
  opacity: 1;
}

.overlay-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  transform: translateY(-6px);
  transition: transform 0.32s cubic-bezier(0.22, 1, 0.36, 1);
}

.overlay-bottom {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  transform: translateY(6px);
  transition: transform 0.32s cubic-bezier(0.22, 1, 0.36, 1);
}

.image-card:hover .overlay-top,
.image-card:hover .overlay-bottom {
  transform: translateY(0);
}

.frame-badge {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 1.5px;
  color: #fff;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(8px);
  padding: 4px 10px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.18);
}

.expand-trigger {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.92);
  color: #111;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.meta-info {
  display: flex;
  align-items: center;
  gap: 6px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 12px;
  letter-spacing: 0.4px;
}

.meta-ratio {
  font-weight: 600;
}

.meta-dims {
  opacity: 0.8;
  font-variant-numeric: tabular-nums;
}

.view-hint {
  font-size: 11px;
  letter-spacing: 0.8px;
  color: rgba(255, 255, 255, 0.85);
  text-transform: uppercase;
}

/* 底部归档标识 */
.gallery-footer-mark {
  margin-top: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  color: #999;
  font-size: 11px;
  letter-spacing: 2.5px;
}

.mark-line {
  width: 48px;
  height: 1px;
  background: rgba(0, 0, 0, 0.12);
}

/* 返回顶部按钮 */
.back-to-top-btn {
  position: fixed;
  right: 32px;
  bottom: 32px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: 1px solid rgba(255, 255, 255, 0.15);
  background: rgba(20, 20, 20, 0.88);
  color: #fff;
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 95;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
  transition: all 0.25s ease;
}

.back-to-top-btn:hover {
  background: #000;
  transform: translateY(-3px);
}

/* ======================== 灯箱 (Lightbox) 样式 ======================== */
.lightbox-backdrop {
  position: fixed;
  inset: 0;
  z-index: 9999;
  background: rgba(8, 8, 10, 0.94);
  backdrop-filter: blur(22px);
  -webkit-backdrop-filter: blur(22px);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  user-select: none;
  color: #fff;
}

.lightbox-topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 28px;
  z-index: 10;
  background: linear-gradient(to bottom, rgba(0, 0, 0, 0.55), transparent);
}

.lightbox-counter {
  display: flex;
  align-items: center;
  gap: 8px;
  font-variant-numeric: tabular-nums;
}

.counter-current {
  font-size: 16px;
  font-weight: 600;
  color: #fff;
}

.counter-sep,
.counter-total {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.5);
}

.counter-divider {
  width: 1px;
  height: 14px;
  background: rgba(255, 255, 255, 0.2);
  margin: 0 8px;
}

.counter-meta {
  font-size: 12px;
  letter-spacing: 0.8px;
  color: rgba(255, 255, 255, 0.65);
}

.lightbox-tools {
  display: flex;
  align-items: center;
  gap: 10px;
}

.tool-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  height: 36px;
  padding: 0 14px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.9);
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.tool-btn:hover {
  background: rgba(255, 255, 255, 0.18);
  border-color: rgba(255, 255, 255, 0.3);
  color: #fff;
}

.tool-btn.active {
  background: #fff;
  color: #111;
  border-color: #fff;
}

.tool-btn.close-tool {
  width: 36px;
  padding: 0;
  justify-content: center;
}

/* 灯箱中央图片区 */
.lightbox-stage {
  position: relative;
  flex: 1;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 80px;
  overflow: hidden;
}

/* 黑色背景上的手绘露营车 Loading 层 */
.lightbox-loader-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 12;
  pointer-events: none;
}

.loader-fade-enter-active,
.loader-fade-leave-active {
  transition: opacity 0.22s ease;
}

.loader-fade-enter-from,
.loader-fade-leave-to {
  opacity: 0;
}

.stage-image-wrapper {
  position: relative;
  max-width: 100%;
  max-height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: zoom-in;
}

.stage-image-wrapper.is-zoomed {
  cursor: grab;
}

.stage-image-wrapper.is-dragging {
  cursor: grabbing;
}

.stage-image {
  max-width: 86vw;
  max-height: calc(100vh - 200px);
  width: auto;
  height: auto;
  object-fit: contain;
  border-radius: 4px;
  box-shadow: 0 25px 70px rgba(0, 0, 0, 0.65);
  opacity: 0;
  transition:
    opacity 0.32s cubic-bezier(0.22, 1, 0.36, 1),
    transform 0.32s cubic-bezier(0.22, 1, 0.36, 1);
}

.stage-image.is-ready {
  opacity: 1;
}

.stage-image-wrapper.is-dragging .stage-image {
  transition: none;
}

/* 左右切换大按钮 */
.stage-nav-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(255, 255, 255, 0.08);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 15;
  backdrop-filter: blur(10px);
  transition: all 0.25s ease;
}

.stage-nav-btn:hover {
  background: rgba(255, 255, 255, 0.22);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-50%) scale(1.08);
}

.stage-nav-btn:active {
  transform: translateY(-50%) scale(0.95);
}

.stage-nav-btn.prev {
  left: 24px;
}

.stage-nav-btn.next {
  right: 24px;
}

/* 底部胶片导览栏 */
.lightbox-filmstrip {
  padding: 12px 28px 18px;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.75), transparent);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  z-index: 10;
}

.filmstrip-track {
  display: flex;
  align-items: center;
  gap: 8px;
  max-width: min(92vw, 1100px);
  overflow-x: auto;
  padding: 6px 8px;
  scrollbar-width: none;
}

.filmstrip-track::-webkit-scrollbar {
  display: none;
}

.filmstrip-item {
  flex: 0 0 auto;
  width: 44px;
  height: 44px;
  border-radius: 5px;
  overflow: hidden;
  border: 2px solid transparent;
  padding: 0;
  background: #222;
  opacity: 0.42;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}

.filmstrip-item:hover {
  opacity: 0.8;
}

.filmstrip-item.active {
  width: 56px;
  height: 44px;
  opacity: 1;
  border-color: #fff;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.45);
}

.filmstrip-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.keyboard-hint {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 11px;
  color: rgba(255, 255, 255, 0.42);
  letter-spacing: 0.5px;
}

/* 过渡动画 */
.lightbox-fade-enter-active,
.lightbox-fade-leave-active {
  transition: opacity 0.28s ease;
}

.lightbox-fade-enter-from,
.lightbox-fade-leave-to {
  opacity: 0;
}

.fade-up-enter-active,
.fade-up-leave-active {
  transition: all 0.25s ease;
}

.fade-up-enter-from,
.fade-up-leave-to {
  opacity: 0;
  transform: translateY(12px);
}

/* 响应式适配 */
@media (max-width: 1024px) {
  .density-switcher {
    display: none;
  }

  .gallery-toolbar-wrapper {
    padding: 12px 20px;
  }

  .gallery-stage {
    padding: 24px 20px 60px;
  }
}

@media (max-width: 768px) {
  .gallery-toolbar-wrapper {
    top: 49px;
    padding: 10px 16px;
  }

  .gallery-toolbar {
    gap: 10px;
  }

  .filter-tabs {
    width: 100%;
    overflow-x: auto;
    justify-content: space-between;
  }

  .filter-tab {
    padding: 6px 12px;
    font-size: 12px;
    white-space: nowrap;
  }

  .toolbar-actions {
    width: 100%;
    justify-content: flex-end;
  }

  .gallery-stage {
    padding: 16px 16px 48px;
  }

  .counter-meta,
  .counter-divider,
  .tool-label,
  .keyboard-hint {
    display: none;
  }

  .lightbox-topbar {
    padding: 14px 16px;
  }

  .lightbox-stage {
    padding: 0 12px;
  }

  .stage-image {
    max-width: 94vw;
    max-height: calc(100vh - 170px);
  }

  .stage-nav-btn {
    width: 40px;
    height: 40px;
  }

  .stage-nav-btn.prev {
    left: 10px;
  }

  .stage-nav-btn.next {
    right: 10px;
  }
}
</style>
