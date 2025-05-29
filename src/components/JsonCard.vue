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
  },
  showJsonExtension: {
    type: Boolean,
    default: false
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

const highlightJson = (jsonStr) => {
  // 使用JSON.parse和递归遍历来确保准确的语法高亮
  try {
    const obj = JSON.parse(jsonStr);
    const formatted = JSON.stringify(obj, null, 2);
    
    return formatted
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .split('\n')
      .map(line => {
        let result = line.replace(/"/g, '&quot;');
        
        // 处理键名（紧跟冒号的引号字符串）- 必须在其他字符串处理之前
        result = result.replace(/(&quot;[^&]*&quot;)(\s*:)/g, '<span class="json-key">$1</span>$2');
        
        // 处理字符串值（不是键名的引号字符串）
        // 匹配冒号后、方括号后、逗号后的字符串
        result = result.replace(/([:[\s,]\s*)(&quot;[^&]*&quot;)/g, '$1<span class="json-string">$2</span>');
        
        // 处理数字（仅匹配未被包装在span中的数字）
        result = result.replace(/([:[\s,]\s*)(-?\d+(?:\.\d+)?(?:[eE][+-]?\d+)?)(?=\s*[,\}\]\s])/g, (match, prefix, number) => {
          // 确保不在已有的span标签内
          if (prefix.includes('<span')) return match;
          return `${prefix}<span class="json-number">${number}</span>`;
        });
        
        // 处理布尔值（仅匹配未被包装在span中的布尔值）
        result = result.replace(/([:[\s,]\s*)(true|false)(?=\s*[,\}\]\s])/g, (match, prefix, bool) => {
          if (prefix.includes('<span')) return match;
          return `${prefix}<span class="json-boolean">${bool}</span>`;
        });
        
        // 处理null值（仅匹配未被包装在span中的null值）
        result = result.replace(/([:[\s,]\s*)(null)(?=\s*[,\}\]\s])/g, (match, prefix, nullVal) => {
          if (prefix.includes('<span')) return match;
          return `${prefix}<span class="json-null">${nullVal}</span>`;
        });
        
        // 处理标点符号（避免影响已有的span标签）
        result = result.replace(/([\{\}\[\],:]+)(?![^<]*>)/g, '<span class="json-punctuation">$1</span>');
        
        return result;
      });
  } catch (error) {
    // 如果JSON解析失败，使用原有方法
    return jsonStr.split('\n').map(line => {
      let result = line
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;');
        
      // 简单的fallback高亮
      result = result.replace(/(&quot;[^&]*&quot;)(\s*:)/g, '<span class="json-key">$1</span>$2');
      result = result.replace(/:\s*(&quot;[^&]*&quot;)/g, ': <span class="json-string">$1</span>');
      result = result.replace(/:\s*(\d+)/g, ': <span class="json-number">$1</span>');
      result = result.replace(/:\s*(true|false)/g, ': <span class="json-boolean">$1</span>');
      result = result.replace(/:\s*(null)/g, ': <span class="json-null">$1</span>');
      result = result.replace(/([\{\}\[\],:]+)/g, '<span class="json-punctuation">$1</span>');
      
      return result;
    });
  }
};

const highlightedJsonLines = computed(() => {
  if (!hasData.value) return [];
  return highlightJson(jsonString.value);
});

