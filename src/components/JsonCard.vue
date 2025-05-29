<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  data: {
    type: Object,
    default: () => ({})
  },
  title: {
    type: String,
    default: 'JSON Card'
  }
})

const fontSize = ref(16);

const hasData = computed(() => {
  return Object.keys(props.data).length > 0
})

const jsonString = computed(() => {
  return JSON.stringify(props.data, null, 2)
})

const jsonLines = computed(() => {
  if (!hasData.value) return []
  return jsonString.value.split('\n')
})

const highlightedJsonLines = computed(() => {
  if (!hasData.value) return [];
  
  return jsonLines.value.map(line => {
    // 首先转义HTML特殊字符
    let escapedLine = line
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#39;');
    
    // 创建一个新的字符串用于高亮处理
    let highlightedLine = escapedLine;
    
    // 处理键名 - 匹配引号中的键名后跟冒号
    highlightedLine = highlightedLine.replace(/(&quot;[\w-]+&quot;)(\s*:)/g, '<span class="json-key">$1</span>$2');
    
    // 处理对象中的字符串值
    highlightedLine = highlightedLine.replace(/:(\s*)(&quot;.*?&quot;)/g, ':$1<span class="json-string">$2</span>');
    
    // 处理数组中的字符串值
    highlightedLine = highlightedLine.replace(/\[(\s*)(&quot;.*?&quot;)/g, '[$1<span class="json-string">$2</span>');
    highlightedLine = highlightedLine.replace(/,(\s*)(&quot;.*?&quot;)/g, ',$1<span class="json-string">$2</span>');
    
    // 处理布尔值 - true/false
    highlightedLine = highlightedLine.replace(/:(\s*)(true|false)\b/g, ':$1<span class="json-boolean">$2</span>');
    
    // 处理null值
    highlightedLine = highlightedLine.replace(/:(\s*)(null)\b/g, ':$1<span class="json-null">$2</span>');
    
    // 处理数字
    highlightedLine = highlightedLine.replace(/:(\s*)(\d+\.?\d*|\.\d+)/g, ':$1<span class="json-number">$2</span>');
    
    // 处理括号和方括号
    highlightedLine = highlightedLine.replace(/([\{\}\[\]])/g, '<span class="json-punctuation">$1</span>');
    
    return highlightedLine;
  });
});

const exportAsImage = async () => {
  try {
    // 动态导入html2canvas库
    const html2canvas = (await import('html2canvas')).default;
    
    // 获取卡片元素
    const card = document.querySelector('.json-card');
    if (!card) {
      console.error('Card element not found');
      return;
    }
    
    // 创建canvas
    const canvas = await html2canvas(card, {
      backgroundColor: '#282a36', // 确保背景色与卡片一致
      scale: 2, // 提高导出图片的清晰度
      logging: false,
      useCORS: true
    });
    
    // 转换为图片并下载
    const imgData = canvas.toDataURL('image/png');
    const a = document.createElement('a');
    a.href = imgData;
    a.download = `${props.title}.png`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  } catch (error) {
    console.error('Failed to export image:', error);
    alert('导出图片失败，请确保已安装html2canvas库。');
  }
}

const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(jsonString.value)
    // Add user feedback here, e.g., a toast notification
    alert('JSON copied to clipboard!');
  } catch (err) {
    console.error('Failed to copy JSON:', err)
    alert('Failed to copy JSON.');
  }
}
</script>

<template>
  <div class="json-card-container">
    <div class="json-card">
      <div class="card-header-controls">
        <div class="control-btn red"></div>
        <div class="control-btn yellow"></div>
        <div class="control-btn green"></div>
        <div class="card-title-text">{{ title }}</div>
         <div class="card-actions">
          <button @click="copyToClipboard" class="action-btn copy-btn" title="复制JSON">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path></svg>
          </button>
          <button @click="exportAsImage" class="action-btn export-btn" title="保存为图片">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline></svg>
          </button>
          <input type="range" min="12" max="24" v-model="fontSize" class="font-size-slider" title="调整字体大小">
        </div>
      </div>

      <div class="json-code-section" v-if="hasData">
        <div class="line-numbers">
          <div v-for="(_, index) in jsonLines" :key="index" class="line-number">
            {{ index + 1 }}
          </div>
        </div>
        <pre class="json-code" :style="{ fontSize: fontSize + 'px' }"><code v-for="(line, index) in highlightedJsonLines" :key="index" v-html="line"></code></pre>
      </div>

      <div class="empty-state" v-else>
        <p>No data to display.</p>
        <p class="empty-hint">Add key-value pairs in the generator to see the JSON card.</p>
      </div>
    </div>
  </div>
</template>

<style>
.json-card-container {
  width: 100%;
  max-width: 600px; /* 最大宽度 */
  margin: 20px auto;
  /* 使用aspect-ratio保持名片比例 */
  aspect-ratio: 1.618 / 1; /* 黄金比例 */
  display: flex;
  align-items: center;
  justify-content: center;
}

.json-card {
  background: #282a36; /* Dracula主题风格背景色 */
  border-radius: 8px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
  color: #f8f8f2; /* 亮色文本 */
  font-family: 'SF Mono', 'Monaco', 'Inconsolata', 'Roboto Mono', monospace;
  /* 填充容器但保持比例 */
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  /* 确保内容不会溢出 */
  max-height: 100%;
}

.card-header-controls {
  display: flex;
  align-items: center;
  padding: 10px 15px;
  background: #1e1f29; /* 标题栏背景色 */
  border-bottom: 1px solid #191a21;
  /* 确保标题栏不会伸缩 */
  flex-shrink: 0;
}

