<template>
  <div class="gallery-container">
    <div class="gallery" ref="gallery">
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
      <img v-for="(image, index) in images" :key="index" :src="image" alt="Gallery image" :style="{ width: imageWidth + '%' }" />
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
const speed = ref(0.5);
const imageWidth = ref(100);

onMounted(async () => {
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
  }
};

const startAutoScroll = () => {
  isScrolling.value = true;
  
  scrollInterval.value = setInterval(() => {
    if (gallery.value) {
      gallery.value.scrollBy(0, speed.value);
      
      if (gallery.value.scrollTop >= gallery.value.scrollHeight - gallery.value.clientHeight) {
        gallery.value.scrollTop = 0;
      }
    }
  }, 6);
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
  height: 96vh;
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
</style>