<template>
  <div
    ref="swiperContainer"
    class="relative w-full overflow-hidden bg-gray-900 rounded-lg"
    :style="{ height: containerHeight }"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
    @touchstart="handleTouchStart"
    @touchmove="handleTouchMove"
    @touchend="handleTouchEnd"
  >
    <!-- 轮播图片容器 -->
    <div
      class="flex transition-transform duration-300 ease-in-out h-full"
      :style="{ transform: `translateX(-${currentIndex * 100}%)` }"
    >
      <div
        v-for="(item, index) in images"
        :key="item.id"
        class="w-full h-full flex-shrink-0 relative overflow-hidden cursor-pointer"
        @click="handleImageClick(item)"
      >
        <img
          v-if="shouldLoadImage(index)"
          :src="item.url"
          :alt="item.title"
          class="w-full h-full object-cover transition-transform duration-300 ease-in-out"
          :class="{ 'scale-110': isHovered && index === currentIndex }"
          @load="handleImageLoad(index)"
          @error="handleImageError(index)"
        />
        <div
          v-else
          class="w-full h-full bg-gray-200 flex items-center justify-center"
        >
          <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-gray-600"></div>
        </div>
        
        <!-- 图片标题覆盖层 -->
        <div
          v-if="item.title"
          class="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/60 to-transparent p-4"
        >
          <h3 class="text-white text-lg font-medium truncate">{{ item.title }}</h3>
        </div>
      </div>
    </div>

    <!-- 左右切换箭头 -->
    <transition name="fade">
      <div v-if="isHovered && images.length > 1" class="absolute inset-y-0 left-0 right-0 flex items-center justify-between px-4 pointer-events-none">
        <button
          class="bg-white/50 hover:bg-white/80 rounded-full p-2 transition-all duration-200 pointer-events-auto"
          @click="prevSlide"
          :disabled="isTransitioning"
        >
          <svg class="w-6 h-6 text-gray-800" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
          </svg>
        </button>
        <button
          class="bg-white/50 hover:bg-white/80 rounded-full p-2 transition-all duration-200 pointer-events-auto"
          @click="nextSlide"
          :disabled="isTransitioning"
        >
          <svg class="w-6 h-6 text-gray-800" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
          </svg>
        </button>
      </div>
    </transition>

    <!-- 底部指示器 -->
    <div
      v-if="images.length > 1"
      class="absolute bottom-4 left-1/2 transform -translate-x-1/2 flex space-x-2"
    >
      <button
        v-for="(_, index) in images"
        :key="index"
        class="w-3 h-3 rounded-full transition-all duration-200"
        :class="[
          index === currentIndex
            ? 'opacity-100'
            : 'opacity-80',
          index === currentIndex
            ? `bg-[${activeIndicatorColor}]`
            : 'bg-white'
        ]"
        @click="goToSlide(index)"
        :disabled="isTransitioning"
      ></button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, nextTick, watch } from 'vue'

// 类型定义
interface SwiperImage {
  id: number | string
  url: string
  title?: string
  link?: string
}

interface SwiperProps {
  images: SwiperImage[]
  autoplay?: boolean
  interval?: number
  height?: string
  mobileHeight?: string
  activeIndicatorColor?: string
}

interface SwiperEmits {
  (e: 'change', currentIndex: number, currentImage: SwiperImage): void
  (e: 'click', image: SwiperImage): void
}

// Props定义
const props = withDefaults(defineProps<SwiperProps>(), {
  autoplay: true,
  interval: 5000,
  height: '400px',
  mobileHeight: '200px',
  activeIndicatorColor: '#ffffff'
})

// Emits定义
const emit = defineEmits<SwiperEmits>()

// 响应式数据
const currentIndex = ref(0)
const isHovered = ref(false)
const isTransitioning = ref(false)
const autoplayTimer = ref<NodeJS.Timeout | null>(null)
const swiperContainer = ref<HTMLElement | null>(null)
const loadedImages = ref(new Set<number>())
const isMobile = ref(false)

// 触摸相关
const touchStartX = ref(0)
const touchEndX = ref(0)
const touchStartY = ref(0)
const touchEndY = ref(0)
const isTouching = ref(false)

// 计算属性
const containerHeight = computed(() => {
  return isMobile.value ? props.mobileHeight : props.height
})

// 检查是否应该加载图片（懒加载逻辑）
const shouldLoadImage = (index: number): boolean => {
  // 加载当前图片、前一张和后一张
  const loadRange = 1
  const start = Math.max(0, currentIndex.value - loadRange)
  const end = Math.min(props.images.length - 1, currentIndex.value + loadRange)
  return index >= start && index <= end
}

