<script setup>
import { ref, onMounted, onBeforeUnmount, computed, watch } from 'vue';

// 使用Vite的glob导入功能批量导入图片
const imageModules = import.meta.glob('../img/*.jpg');

// 处理图片数据
const photos = ref([]);
const loadingPhotos = ref(false);
const loadedImages = ref(new Map()); // 图片缓存
const loadingProgress = ref(0); // 加载进度

// 3D照片墙参数
const currentIndex = ref(0);
const autoRotate = ref(true);
const rotateTimeout = ref(null);
const photoWallRef = ref(null);

// 格式化图片标题
const formatTitle = (filename) => {
  // 移除文件扩展名和路径
  const name = filename.split('/').pop().replace('.jpg', '');
  // 如果文件名是日期格式，生成对应的标题
  if (name.startsWith('IMG_')) {
    const date = name.slice(4, 12);
    const year = date.slice(0, 4);
    const month = date.slice(4, 6);
    const day = date.slice(6, 8);
    return `${year}年${month}月${day}日的回忆`;
  }
  // 如果是其他格式的文件名，直接返回
  return name;
};

// 分批加载图片，避免一次性加载所有图片
const loadPhotosInBatches = async (entries, batchSize = 3) => {
  const totalImages = entries.length;
  const batches = Math.ceil(totalImages / batchSize);
  
  // 预加载所有图片的基本信息，但不等待图片内容加载完成
  const allPhotosInfo = entries.map(([path], index) => ({
    id: index + 1,
    url: '', // 暂时为空，稍后填充
    title: formatTitle(path),
    loaded: false,
    path: path // 保存路径以便后续加载
  }));
  
  // 先创建所有照片的占位，使DOM结构完整
  photos.value = allPhotosInfo;
  
  // 延迟一点时间后设置照片墙基本结构
  setTimeout(() => {
    loadingPhotos.value = false;
    setupPhotoWall();
  }, 300);
  
  // 然后分批加载实际图片内容
  for (let i = 0; i < batches; i++) {
    const start = i * batchSize;
    const end = Math.min(start + batchSize, totalImages);
    const batchEntries = entries.slice(start, end);
    
    // 加载当前批次的图片
    await Promise.all(
      batchEntries.map(async ([path, loadModule], batchIndex) => {
        const index = start + batchIndex;
        
        // 检查缓存中是否已有该图片
        if (loadedImages.value.has(path)) {
          const cachedPhoto = loadedImages.value.get(path);
          // 更新对应索引的照片信息
          photos.value[index] = { ...photos.value[index], ...cachedPhoto };
          return;
        }
        
        // 加载图片
        const module = await loadModule();
        const photoData = {
          url: module.default,
          loaded: true
        };
        
        // 存入缓存
        loadedImages.value.set(path, { ...photos.value[index], ...photoData });
        
        // 更新照片数组中对应索引的照片
        photos.value[index] = { ...photos.value[index], ...photoData };
        
        // 更新加载进度
        loadingProgress.value = Math.round(((i * batchSize) + batchIndex + 1) / totalImages * 100);
      })
    );
    
    // 第一批加载完成后开始自动旋转
    if (i === 0 && autoRotate.value) {
      setTimeout(() => {
        startAutoRotate();
      }, 500);
    }
  }
};

// 加载图片
onMounted(async () => {
  loadingPhotos.value = true;
  try {
    const entries = Object.entries(imageModules);
    photos.value = []; // 清空现有照片
    await loadPhotosInBatches(entries);
  } catch (error) {
    console.error('加载照片失败:', error);
    loadingPhotos.value = false;
  }
});

// 获取随机图片
const fetchRandomPhotos = async () => {
  try {
    loadingPhotos.value = true;
    // 使用Unsplash API获取随机情侣照片
    // 这里使用公共API获取随机图片，后期可以替换为本地图片
    const urls = [];
    for (let i = 0; i < 15; i++) {
      // 使用不同尺寸和主题的随机图片
      urls.push(`https://picsum.photos/400/300?random=${i}`);
    }
    photos.value = urls.map((url, index) => ({
      id: index,
      url,
      title: `照片 ${index + 1}`,
    }));
  } catch (error) {
    console.error('获取照片失败:', error);
  } finally {
    loadingPhotos.value = false;
  }
};

