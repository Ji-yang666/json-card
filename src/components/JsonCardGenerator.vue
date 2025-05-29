<script setup>
import { ref, watch } from 'vue'

const emit = defineEmits(['data-update', 'title-update'])

const cardTitle = ref('我的JSON卡片')
const keyValuePairs = ref([
  { key: 'name', value: '张三', type: 'string' },
  { key: 'age', value: '25', type: 'number' },
  { key: 'city', value: '北京', type: 'string' }
])

const addKeyValue = () => {
  keyValuePairs.value.push({ key: '', value: '', type: 'string' })
}

const removeKeyValue = (index) => {
  keyValuePairs.value.splice(index, 1)
  updateData()
}

const updateData = () => {
  const data = {}
  keyValuePairs.value.forEach(pair => {
    if (pair.key.trim()) {
      let value = pair.value
      if (pair.type === 'number') {
        value = Number(value) || 0
      } else if (pair.type === 'boolean') {
        value = value.toLowerCase() === 'true'
      } else if (pair.type === 'array') {
        try {
          value = JSON.parse(value)
        } catch {
          value = value.split(',').map(item => item.trim())
        }
      } else if (pair.type === 'object') {
        try {
          value = JSON.parse(value)
        } catch {
          value = value
        }
      }
      data[pair.key] = value
    }
  })
  emit('data-update', data)
}

const updateTitle = () => {
  emit('title-update', cardTitle.value)
}

const getPlaceholder = (type) => {
  switch(type) {
    case 'string': return '输入文本'
    case 'number': return '输入数字'
    case 'boolean': return 'true 或 false'
    case 'array': return '["item1", "item2"] 或 item1,item2'
    case 'object': return '{"key": "value"}'
    default: return '输入值'
  }
}

const loadUserTemplate = () => {
  cardTitle.value = '用户信息卡片'
  keyValuePairs.value = [
    { key: 'name', value: '张三', type: 'string' },
    { key: 'age', value: '28', type: 'number' },
    { key: 'email', value: 'zhangsan@example.com', type: 'string' },
    { key: 'city', value: '北京', type: 'string' },
    { key: 'isActive', value: 'true', type: 'boolean' },
    { key: 'skills', value: 'Vue.js,JavaScript,Python', type: 'array' }
  ]
}

const loadProductTemplate = () => {
  cardTitle.value = '产品信息卡片'
  keyValuePairs.value = [
    { key: 'productName', value: 'iPhone 15 Pro', type: 'string' },
    { key: 'price', value: '7999', type: 'number' },
    { key: 'category', value: '智能手机', type: 'string' },
    { key: 'inStock', value: 'true', type: 'boolean' },
    { key: 'features', value: '钛金属设计,A17 Pro芯片,48MP摄像头', type: 'array' },
    { key: 'rating', value: '4.8', type: 'number' }
  ]
}

const loadConfigTemplate = () => {
  cardTitle.value = '系统配置'
  keyValuePairs.value = [
    { key: 'appName', value: 'MyApp', type: 'string' },
    { key: 'version', value: '2.1.0', type: 'string' },
    { key: 'debugMode', value: 'false', type: 'boolean' },
    { key: 'maxUsers', value: '1000', type: 'number' },
    { key: 'supportedLangs', value: 'zh-CN,en-US,ja-JP', type: 'array' }
  ]
}

const loadProjectTemplate = () => {
  cardTitle.value = '项目信息卡片'
  keyValuePairs.value = [
    { key: 'projectName', value: 'Vue3应用', type: 'string' },
    { key: 'version', value: '1.0.0', type: 'string' },
    { key: 'isPublic', value: 'true', type: 'boolean' },
    { key: 'author', value: '张三', type: 'string' },
    { key: 'dependencies', value: '{"vue":"^3.2.0","vite":"^2.7.0","pinia":"^2.0.0"}', type: 'object' },
    { key: 'tags', value: 'Vue,JavaScript,前端', type: 'array' },
    { key: 'stats', value: '{"stars":120,"forks":35,"issues":5}', type: 'object' }
  ]
}