// 检测移动设备
const checkMobile = () => {
  isMobile.value = window.innerWidth < 768
}

// 自动播放控制
const startAutoplay = () => {
  if (!props.autoplay || props.images.length <= 1) return
  
  stopAutoplay()
  autoplayTimer.value = setInterval(() => {
    if (!isHovered.value && !isTouching.value) {
      nextSlide()
    }
  }, props.interval)
}

const stopAutoplay = () => {
  if (autoplayTimer.value) {
    clearInterval(autoplayTimer.value)
    autoplayTimer.value = null
  }
}

// 切换到下一张
const nextSlide = () => {
  if (isTransitioning.value || props.images.length <= 1) return
  
  isTransitioning.value = true
  currentIndex.value = (currentIndex.value + 1) % props.images.length
  
  setTimeout(() => {
    isTransitioning.value = false
  }, 300)
}

// 切换到上一张
const prevSlide = () => {
  if (isTransitioning.value || props.images.length <= 1) return
  
  isTransitioning.value = true
  currentIndex.value = currentIndex.value === 0 ? props.images.length - 1 : currentIndex.value - 1
  
  setTimeout(() => {
    isTransitioning.value = false
  }, 300)
}

// 跳转到指定幻灯片
const goToSlide = (index: number) => {
  if (isTransitioning.value || index === currentIndex.value || props.images.length <= 1) return
  
  isTransitioning.value = true
  currentIndex.value = index
  
  setTimeout(() => {
    isTransitioning.value = false
  }, 300)
}

// 鼠标事件处理
const handleMouseEnter = () => {
  isHovered.value = true
}

const handleMouseLeave = () => {
  isHovered.value = false
}

// 触摸事件处理
const handleTouchStart = (e: TouchEvent) => {
  if (props.images.length <= 1) return
  
  isTouching.value = true
  touchStartX.value = e.touches[0].clientX
  touchStartY.value = e.touches[0].clientY
}

const handleTouchMove = (e: TouchEvent) => {
  if (!isTouching.value) return
  
  touchEndX.value = e.touches[0].clientX
  touchEndY.value = e.touches[0].clientY
  
  // 计算滑动距离
  const deltaX = Math.abs(touchEndX.value - touchStartX.value)
  const deltaY = Math.abs(touchEndY.value - touchStartY.value)
  
  // 如果水平滑动距离大于垂直滑动距离，阻止默认滚动行为
  if (deltaX > deltaY && deltaX > 10) {
    e.preventDefault()
  }
}

const handleTouchEnd = () => {
  if (!isTouching.value) return
  
  isTouching.value = false
  
  const deltaX = touchEndX.value - touchStartX.value
  const deltaY = Math.abs(touchEndY.value - touchStartY.value)
  
  // 只有在水平滑动距离大于垂直滑动距离且超过阈值时才切换
  if (Math.abs(deltaX) > deltaY && Math.abs(deltaX) > 50) {
    if (deltaX > 0) {
      prevSlide()
    } else {
      nextSlide()
    }
  }
}

// 图片点击事件
const handleImageClick = (image: SwiperImage) => {
  emit('click', image)
  if (image.link) {
    window.open(image.link, '_blank')
  }
}

// 图片加载事件
const handleImageLoad = (index: number) => {
  loadedImages.value.add(index)
}

const handleImageError = (index: number) => {
  console.warn(`Failed to load image at index ${index}`)
}

// 监听当前索引变化，触发change事件
watch(currentIndex, (newIndex) => {
  if (props.images[newIndex]) {
    emit('change', newIndex, props.images[newIndex])
  }
})

// 监听图片数组变化
watch(() => props.images, () => {
  if (currentIndex.value >= props.images.length) {
    currentIndex.value = 0
  }
  loadedImages.value.clear()
}, { deep: true })

// 暴露给父组件的方法
defineExpose({
  next: nextSlide,
  prev: prevSlide,
  goTo: goToSlide,
  getCurrentIndex: () => currentIndex.value,
  getCurrentImage: () => props.images[currentIndex.value]
})

// 生命周期
onMounted(() => {
  checkMobile()
  window.addEventListener('resize', checkMobile)
  
  nextTick(() => {
    startAutoplay()
  })
})

onUnmounted(() => {
  stopAutoplay()
  window.removeEventListener('resize', checkMobile)
})

// 监听自动播放设置变化
watch(() => props.autoplay, (newVal) => {
  if (newVal) {
    startAutoplay()
  } else {
    stopAutoplay()
  }
})

watch(() => props.interval, () => {
  if (props.autoplay) {
    startAutoplay()
  }
})
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* 确保指示器颜色正确应用 */
.indicator-active {
  opacity: 1;
}

.indicator-inactive {
  opacity: 0.8;
}
</style>