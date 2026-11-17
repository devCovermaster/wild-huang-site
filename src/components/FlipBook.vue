<script setup lang="ts">
import {
  onMounted,
  onBeforeUnmount,
  ref,
  nextTick,
  computed,
  watch
} from 'vue'
import VanLoader from '@/components/VanLoader.vue'

declare global {
  interface Window {
    jQuery: any
    $: any
  }
  interface JQuery {
    turn(options?: any): JQuery
    turn(method: string, ...args: any[]): any
  }
}

export interface BookPageMeta {
  pageNumber: number
  title: string
  subtitle?: string
  thumb?: string
}

interface Props {
  pagesMeta?: BookPageMeta[]
}

const props = withDefaults(defineProps<Props>(), {
  pagesMeta: () => []
})

const bookRef = ref<HTMLElement | null>(null)
const containerRef = ref<HTMLElement | null>(null)
const stageRef = ref<HTMLElement | null>(null)
const thumbStripRef = ref<HTMLElement | null>(null)

const currentPage = ref(1)
const totalPages = ref(0)
const isLoaded = ref(false)
const loadingProgress = ref(10)
const isFullscreen = ref(false)
const isDarkStudio = ref(false)
const isAutoPlaying = ref(false)
const showIndexDrawer = ref(false)
const isMobileView = ref(false)

let autoPlayTimer: ReturnType<typeof setInterval> | null = null

// 按需安全加载外部脚本，防止路由切换时重复注入覆盖 jQuery 实例
const loadScriptOnce = (src: string, checkFn: () => boolean): Promise<void> => {
  if (checkFn()) return Promise.resolve()

  return new Promise((resolve, reject) => {
    const existing = document.querySelector(`script[src="${src}"]`) as HTMLScriptElement | null
    if (existing) {
      if (checkFn()) {
        resolve()
        return
      }
      existing.addEventListener('load', () => resolve(), { once: true })
      existing.addEventListener('error', reject, { once: true })
      return
    }

    const script = document.createElement('script')
    script.src = src
    script.onload = () => resolve()
    script.onerror = reject
    document.head.appendChild(script)
  })
}

// 预加载画册前几页核心图片并更新进度条（避免页数增多时一次性阻塞等待全部图片）
const preloadBookImages = async (): Promise<void> => {
  if (!bookRef.value) return
  const allImgs = Array.from(bookRef.value.querySelectorAll('img'))
  const imgElements = allImgs.slice(0, 8)
  if (imgElements.length === 0) {
    loadingProgress.value = 100
    return
  }

  let loadedCount = 0
  const total = imgElements.length

  const promises = imgElements.map((imgEl) => {
    return new Promise<void>((resolve) => {
      const done = () => {
        loadedCount++
        loadingProgress.value = Math.min(
          100,
          25 + Math.round((loadedCount / total) * 75)
        )
        resolve()
      }

      if (imgEl.complete && imgEl.naturalWidth > 0) {
        done()
      } else {
        imgEl.addEventListener('load', done, { once: true })
        imgEl.addEventListener('error', done, { once: true })
      }
    })
  })

  const timeoutPromise = new Promise<void>((resolve) => setTimeout(resolve, 3500))
  await Promise.race([Promise.all(promises), timeoutPromise])
  loadingProgress.value = 100
}

