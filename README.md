# 使用 Trae 设计的 Vue3 轮播图组件

Trae 设计的 Vue3 轮播图组件，专为数据分析领域企业网站设计，支持自动播放、触摸滑动、响应式布局等特性。

## 🚀 特性

- ✅ **Vue3 Composition API** - 使用最新的Vue3语法
- ✅ **TypeScript支持** - 完整的类型定义
- ✅ **TailwindCSS样式** - 现代化的样式系统
- ✅ **自动播放** - 支持自定义间隔时间
- ✅ **鼠标交互** - 悬浮暂停、显示切换箭头
- ✅ **触摸支持** - 移动端左右滑动
- ✅ **响应式设计** - PC端和移动端适配
- ✅ **图片懒加载** - 性能优化
- ✅ **淡入淡出动画** - 流畅的切换效果
- ✅ **图片缩放效果** - 鼠标悬浮时图片放大
- ✅ **指示器** - 底部圆点导航
- ✅ **自定义配置** - 高度、颜色、时间等可配置

## 📦 安装

```bash
# 克隆项目
git clone <repository-url>
cd swiper-trae

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

## 🎯 使用方法

### 基础用法

```vue
<template>
  <Swiper
    :images="images"
    @change="handleChange"
    @click="handleClick"
  />
</template>

<script setup>
import Swiper from './Swiper.vue'

const images = [
  {
    id: 1,
    url: 'https://example.com/image1.jpg',
    title: '图片标题1',
    link: 'https://example.com/link1'
  },
  {
    id: 2,
    url: 'https://example.com/image2.jpg',
    title: '图片标题2',
    link: 'https://example.com/link2'
  }
]

const handleChange = (index, image) => {
  console.log('当前索引:', index, '当前图片:', image)
}

const handleClick = (image) => {
  console.log('点击图片:', image)
}
</script>
```

### 高级配置

```vue
<template>
  <Swiper
    :images="images"
    :autoplay="true"
    :interval="3000"
    height="500px"
    mobile-height="250px"
    active-indicator-color="#3b82f6"
    @change="handleChange"
    ref="swiperRef"
  />
</template>

<script setup>
import { ref } from 'vue'
import Swiper from './Swiper.vue'

const swiperRef = ref()

// 手动控制
const goToNext = () => {
  swiperRef.value.next()
}

const goToPrev = () => {
  swiperRef.value.prev()
}

const goToSlide = (index) => {
  swiperRef.value.goTo(index)
}
</script>
```

## 📋 Props

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `images` | `SwiperImage[]` | `[]` | 图片数组 |
| `autoplay` | `boolean` | `true` | 是否自动播放 |
| `interval` | `number` | `5000` | 自动播放间隔时间(ms) |
| `height` | `string` | `'400px'` | PC端高度 |
| `mobileHeight` | `string` | `'200px'` | 移动端高度 |
| `activeIndicatorColor` | `string` | `'#ffffff'` | 当前指示器颜色 |

### SwiperImage 类型定义

```typescript
interface SwiperImage {
  id: number | string    // 唯一标识
  url: string           // 图片地址
  title?: string        // 图片标题（可选）
  link?: string         // 点击跳转链接（可选）
}
```

## 🎪 Events

| 事件名 | 参数 | 说明 |
|--------|------|------|
| `change` | `(index: number, image: SwiperImage)` | 轮播图切换时触发 |
| `click` | `(image: SwiperImage)` | 点击图片时触发 |

## 🔧 Methods

通过 `ref` 可以调用以下方法：

| 方法名 | 参数 | 说明 |
|--------|------|------|
| `next()` | - | 切换到下一张 |
| `prev()` | - | 切换到上一张 |
| `goTo(index)` | `index: number` | 跳转到指定索引 |
| `getCurrentIndex()` | - | 获取当前索引 |
| `getCurrentImage()` | - | 获取当前图片对象 |

## 🎨 样式特性

### 尺寸规格
- **宽度**: 100%（自适应容器）
- **高度**: PC端默认400px，移动端默认200px
- **图片比例**: 16:9，多余部分自动裁切

### 交互效果
- **切换动画**: 300ms淡入淡出效果
- **鼠标悬浮**: 图片放大110%，箭头显示
- **指示器**: 默认80%透明度，当前项100%透明度
- **箭头**: 半透明白色背景，悬浮时增加不透明度

### 响应式设计
- **移动端**: 支持触摸滑动，自动调整高度
- **PC端**: 鼠标悬浮交互，键盘导航支持

## 🏗️ 设计过程

### 1. 架构设计

#### 组件结构
```
Swiper.vue
├── 图片容器 (flex布局)
│   ├── 图片项 (懒加载)
│   └── 标题覆盖层
├── 左右箭头 (条件渲染)
└── 底部指示器
```

#### 状态管理
- `currentIndex`: 当前显示的图片索引
- `isHovered`: 鼠标悬浮状态
- `isTransitioning`: 切换动画状态
- `autoplayTimer`: 自动播放定时器
- `loadedImages`: 已加载图片集合

### 2. 核心功能实现

#### 自动播放机制
```typescript
const startAutoplay = () => {
  if (!props.autoplay || props.images.length <= 1) return
  
  stopAutoplay()
  autoplayTimer.value = setInterval(() => {
    if (!isHovered.value && !isTouching.value) {
      nextSlide()
    }
  }, props.interval)
}
```

#### 触摸滑动处理
```typescript
const handleTouchEnd = () => {
  const deltaX = touchEndX.value - touchStartX.value
  const deltaY = Math.abs(touchEndY.value - touchStartY.value)
  
  // 水平滑动距离大于垂直滑动且超过阈值
  if (Math.abs(deltaX) > deltaY && Math.abs(deltaX) > 50) {
    if (deltaX > 0) {
      prevSlide()
    } else {
      nextSlide()
    }
  }
}
```

#### 图片懒加载策略
```typescript
const shouldLoadImage = (index: number): boolean => {
  // 加载当前图片、前一张和后一张
  const loadRange = 1
  const start = Math.max(0, currentIndex.value - loadRange)
  const end = Math.min(props.images.length - 1, currentIndex.value + loadRange)
  return index >= start && index <= end
}
```

### 3. 性能优化

#### 懒加载实现
- 只加载当前显示图片及相邻图片
- 减少初始加载时间和内存占用
- 提升用户体验

#### 动画优化
- 使用CSS transform进行硬件加速
- 防抖处理避免频繁切换
- 合理的过渡时间设置

#### 内存管理
- 及时清理定时器
- 移除事件监听器
- 响应式数据合理使用

### 4. 响应式设计

#### 移动端适配
```typescript
const checkMobile = () => {
  isMobile.value = window.innerWidth < 768
}

