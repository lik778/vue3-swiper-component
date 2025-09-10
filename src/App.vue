<template>
  <div id="app" class="min-h-screen bg-gray-100 py-8">
    <div class="container mx-auto px-4">
      <h1 class="text-3xl font-bold text-center mb-8 text-gray-800">
        Vue3 轮播图组件示例
      </h1>

      <!-- 基础轮播图 -->
      <div class="mb-12">
        <h2 class="text-xl font-semibold mb-4 text-gray-700">基础轮播图</h2>
        <div class="max-w-4xl mx-auto">
          <Swiper
            :images="sampleImages"
            :autoplay="true"
            :interval="5000"
            @change="handleSwiperChange"
            @click="handleImageClick"
            ref="swiperRef"
          />
        </div>
      </div>

      <!-- 自定义配置轮播图 -->
      <div class="mb-12">
        <h2 class="text-xl font-semibold mb-4 text-gray-700">
          自定义配置轮播图
        </h2>
        <div class="max-w-4xl mx-auto">
          <Swiper
            :images="customImages"
            :autoplay="autoplayEnabled"
            :interval="customInterval"
            height="300px"
            mobile-height="150px"
            active-indicator-color="#3b82f6"
            @change="handleCustomSwiperChange"
          />
        </div>

        <!-- 控制面板 -->
        <div class="mt-6 p-4 bg-white rounded-lg shadow-md max-w-md mx-auto">
          <h3 class="text-lg font-medium mb-4">控制面板</h3>
          <div class="space-y-4">
            <div class="flex items-center justify-between">
              <label class="text-sm font-medium text-gray-700">自动播放</label>
              <button
                @click="autoplayEnabled = !autoplayEnabled"
                :class="[
                  'relative inline-flex h-6 w-11 items-center rounded-full transition-colors',
                  autoplayEnabled ? 'bg-blue-600' : 'bg-gray-200',
                ]"
              >
                <span
                  :class="[
                    'inline-block h-4 w-4 transform rounded-full bg-white transition-transform',
                    autoplayEnabled ? 'translate-x-6' : 'translate-x-1',
                  ]"
                />
              </button>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-700 mb-2">
                切换间隔: {{ customInterval }}ms
              </label>
              <input
                v-model="customInterval"
                type="range"
                min="1000"
                max="10000"
                step="500"
                class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
              />
            </div>

            <div class="flex space-x-2">
              <button
                @click="goToPrev"
                class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
              >
                上一张
              </button>
              <button
                @click="goToNext"
                class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
              >
                下一张
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- 当前状态显示 -->
      <div class="max-w-2xl mx-auto bg-white p-6 rounded-lg shadow-md">
        <h3 class="text-lg font-medium mb-4">当前状态</h3>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm">
          <div>
            <span class="font-medium text-gray-700">当前索引:</span>
            <span class="ml-2 text-blue-600">{{ currentIndex }}</span>
          </div>
          <div>
            <span class="font-medium text-gray-700">当前图片:</span>
            <span class="ml-2 text-blue-600">{{ 
               currentImage && currentImage.title ? currentImage.title : "N/A" 
             }}</span>
          </div>
          <div>
            <span class="font-medium text-gray-700">自动播放:</span>
            <span
              class="ml-2"
              :class="autoplayEnabled ? 'text-green-600' : 'text-red-600'"
            >
              {{ autoplayEnabled ? "开启" : "关闭" }}
            </span>
          </div>
          <div>
            <span class="font-medium text-gray-700">切换间隔:</span>
            <span class="ml-2 text-blue-600">{{ customInterval }}ms</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import Swiper from "../Swiper.vue";

// 类型定义
interface ImageType {
  id: number;
  url: string;
  title: string;
  link: string;
}

// 轮播图引用
const swiperRef = ref();

// 控制状态
const autoplayEnabled = ref(true);
const customInterval = ref(5000);
const currentIndex = ref(0);
const currentImage = ref<ImageType | null>(null);

// 示例图片数据
const sampleImages = reactive([
  {
    id: 1,
    url: "https://images.unsplash.com/photo-1551434678-e076c223a692?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2070&q=80",
    title: "数据分析仪表板",
    link: "https://example.com/dashboard",
  },
  {
    id: 2,
    url: "https://images.unsplash.com/photo-1460925895917-afdab827c52f?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2015&q=80",
    title: "商业智能解决方案",
    link: "https://example.com/business-intelligence",
  },
  {
    id: 3,
    url: "https://images.unsplash.com/photo-1504868584819-f8e8b4b6d7e3?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2076&q=80",
    title: "数据可视化工具",
    link: "https://example.com/visualization",
  },
  {
    id: 4,
    url: "https://images.unsplash.com/photo-1518186285589-2f7649de83e0?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2074&q=80",
    title: "机器学习平台",
    link: "https://example.com/machine-learning",
  },
]);

const customImages = reactive([
  {
    id: 5,
    url: "https://images.unsplash.com/photo-1551288049-bebda4e38f71?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2070&q=80",
    title: "企业级数据分析",
    link: "https://example.com/enterprise",
  },
  {
    id: 6,
    url: "https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2070&q=80",
    title: "实时数据监控",
    link: "https://example.com/monitoring",
  },
  {
    id: 7,
    url: "https://images.unsplash.com/photo-1590650153855-d9e808231d41?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2070&q=80",
    title: "云端数据处理",
    link: "https://example.com/cloud",
  },
]);

// 事件处理函数
const handleSwiperChange = (index: number, image: any) => {
  console.log("轮播图切换:", index, image);
};

const handleCustomSwiperChange = (index: number, image: any) => {
  currentIndex.value = index;
  currentImage.value = image;
  console.log("自定义轮播图切换:", index, image);
};

const handleImageClick = (image: any) => {
  console.log("图片点击:", image);
  alert(`点击了图片: ${image.title}`);
};

// 控制方法
const goToPrev = () => {
  if (swiperRef.value) {
    swiperRef.value.prev();
  }
};

const goToNext = () => {
  if (swiperRef.value) {
    swiperRef.value.next();
  }
};
</script>

<style scoped>
/* 自定义滑块样式 */
input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  height: 20px;
  width: 20px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
}

input[type="range"]::-moz-range-thumb {
  height: 20px;
  width: 20px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: none;
}
</style>