// 计算理想画册尺寸：去除顶部 Header 后最大化利用视口空间
const computeBookMetrics = () => {
  const mobile = window.innerWidth < 768
  const stageW = stageRef.value?.clientWidth || window.innerWidth - 64
  const stageH = stageRef.value?.clientHeight || window.innerHeight - 110
  const pageRatio = 0.75 // 单页 3:4，双页 3:2

  if (mobile) {
    const maxW = Math.max(260, stageW - 20)
    const maxH = Math.max(360, stageH - 12)
    let bookH = Math.min(maxH, maxW / pageRatio, 680)
    let bookW = Math.round(bookH * pageRatio)
    if (bookW > maxW) {
      bookW = Math.round(maxW)
      bookH = Math.round(bookW / pageRatio)
    }
    return { mobile, bookW, bookH, display: 'single' as const }
  } else {
    const spreadRatio = pageRatio * 2 // 1.5
    const maxW = Math.max(580, stageW - 120)
    const maxH = Math.max(440, stageH - 12)
    const capH = isFullscreen.value ? 920 : 840

    const bookH = Math.round(Math.min(maxH, maxW / spreadRatio, capH))
    let bookW = Math.round(bookH * spreadRatio)
    if (bookW % 2 !== 0) bookW += 1

    return { mobile, bookW, bookH, display: 'double' as const }
  }
}

const resizeBook = () => {
  if (!stageRef.value || !bookRef.value || !window.$) return
  const $book = window.$(bookRef.value)
  if (!$book.data || !$book.data('turn')) return

  const { mobile, bookW, bookH, display } = computeBookMetrics()
  isMobileView.value = mobile

  const currentDisplay = $book.turn('display')
  if (currentDisplay !== display) {
    $book.turn('display', display)
  }

  $book.turn('size', bookW, bookH)
}

// 切换画室日夜主题
const toggleStudioTheme = () => {
  isDarkStudio.value = !isDarkStudio.value
}

// 全屏切换
const toggleFullscreen = async () => {
  if (!containerRef.value) return

  try {
    if (!isFullscreen.value) {
      if (containerRef.value.requestFullscreen) {
        await containerRef.value.requestFullscreen()
      } else if ((containerRef.value as any).webkitRequestFullscreen) {
        await (containerRef.value as any).webkitRequestFullscreen()
      }
    } else {
      if (document.exitFullscreen) {
        await document.exitFullscreen()
      } else if ((document as any).webkitExitFullscreen) {
        await (document as any).webkitExitFullscreen()
      }
    }
  } catch (error) {
    console.error('Fullscreen toggle failed:', error)
  }
}

const handleFullscreenChange = () => {
  isFullscreen.value = !!(
    document.fullscreenElement || (document as any).webkitFullscreenElement
  )
  setTimeout(() => {
    resizeBook()
  }, 120)
}

// 翻页控制
const prevPage = () => {
  if (bookRef.value && window.$) {
    window.$(bookRef.value).turn('previous')
  }
}

const nextPage = () => {
  if (bookRef.value && window.$) {
    window.$(bookRef.value).turn('next')
  }
}

const goToPage = (page: number) => {
  if (bookRef.value && window.$) {
    const target = Math.max(1, Math.min(totalPages.value, page))
    window.$(bookRef.value).turn('page', target)
  }
}

// 自动翻阅功能
const startAutoPlay = () => {
  stopAutoPlay()
  isAutoPlaying.value = true
  autoPlayTimer = setInterval(() => {
    if (currentPage.value >= totalPages.value) {
      goToPage(1)
    } else {
      nextPage()
    }
  }, 3400)
}

const stopAutoPlay = () => {
  isAutoPlaying.value = false
  if (autoPlayTimer) {
    clearInterval(autoPlayTimer)
    autoPlayTimer = null
  }
}

const toggleAutoPlay = () => {
  if (isAutoPlaying.value) {
    stopAutoPlay()
  } else {
    startAutoPlay()
  }
}

const canGoPrev = computed(() => currentPage.value > 1)
const canGoNext = computed(() => currentPage.value < totalPages.value)

// 合上书时的精确居中状态计算：
// 双页模式下，P.1 位于双页容器右半侧，向左平移 25% 即严格处于屏幕正中央；
// 偶数封底页位于双页容器左半侧，向右平移 25% 即严格处于屏幕正中央。
const bookPositionState = computed(() => {
  if (isMobileView.value) return 'is-single'
  if (currentPage.value === 1) return 'is-closed-front'
  if (
    totalPages.value > 1 &&
    currentPage.value === totalPages.value &&
    totalPages.value % 2 === 0
  ) {
    return 'is-closed-back'
  }
  return 'is-spread'
})

