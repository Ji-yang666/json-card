# JSON 卡片生成器 (JSON Card Generator)

一个基于 Vue 3 和 Vite 构建的现代化 JSON 可视化卡片生成工具。它可以帮助您轻松地将 JSON 数据转换为美观、可定制的卡片，并支持导出为图片。

## ✨ 功能特性

*   **实时 JSON 可视化**: 将输入的 JSON 数据实时渲染为结构清晰的卡片。
*   **语法高亮**: 自动对 JSON 键、字符串、数字、布尔值和 null 进行语法高亮，提高可读性。
*   **macOS 风格窗口**: 精致的 macOS 窗口样式，包含窗口控制按钮和可定制的标题。
*   **自定义标题**: 可以为生成的卡片设置自定义标题。
*   **显示 `.json` 后缀**: 可选是否在卡片标题后显示 `.json` 文件扩展名。
*   **字体大小调整**: 允许用户动态调整 JSON 内容的字体大小。
*   **图片导出**: 支持将生成的 JSON 卡片导出为 PNG 图片，方便分享和展示。
*   **复制 JSON**: 一键复制原始 JSON 数据到剪贴板。
*   **响应式设计**: 界面在不同设备尺寸下均有良好表现。
*   **预设模板**: 包含一些预设的 JSON 数据模板，例如个人名片。
*   **支持 `null` 类型**: 正确处理和显示 JSON 中的 `null` 值。
*   **数组类型配置**: 支持为数组类型的字段配置其元素的具体类型（例如，字符串数组、数字数组等）。

## 🛠️ 技术栈

*   **Vue 3**: 用于构建用户界面的渐进式 JavaScript 框架。
*   **Vite**: 下一代前端构建工具，提供极速的冷启动和模块热更新。
*   **JavaScript (ES6+)**: 主要编程语言。
*   **HTML5 & CSS3**: 用于结构和样式。
*   **html2canvas**: 用于将 DOM 元素转换为 Canvas，实现图片导出功能。

## 🚀 快速开始

### 环境要求

*   Node.js (建议版本 16.x 或更高)
*   npm 或 yarn

### 安装与运行

1.  **克隆仓库**:
    ```bash
    git clone <your-repository-url>
    cd json-card
    ```

2.  **安装依赖**:
    ```bash
    npm install
    # 或者
    # yarn install
    ```

3.  **启动开发服务器**:
    ```bash
    npm run dev
    # 或者
    # yarn dev
    ```
    应用将在本地启动，通常访问 `http://localhost:5173` (具体端口请查看终端输出)。

4.  **构建生产版本**:
    ```bash
    npm run build
    # 或者
    # yarn build
    ```
    构建后的文件将输出到 `dist` 目录。

## 📝 使用说明

1.  **打开应用**: 在浏览器中访问本地开发服务器地址或部署后的应用地址。
2.  **输入 JSON 数据**:
    *   在左侧的 "JSON 数据生成器" 区域，您可以手动添加、编辑或删除键值对。
    *   选择每个值的类型（字符串、数字、布尔、null、数组、对象）。
    *   对于数组类型，可以选择数组内元素的类型。
    *   也可以直接使用预设的模板。
3.  **自定义卡片**:
    *   **卡片标题**: 在 "卡片配置" 部分修改标题文本。
    *   **显示 .json 后缀**: 勾选或取消勾选 "显示 .json 后缀" 复选框。
4.  **调整显示**:
    *   **字体大小**: 在 JSON 卡片下方的控制面板中，拖动滑块调整代码区域的字体大小。
5.  **操作**:
    *   **复制 JSON**: 点击 "复制" 按钮，将当前卡片显示的 JSON 数据复制到剪贴板。
    *   **导出图片**: 点击 "导出" 按钮，将当前 JSON 卡片保存为 PNG 图片。

## 🎨 样式与定制

*   项目使用了现代化的 CSS 样式，包括 Flexbox 和 Grid 布局。
*   颜色主题和组件样式定义在 `.vue` 文件中的 `<style>` 部分。
*   您可以根据自己的喜好修改 `src/components/JsonCard.vue` 和 `src/components/JsonCardGenerator.vue` 中的样式。

## 📸 截图 (可选)

*(在此处可以添加应用的截图，展示其界面和功能)*

*   主界面截图
*   导出图片示例

## 🤝 贡献

欢迎提交 Pull Request 或 Issue 来改进此项目。

## 📄 开源许可

(如果您的项目有开源许可，请在此处说明，例如 MIT License)

---

希望这个 README 对您有所帮助！
