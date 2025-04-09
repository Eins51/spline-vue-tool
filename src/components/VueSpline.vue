<template>
  <div class="spline-container" ref="container">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import { Application } from '@splinetool/runtime';

const props = defineProps({
  url: {
    type: String,
    required: true
  },
  background: {
    type: String,
    default: 'transparent'
  },
  containerStyle: {
    type: Object,
    default: () => ({})
  },
  canvasStyle: {
    type: Object,
    default: () => ({})
  },
  options: {
    type: Object,
    default: () => ({})
  }
});

const emit = defineEmits(['load', 'error']);

const container = ref(null);
const canvas = ref(null);
let splineApp = null;

// 初始化 Spline 应用
const initSpline = async () => {
  try {
    if (!container.value || !canvas.value) return;

    // 创建 Spline 应用实例
    splineApp = new Application(canvas.value, {
      ...props.options,
      background: props.background
    });

    // 加载场景
    await splineApp.load(props.url);

    // 应用容器样式
    Object.assign(container.value.style, props.containerStyle);
    
    // 应用画布样式
    Object.assign(canvas.value.style, props.canvasStyle);

    // 触发加载完成事件
    emit('load', splineApp);
  } catch (error) {
    console.error('Failed to load Spline scene:', error);
    emit('error', error);
  }
};

// 监听 URL 变化
watch(() => props.url, () => {
  if (splineApp) {
    splineApp.load(props.url);
  }
});

// 监听背景变化
watch(() => props.background, (newBackground) => {
  if (splineApp && typeof splineApp.setBackgroundColor === 'function') {
    splineApp.setBackgroundColor(newBackground);
  }
});

// 监听选项变化
watch(() => props.options, (newOptions) => {
  if (splineApp) {
    // 更新相机位置
    if (newOptions.cameraPosition && splineApp._camera) {
      const { x, y, z } = newOptions.cameraPosition;
      splineApp._camera.position.set(x, y, z);
    }
    
    // 更新对象缩放
    if (newOptions.objectScale && splineApp._scene) {
      splineApp._scene.traverse((object) => {
        if (object.scale) {
          const { x, y, z } = newOptions.objectScale;
          object.scale.set(x, y, z);
        }
      });
    }
    
    // 更新缩放
    if (newOptions.zoom !== undefined && typeof splineApp.setZoom === 'function') {
      splineApp.setZoom(newOptions.zoom);
    }
  }
}, { deep: true });

// 组件挂载时初始化
onMounted(() => {
  initSpline();
});

// 组件卸载时清理
onBeforeUnmount(() => {
  if (splineApp) {
    splineApp.dispose();
    splineApp = null;
  }
});

// 暴露方法给父组件
defineExpose({
  getApp: () => splineApp
});
</script>

<style scoped>
.spline-container {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
}

canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style> 