<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

// 使用Vite的glob导入功能批量导入图片
const imageModules = import.meta.glob('../img/*.jpg');

// 处理图片数据
const photos = ref([]);
const loadingPhotos = ref(false);

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

// 加载图片
onMounted(async () => {
  loadingPhotos.value = true;
  try {
    const entries = Object.entries(imageModules);
    const loadedPhotos = await Promise.all(
      entries.map(async ([path, loadModule], index) => {
        const module = await loadModule();
        return {
          id: index + 1,
          url: module.default,
          title: formatTitle(path)
        };
      })
    );
    photos.value = loadedPhotos;
  } catch (error) {
    console.error('加载照片失败:', error);
  } finally {
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
  
  const photoItems = document.querySelectorAll('.photo-item');
  const photoCount = photoItems.length;
  const baseAngle = 360 / photoCount;
  
  // 计算合适的半径，根据照片数量动态调整
  // 照片数量越多，半径越大，避免照片重叠
  const baseRadius = 450;
  const radius = Math.max(baseRadius, baseRadius + (photoCount - 8) * 30);
  
  photoItems.forEach((item, index) => {
    // 计算每个照片的旋转角度和Z轴位移
    const rotateY = baseAngle * index;
    const translateZ = radius; // 使用动态计算的半径
    
    const translateY = 0;  
    
    // 设置照片的3D变换
    item.style.transform = `rotateY(${rotateY}deg) translateZ(${translateZ}px) translateY(${translateY}px)`;
  });
};

// 旋转照片墙
const rotatePhotoWall = (direction = 1) => {
  if (!photoWallRef.value) return;
  
  currentIndex.value = (currentIndex.value + direction + photos.value.length) % photos.value.length;
  const rotateY = (360 / photos.value.length) * currentIndex.value * -1;
  
  photoWallRef.value.style.transform = `rotateY(${rotateY}deg)`;
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
  setTimeout(() => {
    setupPhotoWall();
    if (autoRotate.value) startAutoRotate();
  }, 500); // 延迟一点时间确保DOM已更新
});

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
        <p>正在加载照片...</p>
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
              <img :src="photo.url" :alt="photo.title" />
            </div>
          </div>
        </div>
        <div class="controls">
          <button @click="rotatePhotoWall(-1)" class="control-btn prev">上一张</button>
          <button @click="autoRotate = !autoRotate; autoRotate ? startAutoRotate() : stopAutoRotate()" class="control-btn play">
            {{ autoRotate ? '暂停' : '播放' }}
          </button>
          <button @click="rotatePhotoWall(1)" class="control-btn next">下一张</button>
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

.photo-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 600px;
  position: relative;
}

.scene {
  width: 100%;
  height: 500px;
  perspective: 1000px;
  position: relative;
  overflow: hidden;
}

.photo-wall-3d {
  width: 100%;
  height: 100%;
  position: absolute;
  transform-style: preserve-3d;
  transition: transform 1s ease;
  transform: rotateX(5deg);
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
  transition: transform 0.5s ease, box-shadow 0.5s ease;
  transform-origin: center center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
  border-radius: 8px;
  background-color: #fff;
  z-index: 1;
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