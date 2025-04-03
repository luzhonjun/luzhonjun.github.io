<script setup>
import { ref } from 'vue';

// 日常生活内容
const dailyMoments = ref([
  {
    id: 1,
    title: '第一次确定关系',
    date: '2024-07-19',
    description: '朦朦胧胧中，我们第一次确定了彼此',
    emoji: '☕️'
  },
  {
    id: 2,
    title: '一起看动物',
    date: '2024-07-20',
    description: '我们在动物园逛了一整天，那天你满头大汗的跑来跑去，像个小精灵',
    emoji: '🎬'
  },
  {
    id: 3,
    title: '第一次牵手',
    date: '2024-07-27',
    description: '距离上一次见面只隔了一个礼拜，感觉像是过了好久，这次终于牵起了你的小手',
    emoji: '🧺'
  },
  {
    id: 4,
    title: '过七夕',
    date: '2024-08-10',
    description: '收到了你的七夕告白情书，有种特别的情绪在我心里酝酿着',
    emoji: '🍳'
  }
]);
</script>

<template>
  <div class="daily-life-container">
    <h2>我们的日常</h2>
    <div class="timeline">
      <div 
        v-for="moment in dailyMoments" 
        :key="moment.id"
        class="timeline-item"
      >
        <div class="timeline-emoji">{{ moment.emoji }}</div>
        <div class="timeline-content">
          <h3>{{ moment.title }}</h3>
          <div class="timeline-date">{{ moment.date }}</div>
          <p>{{ moment.description }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.daily-life-container {
  width: 100%;
  /* max-width: 800px; */
  margin: 40px auto;
  padding: 20px;
}

h2 {
  text-align: center;
  margin-bottom: 30px;
  color: #333;
}

.timeline {
  position: relative;
  padding: 20px 0;
}

.timeline::before {
  content: '';
  position: absolute;
  top: 0;
  bottom: 0;
  left: 50px;
  width: 2px;
  background: linear-gradient(to bottom, #ff6b6b, #4ecdc4);
}

.timeline-item {
  position: relative;
  margin-bottom: 30px;
  padding-left: 100px;
}

.timeline-emoji {
  position: absolute;
  left: 30px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #fff;
  box-shadow: 0 0 0 4px #ff6b6b;
  text-align: center;
  line-height: 40px;
  font-size: 20px;
  z-index: 1;
}

.timeline-content {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

.timeline-content:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.timeline-content h3 {
  margin-top: 0;
  color: #ff6b6b;
}

.timeline-date {
  font-size: 14px;
  color: #888;
  margin-bottom: 10px;
}

/* 响应式设计 */
@media (max-width: 600px) {
  .timeline::before {
    left: 30px;
  }
  
  .timeline-emoji {
    left: 10px;
  }
  
  .timeline-item {
    padding-left: 60px;
  }
}
</style>