// 设置3D照片墙效果
const setupPhotoWall = () => {
  if (photos.value.length === 0) return;
  
  // 确保DOM已完全渲染
  setTimeout(() => {
    const photoItems = document.querySelectorAll('.photo-item');
    const photoCount = photoItems.length;
    const baseAngle = 360 / photoCount;
    
    // 计算合适的半径，根据照片数量动态调整
    // 照片数量越多，半径越大，避免照片重叠
    const baseRadius = 480;
    // 确保半径足够大，使所有照片都能在视野中
    const radius = Math.max(baseRadius, baseRadius + (photoCount - 8) * 20);
    
    // 先设置容器样式，确保3D环境准备好
    if (photoWallRef.value) {
      photoWallRef.value.style.transform = 'rotateY(0deg)';
      // 确保3D变换样式应用
      photoWallRef.value.style.transformStyle = 'preserve-3d';
      photoWallRef.value.style.webkitTransformStyle = 'preserve-3d';
    }
    
    // 然后设置每个照片的样式
    photoItems.forEach((item, index) => {
      // 计算每个照片的旋转角度和Z轴位移
      const rotateY = baseAngle * index;
      const translateZ = radius; // 使用动态计算的半径
      
      // 设置照片的3D变换
      item.style.transform = `rotateY(${rotateY}deg) translateZ(${translateZ}px)`;
      // 确保所有照片都可见
      item.style.backfaceVisibility = 'hidden';
      item.style.webkitBackfaceVisibility = 'hidden';
      item.style.visibility = 'visible';
      // 重置其他可能影响显示的样式
      item.style.opacity = '1';
      item.style.pointerEvents = 'auto';
      // 确保3D变换样式应用到每个照片
      item.style.transformStyle = 'preserve-3d';
      item.style.webkitTransformStyle = 'preserve-3d';
    });
    
    console.log(`照片墙已设置，共有${photoCount}张照片`);
  }, 100); // 短暂延迟确保DOM已完全渲染
};

// 旋转照片墙
const rotatePhotoWall = (direction = 1) => {
  if (!photoWallRef.value || photos.value.length === 0) return;
  
  currentIndex.value = (currentIndex.value + direction + photos.value.length) % photos.value.length;
  const rotateY = (360 / photos.value.length) * currentIndex.value * -1;
  
  // 确保所有照片元素都已经渲染
  const photoItems = document.querySelectorAll('.photo-item');
  if (photoItems.length !== photos.value.length) {
    console.log('照片元素数量与数据不匹配，重新设置照片墙');
    setupPhotoWall();
    setTimeout(() => rotatePhotoWall(direction), 300);
    return;
  }
  
  // 平滑旋转照片墙 - 保持照片墙水平
  photoWallRef.value.style.transition = 'transform 0.8s ease';
  photoWallRef.value.style.transform = `rotateY(${rotateY}deg)`;
  
  // 更新当前照片的状态
  photoItems.forEach((item, index) => {
    if (index === currentIndex.value) {
      
    } else {
      // 恢复其他照片的原始位置
      const baseAngle = 360 / photos.value.length;
      const rotateY = baseAngle * index;
      const radius = Math.max(480, 480 + (photos.value.length - 8) * 20);
      item.style.transform = `rotateY(${rotateY}deg) translateZ(${radius}px)`;
      item.style.zIndex = 'auto';
      item.style.boxShadow = '0 10px 30px rgba(0, 0, 0, 0.3)';
    }
    // 确保所有照片在旋转过程中保持可见
    item.style.backfaceVisibility = 'hidden';
    item.style.webkitBackfaceVisibility = 'hidden';
    item.style.visibility = 'visible';
  });

};

// 自动旋转
const startAutoRotate = () => {
  if (rotateTimeout.value) clearTimeout(rotateTimeout.value);
  
  rotateTimeout.value = setTimeout(() => {
    rotatePhotoWall(1);
    if (autoRotate.value) startAutoRotate();
  }, 3000);
};

// 停止自动旋转
const stopAutoRotate = () => {
  if (rotateTimeout.value) {
    clearTimeout(rotateTimeout.value);
    rotateTimeout.value = null;
  }
};

// 键盘控制
const handleKeyDown = (e) => {
  if (e.key === 'ArrowLeft') {
    stopAutoRotate();
    rotatePhotoWall(-1);
  } else if (e.key === 'ArrowRight') {
    stopAutoRotate();
    rotatePhotoWall(1);
  } else if (e.key === ' ') {
    // 空格键切换自动旋转
    autoRotate.value = !autoRotate.value;
    if (autoRotate.value) {
      startAutoRotate();
    } else {
      stopAutoRotate();
    }
  }
};

// 组件挂载时设置3D照片墙
onMounted(() => {
  // 设置键盘事件监听
  window.addEventListener('keydown', handleKeyDown);
  
  // 设置照片墙并开始自动旋转
  // 增加延迟时间，确保DOM完全渲染
  setTimeout(() => {
    setupPhotoWall();
    // 再次延迟自动旋转，确保照片墙设置完成
    setTimeout(() => {
      if (autoRotate.value) startAutoRotate();
    }, 500);
  }, 1000); // 延长延迟时间确保DOM完全更新
});

// 监听照片数组变化，确保照片加载后重新设置照片墙
const photoLoadedHandler = () => {
  // 检查是否所有照片都已加载
  const allLoaded = photos.value.every(photo => photo.loaded || photo.url === '');
  if (allLoaded && photos.value.length > 0) {
    // 重新设置照片墙
    setupPhotoWall();
  }
};

// 使用watch监听照片加载状态变化
watch(() => [...photos.value], photoLoadedHandler, { deep: true });