// 页码指示文案
const spreadLabel = computed(() => {
  if (totalPages.value === 0) return '01'
  if (currentPage.value === 1) return 'COVER · 封面'
  if (currentPage.value === totalPages.value) return `BACK COVER · 封底`
  if (isMobileView.value) {
    return `PAGE ${formatNum(currentPage.value)} / ${formatNum(totalPages.value)}`
  }
  const evenLeft = currentPage.value % 2 === 0 ? currentPage.value : currentPage.value - 1
  const oddRight = Math.min(totalPages.value, evenLeft + 1)
  return `P. ${formatNum(evenLeft)} — ${formatNum(oddRight)} / ${formatNum(totalPages.value)}`
})

const progressPercent = computed(() => {
  if (totalPages.value <= 1) return 0
  return ((currentPage.value - 1) / (totalPages.value - 1)) * 100
})

const formatNum = (n: number) => (n < 10 ? `0${n}` : `${n}`)

// 同步缩略图抽屉滚动位置
watch([currentPage, showIndexDrawer], ([page, open]) => {
  if (!open) return
  nextTick(() => {
    if (!thumbStripRef.value) return
    const activeEl = thumbStripRef.value.children[page - 1] as HTMLElement | undefined
    if (activeEl) {
      activeEl.scrollIntoView({ behavior: 'smooth', inline: 'center', block: 'nearest' })
    }
  })
})

// 键盘快捷键
const handleKeydown = (event: KeyboardEvent) => {
  if (!isLoaded.value) return
  if (event.key === 'ArrowLeft') {
    event.preventDefault()
    prevPage()
  } else if (event.key === 'ArrowRight') {
    event.preventDefault()
    nextPage()
  } else if (event.key === 'Escape' && showIndexDrawer.value) {
    showIndexDrawer.value = false
  }
}

onMounted(async () => {
  const startTime = performance.now()
  try {
    await loadScriptOnce(
      'https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js',
      () => !!window.jQuery
    )
    loadingProgress.value = 18

    await loadScriptOnce(
      `${location.protocol}//${location.host}/turn/turn.min.js`,
      () => !!(window.jQuery && window.jQuery.fn && window.jQuery.fn.turn)
    )
    loadingProgress.value = 25

    await nextTick()
    await preloadBookImages()

    if (bookRef.value && window.$) {
      const $book = window.$(bookRef.value)
      const { mobile, bookW, bookH, display } = computeBookMetrics()
      isMobileView.value = mobile

      $book.turn({
        width: bookW,
        height: bookH,
        autoCenter: false, // 由外层 CSS translate3d 精确控制合上/展开时的居中过渡
        display,
        gradients: true,
        elevation: 65,
        acceleration: true,
        duration: 850,
        pages: $book.children().length,
        when: {
          turning: (_event: any, page: number) => {
            currentPage.value = page
          },
          turned: (_event: any, page: number) => {
            currentPage.value = page
          }
        }
      })

      totalPages.value = $book.turn('pages')
      currentPage.value = 1
      resizeBook()
    }

    // 保证露营车 Loading 至少展示 850ms，过渡自然从容
    const elapsed = performance.now() - startTime
    if (elapsed < 850) {
      await new Promise((r) => setTimeout(r, 850 - elapsed))
    }

    isLoaded.value = true
    await nextTick()
    resizeBook()

    window.addEventListener('resize', resizeBook)
    window.addEventListener('keydown', handleKeydown)
    document.addEventListener('fullscreenchange', handleFullscreenChange)
    document.addEventListener('webkitfullscreenchange', handleFullscreenChange)
  } catch (error) {
    console.error('Failed to initialize photobook:', error)
    isLoaded.value = true
  }
})