const containerHeight = computed(() => {
  return isMobile.value ? props.mobileHeight : props.height
})
```

#### 触摸事件处理
- 区分水平和垂直滑动
- 防止误触发
- 流畅的手势响应

### 5. 可访问性考虑

- 键盘导航支持
- 语义化HTML结构
- 适当的ARIA属性
- 颜色对比度优化

## 🔧 开发脚本

```bash
# 开发模式
npm run dev

# 构建生产版本
npm run build

# 预览构建结果
npm run preview

# 类型检查
npm run type-check
```

## 📁 项目结构

```
swiper-trae/
├── src/
│   ├── App.vue          # 示例应用
│   ├── main.ts          # 入口文件
│   └── style.css        # 全局样式
├── Swiper.vue           # 轮播图组件
├── index.html           # HTML模板
├── package.json         # 项目配置
├── tsconfig.json        # TypeScript配置
├── vite.config.ts       # Vite配置
├── tailwind.config.js   # TailwindCSS配置
└── README.md           # 项目文档
```

## 🌟 最佳实践

### 图片优化
- 使用适当的图片格式（WebP、AVIF）
- 提供多种尺寸的图片
- 设置合理的图片压缩比例

### 性能监控
- 监控图片加载时间
- 跟踪用户交互行为
- 优化关键渲染路径

### 用户体验
- 提供加载状态指示
- 合理的动画时长
- 直观的交互反馈

## 🤝 贡献

欢迎提交Issue和Pull Request来改进这个组件！

## 📄 许可证

MIT License

---

T## 📝 项目设计提示词

以下是本项目的完整设计需求和提示词，记录了项目的原始需求和设计思路：

### 原始需求

**使用场景**: 数据分析领域企业网站的首页轮播图展示

### 具体需求

#### 1. 基本功能
- 自动播放，默认间隔5s自动切换一次
- 鼠标悬浮时自动播放暂停，展示左右切换箭头
- 底部有小圆点指示器
- 适配手机端、支持在手机端左右滑动

#### 2. 样式设计
- 轮播图尺寸：宽度100%，高度pc端默认400px，移动端默认200px
- 图片保持16:9 比例，多余部分裁切
- 左右箭头：半透透明白色背景，hover时不透明度增加
- 指示器：默认80%透明白色小圆点，当前项时透明度默认为100%
- 切换动画：淡入淡出效果，持续时间为300ms
- 鼠标悬浮在轮播图上时，当前轮播图略微放大10%，放大动画为淡入淡出

#### 3. 技术要求
- 使用Vue3 的composition api
- 不依赖第三方轮播库实现，原生实现
- 使用typescript
- 样式使用tailwindcss

#### 4. 数组结构
传入组件的图片数组格式：
```javascript
[{
  id: 1,
  url: '图片地址',
  title: '图片标题',
  link: '点击跳转链接'
}]
```

#### 5. 额外功能
- 图片懒加载
- 支持传入自定义的轮播图切换时间
- 支持传入自定义的轮播图高度
- 支持传入小圆点指示器当前项的背景颜色
- 暴露切换方法给父组件调用
- 切换时触发onChange事件

### 设计目标

本项目旨在创建一个功能完整、性能优秀的Vue3轮播图组件，专门为数据分析领域的企业网站设计。组件需要具备现代化的交互体验、响应式设计、以及企业级的代码质量。

### 实现亮点

1. **原生实现**: 完全不依赖第三方轮播库，确保代码的可控性和轻量化
2. **性能优化**: 实现图片懒加载、硬件加速动画等性能优化策略
3. **用户体验**: 流畅的触摸交互、直观的视觉反馈、合理的动画时长
4. **开发体验**: 完整的TypeScript支持、清晰的API设计、详细的文档说明
5. **企业级品质**: 考虑可访问性、浏览器兼容性、代码可维护性

---

**注意**: 这是一个演示项目，用于展示Vue3轮播图组件的完整实现。在生产环境中使用时，请根据具体需求进行调整和优化。