<script setup lang="ts">
import { ref, onMounted, reactive } from 'vue';

// 生日祝福，各种语言
const birthdayWishes = [
  '生日快乐！', 'Happy Birthday!', 'Joyeux Anniversaire!', 
  'Alles Gute zum Geburtstag!', '¡Feliz Cumpleaños!', 
  'お誕生日おめでとう!', '생일 축하해요!', 'Buon Compleanno!',
  'С Днем Рождения!', 'Χαρούμενα γενέθλια!', 'जन्मदिन मुबारक!'
];

// 点击计数器
const clickCount = ref(0);

// 蛋糕位置
const cakePosition = reactive({
  top: 0,
  left: 0
});

// 点击蛋糕的动画状态
const isClicked = ref(false);
const wishBubbles = ref<{ 
  id: number; 
  text: string; 
  top: number; 
  left: number; 
  acceleration: number; 
  velocity: number; 
  active: boolean; 
  width?: number 
}[]>([]);
let bubbleId = 0;

// 获取蛋糕元素位置
onMounted(() => {
  const updateCakePosition = () => {
    const cakeElement = document.querySelector('.cake');
    if (cakeElement) {
      const rect = cakeElement.getBoundingClientRect();
      const containerRect = document.querySelector('.birthday-container')?.getBoundingClientRect();
      
      if (containerRect) {
        // 计算相对于容器的百分比位置
        cakePosition.top = (rect.top - containerRect.top) / containerRect.height * 100;
        cakePosition.left = (rect.left - containerRect.left + rect.width / 2) / containerRect.width * 100;
      }
    }
  };
  
  // 初始化位置
  updateCakePosition();
  
  // 窗口大小变化时更新位置
  window.addEventListener('resize', updateCakePosition);
  
  // 气泡动画 - 添加加速度效果
  const animateBubbles = () => {
    wishBubbles.value.forEach(bubble => {
      // 只有当气泡激活后才开始移动
      if (bubble.active) {
        // 更新速度（加速度效果）
        bubble.velocity += bubble.acceleration;
        
        // 更新位置（直线下落）
        bubble.top += bubble.velocity;
      }
    });
    requestAnimationFrame(animateBubbles);
  };
  
  requestAnimationFrame(animateBubbles);
});

// 点击蛋糕事件处理
const handleCakeClick = () => {
  // 增加点击计数
  clickCount.value++;
  
  isClicked.value = true;
  
  // 创建新的祝福气泡
  const randomWish = birthdayWishes[Math.floor(Math.random() * birthdayWishes.length)];
  
  // 估算气泡宽度（考虑不同语言的字符宽度差异）
  const containerWidth = document.querySelector('.birthday-container')?.getBoundingClientRect().width || 1000;
  const bubbleWidthPercent = (randomWish.length * 16) / containerWidth * 100 / 2; // 气泡宽度的一半，以百分比表示
  
  const newBubble = {
    id: bubbleId++,
    text: randomWish,
    top: cakePosition.top - 10, // 蛋糕上方位置
    left: cakePosition.left - bubbleWidthPercent + (Math.random() * 10 - 5), // 居中并添加随机偏移
    velocity: 0.05, // 初始速度很慢
    acceleration: 0.02, // 加速度
    active: false, // 初始状态为不活动（停留状态）
    width: bubbleWidthPercent * 2 // 存储气泡宽度以备后用
  };
  
  wishBubbles.value.push(newBubble);
  
  // 重置蛋糕动画状态
  setTimeout(() => {
    isClicked.value = false;
  }, 500);

  // 气泡停留1秒后开始掉落
  setTimeout(() => {
    const bubble = wishBubbles.value.find(b => b.id === newBubble.id);
    if (bubble) {
      bubble.active = true;
    }
  }, 1000);
  
  // 移除气泡（当气泡移出屏幕）
  setTimeout(() => {
    wishBubbles.value = wishBubbles.value.filter(bubble => bubble.id !== newBubble.id);
  }, 4000); // 总时间也相应延长
};
</script>

<template>
  <div class="birthday-container">
    <div class="cake-container" @click="handleCakeClick">
      <div class="cake" :class="{ 'cake-clicked': isClicked }">
        🎂
      </div>
    </div>
    
    <div v-for="bubble in wishBubbles" :key="bubble.id" 
         class="wish-bubble"
         :class="{ 'wish-bubble-active': bubble.active }"
         :style="{ top: `${bubble.top}%`, left: `${bubble.left}%` }">
      {{ bubble.text }}
    </div>
    
    <h1 class="birthday-title">生日快乐！x{{ clickCount }}</h1>
  </div>
</template>

<style scoped>
.birthday-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(to bottom, #1a1a2e, #16213e);
}

.cake-container {
  cursor: pointer;
  z-index: 1;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cake {
  font-size: 8rem;
  transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  user-select: none;
}

.cake-clicked {
  transform: scale(0.8);
}

.wish-bubble {
  position: absolute;
  padding: 8px 16px;
  background-color: rgba(255, 255, 255, 0.9);
  color: #ff6b6b;
  border-radius: 20px;
  font-weight: bold;
  font-size: 1rem;
  white-space: nowrap;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  animation: appear 0.3s ease-out;
  z-index: 0;
  transform: translateY(0);
  transform-origin: bottom center;
}

.wish-bubble-active {
  transition: transform 0.05s ease-out;
  animation: float 0.5s ease-in-out;
}

.birthday-title {
  margin-top: 2rem;
  font-size: 2.5rem;
  color: #ff8e8e;
  text-shadow: 0 0 10px rgba(255, 142, 142, 0.5);
}

@keyframes appear {
  from {
    opacity: 0;
    transform: scale(0.5) translateY(20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

@keyframes float {
  0% {
    transform: translateY(0);
  }
  10% {
    transform: translateY(-10px);
  }
  100% {
    transform: translateY(0);
  }
}
</style>

<style>
/* 添加全局样式以确保背景铺满整个屏幕 */
html, body, #app {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
</style>