onBeforeUnmount(() => {
  stopAutoPlay()
  window.removeEventListener('resize', resizeBook)
  window.removeEventListener('keydown', handleKeydown)
  document.removeEventListener('fullscreenchange', handleFullscreenChange)
  document.removeEventListener('webkitfullscreenchange', handleFullscreenChange)
})

defineExpose({
  goToPage,
  nextPage,
  prevPage
})
</script>

<template>
  <div
    ref="containerRef"
    class="flipbook-studio"
    :class="{
      'is-fullscreen': isFullscreen,
      'is-dark': isDarkStudio
    }"
  >
    <!-- 手绘露营车 Loading 遮罩 -->
    <Transition name="studio-loader-fade">
      <div v-if="!isLoaded" class="studio-loading-overlay">
        <VanLoader
          :theme="isDarkStudio ? 'dark' : 'light'"
          :progress="loadingProgress"
          :show-progress="true"
          text="BINDING ZINE · 正在装帧荒野画册..."
        />
      </div>
    </Transition>

    <!-- 画册顶部极简状态与控制栏 -->
    <div class="studio-subbar" :class="{ 'is-ready': isLoaded }">
      <div class="subbar-left">
        <router-link to="/" class="studio-brand-link" title="返回 Wild Huang 首页">
          WILD HUANG
        </router-link>
        <span class="subbar-sep">/</span>
        <span class="edition-tag">VANLIFE MONOGRAPH · 视觉画册</span>
      </div>

      <div class="subbar-actions">
        <!-- 目录抽屉开关 -->
        <button
          v-if="pagesMeta.length > 0"
          class="studio-action-btn"
          :class="{ active: showIndexDrawer }"
          @click="showIndexDrawer = !showIndexDrawer"
          title="展开/收起页面目录"
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="3" y="3" width="7" height="7" rx="1"></rect>
            <rect x="14" y="3" width="7" height="7" rx="1"></rect>
            <rect x="14" y="14" width="7" height="7" rx="1"></rect>
            <rect x="3" y="14" width="7" height="7" rx="1"></rect>
          </svg>
          <span>画册目录</span>
        </button>

        <!-- 自动翻阅开关 -->
        <button
          class="studio-action-btn"
          :class="{ active: isAutoPlaying }"
          @click="toggleAutoPlay"
          title="自动翻页阅读"
        >
          <svg v-if="!isAutoPlaying" width="14" height="14" viewBox="0 0 24 24" fill="currentColor">
            <polygon points="6 4 20 12 6 20 6 4"></polygon>
          </svg>
          <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="currentColor">
            <rect x="6" y="4" width="4" height="16"></rect>
            <rect x="14" y="4" width="4" height="16"></rect>
          </svg>
          <span>{{ isAutoPlaying ? '翻阅中' : '自动翻阅' }}</span>
        </button>

        <!-- 画室环境光切换（日间画室 / 夜间暗房） -->
        <button
          class="studio-action-btn"
          @click="toggleStudioTheme"
          :title="isDarkStudio ? '切换到日间画室' : '切换到夜间暗房'"
        >
          <svg v-if="!isDarkStudio" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
          </svg>
          <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="5"></circle>
            <line x1="12" y1="1" x2="12" y2="3"></line>
            <line x1="12" y1="21" x2="12" y2="23"></line>
            <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
            <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
            <line x1="1" y1="12" x2="3" y2="12"></line>
            <line x1="21" y1="12" x2="23" y2="12"></line>
          </svg>
          <span>{{ isDarkStudio ? '日间画室' : '暗房模式' }}</span>
        </button>

        <!-- 全屏阅读按钮 -->
        <button
          class="studio-action-btn icon-only"
          @click="toggleFullscreen"
          :title="isFullscreen ? '退出全屏' : '全屏阅读'"
          :aria-label="isFullscreen ? '退出全屏' : '全屏阅读'"
        >
          <svg v-if="!isFullscreen" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
          <svg v-else width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M8 3v3a2 2 0 0 1-2 2H3m18 0h-3a2 2 0 0 1-2-2V3m0 18v-3a2 2 0 0 1 2-2h3M3 16h3a2 2 0 0 1 2 2v3" stroke-linecap="round" stroke-linejoin="round" />
          </svg>
        </button>
      </div>
    </div>

    <!-- 中央阅读舞台 -->
    <div ref="stageRef" class="studio-stage" :class="{ 'is-ready': isLoaded }">
      <!-- 左侧上一页悬浮按钮 -->
      <button
        class="stage-flip-btn prev"
        :disabled="!canGoPrev"
        @click="prevPage"
        aria-label="上一页"
        title="上一页 (←)"
      >
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
          <polyline points="15 18 9 12 15 6"></polyline>
        </svg>
      </button>

      <!-- 拟真书本外框与居中平移动画容器 -->
      <div class="book-pedestal" :class="bookPositionState">
        <div class="flipbook" ref="bookRef">
          <slot />
        </div>
      </div>

      <!-- 右侧下一页悬浮按钮 -->
      <button
        class="stage-flip-btn next"
        :disabled="!canGoNext"
        @click="nextPage"
        aria-label="下一页"
        title="下一页 (→)"
      >
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
          <polyline points="9 18 15 12 9 6"></polyline>
        </svg>
      </button>
    </div>

    <!-- 可展开的画册章节/缩略图快速跳转抽屉 -->
    <Transition name="drawer-slide">
      <div v-if="showIndexDrawer && pagesMeta.length > 0" class="index-drawer">
        <div class="drawer-header">
          <span class="drawer-title">ALL PAGES · 画册全览 ({{ totalPages }}P)</span>
          <button class="drawer-close" @click="showIndexDrawer = false">收起 ×</button>
        </div>
        <div ref="thumbStripRef" class="drawer-strip">
          <button
            v-for="item in pagesMeta"
            :key="item.pageNumber"
            class="drawer-page-card"
            :class="{
              active:
                currentPage === item.pageNumber ||
                (!isMobileView &&
                  currentPage > 1 &&
                  currentPage < totalPages &&
                  Math.floor(currentPage / 2) === Math.floor(item.pageNumber / 2))
            }"
            @click="goToPage(item.pageNumber)"
          >
            <div class="card-thumb">
              <img v-if="item.thumb" :src="item.thumb" :alt="item.title" loading="lazy" />
              <div v-else class="card-text-placeholder">
                <span>{{ item.pageNumber === 1 ? 'COVER' : item.pageNumber === totalPages ? 'END' : 'TEXT' }}</span>
              </div>
            </div>
            <div class="card-meta">
              <span class="card-pnum">P.{{ formatNum(item.pageNumber) }}</span>
              <span class="card-ptitle">{{ item.title }}</span>
            </div>
          </button>
        </div>
      </div>
    </Transition>

    <!-- 底部极简悬浮胶囊导览坞 -->
    <div class="studio-dock-wrapper" :class="{ 'is-ready': isLoaded }">
      <div class="studio-dock">
        <button
          class="dock-text-btn"
          :disabled="!canGoPrev"
          @click="goToPage(1)"
        >
          封面
        </button>

        <button
          class="dock-arrow-btn"
          :disabled="!canGoPrev"
          @click="prevPage"
          aria-label="上一页"
        >
          ‹
        </button>

        <div class="dock-progress-group" @click="showIndexDrawer = !showIndexDrawer" title="点击展开画册目录">
          <div class="dock-spread-label">{{ spreadLabel }}</div>
          <div class="dock-progress-track">
            <div
              class="dock-progress-fill"
              :style="{ width: `${progressPercent}%` }"
            ></div>
          </div>
        </div>

        <button
          class="dock-arrow-btn"
          :disabled="!canGoNext"
          @click="nextPage"
          aria-label="下一页"
        >
          ›
        </button>

        <button
          class="dock-text-btn"
          :disabled="!canGoNext"
          @click="goToPage(totalPages)"
        >
          封底
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.flipbook-studio {
  --studio-bg: #f1eee8;
  --studio-bg-accent: #e5e0d6;
  --studio-text: #1c1b18;
  --studio-muted: #78746c;
  --studio-pill-bg: rgba(255, 255, 255, 0.78);
  --studio-pill-border: rgba(0, 0, 0, 0.08);
  --studio-dock-bg: rgba(255, 255, 255, 0.88);

  position: relative;
  width: 100%;
  height: 100vh;
  padding-top: 0;
  background: radial-gradient(circle at 50% 45%, var(--studio-bg) 0%, var(--studio-bg-accent) 100%);
  color: var(--studio-text);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  overflow: hidden;
  user-select: none;
  transition: background 0.45s ease, color 0.45s ease;
}

