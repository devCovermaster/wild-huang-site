<script setup lang="ts">
import { useRoute } from 'vue-router'
import { ref, onMounted, onUnmounted, computed, watch } from 'vue'

const route = useRoute()
const isHome = computed(() => route.path === '/')
const isPhotos = computed(() => route.path === '/photos')
const isPhotobook = computed(() => route.path === '/photobook')

const bannerRef = ref<HTMLImageElement | null>(null)
const isScrollActive = ref(false)
const isLightMerged = ref(false)
const isMobileNavOpen = ref(false)

const updateHeaderState = () => {
  const y = window.scrollY
  isScrollActive.value = y > 50

  if (isPhotos.value) {
    const headerHeight = window.innerWidth <= 768 ? 50 : 58
    const bannerHeight = bannerRef.value?.offsetHeight || 160
    isLightMerged.value = y >= Math.max(40, bannerHeight - headerHeight)
  } else {
    isLightMerged.value = false
  }
}

const toggleMobileMenu = () => {
  isMobileNavOpen.value = !isMobileNavOpen.value
}

const closeMobileMenu = () => {
  isMobileNavOpen.value = false
}

watch(
  () => route.path,
  () => {
    closeMobileMenu()
    updateHeaderState()
  }
)

onMounted(() => {
  updateHeaderState()
  window.addEventListener('scroll', updateHeaderState, { passive: true })
  window.addEventListener('resize', updateHeaderState)
})

onUnmounted(() => {
  window.removeEventListener('scroll', updateHeaderState)
  window.removeEventListener('resize', updateHeaderState)
})
</script>

<template>
  <template v-if="!isPhotobook">
    <header
      :class="{
        'scroll-active': isScrollActive && !isLightMerged,
        'header-light-merged': isLightMerged
      }"
    >
      <router-link class="brand" to="/" @click="closeMobileMenu">Wild Huang</router-link>
      <nav :class="{ active: isMobileNavOpen }">
        <a href="#projects" v-if="isHome" @click="closeMobileMenu">Projects</a>
        <router-link to="/" v-else @click="closeMobileMenu">Home</router-link>
        <router-link to="/photos" @click="closeMobileMenu">Photos</router-link>
        <router-link
          to="/photobook"
          target="_blank"
          rel="noopener noreferrer"
          @click="closeMobileMenu"
        >
          Book
        </router-link>
        <a href="#contact" v-if="isHome" @click="closeMobileMenu">Contact</a>
      </nav>
      <div class="menu-icon" @click="toggleMobileMenu">
        <span></span>
        <span></span>
        <span></span>
      </div>
    </header>
    <img
      ref="bannerRef"
      class="banner"
      src="@/assets/coms/banner.jpg"
      v-if="isPhotos"
      @load="updateHeaderState"
    />
  </template>
</template>

<style scoped>
img.banner {
  width: 100%;
  aspect-ratio: 2276 / 377;
  object-fit: cover;
  background: #141414;
  display: block;
}
</style>