// 组件卸载前清理
onBeforeUnmount(() => {
  stopAutoRotate();
  window.removeEventListener('keydown', handleKeyDown);
});

</script>

<template>
  <div class="photo-wall-container">
    <h2>我们的照片墙</h2>
    <div class="photo-wall">
      <div v-if="loadingPhotos" class="loading">
        <p>正在加载照片... {{loadingProgress}}%</p>
        <div class="progress-bar">
          <div class="progress" :style="{width: loadingProgress + '%'}"></div>
        </div>
      </div>
      <div v-else class="photo-container">
        <div class="scene">
          <div class="photo-wall-3d" ref="photoWallRef">
            <div 
              v-for="(photo, index) in photos" 
              :key="photo.id"
              class="photo-item"
              @mouseenter="stopAutoRotate"
              @mouseleave="autoRotate ? startAutoRotate() : null"
            >
              <!-- 添加图片加载状态指示器 -->
              <div v-if="!photo.loaded && photo.url" class="photo-loading">
                <div class="loading-spinner"></div>
              </div>
              <img 
                v-if="photo.url" 
                :src="photo.url" 
                :alt="photo.title" 
                loading="lazy"
                @load="photo.loaded = true"
                :class="{'fade-in': photo.loaded}"
              />
              <!-- 未加载时显示占位 -->
              <div v-else class="photo-placeholder">
                <span>加载中...</span>
              </div>
            </div>
          </div>
        </div>
        
      </div>
    </div>
  </div>
</template>

<style scoped>
.photo-wall-container {
  width: 100%;
  margin: 0 auto;
  padding: 20px 0;
  text-align: center;
}

.photo-wall {
  margin: 30px auto;
  position: relative;
}

.loading {
  text-align: center;
  padding: 80px;
  font-size: 18px;
  color: #666;
  background: rgba(255,255,255,0.8);
  border-radius: 12px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.05);
}

.progress-bar {
  width: 80%;
  height: 8px;
  background-color: #eee;
  border-radius: 4px;
  margin: 15px auto;
  overflow: hidden;
}

.progress {
  height: 100%;
  background: linear-gradient(90deg, #ff9a9e 0%, #fad0c4 100%);
  transition: width 0.3s ease;
}

.fade-in {
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.photo-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 600px;
  position: relative;
  /* 确保容器足够大以显示所有照片 */
  min-height: 600px;
}

.scene {
  width: 100%;
  height: 500px;
  perspective: 1200px;
  position: relative;
  overflow: visible;
  /* 增加场景大小以容纳所有照片 */
  min-height: 500px;
  margin: 0 auto;
  /* 确保3D效果正确显示 */
  transform-style: preserve-3d;
}

.photo-wall-3d {
  width: 100%;
  height: 100%;
  position: absolute;
  transform-style: preserve-3d;
  transition: transform 1s ease;
  /* 移除倾斜角度，保持水平 */
  transform: rotateX(0deg);
  transform-origin: center center;
  /* 确保所有照片都能显示 */
  will-change: transform;
  /* 确保3D效果正确显示 */
  -webkit-transform-style: preserve-3d;
  /* 增加渲染优先级 */
  z-index: 1;
}

.photo-item {
  position: absolute;
  width: 280px;
  height: 380px;
  left: 50%;
  top: 50%;
  margin-left: -140px;
  margin-top: -190px;
  cursor: pointer;
  transition: all 0.5s ease;
  transform-origin: center center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
  border-radius: 8px;
  background-color: #fff;
  z-index: auto;
  /* 确保所有照片都能显示 */
  -webkit-transform-style: preserve-3d;
  transform-style: preserve-3d;
  /* 确保照片不会消失 */
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  /* 添加初始可见性 */
  visibility: visible;
  /* 添加GPU加速 */
  will-change: transform, opacity;
  /* 确保照片在加载前有占位 */
  min-width: 280px;
  min-height: 380px;
  /* 添加背景色作为占位 */
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.photo-item:hover {
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.5);
  transform: scale(1.05) translateZ(20px);
  z-index: 10;
}

.photo-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  display: block;
  /* 确保图片在加载过程中不会导致布局变化 */
  min-height: 100%;
}

/* 照片加载状态指示器 */
.photo-loading {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.7);
  z-index: 2;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(255, 157, 157, 0.3);
  border-radius: 50%;
  border-top-color: #ff9d9d;
  animation: spin 1s ease-in-out infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* 照片占位样式 */
.photo-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  color: #666;
  font-size: 16px;
}

.controls {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  gap: 20px;
}

.control-btn {
  background: linear-gradient(135deg, #ff9a9e 0%, #fad0c4 100%);
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 30px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.control-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(0, 0, 0, 0.15);
}

.control-btn:active {
  transform: translateY(1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

@media (max-width: 768px) {
  .photo-container {
    height: 450px;
  }
  
  .scene {
    height: 350px;
  }
  
  .photo-item {
    width: 220px;
    height: 300px;
    margin-left: -110px;
    margin-top: -150px;
  }
}
</style>