.flipbook-studio.is-dark {
  --studio-bg: #18181b;
  --studio-bg-accent: #0e0e10;
  --studio-text: #f3f1ec;
  --studio-muted: #8e8c84;
  --studio-pill-bg: rgba(34, 34, 38, 0.82);
  --studio-pill-border: rgba(255, 255, 255, 0.12);
  --studio-dock-bg: rgba(26, 26, 30, 0.88);
}

/* Loading 遮罩 */
.studio-loading-overlay {
  position: fixed;
  inset: 0;
  z-index: 1500;
  background: var(--studio-bg);
  display: flex;
  align-items: center;
  justify-content: center;
}

.studio-loader-fade-enter-active,
.studio-loader-fade-leave-active {
  transition: opacity 0.5s cubic-bezier(0.22, 1, 0.36, 1);
}

.studio-loader-fade-enter-from,
.studio-loader-fade-leave-to {
  opacity: 0;
}

/* 顶部紧凑工具栏 */
.studio-subbar {
  width: 100%;
  padding: 12px 36px 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  z-index: 20;
  opacity: 0;
  transform: translateY(-6px);
  transition: all 0.5s ease;
}

.studio-subbar.is-ready {
  opacity: 1;
  transform: translateY(0);
}

.subbar-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.studio-brand-link {
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 1.8px;
  color: var(--studio-text);
  text-decoration: none;
  transition: opacity 0.2s ease;
}

