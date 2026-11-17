<script setup lang="ts">
import { ref, computed } from 'vue'
import FlipBook, { type BookPageMeta } from '@/components/FlipBook.vue'
import VanLoader from '@/components/VanLoader.vue'

const getImgUrl = (id: number): string => {
  return new URL(`/src/assets/imgs/${id}.jpg`, import.meta.url).href
}

interface InnerPageConfig {
  pageNumber: number
  layout: 'full-bleed' | 'museum-mat' | 'polaroid-note' | 'portrait-frame' | 'duo-stack'
  imgId: number
  secondaryImgId?: number
  plate: string
  title: string
  subtitle: string
  note?: string
  meta?: string
}

const flipBookRef = ref<InstanceType<typeof FlipBook> | null>(null)

// 56 页完整摄影画册（P.01 封面 + P.02 序言 + P.03~P.55 共 53 页内页 + P.56 封底）
// 精选 src/assets/imgs/ 下 68 张不重复摄影作品，涵盖五种杂志画册版式
const innerPages: InnerPageConfig[] = [
  {
    pageNumber: 3,
    layout: 'full-bleed',
    imgId: 58,
    plate: 'PLATE 01',
    title: 'The Departure',
    subtitle: '启程 · 驶离城市边界',
    meta: 'CH.01 · AUTUMN'
  },
  {
    pageNumber: 4,
    layout: 'museum-mat',
    imgId: 2,
    plate: 'PLATE 02',
    title: 'Highway to Horizon',
    subtitle: '驶向无垠天际',
    note: '清晨六点的公路尚无车流，唯有引擎低鸣与远山晨光相伴。',
    meta: '3840 × 2159 · 16:9'
  },
  {
    pageNumber: 5,
    layout: 'polaroid-note',
    imgId: 6,
    plate: 'PLATE 03',
    title: 'Where Asphalt Ends',
    subtitle: '柏油路尽头的荒野',
    note: 'When the paved road fades into gravel and pine needles, the real vanlife story begins.',
    meta: 'CH.01 · ON THE ROAD'
  },
  {
    pageNumber: 6,
    layout: 'duo-stack',
    imgId: 3,
    secondaryImgId: 4,
    plate: 'PLATE 04',
    title: 'Winding Trails',
    subtitle: '山路十八弯 · 双帧纪实',
    meta: 'STORYBOARD · 16:9'
  },
  {
    pageNumber: 7,
    layout: 'full-bleed',
    imgId: 55,
    plate: 'PLATE 05',
    title: 'Deep in the Woods',
    subtitle: '密林深处的呼吸',
    meta: 'FOREST CAMP'
  },
  {
    pageNumber: 8,
    layout: 'museum-mat',
    imgId: 5,
    plate: 'PLATE 06',
    title: 'Wilderness Camp',
    subtitle: '旷野驻车点',
    note: '选一处背风向阳的河谷停下，把车窗变成装满风景的取景框。',
    meta: '6000 × 4000 · 3:2'
  },
  {
    pageNumber: 9,
    layout: 'portrait-frame',
    imgId: 66,
    plate: 'PLATE 07',
    title: 'Morning Breeze',
    subtitle: '林间清风',
    meta: '3:4 PORTRAIT'
  },
  {
    pageNumber: 10,
    layout: 'polaroid-note',
    imgId: 7,
    plate: 'PLATE 08',
    title: 'Slow Down the Clock',
    subtitle: '拨慢时间的指针',
    note: 'No schedules, no rush — just the rhythm of sunlight moving across the dashboard.',
    meta: 'CH.01 · ROAD LOG'
  },
  {
    pageNumber: 11,
    layout: 'duo-stack',
    imgId: 8,
    secondaryImgId: 9,
    plate: 'PLATE 09',
    title: 'Across the Valley',
    subtitle: '穿越河谷与山脊',
    meta: 'DUAL FRAMES'
  },
  {
    pageNumber: 12,
    layout: 'museum-mat',
    imgId: 10,
    plate: 'PLATE 10',
    title: 'Golden Larch',
    subtitle: '秋日落叶松林',
    note: '金黄的松针铺满车顶与地面，每一次踩踏都发出松软的沙沙声。',
    meta: '4240 × 2832 · 3:2'
  },
  {
    pageNumber: 13,
    layout: 'full-bleed',
    imgId: 69,
    plate: 'PLATE 11',
    title: 'Tall Timber',
    subtitle: '巨木之下',
    meta: 'DEEP FOREST'
  },
  {
    pageNumber: 14,
    layout: 'polaroid-note',
    imgId: 11,
    plate: 'PLATE 12',
    title: 'A Cup of Warmth',
    subtitle: '荒野手冲时刻',
    note: 'Boiling water on a portable stove while watching clouds drift over the ridge.',
    meta: 'CH.02 · CAMP VIBE'
  },
  {
    pageNumber: 15,
    layout: 'duo-stack',
    imgId: 12,
    secondaryImgId: 13,
    plate: 'PLATE 13',
    title: 'Camp Setup',
    subtitle: '天幕与移动小家',
    meta: 'STORYBOARD'
  },
  {
    pageNumber: 16,
    layout: 'museum-mat',
    imgId: 14,
    plate: 'PLATE 14',
    title: 'Cozy Interior',
    subtitle: '车内一方天地',
    note: '原木饰板、暖黄灯光与折叠桌椅，构成了旅途中最安心的避风港。',
    meta: 'VAN INTERIOR'
  },
  {
    pageNumber: 17,
    layout: 'portrait-frame',
    imgId: 70,
    plate: 'PLATE 15',
    title: 'Quietude',
    subtitle: '静谧时刻',
    meta: '2:3 PORTRAIT'
  },
  {
    pageNumber: 18,
    layout: 'polaroid-note',
    imgId: 16,
    plate: 'PLATE 16',
    title: 'Afternoon Nap',
    subtitle: '午后小憩',
    note: 'Doors wide open to the meadow breeze, listening to birdsong from the back seat.',
    meta: 'SLOW LIVING'
  },
  {
    pageNumber: 19,
    layout: 'duo-stack',
    imgId: 17,
    secondaryImgId: 18,
    plate: 'PLATE 17',
    title: 'Details of Living',
    subtitle: '器物与生活质感',
    meta: 'CAMP GEAR'
  },
  {
    pageNumber: 20,
    layout: 'museum-mat',
    imgId: 19,
    plate: 'PLATE 18',
    title: 'Shadows & Light',
    subtitle: '斑驳光影',
    note: '午后三点的阳光斜穿过树梢，在车身侧面投下流动的画作。',
    meta: '4240 × 2832 · 3:2'
  },
  {
    pageNumber: 21,
    layout: 'full-bleed',
    imgId: 71,
    plate: 'PLATE 19',
    title: 'Path Unseen',
    subtitle: '未名小径',
    meta: 'EXPLORATION'
  },
  {
    pageNumber: 22,
    layout: 'museum-mat',
    imgId: 21,
    plate: 'PLATE 20',
    title: 'Mobile Sanctuary',
    subtitle: '移动的舒适栖居',
    note: '推开车门即是山川湖海，关上车门便是温暖治愈的一方小世界。',
    meta: 'TRANSIT · LITTLE GRAY'
  },
  {
    pageNumber: 23,
    layout: 'polaroid-note',
    imgId: 20,
    plate: 'PLATE 21',
    title: ' wandering Companion',
    subtitle: '旅途中的伙伴',
    note: 'Every mile feels lighter when shared with curious eyes and a wagging tail.',
    meta: 'COMPANION'
  },
  {
    pageNumber: 24,
    layout: 'duo-stack',
    imgId: 22,
    secondaryImgId: 23,
    plate: 'PLATE 22',
    title: 'Forest Edge',
    subtitle: '林缘营地纪实',
    meta: 'DUAL FRAMES'
  },
  {
    pageNumber: 25,
    layout: 'portrait-frame',
    imgId: 72,
    plate: 'PLATE 23',
    title: 'Golden Hour',
    subtitle: '落日熔金',
    meta: 'DUSK · 17:45'
  },
  {
    pageNumber: 26,
    layout: 'museum-mat',
    imgId: 24,
    plate: 'PLATE 24',
    title: 'Dust & Distance',
    subtitle: '尘土与远方',
    note: '走过碎石路与泥泞河滩，车身上的泥点是荒野颁发的勋章。',
    meta: 'OFF-ROAD'
  },
  {
    pageNumber: 27,
    layout: 'polaroid-note',
    imgId: 25,
    plate: 'PLATE 25',
    title: 'Pine Needle Carpet',
    subtitle: '松针地毯',
    note: 'Parked beneath century-old pines where the air smells of resin and autumn rain.',
    meta: 'WOODLAND'
  },
  {
    pageNumber: 28,
    layout: 'duo-stack',
    imgId: 26,
    secondaryImgId: 27,
    plate: 'PLATE 26',
    title: 'In Motion',
    subtitle: '在途掠影',
    meta: 'CH.03 · AUTOMOTIVE'
  },
  {
    pageNumber: 29,
    layout: 'full-bleed',
    imgId: 73,
    plate: 'PLATE 27',
    title: 'Valley Shadows',
    subtitle: '山谷暮色',
    meta: '4000 × 6000'
  },
  {
    pageNumber: 30,
    layout: 'museum-mat',
    imgId: 30,
    plate: 'PLATE 28',
    title: 'Morning Mist & Coffee',
    subtitle: '松林晨雾与热咖啡',
    note: '水汽在冷空气中升腾，与远处的山雾融为一体。',
    meta: 'MORNING RITUAL'
  },
  {
    pageNumber: 31,
    layout: 'polaroid-note',
    imgId: 28,
    plate: 'PLATE 29',
    title: 'Little Yellow',
    subtitle: '北斗星 · 小黄的姿态',
    note: 'Boxy silhouette, classic lines, and a bold yellow coat that stands out in any landscape.',
    meta: 'SUZUKI WAGON R'
  },
  {
    pageNumber: 32,
    layout: 'duo-stack',
    imgId: 29,
    secondaryImgId: 31,
    plate: 'PLATE 30',
    title: 'Classic Stance',
    subtitle: '复古方盒子美学',
    meta: 'STANCE & STYLE'
  },
  {
    pageNumber: 33,
    layout: 'portrait-frame',
    imgId: 81,
    plate: 'PLATE 31',
    title: 'Into the Wind',
    subtitle: '迎风而行',
    meta: '3:4 PORTRAIT'
  },
  {
    pageNumber: 34,
    layout: 'museum-mat',
    imgId: 35,
    plate: 'PLATE 32',
    title: 'Stance & Soul',
    subtitle: '姿态与灵魂',
    note: '老车不死，每一道车辙都记录着走过的山路与旷野。',
    meta: 'AUTOMOTIVE ARCHIVE'
  },
  {
    pageNumber: 35,
    layout: 'polaroid-note',
    imgId: 32,
    plate: 'PLATE 33',
    title: 'Tailgate View',
    subtitle: '尾门外的风景',
    note: 'Sitting on the tailgate watching the sky turn from amber to deep indigo.',
    meta: 'EVENING VIBE'
  },
  {
    pageNumber: 36,
    layout: 'duo-stack',
    imgId: 33,
    secondaryImgId: 34,
    plate: 'PLATE 34',
    title: 'Two Ways of Living',
    subtitle: '小黄与小灰的对话',
    meta: 'FLEET ARCHIVE'
  },
  {
    pageNumber: 37,
    layout: 'full-bleed',
    imgId: 82,
    plate: 'PLATE 35',
    title: 'Silent Peaks',
    subtitle: '静默群峰',
    meta: 'HIGHLAND'
  },
  {
    pageNumber: 38,
    layout: 'museum-mat',
    imgId: 36,
    plate: 'PLATE 36',
    title: 'Campfire Stories',
    subtitle: '晚风与营地灯火',
    note: '当夜幕四合，暖黄的营地灯亮起，荒野便有了家的温度。',
    meta: 'CH.04 · NIGHTFALL'
  },
  {
    pageNumber: 39,
    layout: 'polaroid-note',
    imgId: 37,
    plate: 'PLATE 37',
    title: 'Under the Awning',
    subtitle: '侧帐下的慢时光',
    note: 'Raindrops tapping gently on the canvas awning while dinner simmers inside.',
    meta: 'SHELTER'
  },
  {
    pageNumber: 40,
    layout: 'duo-stack',
    imgId: 38,
    secondaryImgId: 39,
    plate: 'PLATE 38',
    title: 'Autumn Palette',
    subtitle: '层林尽染的季节',
    meta: 'SEASONAL LOG'
  },
  {
    pageNumber: 41,
    layout: 'portrait-frame',
    imgId: 85,
    plate: 'PLATE 39',
    title: 'Under the Canopy',
    subtitle: '林冠之下',
    meta: '3:4 MEDIUM'
  },
  {
    pageNumber: 42,
    layout: 'museum-mat',
    imgId: 40,
    plate: 'PLATE 40',
    title: 'Reflections',
    subtitle: '湖畔倒影',
    note: '风停的瞬间，整片山林与云影都安静地落入水面。',
    meta: 'WATERSIDE'
  },
  {
    pageNumber: 43,
    layout: 'polaroid-note',
    imgId: 41,
    plate: 'PLATE 41',
    title: 'Unpaved Freedom',
    subtitle: '非铺装路面的自由',
    note: 'Taking the detour that is not on the map often leads to the most unforgettable camp.',
    meta: 'OFF THE GRID'
  },
  {
    pageNumber: 44,
    layout: 'duo-stack',
    imgId: 42,
    secondaryImgId: 43,
    plate: 'PLATE 42',
    title: 'Dusk to Dawn',
    subtitle: '从暮色到破晓',
    meta: 'TIME LAPSE'
  },
  {
    pageNumber: 45,
    layout: 'full-bleed',
    imgId: 92,
    plate: 'PLATE 43',
    title: 'Horizon Line',
    subtitle: '天际线之外',
    meta: 'EXPEDITION'
  },
  {
    pageNumber: 46,
    layout: 'museum-mat',
    imgId: 44,
    plate: 'PLATE 44',
    title: 'Solitude & Freedom',
    subtitle: '旷野独行',
    note: '在辽阔的天地之间，独处不是孤独，而是与自我的重逢。',
    meta: '4240 × 2832 · 3:2'
  },
  {
    pageNumber: 47,
    layout: 'polaroid-note',
    imgId: 45,
    plate: 'PLATE 45',
    title: 'Cinematic Panorama',
    subtitle: '宽银幕视野',
    note: 'Wide open spaces where the sky takes up three quarters of your vision.',
    meta: '21:9 CINEMA'
  },
  {
    pageNumber: 48,
    layout: 'duo-stack',
    imgId: 46,
    secondaryImgId: 47,
    plate: 'PLATE 46',
    title: 'Wilderness Textures',
    subtitle: '岩石、草甸与车辙',
    meta: 'FIELD STUDY'
  },
  {
    pageNumber: 49,
    layout: 'portrait-frame',
    imgId: 94,
    plate: 'PLATE 47',
    title: 'Starlight Wait',
    subtitle: '静候星河',
    meta: 'HIGH RES · 3:4'
  },
  {
    pageNumber: 50,
    layout: 'museum-mat',
    imgId: 76,
    plate: 'PLATE 48',
    title: 'Miles of Memories',
    subtitle: '里程表上的记忆',
    note: '每一段旅程的终点，都会成为下一段故事的起点。',
    meta: 'VANLIFE JOURNAL'
  },
  {
    pageNumber: 51,
    layout: 'duo-stack',
    imgId: 60,
    secondaryImgId: 61,
    plate: 'PLATE 49',
    title: 'Wide Country',
    subtitle: '广袤原野双帧',
    meta: '16:9 SERIES'
  },
  {
    pageNumber: 52,
    layout: 'polaroid-note',
    imgId: 80,
    plate: 'PLATE 50',
    title: 'Simple Pleasures',
    subtitle: '极简的快乐',
    note: 'Less clutter, more sky. Everything we truly need fits inside four wheels.',
    meta: 'MINIMALISM'
  },
  {
    pageNumber: 53,
    layout: 'duo-stack',
    imgId: 87,
    secondaryImgId: 88,
    plate: 'PLATE 51',
    title: 'Last Light',
    subtitle: '余晖漫染',
    meta: 'GOLDEN SERIES'
  },
  {
    pageNumber: 54,
    layout: 'museum-mat',
    imgId: 15,
    plate: 'PLATE 52',
    title: 'Until the Next Road',
    subtitle: '下一段山海再会',
    note: '旅途从不真正结束，它只是在等待下一次转动钥匙的瞬间。',
    meta: 'EPILOGUE'
  },
  {
    pageNumber: 55,
    layout: 'full-bleed',
    imgId: 95,
    plate: 'PLATE 53',
    title: 'Endless Horizon',
    subtitle: '未完待续的公路',
    meta: 'FINALE'
  }
]

