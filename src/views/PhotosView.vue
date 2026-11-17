<template>
  <main class="photos-page">
    <!-- 顶部极简杂志感导言区 -->
    <section class="archive-hero">
      <div class="archive-hero-inner">
        <div class="archive-title-group">
          <p class="archive-eyebrow">VISUAL ARCHIVE · 99 SELECTED FRAMES</p>
          <h1 class="archive-title">Frames in the Wild</h1>
        </div>
        <p class="archive-subtitle">
          荒野、公路与车内慢生活——记录在途的每一瞬光影与自由。点击任意画幅进入沉浸视图，支持键盘方向键切换与细节缩放。
        </p>
      </div>
    </section>

    <!-- 瀑布流画廊组件 -->
    <ImageGallery :images="galleryImages" :column-count="3" :gap="22" />
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ImageGallery, { type ImageItem } from '@/components/PhotoList.vue'

// 99 张摄影作品的真实像素尺寸表 [width, height]，确保首屏零抖动精确瀑布流排版
const IMAGE_DIMENSIONS: Record<number, [number, number]> = {
  1: [2592, 3888],
  2: [3840, 2159],
  3: [3840, 2159],
  4: [1737, 977],
  5: [6000, 4000],
  6: [6000, 4000],
  7: [4032, 3024],
  8: [6000, 4000],
  9: [6840, 4560],
  10: [4240, 2832],
  11: [4240, 2832],
  12: [1616, 1080],
  13: [4240, 2832],
  14: [4240, 2832],
  15: [4240, 2832],
  16: [4240, 2832],
  17: [4240, 2832],
  18: [4240, 2832],
  19: [4240, 2832],
  20: [4240, 2832],
  21: [4240, 2832],
  22: [4240, 2832],
  23: [6000, 4000],
  24: [4240, 2832],
  25: [4240, 2832],
  26: [4240, 2832],
  27: [4240, 2832],
  28: [4240, 2832],
  29: [4240, 2832],
  30: [4240, 2832],
  31: [4240, 2832],
  32: [4240, 2832],
  33: [4240, 2832],
  34: [6000, 4000],
  35: [4240, 2832],
  36: [4240, 2832],
  37: [4240, 2832],
  38: [4240, 2832],
  39: [4240, 2832],
  40: [4240, 2832],
  41: [4240, 2832],
  42: [4240, 2832],
  43: [4240, 2832],
  44: [4240, 2832],
  45: [3981, 1836],
  46: [3120, 2080],
  47: [3120, 2080],
  48: [1620, 1080],
  49: [2400, 1600],
  50: [2400, 1600],
  51: [1920, 1080],
  52: [1080, 949],
  53: [1080, 948],
  54: [1679, 1216],
  55: [5130, 6840],
  56: [4240, 2832],
  57: [6840, 4560],
  58: [4000, 6000],
  59: [6000, 4000],
  60: [3840, 2159],
  61: [3840, 2159],
  62: [1620, 1080],
  63: [6000, 4000],
  64: [6000, 4000],
  65: [6000, 4000],
  66: [1280, 1707],
  67: [4240, 2832],
  68: [1737, 977],
  69: [1920, 2560],
  70: [1280, 1920],
  71: [1280, 1920],
  72: [4000, 6000],
  73: [4000, 6000],
  74: [6000, 4000],
  75: [6000, 4000],
  76: [6000, 4000],
  77: [4032, 3024],
  78: [4240, 2832],
  79: [4240, 2832],
  80: [4032, 3024],
  81: [3024, 4032],
  82: [3024, 4032],
  83: [2070, 1080],
  84: [4032, 3024],
  85: [3024, 4032],
  86: [1620, 1080],
  87: [6000, 4000],
  88: [6000, 4000],
  89: [4240, 2832],
  90: [4032, 3024],
  91: [4032, 3024],
  92: [3024, 4032],
  93: [5472, 3648],
  94: [8262, 11019],
  95: [3024, 4033],
  96: [4032, 3024],
  97: [4028, 2965],
  98: [4032, 3024],
  99: [4032, 3024]
}

const getDocImage = (name: number): string => {
  return new URL(`/src/assets/imgs/${name}.jpg`, import.meta.url).href
}

// 将竖幅、宽画幅与标准横幅作品均匀穿插编排，赋予首屏极具呼吸感的错落瀑布流节奏
const buildCuratedGallery = (total: number): ImageItem[] => {
  const portraits: ImageItem[] = []
  const landscapes: ImageItem[] = []

  for (let i = 1; i <= total; i++) {
    const [width, height] = IMAGE_DIMENSIONS[i] || [4240, 2832]
    const item: ImageItem = {
      id: i,
      url: getDocImage(i),
      alt: `Wild Huang Photography #${i}`,
      width,
      height
    }
    if (height > width) {
      portraits.push(item)
    } else {
      landscapes.push(item)
    }
  }

  // 每隔约 5 张横图穿插 1 张竖图，形成自然的视觉高低差
  const curated: ImageItem[] = []
  let pIdx = 0
  landscapes.forEach((img, index) => {
    curated.push(img)
    if (index % 5 === 1 && pIdx < portraits.length) {
      const pItem = portraits[pIdx++]
      if (pItem) curated.push(pItem)
    }
  })

  while (pIdx < portraits.length) {
    const pItem = portraits[pIdx++]
    if (pItem) curated.push(pItem)
  }

  return curated
}

const galleryImages = ref<ImageItem[]>(buildCuratedGallery(99))
</script>

<style scoped>
.photos-page {
  min-height: 100vh;
  background: #fafafa;
}

.archive-hero {
  background: #fafafa;
  padding: 56px 40px 28px;
}

.archive-hero-inner {
  max-width: 1600px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: 40px;
  flex-wrap: wrap;
}

.archive-eyebrow {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 2.5px;
  color: #888;
  margin-bottom: 10px;
  text-transform: uppercase;
}

.archive-title {
  font-size: 2.75rem;
  font-weight: 300;
  letter-spacing: -0.5px;
  color: #111;
  line-height: 1.15;
  margin: 0;
}

.archive-subtitle {
  max-width: 440px;
  font-size: 13.5px;
  line-height: 1.75;
  color: #666;
  margin: 0;
}

@media (max-width: 768px) {
  .archive-hero {
    padding: 32px 16px 18px;
  }

  .archive-title {
    font-size: 2rem;
  }

  .archive-hero-inner {
    gap: 14px;
  }
}
</style>