.studio-brand-link:hover {
  opacity: 0.6;
}

.subbar-sep {
  font-size: 11px;
  color: var(--studio-muted);
  opacity: 0.4;
}

.edition-tag {
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 1.6px;
  color: var(--studio-muted);
}

.subbar-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.studio-action-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  height: 32px;
  padding: 0 13px;
  border-radius: 999px;
  border: 1px solid var(--studio-pill-border);
  background: var(--studio-pill-bg);
  color: var(--studio-text);
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  backdrop-filter: blur(12px);
  transition: all 0.22s ease;
}

.studio-action-btn:hover {
  transform: translateY(-1px);
  border-color: var(--studio-text);
}

.studio-action-btn.active {
  background: var(--studio-text);
  color: var(--studio-bg);
  border-color: var(--studio-text);
}

.studio-action-btn.icon-only {
  width: 32px;
  padding: 0;
  justify-content: center;
}

/* 中央阅读舞台：最大化留给画册 */
.studio-stage {
  position: relative;
  flex: 1;
  min-height: 0;
  width: 100%;
  max-width: 1680px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 4px 32px;
  overflow: hidden;
  opacity: 0;
  transform: scale(0.975);
  transition:
    opacity 0.6s cubic-bezier(0.22, 1, 0.36, 1),
    transform 0.6s cubic-bezier(0.22, 1, 0.36, 1);
}