const coverImgUrl = getImgUrl(1)

// 供目录抽屉使用的 56 页元数据
const pagesMeta = computed<BookPageMeta[]>(() => {
  const list: BookPageMeta[] = [
    { pageNumber: 1, title: '封面 · Cover', thumb: coverImgUrl },
    { pageNumber: 2, title: '序言 · Preface' }
  ]
  innerPages.forEach((p) => {
    list.push({
      pageNumber: p.pageNumber,
      title: p.subtitle,
      thumb: getImgUrl(p.imgId)
    })
  })
  list.push({ pageNumber: 56, title: '封底 · Fin' })
  return list
})

// 点击书页内放大按钮可检视高清原图
interface PreviewState {
  imgId: number
  plate: string
  title: string
  subtitle: string
  note?: string
}

const previewItem = ref<PreviewState | null>(null)
const isPreviewLoading = ref(false)
let previewSeq = 0
let previewStart = 0

const openPreview = (item: InnerPageConfig, customImgId?: number) => {
  previewSeq++
  previewStart = performance.now()
  isPreviewLoading.value = true
  previewItem.value = {
    imgId: customImgId ?? item.imgId,
    plate: item.plate,
    title: item.title,
    subtitle: item.subtitle,
    note: item.note
  }
}

const closePreview = () => {
  previewItem.value = null
  isPreviewLoading.value = false
}