.control-btn {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  margin-right: 6px;
}
.control-btn.red { background-color: #ff5f57; }
.control-btn.yellow { background-color: #febc2e; }
.control-btn.green { background-color: #28c840; }

.card-title-text {
  font-weight: 500;
  color: #e6e6e6;
  /* 自适应字体大小 */
  font-size: clamp(0.7rem, 1.2vw, 0.9rem);
  margin-left: 5px; /* 控制按钮后的间距 */
  /* 防止标题过长 */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 60%;
}

.card-actions {
  margin-left: auto; /* 将操作按钮推到右侧 */
  display: flex;
  gap: 8px;
}

.action-btn {
  background: transparent;
  border: none;
  color: #888e99; /* 图标颜色 */
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  /* 确保按钮在小屏幕上不会太大 */
  min-width: 24px;
  min-height: 24px;
  /* 自适应大小 */
  width: clamp(24px, 2vw, 32px);
  height: clamp(24px, 2vw, 32px);
}

.action-btn:hover {
  color: #c8cdd4;
  background: #3a3f4a;
}

.action-btn svg {
  /* 自适应图标大小 */
  width: clamp(12px, 1.5vw, 16px);
  height: clamp(12px, 1.5vw, 16px);
}

.json-code-section {
  display: flex;
  padding: 15px 0px 15px 15px; /* 左侧padding，右侧由pre处理 */
  background: #282a36; /* 与卡片背景色保持一致 */
  overflow-y: auto;
  flex: 1; /* 填充剩余空间 */
  /* 自动调整字体大小 */
  font-size: clamp(0.6rem, 1.5vw, 0.85rem); /* 根据视口宽度自动调整字体大小，但有最小和最大限制 */
}

.line-numbers {
  padding-right: 15px; /* 行号与代码之间的间距 */
  text-align: right;
  color: #6272a4; /* Dracula主题的注释色用于行号 */
  user-select: none; /* 防止选择行号 */
  line-height: 1.6;
  /* 不单独设置字体大小，继承父元素的自适应字体大小 */
  min-width: 2em; /* 确保行号有足够的空间 */
}

.line-number {
  display: block;
}

.json-code {
  margin: 0;
  padding: 0 15px 0 0; /* 右侧为滚动条预留空间 */
  line-height: 1.6;
  white-space: pre;
  flex-grow: 1;
  overflow-x: auto; /* 允许长行水平滚动 */
  /* 不单独设置字体大小，继承父元素的自适应字体大小 */
}

.json-code code {
  display: block; /* Each line of code as a block */
  min-height: 1.6em; /* Ensure empty lines also take up space */
}

/* Syntax Highlighting Styles */
.json-key {
  color: #ff79c6; /* 亮粉色 for keys，更醒目 */
}
.json-string {
  color: #50fa7b; /* 亮绿色 for strings，更鲜明 */
}
.json-number {
  color: #f1fa8c; /* 亮黄色 for numbers，更易辨识 */
}
.json-boolean {
  color: #bd93f9; /* 亮紫色 for booleans，更突出 */
  font-weight: bold; /* 加粗布尔值 */
}
.json-null {
  color: #8be9fd; /* 亮蓝色 for null，更明显 */
  font-style: italic; /* 斜体null值 */
}
.json-punctuation {
  color: #f8f8f2; /* 亮白色 for punctuation，更清晰 */
}

.empty-state {
  padding: 20px;
  text-align: center;
  color: #626a7f;
  /* 自适应字体大小 */
  font-size: clamp(0.7rem, 1.2vw, 0.9rem);
  /* 填充剩余空间并居中内容 */
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.empty-hint {
  font-size: clamp(0.6rem, 1vw, 0.8rem);
  opacity: 0.7;
  margin-top: 10px;
}

/* 媒体查询，确保在不同屏幕尺寸下的响应式布局 */
@media (max-width: 768px) {
  .json-card-container {
    aspect-ratio: 1.2 / 1; /* 在小屏幕上使用更紧凑的比例 */
  }
  
  .json-code-section {
    padding: 10px 0px 10px 10px; /* 减小内边距 */
  }
  
  .line-numbers {
    padding-right: 10px;
    min-width: 1.5em;
  }
}

@media (max-width: 480px) {
  .json-card-container {
    aspect-ratio: 1 / 1; /* 在非常小的屏幕上使用正方形比例 */
  }
  
  .card-title-text {
    max-width: 50%; /* 在小屏幕上减少标题最大宽度 */
  }
}

/* Scrollbar styling for webkit browsers */
.json-code-section::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.json-code-section::-webkit-scrollbar-track {
  background: #1e1f29; /* 与标题栏背景色一致 */
  border-radius: 4px;
}

.json-code-section::-webkit-scrollbar-thumb {
  background: #44475a; /* Dracula主题的选择背景色 */
  border-radius: 4px;
}

.json-code-section::-webkit-scrollbar-thumb:hover {
  background: #6272a4; /* Dracula主题的注释色 */
}

.json-code::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.json-code::-webkit-scrollbar-track {
  background: #1e1f29; /* 与标题栏背景色一致 */
  border-radius: 4px;
}

.json-code::-webkit-scrollbar-thumb {
  background: #44475a; /* Dracula主题的选择背景色 */
  border-radius: 4px;
}

.json-code::-webkit-scrollbar-thumb:hover {
  background: #6272a4; /* Dracula主题的注释色 */
}


.json-code-section {
  display: flex;
  overflow: auto;
}
.line-numbers {
  flex-shrink: 0;
  text-align: right;
  margin-right: 10px;
  font-size: var(--font-size);
  overflow: hidden;
}
.json-code {
  flex-grow: 1;
  font-size: var(--font-size);
  overflow: hidden;
}
:root {
  --font-size: 16px;
}
</style>