.studio-stage.is-ready {
  opacity: 1;
  transform: scale(1);
}

/* 书本底座：处理合上书（封面/封底）与双页展开时的精确丝滑居中 */
.book-pedestal {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  filter: drop-shadow(0 28px 55px rgba(0, 0, 0, 0.18));
  transition:
    transform 0.65s cubic-bezier(0.22, 1, 0.36, 1),
    filter 0.4s ease;
}

.book-pedestal.is-closed-front {
  transform: translate3d(-25%, 0, 0);
}

.book-pedestal.is-closed-back {
  transform: translate3d(25%, 0, 0);
}

.book-pedestal.is-spread,
.book-pedestal.is-single {
  transform: translate3d(0, 0, 0);
}

.flipbook-studio.is-dark .book-pedestal {
  filter: drop-shadow(0 32px 65px rgba(0, 0, 0, 0.68));
}

.flipbook {
  cursor: grab;
  margin-left: 0 !important;
}

.flipbook:active {
  cursor: grabbing;
}

/* 书页通用样式与中缝立体阴影 */
:deep(.page) {
  width: 100%;
  height: 100%;
  background: #fbfaf7;
  color: #1c1b18;
  position: relative;
  overflow: hidden;
  box-sizing: border-box;
}

:deep(.page img) {
  max-width: 100%;
  max-height: 100%;
}

/* 奇数页（右侧页）左侧书脊装订阴影 */
:deep(.page:nth-child(odd)) {
  box-shadow:
    inset 18px 0 28px -12px rgba(0, 0, 0, 0.13),
    inset 1px 0 0 rgba(0, 0, 0, 0.08),
    inset -1px 0 0 rgba(0, 0, 0, 0.06);
}

/* 偶数页（左侧页）右侧书脊装订阴影 */
:deep(.page:nth-child(even)) {
  box-shadow:
    inset -18px 0 28px -12px rgba(0, 0, 0, 0.13),
    inset -1px 0 0 rgba(0, 0, 0, 0.08),
    inset 1px 0 0 rgba(0, 0, 0, 0.06);
}

/* 白色精装封面与封底的精装书槽立体阴影 */
:deep(.page.is-cover-page) {
  box-shadow:
    inset 10px 0 18px -8px rgba(0, 0, 0, 0.12),
    inset 1px 0 0 rgba(0, 0, 0, 0.08),
    inset 0 0 0 1px rgba(0, 0, 0, 0.06);
}

/* 舞台左右翻页按钮 */
.stage-flip-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 46px;
  height: 46px;
  border-radius: 50%;
  border: 1px solid var(--studio-pill-border);
  background: var(--studio-pill-bg);
  color: var(--studio-text);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 25;
  backdrop-filter: blur(12px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}

.stage-flip-btn:hover:not(:disabled) {
  background: var(--studio-text);
  color: var(--studio-bg);
  transform: translateY(-50%) scale(1.08);
}

.stage-flip-btn:disabled {
  opacity: 0.25;
  cursor: not-allowed;
}

.stage-flip-btn.prev {
  left: 28px;
}

.stage-flip-btn.next {
  right: 28px;
}

/* 画册目录快速跳转抽屉 */
.index-drawer {
  position: absolute;
  left: 50%;
  bottom: 66px;
  transform: translateX(-50%);
  width: min(92vw, 980px);
  background: var(--studio-dock-bg);
  border: 1px solid var(--studio-pill-border);
  border-radius: 16px;
  padding: 14px 18px;
  z-index: 40;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.18);
}

.drawer-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.drawer-title {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 1.5px;
  color: var(--studio-muted);
}

.drawer-close {
  border: none;
  background: transparent;
  color: var(--studio-muted);
  font-size: 12px;
  cursor: pointer;
}

.drawer-close:hover {
  color: var(--studio-text);
}