const onPreviewLoaded = (seq: number) => {
  if (seq !== previewSeq) return
  const elapsed = performance.now() - previewStart
  const remain = Math.max(0, 300 - elapsed)
  setTimeout(() => {
    if (seq === previewSeq) {
      isPreviewLoading.value = false
    }
  }, remain)
}

const formatPageNum = (n: number) => (n < 10 ? `0${n}` : `${n}`)
</script>

<template>
  <main class="photobook-page">
    <FlipBook ref="flipBookRef" :pages-meta="pagesMeta">
      <!-- ==================== P.01 纯白精装封面 ==================== -->
      <div class="page is-cover-page cover-front">
        <div class="cover-inner">
          <div class="cover-top">
            <span class="cover-issue">ISSUE NO. 01 · ARCHIVE EDITION</span>
            <span class="cover-year">56 PAGES</span>
          </div>

          <div class="cover-frame">
            <img :src="coverImgUrl" alt="Cover Photography" />
          </div>

          <div class="cover-titles">
            <h1 class="cover-main-title">WILD HUANG</h1>
            <p class="cover-sub-title">LIFE IN THE WILD · 荒野在途影像志</p>
            <div class="cover-rule"></div>
            <p class="cover-caption">A Visual Monograph of Vanlife, Automobiles & Slow Living</p>
          </div>
        </div>
      </div>

      <!-- ==================== P.02 扉页序言 ==================== -->
      <div class="page page-preface">
        <div class="preface-inner">
          <div class="preface-header">
            <span class="preface-eyebrow">FOREWORD · 扉页手记</span>
            <h2 class="preface-heading">On the Road,<br />In the Wild.</h2>
          </div>

          <div class="preface-body">
            <p class="preface-cn">
              喜欢自然原生态生活，探索更多的生活方式。以真实的 Vanlife 体验者视角，在车轮与帐篷之间，讲述一个普通汽车旅行者的旷野故事。
            </p>
            <p class="preface-en">
              Passionate about a back-to-nature lifestyle and cozy vehicle-based living spaces. Every frame in this 56-page monograph was captured along unpaved trails, quiet valleys, and campfires.
            </p>

            <div class="fleet-box">
              <div class="fleet-item">
                <span class="fleet-tag">VEHICLE 01</span>
                <strong>北斗星 · 小黄</strong>
                <span>SUZUKI Wagon R · Tuned for Stance</span>
              </div>
              <div class="fleet-item">
                <span class="fleet-tag">VEHICLE 02</span>
                <strong>全顺 · 小灰</strong>
                <span>FORD Transit · Built for Coziness</span>
              </div>
            </div>
          </div>

          <div class="preface-footer">
            <span>PAGE 02</span>
            <span>WILD HUANG STUDIO</span>
          </div>
        </div>
      </div>

      <!-- ==================== P.03 ~ P.55 多版式摄影内页 ==================== -->
      <div
        v-for="item in innerPages"
        :key="item.pageNumber"
        class="page inner-art-page"
        :class="`layout-${item.layout}`"
      >
        <!-- 版式 1：画廊大图版式 (full-bleed) -->
        <template v-if="item.layout === 'full-bleed'">
          <div class="bleed-wrap">
            <div class="bleed-img-stage">
              <img :src="getImgUrl(item.imgId)" :alt="item.title" loading="lazy" />
              <button
                class="inspect-btn"
                @click.stop="openPreview(item)"
                title="点击检视高清大图"
              >
                ⤢
              </button>
            </div>
            <div class="bleed-caption-bar">
              <div class="bleed-caption-left">
                <span class="bleed-plate">{{ item.plate }} · {{ item.meta }}</span>
                <h3 class="bleed-title">{{ item.title }} · {{ item.subtitle }}</h3>
              </div>
              <span class="bleed-pnum">{{ formatPageNum(item.pageNumber) }}</span>
            </div>
          </div>
        </template>

        <!-- 版式 2：美术馆留白装裱 (museum-mat) -->
        <template v-else-if="item.layout === 'museum-mat'">
          <div class="mat-wrap">
            <div class="mat-top-meta">
              <span>{{ item.plate }}</span>
              <span>{{ item.meta }}</span>
            </div>

            <div class="mat-photo-box">
              <img :src="getImgUrl(item.imgId)" :alt="item.title" loading="lazy" />
              <button
                class="inspect-btn"
                @click.stop="openPreview(item)"
                title="点击检视高清大图"
              >
                ⤢
              </button>
            </div>

            <div class="mat-text-box">
              <h3 class="mat-title">{{ item.title }}</h3>
              <p class="mat-subtitle">{{ item.subtitle }}</p>
              <p v-if="item.note" class="mat-note">{{ item.note }}</p>
            </div>

            <div class="page-foot-line">
              <span>WILD HUANG ARCHIVE</span>
              <span>{{ formatPageNum(item.pageNumber) }}</span>
            </div>
          </div>
        </template>

        <!-- 版式 3：手札随笔版式 (polaroid-note) -->
        <template v-else-if="item.layout === 'polaroid-note'">
          <div class="polaroid-wrap">
            <div class="polaroid-card">
              <div class="polaroid-img-holder">
                <img :src="getImgUrl(item.imgId)" :alt="item.title" loading="lazy" />
                <button
                  class="inspect-btn"
                  @click.stop="openPreview(item)"
                  title="点击检视高清大图"
                >
                  ⤢
                </button>
              </div>
              <div class="polaroid-stamp">
                <span>{{ item.plate }}</span>
                <span>{{ item.meta }}</span>
              </div>
            </div>

            <div class="journal-quote">
              <span class="quote-mark">“</span>
              <p class="quote-en">{{ item.note }}</p>
              <div class="quote-author">— {{ item.title }} ({{ item.subtitle }})</div>
            </div>

            <div class="page-foot-line">
              <span>FIELD NOTES</span>
              <span>{{ formatPageNum(item.pageNumber) }}</span>
            </div>
          </div>
        </template>

        <!-- 版式 4：电影分镜上下双图版式 (duo-stack) -->
        <template v-else-if="item.layout === 'duo-stack'">
          <div class="duo-wrap">
            <div class="mat-top-meta">
              <span>{{ item.plate }} · {{ item.meta }}</span>
              <span>{{ item.title }}</span>
            </div>

            <div class="duo-frames">
              <div class="duo-frame-item">
                <img :src="getImgUrl(item.imgId)" :alt="item.title" loading="lazy" />
                <span class="scene-tag">SCENE A</span>
                <button
                  class="inspect-btn"
                  @click.stop="openPreview(item, item.imgId)"
                  title="点击检视高清大图"
                >
                  ⤢
                </button>
              </div>

              <div v-if="item.secondaryImgId" class="duo-frame-item">
                <img :src="getImgUrl(item.secondaryImgId)" :alt="item.subtitle" loading="lazy" />
                <span class="scene-tag">SCENE B</span>
                <button
                  class="inspect-btn"
                  @click.stop="openPreview(item, item.secondaryImgId)"
                  title="点击检视高清大图"
                >
                  ⤢
                </button>
              </div>
            </div>

            <div class="page-foot-line">
              <span>{{ item.subtitle }}</span>
              <span>{{ formatPageNum(item.pageNumber) }}</span>
            </div>
          </div>
        </template>

        <!-- 版式 5：竖幅特写画框 (portrait-frame) -->
        <template v-else>
          <div class="portrait-wrap">
            <div class="portrait-side-label">
              <span>{{ item.plate }} · {{ item.meta }}</span>
            </div>

            <div class="portrait-main">
              <div class="portrait-img-box">
                <img :src="getImgUrl(item.imgId)" :alt="item.title" loading="lazy" />
                <button
                  class="inspect-btn"
                  @click.stop="openPreview(item)"
                  title="点击检视高清大图"
                >
                  ⤢
                </button>
              </div>
              <div class="portrait-caption">
                <div>
                  <h4>{{ item.title }}</h4>
                  <p>{{ item.subtitle }}</p>
                </div>
                <span class="portrait-pnum">{{ formatPageNum(item.pageNumber) }}</span>
              </div>
            </div>
          </div>
        </template>
      </div>

      <!-- ==================== P.56 纯白精装封底 ==================== -->
      <div class="page is-cover-page cover-back">
        <div class="back-inner">
          <div class="back-top">
            <span class="back-badge">FIN · 终章 (P.56)</span>
          </div>

          <div class="back-center">
            <p class="back-quote">“The road is life.”</p>
            <p class="back-quote-cn">山海自有归期，风雨自有相逢。</p>
            <button
              class="replay-book-btn"
              @click.stop="flipBookRef?.goToPage(1)"
            >
              ↻ 重新翻阅 · Back to Cover
            </button>
          </div>

          <div class="back-colophon">
            <p>PHOTOGRAPHED & CURATED BY WILD HUANG</p>
            <p>56 PAGES · 68 ORIGINAL FRAMES · ALL RIGHTS RESERVED</p>
          </div>
        </div>
      </div>
    </FlipBook>

    <!-- 画册单张高清大图检视弹窗（带手绘露营车 Loading） -->
    <Teleport to="body">
      <Transition name="fade">
        <div v-if="previewItem" class="book-preview-modal" @click="closePreview">
          <button class="preview-close-btn" @click="closePreview" title="关闭">×</button>

          <Transition name="fade">
            <div v-if="isPreviewLoading" class="preview-loader-box" @click.stop>
              <VanLoader
                theme="dark"
                compact
                :text="`DEVELOPING ${previewItem.plate}...`"
              />
            </div>
          </Transition>

          <div class="preview-content" @click.stop>
            <img
              :src="getImgUrl(previewItem.imgId)"
              :alt="previewItem.title"
              class="preview-full-img"
              :class="{ 'is-ready': !isPreviewLoading }"
              @load="onPreviewLoaded(previewSeq)"
              @error="onPreviewLoaded(previewSeq)"
            />
            <div v-if="!isPreviewLoading" class="preview-caption">
              <span class="preview-plate">{{ previewItem.plate }}</span>
              <strong>{{ previewItem.title }} · {{ previewItem.subtitle }}</strong>
              <span v-if="previewItem.note" class="preview-note">{{ previewItem.note }}</span>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>
  </main>
