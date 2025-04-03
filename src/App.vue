<script setup>
import PhotoWall from './components/PhotoWall.vue';
import DailyLife from './components/DailyLife.vue';
</script>

<template>
  <div class="app-container">
    <header>
      <h1>我们的故事</h1>
      <p class="subtitle">记录美好时光</p>
    </header>
    
    <main>
      <PhotoWall />
      <DailyLife />
    </main>
    
    <footer>
      <p>© {{ new Date().getFullYear() }} 鲁俊&王小麦日常 - 用爱记录每一刻</p>
    </footer>
  </div>
</template>

<style>
/* 全局样式 */
body {
  margin: 0;
  font-family: 'Helvetica Neue', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #f8f9fa;
  color: #333;
  line-height: 1.6;
}
</style>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: 100%;
  overflow-x: hidden;
}

header {
  text-align: center;
  padding: 60px 0;
  background: linear-gradient(135deg, #ff9a9e 0%, #fad0c4 100%);
  color: white;
  width: 100vw;
  margin-left: calc(-50vw + 50%);
  box-sizing: border-box;
}

h1 {
  margin: 0;
  font-size: 3rem;
  font-weight: 700;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.subtitle {
  font-size: 1.2rem;
  margin-top: 10px;
  opacity: 0.9;
}

main {
  flex: 1;
  padding: 20px;
}

footer {
  text-align: center;
  padding: 20px;
  background-color: #333;
  color: white;
  font-size: 0.9rem;
}

/* 响应式设计 */
@media (max-width: 768px) {
  h1 {
    font-size: 2.5rem;
  }
  
  .subtitle {
    font-size: 1rem;
  }
}
</style>