.drawer-strip {
  display: flex;
  gap: 10px;
  overflow-x: auto;
  padding-bottom: 4px;
  scrollbar-width: thin;
}

.drawer-page-card {
  flex: 0 0 auto;
  width: 76px;
  border: 1px solid var(--studio-pill-border);
  border-radius: 8px;
  background: rgba(128, 128, 128, 0.06);
  padding: 5px;
  cursor: pointer;
  text-align: left;
  color: var(--studio-text);
  transition: all 0.2s ease;
}

.drawer-page-card:hover {
  transform: translateY(-2px);
  border-color: var(--studio-text);
}

.drawer-page-card.active {
  border-color: #b85d38;
  background: rgba(184, 93, 56, 0.12);
}

.card-thumb {
  width: 100%;
  aspect-ratio: 3 / 4;
  border-radius: 4px;
  overflow: hidden;
  background: #eee;
  margin-bottom: 5px;
}

.card-thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.card-text-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 9px;
  letter-spacing: 1px;
  color: #777;
  background: #f5f3ee;
}

.card-meta {
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.card-pnum {
  font-size: 9px;
  color: var(--studio-muted);
  font-variant-numeric: tabular-nums;
}

.card-ptitle {
  font-size: 10px;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.drawer-slide-enter-active,
.drawer-slide-leave-active {
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}

.drawer-slide-enter-from,
.drawer-slide-leave-to {
  opacity: 0;
  transform: translateX(-50%) translateY(12px);
}

/* 底部悬浮控制胶囊坞 */
.studio-dock-wrapper {
  width: 100%;
  padding: 2px 20px 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 30;
  opacity: 0;
  transform: translateY(8px);
  transition: all 0.5s ease 0.1s;
}

.studio-dock-wrapper.is-ready {
  opacity: 1;
  transform: translateY(0);
}

.studio-dock {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 5px 14px;
  border-radius: 999px;
  background: var(--studio-dock-bg);
  border: 1px solid var(--studio-pill-border);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
}

.dock-text-btn {
  border: none;
  background: transparent;
  color: var(--studio-muted);
  font-size: 12px;
  font-weight: 500;
  padding: 4px 8px;
  border-radius: 999px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.dock-text-btn:hover:not(:disabled) {
  color: var(--studio-text);
  background: rgba(128, 128, 128, 0.1);
}

.dock-text-btn:disabled {
  opacity: 0.32;
  cursor: not-allowed;
}

.dock-arrow-btn {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  border: none;
  background: rgba(128, 128, 128, 0.1);
  color: var(--studio-text);
  font-size: 16px;
  line-height: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
}

.dock-arrow-btn:hover:not(:disabled) {
  background: var(--studio-text);
  color: var(--studio-bg);
}

.dock-arrow-btn:disabled {
  opacity: 0.28;
  cursor: not-allowed;
}

.dock-progress-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  min-width: 170px;
  padding: 2px 10px;
  cursor: pointer;
}

.dock-spread-label {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 1.4px;
  color: var(--studio-text);
  font-variant-numeric: tabular-nums;
}

.dock-progress-track {
  width: 100%;
  height: 3px;
  border-radius: 999px;
  background: rgba(128, 128, 128, 0.2);
  overflow: hidden;
}

.dock-progress-fill {
  height: 100%;
  background: #b85d38;
  border-radius: 999px;
  transition: width 0.45s cubic-bezier(0.22, 1, 0.36, 1);
}

@media (max-width: 768px) {
  .studio-subbar {
    padding: 10px 14px 4px;
  }

  .edition-tag,
  .subbar-sep {
    display: none;
  }

  .studio-stage {
    padding: 4px 10px;
  }

  .stage-flip-btn {
    width: 36px;
    height: 36px;
  }

  .stage-flip-btn.prev {
    left: 8px;
  }

  .stage-flip-btn.next {
    right: 8px;
  }

  .dock-progress-group {
    min-width: 120px;
  }
}
</style>