</template>

<style scoped>
.photobook-page {
  width: 100%;
  min-height: 100vh;
}

/* ==================== P.01 纯白精装封面设计 ==================== */
.cover-front {
  background: #ffffff !important;
  color: #181715 !important;
}

.cover-inner {
  position: absolute;
  inset: 14px;
  padding: 6% 8%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  border: 1px solid rgba(0, 0, 0, 0.08);
  background: linear-gradient(135deg, #ffffff 0%, #faf8f4 100%);
  overflow: hidden;
  box-sizing: border-box;
}

.cover-top {
  flex-shrink: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 2px;
  color: #88847c;
}

.cover-frame {
  position: relative;
  flex: 1;
  min-height: 0;
  width: 78%;
  margin: 14px auto;
  background: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.08);
  box-shadow: 0 12px 28px rgba(0, 0, 0, 0.09);
  overflow: hidden;
}

.cover-frame img {
  position: absolute;
  inset: 8px;
  width: calc(100% - 16px);
  height: calc(100% - 16px);
  object-fit: cover;
  display: block;
}

.cover-titles {
  flex-shrink: 0;
  text-align: center;
}

.cover-main-title {
  font-size: 1.5rem;
  font-weight: 500;
  letter-spacing: 5px;
  margin: 0 0 4px;
  color: #141413;
}

