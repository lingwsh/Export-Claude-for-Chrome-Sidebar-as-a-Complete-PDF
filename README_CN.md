# 强制无损导出 Claude for Chrome 侧边栏为完整 PDF

**这是一个稳定且半自动的解决方案，用于将 Claude for Chrome 侧边栏的对话记录完美导出为 PDF。它不仅能完整保留所有 CSS 样式，还能彻底解决浏览器打印时的“分页截断”问题。**

## 🛑 核心痛点
官方的 Claude for Chrome 扩展程序缺乏原生的导出功能。如果尝试强制打印或保存 HTML，会遇到以下三大技术阻碍：
1. **侧边栏限制 (Side Panel Restrictions):** Chrome 严格的安全沙盒机制禁止在扩展程序的侧边栏上下文中直接调用 `window.print()`。
2. **执行上下文隔离 (Execution Context Isolation):** 普通的书签脚本 (Bookmarklets) 或外部注入脚本无法穿透 `chrome-extension://` 环境。
3. **打印截断 (Print Truncation):** 插件前端使用了 `height: 100vh` 和内部滚动限制。打印机只能识别物理高度，导致多页对话在打印时被硬生生截断，只能保存下来第一页。

## 💡 解决思路
本脚本会将当前渲染好的 HTML DOM 和 CSS 样式完整克隆到一个不受限制的 `about:blank` 空白标签页中。最关键的是，它向新网页注入了暴力的 `@media print` 打印样式，解除了所有高度限制和滚动锁定，强制浏览器完美渲染并打印每一页内容。

为了避免每次都需要繁琐地复制粘贴代码，我们可以借助 **Chrome DevTools Snippets (代码片段)** 功能，实现只需两下快捷键的高效工作流。

---

## 🛠️ 一次性设置 (使用 Snippets 实现自动化)

不要每次都手动粘贴代码，请将其永久保存在 Chrome 内置的 Snippets 管理器中。

1. 在 Claude 侧边栏的空白处（避开文字输入框）**右键**，选择 **检查 (Inspect)**。
2. 在弹出的开发者工具 (DevTools) 窗口顶部，切换到 **Sources (源代码)** 标签页。
3. 在左侧面板中，点击 **Snippets (代码片段)** 标签（如果没有看到，请点击 `>>` 箭头展开隐藏的标签页）。
4. 点击 **+ New snippet (新建代码片段)**，将其命名为 `Claude-to-PDF`。
5. 将以下完整的 JavaScript 代码复制并粘贴到右侧的代码编辑器中：

```javascript
// 1. 打开一个完全不受限制的全新空白标签页
let printWindow = window.open('', '_blank');

// 2. 将侧边栏的 DOM 和 CSS 克隆进去，并注入重置打印样式的 CSS
printWindow.document.write(`
  <html>
    <head>
      <base href="${location.origin}${location.pathname}">
      ${document.head.innerHTML}
      <style>
        /* 终极打印截断修复：强制所有元素释放高度并允许内容溢出 */
        @media print {
            * {
                overflow: visible !important;
                max-height: none !important;
                box-shadow: none !important;
            }
            html, body, #root, main, div {
                height: auto !important;
                position: static !important;
            }
            /* 隐藏底部输入框等不需要打印的无用 UI 元素 */
            fieldset, form, textarea {
                display: none !important;
            }
        }
      </style>
    </head>
    <body>
      ${document.body.innerHTML}
    </body>
  </html>
`);

// 3. 关闭文档写入流，确保 DOM 完全解析完成
printWindow.document.close();

// 4. 等待 2 秒，让外部 CSS 和字体资源加载完毕，然后触发系统原生打印弹窗
setTimeout(() => {
    printWindow.print();
}, 2000);
```

6. 按下 `Ctrl + S` (Windows/Linux) 或 `Cmd + S` (Mac) 保存该代码片段。

---

## 🚀 日常使用方法

既然脚本已经永久保存在你的浏览器中，以后导出时只需几秒钟：

1. 在 Claude 侧边栏右键并点击 **检查 (Inspect)**。
2. 按下 `Ctrl + P` (Mac 上为 `Cmd + P`)，输入 `!` 唤出 Snippets 菜单，然后选择 `Claude-to-PDF` 并回车。
   *(或者，如果你正好在 Sources 标签页，直接按 `Ctrl + Enter` 即可运行)*。
3. 浏览器会自动弹出一个新标签页，完美克隆你的对话内容，并在 2 秒后自动唤出系统打印窗口。

### ⚠️ 关键打印设置
当 Chrome 的打印预览窗口弹出时，请务必确认以下设置：
* 目标打印机选择 **另存为 PDF (Save as PDF)**。
* 展开 **更多设置 (More settings)**。
* **你必须勾选“背景图形” (Background graphics)**。如果你跳过这一步，深色/浅色模式的背景、代码块的底色以及聊天气泡的颜色都会全部丢失，导致排版混乱。