watch([keyValuePairs, cardTitle], () => {
  updateData()
  updateTitle()
}, { deep: true })

// 初始化数据
updateData()
updateTitle()
</script>

<template>
  <div class="generator">
    <div class="generator-card">
      <h2>配置卡片</h2>
      
      <!-- 卡片标题 -->
      <div class="form-group">
        <label>卡片标题</label>
        <input 
          v-model="cardTitle"
          type="text" 
          class="input-field"
          placeholder="输入卡片标题"
        />
      </div>

      <!-- 键值对列表 -->
      <div class="form-group">
        <label>数据字段</label>
        <div class="key-value-list">
          <div 
            v-for="(pair, index) in keyValuePairs" 
            :key="index"
            class="key-value-item"
          >
            <input 
              v-model="pair.key"
              type="text" 
              class="input-field key-input"
              placeholder="键名"
            />
            
            <select v-model="pair.type" class="type-select">
              <option value="string">文本</option>
              <option value="number">数字</option>
              <option value="boolean">布尔值</option>
              <option value="array">数组</option>
              <option value="object">对象</option>
            </select>
            
            <input 
              v-model="pair.value"
              type="text" 
              class="input-field value-input"
              :placeholder="getPlaceholder(pair.type)"
            />
            
            <button 
              @click="removeKeyValue(index)"
              class="remove-btn"
              type="button"
            >
              ✕
            </button>
          </div>
        </div>
        
        <button @click="addKeyValue" class="add-btn">
          + 添加字段
        </button>
      </div>

      <!-- 预设模板 -->
      <div class="form-group">
        <label>快速模板</label>
        <div class="template-buttons">
          <button @click="loadUserTemplate" class="template-btn">用户信息</button>
          <button @click="loadProductTemplate" class="template-btn">产品信息</button>
          <button @click="loadConfigTemplate" class="template-btn">配置信息</button>
          <button @click="loadProjectTemplate" class="template-btn">项目信息</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.generator {
  width: 100%;
}

.generator-card {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 16px;
  padding: 30px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.generator-card h2 {
  margin: 0 0 25px 0;
  color: #333;
  font-size: 1.5rem;
  font-weight: 600;
}

.form-group {
  margin-bottom: 25px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: #555;
  font-weight: 500;
  font-size: 0.9rem;
}

.input-field {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e1e5e9;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.2s ease;
  background: white;
  box-sizing: border-box;
}

.input-field:focus {
  outline: none;
  border-color: #007aff;
  box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.1);
}

.key-value-list {
  margin-bottom: 15px;
}

.key-value-item {
  display: grid;
  grid-template-columns: 1fr auto 2fr auto;
  gap: 8px;
  align-items: center;
  margin-bottom: 12px;
}

.key-input {
  min-width: 120px;
}

.type-select {
  padding: 8px 12px;
  border: 2px solid #e1e5e9;
  border-radius: 6px;
  font-size: 12px;
  background: white;
  min-width: 80px;
}

.value-input {
  min-width: 150px;
}

.remove-btn {
  width: 32px;
  height: 32px;
  border: none;
  background: #ff3b30;
  color: white;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  transition: all 0.2s ease;
}

.remove-btn:hover {
  background: #d70015;
  transform: scale(1.05);
}

.add-btn {
  width: 100%;
  padding: 12px;
  border: 2px dashed #007aff;
  background: transparent;
  color: #007aff;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.add-btn:hover {
  background: rgba(0, 122, 255, 0.05);
  border-style: solid;
}

.template-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.template-btn {
  padding: 8px 16px;
  border: 1px solid #007aff;
  background: white;
  color: #007aff;
  border-radius: 20px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.template-btn:hover {
  background: #007aff;
  color: white;
  transform: translateY(-1px);
}

@media (max-width: 768px) {
  .key-value-item {
    grid-template-columns: 1fr;
    gap: 8px;
  }
  
  .template-buttons {
    flex-direction: column;
  }
  
  .template-btn {
    text-align: center;
  }
}
</style>
