<script setup>
import { ref, watch } from 'vue'

const emit = defineEmits(['data-update', 'title-update', 'extension-update'])

const cardTitle = ref('我的JSON卡片')
const showJsonExtension = ref(false)

// 为每个键值对生成唯一ID
let nextId = 1
const keyValuePairs = ref([
  { id: nextId++, key: 'name', value: '张三', type: 'string', arrayType: 'string' },
  { id: nextId++, key: 'age', value: '25', type: 'number', arrayType: 'string' },
  { id: nextId++, key: 'city', value: '北京', type: 'string', arrayType: 'string' }
])

const addKeyValue = () => {
  keyValuePairs.value.push({ id: nextId++, key: '', value: '', type: 'string', arrayType: 'string' })
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
      } else if (pair.type === 'null') {
        value = null
      } else if (pair.type === 'array') {
        try {
          // 尝试解析为JSON
          value = JSON.parse(value)
        } catch {
          // 如果JSON解析失败，根据数组类型处理
          const items = value.split(',').map(item => item.trim()).filter(item => item)
          if (pair.arrayType === 'number') {
            value = items.map(item => Number(item) || 0)
          } else if (pair.arrayType === 'boolean') {
            value = items.map(item => item.toLowerCase() === 'true')
          } else {
            value = items // 字符串数组
          }
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

const updateExtension = () => {
  emit('extension-update', showJsonExtension.value)
}

const getPlaceholder = (type, arrayType) => {
  switch(type) {
    case 'string': return '输入文本'
    case 'number': return '输入数字'
    case 'boolean': return 'true 或 false'
    case 'null': return '将自动设置为 null（无需输入）'
    case 'array': 
      switch(arrayType) {
        case 'string': return '项目1,项目2,项目3 或 ["项目1","项目2"]'
        case 'number': return '1,2,3 或 [1,2,3]'
        case 'boolean': return 'true,false,true 或 [true,false,true]'
        default: return '项目1,项目2,项目3'
      }
    case 'object': return '{"key": "value"}'
    default: return '输入值'
  }
}

const loadPersonalCardTemplate = () => {
  cardTitle.value = '个人名片'
  keyValuePairs.value = [
    { id: nextId++, key: 'name', value: '李明', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'title', value: '前端开发工程师', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'company', value: '科技有限公司', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'phone', value: '+86 138-0000-0000', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'email', value: 'liming@example.com', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'location', value: '深圳市南山区', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'website', value: 'https://liming.dev', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'specialties', value: 'Vue.js,React,TypeScript', type: 'array', arrayType: 'string' },
    { id: nextId++, key: 'experience', value: '5', type: 'number', arrayType: 'string' },
    { id: nextId++, key: 'available', value: 'true', type: 'boolean', arrayType: 'string' }
  ]
}

const loadUserTemplate = () => {
  cardTitle.value = '用户信息卡片'
  keyValuePairs.value = [
    { id: nextId++, key: 'name', value: '张三', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'age', value: '28', type: 'number', arrayType: 'string' },
    { id: nextId++, key: 'email', value: 'zhangsan@example.com', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'city', value: '北京', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'isActive', value: 'true', type: 'boolean', arrayType: 'string' },
    { id: nextId++, key: 'skills', value: 'Vue.js,JavaScript,Python', type: 'array', arrayType: 'string' },
    { id: nextId++, key: 'avatar', value: '', type: 'null', arrayType: 'string' }
  ]
}

const loadProductTemplate = () => {
  cardTitle.value = '产品信息卡片'
  keyValuePairs.value = [
    { id: nextId++, key: 'productName', value: 'iPhone 15 Pro', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'price', value: '7999', type: 'number', arrayType: 'string' },
    { id: nextId++, key: 'category', value: '智能手机', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'inStock', value: 'true', type: 'boolean', arrayType: 'string' },
    { id: nextId++, key: 'features', value: '钛金属设计,A17 Pro芯片,48MP摄像头', type: 'array', arrayType: 'string' },
    { id: nextId++, key: 'rating', value: '4.8', type: 'number', arrayType: 'string' },
    { id: nextId++, key: 'scores', value: '95,88,92,90', type: 'array', arrayType: 'number' }
  ]
}

const loadConfigTemplate = () => {
  cardTitle.value = '系统配置'
  keyValuePairs.value = [
    { id: nextId++, key: 'appName', value: 'MyApp', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'version', value: '2.1.0', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'debugMode', value: 'false', type: 'boolean', arrayType: 'string' },
    { id: nextId++, key: 'maxUsers', value: '1000', type: 'number', arrayType: 'string' },
    { id: nextId++, key: 'supportedLangs', value: 'zh-CN,en-US,ja-JP', type: 'array', arrayType: 'string' },
    { id: nextId++, key: 'ports', value: '8080,3000,5000', type: 'array', arrayType: 'number' },
    { id: nextId++, key: 'secretKey', value: '', type: 'null', arrayType: 'string' }
  ]
}

const loadProjectTemplate = () => {
  cardTitle.value = '项目信息卡片'
  keyValuePairs.value = [
    { id: nextId++, key: 'projectName', value: 'Vue3应用', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'version', value: '1.0.0', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'isPublic', value: 'true', type: 'boolean', arrayType: 'string' },
    { id: nextId++, key: 'author', value: '张三', type: 'string', arrayType: 'string' },
    { id: nextId++, key: 'dependencies', value: '{"vue":"^3.2.0","vite":"^2.7.0","pinia":"^2.0.0"}', type: 'object', arrayType: 'string' },
    { id: nextId++, key: 'tags', value: 'Vue,JavaScript,前端', type: 'array', arrayType: 'string' },
    { id: nextId++, key: 'stats', value: '{"stars":120,"forks":35,"issues":5}', type: 'object', arrayType: 'string' },
    { id: nextId++, key: 'testResults', value: 'true,false,true,true', type: 'array', arrayType: 'boolean' }
  ]
}

watch([keyValuePairs, cardTitle], () => {
  updateData()
  updateTitle()
}, { deep: true })

watch(showJsonExtension, () => {
  updateExtension()
})

// 监听类型变化，为新添加的数组类型设置默认值
watch(keyValuePairs, (newPairs) => {
  newPairs.forEach(pair => {
    if (pair.type === 'array' && !pair.arrayType) {
      pair.arrayType = 'string'
    }
  })
}, { deep: true })

// 初始化数据
updateData()
updateTitle()
updateExtension()
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

      <!-- JSON扩展名配置 -->
      <div class="form-group">
        <label class="checkbox-wrapper">
          <input 
            v-model="showJsonExtension"
            type="checkbox" 
            class="checkbox-input"
          />
          <span class="checkbox-custom"></span>
          <span class="checkbox-label">在标题末尾显示 .json</span>
        </label>
      </div>

      <!-- 键值对列表 -->
      <div class="form-group">
        <label>数据字段</label>
        <div class="key-value-list">
          <div 
            v-for="(pair, index) in keyValuePairs" 
            :key="pair.id"
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
              <option value="null">null</option>
            </select>
            
            <!-- 数组类型选择器占位符 -->
            <div class="array-type-container">
              <select 
                v-if="pair.type === 'array'" 
                v-model="pair.arrayType" 
                class="array-type-select"
              >
                <option value="string">字符串数组</option>
                <option value="number">数字数组</option>
                <option value="boolean">布尔数组</option>
              </select>
            </div>
            
            <input 
              v-model="pair.value"
              type="text" 
              class="input-field value-input"
              :placeholder="getPlaceholder(pair.type, pair.arrayType)"
              :disabled="pair.type === 'null'"
              :class="{ 'disabled': pair.type === 'null' }"
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
          <button @click="loadPersonalCardTemplate" class="template-btn">个人名片</button>
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

.form-group label.checkbox-wrapper {
  display: inline-flex;
  margin-bottom: 0;
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

.input-field.disabled {
  background: #f5f5f5;
  color: #999;
  cursor: not-allowed;
  border-color: #ddd;
}

.input-field.disabled:focus {
  border-color: #ddd;
  box-shadow: none;
}

.key-value-list {
  margin-bottom: 15px;
}

.key-value-item {
  display: grid;
  grid-template-columns: 120px 80px auto 1fr 32px;
  gap: 8px;
  align-items: center;
  margin-bottom: 12px;
}

.array-type-container {
  width: 100px;
  display: flex;
  align-items: center;
}

.array-type-select {
  width: 100px;
  padding: 8px 12px;
  border: 2px solid #e1e5e9;
  border-radius: 6px;
  font-size: 12px;
  background: white;
  color: #666;
  box-sizing: border-box;
}

.key-input {
  width: 100%;
}

.type-select {
  padding: 8px 12px;
  border: 2px solid #e1e5e9;
  border-radius: 6px;
  font-size: 12px;
  background: white;
  width: 100%;
  box-sizing: border-box;
}

.value-input {
  width: 100%;
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

.checkbox-wrapper {
  display: inline-flex;
  align-items: center;
  cursor: pointer;
  gap: 8px;
  font-size: 14px;
  position: relative;
  margin: 0;
}

.checkbox-input {
  margin: 0;
  width: 16px;
  height: 16px;
  cursor: pointer;
  opacity: 0;
  position: absolute;
  z-index: 2;
}

.checkbox-custom {
  width: 16px;
  height: 16px;
  border: 2px solid #ddd;
  border-radius: 50%;
  background: white;
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  flex-shrink: 0;
}

.checkbox-custom::after {
  content: '';
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: white;
  opacity: 0;
  transition: all 0.2s ease;
}

.checkbox-input:checked + .checkbox-custom {
  background: #007aff;
  border-color: #007aff;
}

.checkbox-input:checked + .checkbox-custom::after {
  opacity: 1;
}

.checkbox-wrapper:hover .checkbox-custom {
  border-color: #007aff;
}

.checkbox-label {
  color: #555;
  font-weight: 500;
  user-select: none;
  line-height: 1.4;
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
  
  .key-input,
  .type-select,
  .array-type-select,
  .value-input {
    width: 100%;
  }
  
  .array-type-container {
    width: 100%;
  }
  
  .template-buttons {
    flex-direction: column;
  }
  
  .template-btn {
    text-align: center;
  }
}
</style>
