<template>
  <div class="gallery-container" :style="{ background: backgroundColor }">
    <div class="gallery" ref="gallery">
      <div class="upload-section">
        <div class="header-controls" @mouseenter="showTooltip = true" @mouseleave="showTooltip = false">
          <div class="control-group">
            <label for="color-picker">Фон:</label>
            <input 
              id="color-picker"
              v-model="backgroundColor" 
              type="color"
              class="color-picker"
            />
          </div>
          <input 
            ref="fileInput" 
            type="file" 
            multiple 
            accept="image/*"
            @change="handleFileUpload"
            style="display: none"
          />
          <button @click="triggerFileInput" class="upload-btn">Завантажити файли</button>
          <div class="instructions" :class="{ 'visible': showTooltip }">
            <div class="instruction-item"><kbd>S</kbd> - Почати прокрутку</div>
            <div class="instruction-item"><kbd>F</kbd> - Зупинити прокрутку</div>
            <div class="instruction-item"><kbd>↑</kbd> - Збільшити швидкість</div>
            <div class="instruction-item"><kbd>↓</kbd> - Зменшити швидкість</div>
            <div class="instruction-item"><kbd>W</kbd> - Збільшити ширину</div>
            <div class="instruction-item"><kbd>Q</kbd> - Зменшити ширину</div>
            <div class="instruction-item"><kbd>R</kbd> - Видалити картинку (при ховері)</div>
          </div>
        </div>
      </div>
      <img 
        v-for="(image, index) in images" 
        :key="index" 
        :src="image" 
        alt="Gallery image" 
        :style="{ width: imageWidth + '%' }" 
        @mouseenter="hoveredImageIndex = index"
        @mouseleave="hoveredImageIndex = null"
        :class="{ 'hovered': hoveredImageIndex === index }"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const gallery = ref(null);
const fileInput = ref(null);
const images = ref([]);
const isScrolling = ref(false);
let scrollAnimationId = null;
const speed = ref(0.5);
const imageWidth = ref(100);
const backgroundColor = ref('#f0f0f0');
const hoveredImageIndex = ref(null);
const showTooltip = ref(false);

onMounted(async () => {
  window.addEventListener('keydown', handleKeyPress);
});

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyPress);
  if (scrollAnimationId !== null) {
    cancelAnimationFrame(scrollAnimationId);
  }
});

const triggerFileInput = () => {
  fileInput.value.click();
};

const handleFileUpload = async (event) => {
  const files = Array.from(event.target.files);

  const readFiles = files.map(file => {
    return new Promise((resolve) => {
      const reader = new FileReader();
      reader.onload = (e) => resolve(e.target.result);
      reader.readAsDataURL(file);
    });
  });

  const results = await Promise.all(readFiles);
  images.value.push(...results);
  
  fileInput.value.value = '';
};

const handleKeyPress = (event) => {
  const key = event.key.toLowerCase();
  
  if (key === 's' && !isScrolling.value) {
    startAutoScroll();
  } else if (key === 'f' && isScrolling.value) {
    stopAutoScroll();
  } else if (event.key === 'ArrowUp') {
    event.preventDefault();
    speed.value = Math.min(speed.value + 0.5, 20);
  } else if (event.key === 'ArrowDown') {
    event.preventDefault();
    speed.value = Math.max(speed.value - 0.5, 0.5);
  } else if (key === 'w') {
    event.preventDefault();
    imageWidth.value = Math.min(imageWidth.value + 5, 200);
  } else if (key === 'q') {
    event.preventDefault();
    imageWidth.value = Math.max(imageWidth.value - 5, 20);
  } else if (key === 'r' && hoveredImageIndex.value !== null) {
    event.preventDefault();
    images.value.splice(hoveredImageIndex.value, 1);
    hoveredImageIndex.value = null;
  }
};

const startAutoScroll = () => {
  isScrolling.value = true;
  
  const scroll = () => {
    if (gallery.value && isScrolling.value) {
      gallery.value.scrollBy(0, speed.value);
      
      if (gallery.value.scrollTop >= gallery.value.scrollHeight - gallery.value.clientHeight) {
        gallery.value.scrollTop = 0;
      }
      
      scrollAnimationId = requestAnimationFrame(scroll);
    }
  };
  
  scrollAnimationId = requestAnimationFrame(scroll);
};

const stopAutoScroll = () => {
  isScrolling.value = false;
  if (scrollAnimationId !== null) {
    cancelAnimationFrame(scrollAnimationId);
    scrollAnimationId = null;
  }
};
</script>

<style scoped>
.gallery-container {
  height: 100vh;
  max-height: 1080px;
  display: flex;
  flex-direction: column;
  padding-left: 8px;
  background: #f0f0f0;
}

.upload-section {
  padding: 15px;
  background: #fff;
  border-bottom: 2px solid #ddd;
  text-align: center;
  margin-bottom: 5px;
}

.header-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  position: relative;
}

.control-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

.control-group label {
  font-size: 14px;
  font-weight: bold;
  color: #333;
}

.color-picker {
  width: 50px;
  height: 40px;
  border: 2px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  transition: border-color 0.3s;
}

.color-picker:hover {
  border-color: #0066cc;
}

.instructions {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border: 2px solid #ddd;
  border-top: none;
  border-radius: 0 0 8px 8px;
  padding: 15px 20px;
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  font-size: 13px;
  color: #555;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s, visibility 0.3s;
  pointer-events: none;
  z-index: 10;
}

.instructions.visible {
  opacity: 1;
  visibility: visible;
  pointer-events: auto;
}

.instruction-item {
  white-space: nowrap;
}

.instruction-item kbd {
  background: #f0f0f0;
  border: 1px solid #999;
  border-radius: 3px;
  padding: 2px 6px;
  font-family: monospace;
  font-weight: bold;
  color: #333;
  margin-right: 5px;
}

.upload-btn {
  padding: 10px 20px;
  font-size: 14px;
  font-weight: bold;
  color: white;
  background: #0066cc;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.3s;
}

.upload-btn:hover {
  background: #0052a3;
}

.upload-btn:active {
  background: #003d7a;
}

.gallery {
  flex: 1;
  overflow-y: auto;
  padding: 15px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  scrollbar-width: thin;
  padding-bottom: 500px;
}

.gallery img {
  width: 100%;
  max-width: 1920px;
  height: auto;
  border-radius: 8px;
  margin: 0 auto;
  cursor: pointer;
}
</style>