.cover-sub-title {
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 2.5px;
  color: #b85d38;
  margin: 0;
}

.cover-rule {
  width: 32px;
  height: 1px;
  background: rgba(0, 0, 0, 0.15);
  margin: 8px auto;
}

.cover-caption {
  font-size: 9px;
  letter-spacing: 0.8px;
  color: #8a867e;
  margin: 0;
}

/* ==================== P.02 扉页序言 ==================== */
.page-preface {
  background: #f9f7f2 !important;
}

.preface-inner {
  position: absolute;
  inset: 0;
  padding: 10% 11%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  overflow: hidden;
  box-sizing: border-box;
}

.preface-eyebrow {
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 2.2px;
  color: #b85d38;
  display: block;
  margin-bottom: 8px;
}

.preface-heading {
  font-size: 1.5rem;
  font-weight: 300;
  line-height: 1.25;
  color: #191816;
  margin: 0;
}

.preface-body {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.preface-cn {
  font-size: 11.5px;
  line-height: 1.8;
  color: #3a3834;
  margin: 0;
}

.preface-en {
  font-size: 10.5px;
  line-height: 1.65;
  color: #77736b;
  font-style: italic;
  margin: 0;
}

.fleet-box {
  margin-top: 6px;
  padding-top: 12px;
  border-top: 1px solid rgba(0, 0, 0, 0.08);
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.fleet-item {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.fleet-tag {
  font-size: 8px;
  letter-spacing: 1.5px;
  color: #999;
}

.fleet-item strong {
  font-size: 11px;
  color: #1c1b18;
}

.fleet-item span:last-child {
  font-size: 9px;
  color: #777;
}

.preface-footer,
.page-foot-line {
  flex-shrink: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 9px;
  letter-spacing: 1.8px;
  color: #99958c;
  border-top: 1px solid rgba(0, 0, 0, 0.06);
  padding-top: 8px;
}

/* ==================== 内页通用放大按钮 ==================== */
.inspect-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 26px;
  height: 26px;
  border-radius: 50%;
  border: none;
  background: rgba(255, 255, 255, 0.9);
  color: #111;
  font-size: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  opacity: 0;
  transform: scale(0.9);
  transition: all 0.22s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.18);
  z-index: 5;
}

.inner-art-page:hover .inspect-btn {
  opacity: 1;
  transform: scale(1);
}

.inspect-btn:hover {
  background: #111;
  color: #fff;
}

/* ==================== 版式 1：画廊大图版式 (full-bleed) ==================== */
.bleed-wrap {
  position: absolute;
  inset: 0;
  padding: 5% 6% 4.5%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: #fbfaf7;
  overflow: hidden;
  box-sizing: border-box;
}

.bleed-img-stage {
  position: relative;
  flex: 1;
  min-height: 0;
  width: 100%;
  overflow: hidden;
  border-radius: 2px;
  background: #f0eee9;
}

.bleed-img-stage img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.bleed-caption-bar {
  flex-shrink: 0;
  padding-top: 10px;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

.bleed-plate {
  display: block;
  font-size: 8.5px;
  letter-spacing: 1.8px;
  color: #b85d38;
  margin-bottom: 2px;
}

.bleed-title {
  font-size: 11.5px;
  font-weight: 500;
  letter-spacing: 0.6px;
  color: #1c1b18;
  margin: 0;
}

.bleed-pnum {
  font-size: 9.5px;
  letter-spacing: 1.5px;
  color: #99958c;
}

/* ==================== 版式 2：美术馆装裱 (museum-mat) ==================== */
.mat-wrap {
  position: absolute;
  inset: 0;
  padding: 8% 9%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: #fbfaf7;
  overflow: hidden;
  box-sizing: border-box;
}

.mat-top-meta {
  flex-shrink: 0;
  display: flex;
  justify-content: space-between;
  font-size: 8.5px;
  letter-spacing: 1.8px;
  color: #99958c;
}

.mat-photo-box {
  position: relative;
  width: 100%;
  height: 48%;
  flex-shrink: 0;
  background: #ffffff;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.06);
  overflow: hidden;
}

.mat-photo-box img {
  position: absolute;
  inset: 8px;
  width: calc(100% - 16px);
  height: calc(100% - 16px);
  object-fit: cover;
  display: block;
}

.mat-text-box {
  flex-shrink: 0;
  text-align: center;
  padding: 0 4%;
}

.mat-title {
  font-size: 13.5px;
  font-weight: 500;
  letter-spacing: 1px;
  color: #1c1b18;
  margin: 0 0 2px;
}

.mat-subtitle {
  font-size: 10px;
  color: #b85d38;
  letter-spacing: 1.2px;
  margin: 0 0 6px;
}

.mat-note {
  font-size: 10px;
  line-height: 1.6;
  color: #66625a;
  margin: 0;
}

/* ==================== 版式 3：手札随笔 (polaroid-note) ==================== */
.polaroid-wrap {
  position: absolute;
  inset: 0;
  padding: 8% 9%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: #f7f4ed;
  overflow: hidden;
  box-sizing: border-box;
}

.polaroid-card {
  height: 55%;
  flex-shrink: 0;
  background: #ffffff;
  padding: 10px 10px 12px;
  box-shadow: 0 10px 26px rgba(0, 0, 0, 0.07);
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
}

.polaroid-img-holder {
  position: relative;
  flex: 1;
  min-height: 0;
  width: 100%;
  overflow: hidden;
  margin-bottom: 8px;
}

.polaroid-img-holder img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.polaroid-stamp {
  flex-shrink: 0;
  display: flex;
  justify-content: space-between;
  font-size: 8.5px;
  letter-spacing: 1.5px;
  color: #8c877e;
}

.journal-quote {
  flex-shrink: 0;
  position: relative;
  padding: 2px 4px;
}

.quote-mark {
  font-family: Georgia, serif;
  font-size: 26px;
  line-height: 1;
  color: #d39b78;
  display: block;
  margin-bottom: -6px;
}

.quote-en {
  font-size: 10.5px;
  line-height: 1.65;
  color: #3d3b37;
  font-style: italic;
  margin: 0 0 6px;
}

.quote-author {
  font-size: 9px;
  letter-spacing: 1px;
  color: #888379;
}

/* ==================== 版式 4：电影分镜上下双图 (duo-stack) ==================== */
.duo-wrap {
  position: absolute;
  inset: 0;
  padding: 7% 8.5%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: #fbfaf7;
  overflow: hidden;
  box-sizing: border-box;
}

.duo-frames {
  flex: 1;
  min-height: 0;
  margin: 10px 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.duo-frame-item {
  position: relative;
  flex: 1;
  min-height: 0;
  width: 100%;
  background: #ffffff;
  padding: 6px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.06);
  overflow: hidden;
  box-sizing: border-box;
}

.duo-frame-item img {
  position: absolute;
  inset: 6px;
  width: calc(100% - 12px);
  height: calc(100% - 12px);
  object-fit: cover;
  display: block;
}

.scene-tag {
  position: absolute;
  left: 14px;
  bottom: 12px;
  font-size: 8px;
  font-weight: 600;
  letter-spacing: 1.6px;
  color: #fff;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(4px);
  padding: 2px 7px;
  border-radius: 3px;
  pointer-events: none;
}

/* ==================== 版式 5：竖幅画框 (portrait-frame) ==================== */
.portrait-wrap {
  position: absolute;
  inset: 0;
  padding: 8% 9%;
  display: flex;
  gap: 12px;
  background: #fbfaf7;
  overflow: hidden;
  box-sizing: border-box;
}

.portrait-side-label {
  flex-shrink: 0;
  writing-mode: vertical-rl;
  font-size: 8.5px;
  letter-spacing: 2.2px;
  color: #a39e93;
  display: flex;
  align-items: center;
  border-right: 1px solid rgba(0, 0, 0, 0.06);
  padding-right: 8px;
}

.portrait-main {
  flex: 1;
  min-width: 0;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.portrait-img-box {
  position: relative;
  width: 100%;
  flex: 1;
  min-height: 0;
  overflow: hidden;
  border-radius: 2px;
  margin-bottom: 10px;
}

.portrait-img-box img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.portrait-caption {
  flex-shrink: 0;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

.portrait-caption h4 {
  font-size: 12px;
  font-weight: 500;
  margin: 0 0 2px;
  color: #1c1b18;
}

.portrait-caption p {
  font-size: 10px;
  color: #77736b;
  margin: 0;
}

.portrait-pnum {
  font-size: 9.5px;
  letter-spacing: 1.5px;
  color: #99958c;
}

/* ==================== P.56 纯白精装封底设计 ==================== */
.cover-back {
  background: #ffffff !important;
  color: #181715 !important;
}

.back-inner {
  position: absolute;
  inset: 14px;
  padding: 12% 10%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  text-align: center;
  border: 1px solid rgba(0, 0, 0, 0.08);
  background: linear-gradient(135deg, #ffffff 0%, #faf8f4 100%);
  overflow: hidden;
  box-sizing: border-box;
}

.back-badge {
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 2.5px;
  color: #b85d38;
}

.back-quote {
  font-family: Georgia, serif;
  font-size: 1.35rem;
  font-style: italic;
  margin: 0 0 6px;
  color: #181715;
}

.back-quote-cn {
  font-size: 11px;
  letter-spacing: 2px;
  color: #6e6a63;
  margin: 0 0 24px;
}

.replay-book-btn {
  padding: 8px 18px;
  border-radius: 999px;
  border: 1px solid rgba(0, 0, 0, 0.18);
  background: #ffffff;
  color: #181715;
  font-size: 11px;
  letter-spacing: 1px;
  cursor: pointer;
  transition: all 0.25s ease;
}

.replay-book-btn:hover {
  background: #181715;
  color: #ffffff;
  border-color: #181715;
}

.back-colophon p {
  font-size: 8.5px;
  letter-spacing: 2px;
  color: #99958c;
  margin: 3px 0;
}

/* ==================== 高清大图检视弹窗 ==================== */
.book-preview-modal {
  position: fixed;
  inset: 0;
  z-index: 9999;
  background: rgba(8, 8, 10, 0.94);
  backdrop-filter: blur(20px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 32px;
}

.preview-close-btn {
  position: absolute;
  top: 20px;
  right: 24px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.08);
  color: #fff;
  font-size: 22px;
  cursor: pointer;
  z-index: 10;
}

.preview-loader-box {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.preview-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  max-width: 90vw;
  max-height: 90vh;
}

.preview-full-img {
  max-width: 88vw;
  max-height: 78vh;
  object-fit: contain;
  border-radius: 4px;
  box-shadow: 0 24px 60px rgba(0, 0, 0, 0.65);
  opacity: 0;
  transform: scale(0.97);
  transition: all 0.3s ease;
}

.preview-full-img.is-ready {
  opacity: 1;
  transform: scale(1);
}

.preview-caption {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  color: #fff;
  text-align: center;
}

.preview-plate {
  font-size: 10px;
  letter-spacing: 2px;
  color: #d39b78;
}

.preview-caption strong {
  font-size: 14px;
  font-weight: 500;
}

.preview-note {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.65);
  max-width: 520px;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
