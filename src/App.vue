<script setup>
import { ref, computed } from 'vue'
import JsonCardGenerator from './components/JsonCardGenerator.vue'
import JsonCard from './components/JsonCard.vue'

const jsonData = ref({})
const cardTitle = ref('JSON Card')

const handleDataUpdate = (data) => {
  jsonData.value = data
}

const handleTitleUpdate = (title) => {
  cardTitle.value = title
}
</script>

<template>
  <div class="app">
    <header class="app-header">
      <h1>macOS风格 JSON卡片生成器</h1>
      <p>输入键值对数据，生成精美的macOS风格卡片</p>
    </header>

    <main class="app-main">
      <div class="content-grid">
        <div class="generator-section">
          <JsonCardGenerator 
            @data-update="handleDataUpdate"
            @title-update="handleTitleUpdate"
          />
        </div>
        
        <div class="preview-section">
          <JsonCard 
            :data="jsonData"
            :title="cardTitle"
          />
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.app-header {
  text-align: center;
  color: white;
  margin-bottom: 40px;
}

.app-header h1 {
  font-size: 2.5rem;
  font-weight: 600;
  margin-bottom: 10px;
  text-shadow: 0 2px 4px rgba(0,0,0,0.3);
}

.app-header p {
  font-size: 1.1rem;
  opacity: 0.9;
}

.app-main {
  max-width: 1400px;
  margin: 0 auto;
}

.content-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 40px;
  align-items: start;
}

@media (max-width: 768px) {
  .content-grid {
    grid-template-columns: 1fr;
    gap: 30px;
  }
  
  .app-header h1 {
    font-size: 2rem;
  }
}
</style>
