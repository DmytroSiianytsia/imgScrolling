<template>
  <div class="gallery-container">
    <div class="upload-section">
      <input 
        ref="fileInput" 
        type="file" 
        multiple 
        accept="image/*"
        @change="handleFileUpload"
        style="display: none"
      />
      <button @click="triggerFileInput" class="upload-btn">Завантажити файли</button>
    </div>

    <div class="gallery" ref="gallery">
      <img v-for="(image, index) in images" :key="index" :src="image" alt="Gallery image" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const gallery = ref(null);
const fileInput = ref(null);
const images = ref([]);
const isScrolling = ref(false);
const scrollInterval = ref(null);
const speed = ref(2);

onMounted(async () => {
  // Завантажуємо картинки з папки
  const modules = import.meta.glob('./assets/images/**/*', { 
    query: '?url',
    import: 'default' 
  });
  
  const imageFiles = Object.keys(modules).filter(key => 
    /\.(jpg|jpeg|png|gif|webp|svg)$/i.test(key)
  );
  
  const loadedImages = await Promise.all(
    imageFiles.map(key => modules[key]())
  );
  
  images.value = loadedImages;
  
  // Слухач клавіш
  window.addEventListener('keydown', handleKeyPress);
});

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyPress);
  if (scrollInterval.value) {
    clearInterval(scrollInterval.value);
  }
});

const triggerFileInput = () => {
  fileInput.value.click();
};

const handleFileUpload = (event) => {
  const files = Array.from(event.target.files);
  
  files.forEach(file => {
    const reader = new FileReader();
    
    reader.onload = (e) => {
      images.value.push(e.target.result);
    };
    
    reader.readAsDataURL(file);
  });
  
  // Очищаємо input
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
  }
};

const startAutoScroll = () => {
  isScrolling.value = true;
  
  scrollInterval.value = setInterval(() => {
    if (gallery.value) {
      gallery.value.scrollBy(0, speed.value);
      
      // Циклічна прокрутка
      if (gallery.value.scrollTop >= gallery.value.scrollHeight - gallery.value.clientHeight) {
        gallery.value.scrollTop = 0;
      }
    }
  }, 30);
};

const stopAutoScroll = () => {
  isScrolling.value = false;
  if (scrollInterval.value) {
    clearInterval(scrollInterval.value);
    scrollInterval.value = null;
  }
};
</script>

<style scoped>
.gallery-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.upload-section {
  padding: 15px;
  background: #fff;
  border-bottom: 2px solid #ddd;
  text-align: center;
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
}

.gallery img {
  width: 100%;
  max-width: 1920px;
  height: auto;
  border-radius: 8px;
  margin: 0 auto;
}

.controls {
  padding: 15px;
  background: #f0f0f0;
  text-align: center;
  border-top: 1px solid #ccc;
}

.controls p {
  margin: 5px 0;
  font-size: 14px;
  color: #666;
}

.controls strong {
  color: #333;
}

.speed-display {
  font-size: 16px;
  color: #0066cc;
  margin-top: 10px;
}
</style>