const exportAsImage = async () => {
  try {
    // 动态导入html2canvas库
    const html2canvas = (await import('html2canvas')).default;
    
    // 获取卡片元素
    const card = document.querySelector('#json-card-export');
    if (!card) {
      console.error('Card element not found');
      return;
    }

    console.log('Card element found:', card);
    console.log('Card dimensions:', card.offsetWidth, 'x', card.offsetHeight);
    console.log('Current font size:', fontSize.value + 'px');

    // 等待一小段时间确保DOM渲染完成
    await new Promise(resolve => setTimeout(resolve, 100));
    
    // 使用onclone回调来确保字体大小正确应用
    const canvas = await html2canvas(card, {
      backgroundColor: '#282a36',
      scale: 2, // 提高清晰度
      logging: false, // 关闭日志以避免干扰
      useCORS: true,
      allowTaint: true,
      width: card.offsetWidth,
      height: card.offsetHeight,
      scrollX: 0,
      scrollY: 0,
      windowWidth: window.innerWidth,
      windowHeight: window.innerHeight,
      onclone: (clonedDoc) => {
        console.log('Cloning document for export...');
        
        // 在克隆的文档中找到JSON代码区域并确保字体大小正确
        const clonedCodeSection = clonedDoc.querySelector('.json-code-section');
        if (clonedCodeSection) {
          // 强制设置字体大小，确保继承正确
          clonedCodeSection.style.fontSize = fontSize.value + 'px';
          clonedCodeSection.style.fontFamily = '"SF Mono", "Monaco", "Inconsolata", "Roboto Mono", monospace';
          console.log('Applied font size to cloned code section:', fontSize.value + 'px');
        }
        
        // 确保所有相关元素都应用正确的字体大小
        const clonedCodeContainer = clonedDoc.querySelector('.json-code-container');
        if (clonedCodeContainer) {
          clonedCodeContainer.style.fontSize = fontSize.value + 'px';
        }
        
        // 应用字体大小到所有代码相关元素
        const clonedCodeElements = clonedDoc.querySelectorAll('.json-code, .line-content, .line-number');
        clonedCodeElements.forEach(element => {
          element.style.fontSize = fontSize.value + 'px';
          element.style.fontFamily = '"SF Mono", "Monaco", "Inconsolata", "Roboto Mono", monospace';
        });
        
        // 移除可能影响渲染的复杂样式
        const clonedCard = clonedDoc.querySelector('#json-card-export');
        if (clonedCard) {
          // 移除backdrop-filter和复杂的阴影
          clonedCard.style.backdropFilter = 'none';
          clonedCard.style.boxShadow = '0 4px 20px rgba(0, 0, 0, 0.4)';
          clonedCard.style.background = '#282a36';
          clonedCard.style.position = 'relative';
          // 确保没有伪元素干扰
          clonedCard.style.setProperty('--before-display', 'none');
        }
        
        // 添加样式来覆盖任何可能的继承问题
        const style = clonedDoc.createElement('style');
        style.textContent = `
          /* 完全移除所有伪元素和复杂样式 */
          .json-card::before,
          .json-card::after,
          .card-header-controls::before,
          .card-header-controls::after,
          .control-btn::after,
          .control-btn::before {
            display: none !important;
            content: none !important;
            visibility: hidden !important;
            opacity: 0 !important;
          }
          
          /* 强制移除任何可能的背景渐变和复杂效果 */
          .json-card {
            background: #282a36 !important;
            backdrop-filter: none !important;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4) !important;
            position: relative !important;
          }
          
          /* 确保header样式完全统一 */
          .card-header-controls {
            background: #2d2f3a !important;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1) !important;
            position: relative !important;
            box-shadow: none !important; /* 修改此处 */
            z-index: 10 !important;
          }
          
          /* 确保字体大小一致 */
          .json-code-section,
          .json-code-container,
          .json-code,
          .line-content,
          .line-number {
            font-size: ${fontSize.value}px !important;
            font-family: "SF Mono", "Monaco", "Inconsolata", "Roboto Mono", monospace !important;
          }
          
          /* 确保控制按钮样式正确 */
          .control-btn {
            position: relative !important;
            z-index: 11 !important;
          }
          
          .control-btn.red { 
            background: linear-gradient(135deg, #ff6b5a 0%, #ff4136 100%) !important;
          }
          
          .control-btn.yellow { 
            background: linear-gradient(135deg, #ffdc3e 0%, #ffcc02 100%) !important;
          }
          
          .control-btn.green { 
            background: linear-gradient(135deg, #2ecc40 0%, #28a838 100%) !important;
          }
          
          /* 确保标题文字样式正确 */
          .card-title-text {
            color: #f8f8f2 !important;
            z-index: 11 !important;
            position: relative !important;
          }
        `;
        clonedDoc.head.appendChild(style);
        
        // 额外确保清理所有伪元素和复杂样式
        const clonedHeaderControls = clonedDoc.querySelector('.card-header-controls');
        if (clonedHeaderControls) {
          // 确保header背景正确统一
          clonedHeaderControls.style.background = '#2d2f3a';
          clonedHeaderControls.style.borderBottom = '1px solid rgba(255, 255, 255, 0.1)';
          clonedHeaderControls.style.position = 'relative';
          clonedHeaderControls.style.boxShadow = 'none'; // 修改此处
        }
        
        // 确保控制按钮样式正确
        const clonedControlBtns = clonedDoc.querySelectorAll('.control-btn');
        clonedControlBtns.forEach(btn => {
          btn.style.position = 'relative';
          if (btn.classList.contains('red')) {
            btn.style.background = 'linear-gradient(135deg, #ff6b5a 0%, #ff4136 100%)';
          } else if (btn.classList.contains('yellow')) {
            btn.style.background = 'linear-gradient(135deg, #ffdc3e 0%, #ffcc02 100%)';
          } else if (btn.classList.contains('green')) {
            btn.style.background = 'linear-gradient(135deg, #2ecc40 0%, #28a838 100%)';
          }
        });
        
        console.log('Applied styles to cloned document');
      }
    });
    
    console.log('Canvas created:', canvas.width, 'x', canvas.height);
    
    // 检查canvas是否为空
    if (canvas.width === 0 || canvas.height === 0) {
      throw new Error('Canvas has zero dimensions');
    }
    
    // 转换为图片并下载
    const imgData = canvas.toDataURL('image/png', 1.0);
    
    // 检查图片数据
    if (imgData.length < 100) {
      throw new Error('Generated image data is too small');
    }
    
    console.log('Image data length:', imgData.length);
    
    const a = document.createElement('a');
    a.href = imgData;
    a.download = `${props.title}.png`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    
    console.log('Image exported successfully with font size:', fontSize.value + 'px');
    
  } catch (error) {
    console.error('Failed to export image:', error);
    alert(`导出图片失败: ${error.message}。请尝试刷新页面重试。`);
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
    <div class="json-card" id="json-card-export">
      <div class="card-header-controls">
        <div class="control-btn red"></div>
        <div class="control-btn yellow"></div>
        <div class="control-btn green"></div>
        <div class="card-title-text">{{ title }}{{ showJsonExtension ? '.json' : '' }}</div>
      </div>

      <div class="json-code-section" v-if="hasData" :style="{ fontSize: fontSize + 'px' }">
        <pre class="json-code-container"><code class="json-code" v-for="(line, index) in highlightedJsonLines" :key="index"><span class="line-number">{{ index + 1 }}</span><span class="line-content" v-html="line"></span></code></pre>
      </div>

      <div class="empty-state" v-else>
        <p>No data to display.</p>
        <p class="empty-hint">Add key-value pairs in the generator to see the JSON card.</p>
      </div>
    </div>
    
    <!-- 控制面板 -->
    <div class="controls-panel">
      <div class="font-size-control">
        <label class="control-label">字体大小</label>
        <input type="range" min="10" max="28" v-model="fontSize" class="font-size-slider" title="调整字体大小">
        <span class="font-size-value">{{ fontSize }}px</span>
      </div>
      <div class="action-controls">
        <button @click="copyToClipboard" class="action-btn copy-btn" title="复制JSON">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2 2v1"></path></svg>
          复制
        </button>
        <button @click="exportAsImage" class="action-btn export-btn" title="保存为图片">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline></svg>
          导出
        </button>
      </div>
    </div>
  </div>
</template>

<style>
.json-card-container {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.controls-panel {
  display: flex;
  gap: 20px;
  align-items: center;
  justify-content: space-between;
  background: rgba(40, 42, 54, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 8px;
  padding: 12px 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  border: 1px solid #44475a;
}

.font-size-control {
  display: flex;
  align-items: center;
  gap: 8px;
}

.control-label {
  font-size: 12px;
  color: #6272a4;
  white-space: nowrap;
}

.font-size-value {
  font-size: 12px;
  color: #f8f8f2;
  min-width: 30px;
  text-align: center;
}

.action-controls {
  display: flex;
  gap: 8px;
}

.json-card {
  background: #282a36;
  border-radius: 12px;
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.4),
    0 0 0 1px rgba(255, 255, 255, 0.05),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  overflow: hidden;
  color: #f8f8f2;
  font-family: 'SF Mono', 'Monaco', 'Inconsolata', 'Roboto Mono', monospace;
  width: 100%;
  aspect-ratio: 1.618 / 1; /* 黄金比例 */
  display: flex;
  flex-direction: column;
  max-height: 500px; /* 设置最大高度 */
  position: relative;
  /* backdrop-filter: blur(20px); */ /* 移除此行 */
}

.json-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  /* background: linear-gradient(135deg, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0.05) 50%, rgba(0, 0, 0, 0.05) 100%); */ /* 移除此行 */
  pointer-events: none;
  border-radius: 12px;
  z-index: 1;
}

.card-header-controls {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  background: #2d2f3a;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  flex-shrink: 0;
  position: relative;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.05);
  z-index: 2;
}

.control-btn {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 8px;
  position: relative;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 
    inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 1px 2px rgba(0, 0, 0, 0.3);
}

.control-btn:hover {
  transform: scale(1.1);
  box-shadow: 
    inset 0 1px 0 rgba(255, 255, 255, 0.3),
    0 2px 4px rgba(0, 0, 0, 0.4);
}

.control-btn.red { 
  background: linear-gradient(135deg, #ff6b5a 0%, #ff4136 100%);
}

.control-btn.yellow { 
  background: linear-gradient(135deg, #ffdc3e 0%, #ffcc02 100%);
}

.control-btn.green { 
  background: linear-gradient(135deg, #2ecc40 0%, #28a838 100%);
}

.control-btn::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.2);
  opacity: 0;
  transition: opacity 0.2s ease;
}

.control-btn:hover::after {
  opacity: 1;
}

.card-title-text {
  font-weight: 600;
  color: #f8f8f2;
  font-size: clamp(0.75rem, 1.2vw, 0.95rem);
  margin-left: 8px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
  text-align: left;
  letter-spacing: 0.5px;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
}

.action-btn {
  background: rgba(98, 114, 164, 0.2);
  border: 1px solid #6272a4;
  color: #f8f8f2;
  cursor: pointer;
  padding: 8px 12px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  font-size: 12px;
  font-family: inherit;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.action-btn:hover {
  background: rgba(189, 147, 249, 0.3);
  border-color: #bd93f9;
  color: #ffffff;
  transform: translateY(-1px);
}

.action-btn svg {
  width: 14px;
  height: 14px;
  flex-shrink: 0;
}

.font-size-slider {
  width: 100px;
  height: 6px;
  background: #44475a;
  border-radius: 3px;
  outline: none;
  appearance: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.font-size-slider:hover {
  background: #6272a4;
}

.font-size-slider::-webkit-slider-thumb {
  appearance: none;
  width: 18px;
  height: 18px;
  background: #bd93f9;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 6px rgba(189, 147, 249, 0.3);
}

.font-size-slider::-webkit-slider-thumb:hover {
  background: #ff79c6;
  transform: scale(1.1);
  box-shadow: 0 3px 8px rgba(255, 121, 198, 0.4);
}

.font-size-slider::-moz-range-thumb {
  width: 18px;
  height: 18px;
  background: #bd93f9;
  border-radius: 50%;
  cursor: pointer;
  border: none;
  transition: all 0.2s ease;
  box-shadow: 0 2px 6px rgba(189, 147, 249, 0.3);
}

.font-size-slider::-moz-range-thumb:hover {
  background: #ff79c6;
  transform: scale(1.1);
}

.json-code-section {
  padding: 15px;
  background: #282a36;
  overflow: auto;
  flex: 1;
  font-family: 'SF Mono', 'Monaco', 'Inconsolata', 'Roboto Mono', monospace;
}

.json-code-container {
  margin: 0;
  padding: 0;
  line-height: 1.6;
  white-space: pre;
  background: transparent;
  border: none;
  width: 100%;
  overflow: visible;
}

.json-code {
  display: flex;
  align-items: flex-start;
  min-height: 1.6em;
  width: 100%;
}

.line-number {
  display: inline-block;
  text-align: right;
  color: #6272a4;
  user-select: none;
  padding-right: 15px;
  min-width: 3em;
  flex-shrink: 0;
}

.line-content {
  flex: 1;
  white-space: pre-wrap;
  word-break: break-word;
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
  .json-card {
    aspect-ratio: 1.2 / 1;
    max-height: 400px;
  }
  
  .controls-panel {
    flex-direction: column;
    gap: 12px;
    align-items: stretch;
  }
  
  .font-size-control {
    justify-content: space-between;
  }
  
  .action-controls {
    justify-content: center;
  }
  
  .json-code-section {
    padding: 10px;
  }
  
  .line-number {
    min-width: 2.5em;
    padding-right: 10px;
  }
}

@media (max-width: 480px) {
  .json-card {
    aspect-ratio: 1 / 1;
    max-height: 350px;
  }
  
  .controls-panel {
    padding: 10px 12px;
  }
  
  .action-btn {
    padding: 6px 8px;
    font-size: 11px;
  }
  
  .font-size-slider {
    width: 80px;
  }
  
  .card-title-text {
    max-width: 70%;
  }
  
  .line-number {
    min-width: 2em;
    padding-right: 8px;
  }
}

/* 优化滚动条样式 */
.json-code-section::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.json-code-section::-webkit-scrollbar-track {
  background: transparent;
  border-radius: 3px;
}

.json-code-section::-webkit-scrollbar-thumb {
  background: rgba(98, 114, 164, 0.5);
  border-radius: 3px;
  transition: all 0.2s ease;
}

.json-code-section::-webkit-scrollbar-thumb:hover {
  background: rgba(189, 147, 249, 0.8);
}

.json-code-section::-webkit-scrollbar-corner {
  background: transparent;
}
</style>
