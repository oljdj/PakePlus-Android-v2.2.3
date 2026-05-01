# 子任务 01：HTML 骨架与移动端适配

## 执行目标
创建单页 HTML 应用的骨架文件 `index.html`，包含完整的 viewport 适配、触控优化、全局 CSS 变量与基础布局结构。此文件作为所有后续子任务的基础容器。

## 详细 Prompt

你正在开发一个面向移动端的单文件 HTML 应用——**视功能分析工具（Oculus）**，用于眼视光专业人员进行视功能检查数据分析与诊断。

请创建 `index.html` 文件，要求如下：

### 1. HTML 骨架
- 单文件应用，所有 CSS 内联在 `<style>` 中，所有 JS 内联在 `<script>` 中
- `<!DOCTYPE html>` 声明，`<html lang="zh-CN">`
- `<meta charset="UTF-8">`
- `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">`
- `<title>视功能分析工具 - Oculus</title>`

### 2. 全局 CSS 变量（设计令牌）
在 `:root` 中定义以下 CSS 变量：
- **主色调**：--primary: #1A73E8（医疗蓝）；--primary-light: #E8F0FE
- **语义色**：--success: #34A853（正常/绿）；--warning: #FBBC04（偏低偏高/黄）；--danger: #EA4335（显著异常/红）；--info: #4285F4（需关注/蓝）
- **文字**：--text-primary: #202124；--text-secondary: #5F6368；--text-hint: #9AA0A6
- **间距**：--spacing-xs: 4px；--spacing-sm: 8px；--spacing-md: 16px；--spacing-lg: 24px；--spacing-xl: 32px
- **圆角**：--radius-sm: 4px；--radius-md: 8px；--radius-lg: 12px
- **字号**：--font-xs: 12px；--font-sm: 14px；--font-md: 16px；--font-lg: 18px；--font-xl: 20px；--font-xxl: 24px
- **安全区**：--safe-top: env(safe-area-inset-top)；--safe-bottom: env(safe-area-inset-bottom)

### 3. 基础布局结构
页面采用多视图单页架构，通过 JS 切换显示不同 section：
```
<body>
  <header> 顶部导航栏：应用名称 + 当前步骤指示器 </header>
  <main id="app">
    <section id="page-home"> 首页：新建检查 / 历史记录入口 </section>
    <section id="page-input"> 信息录入页（3个Tab） </section>
    <section id="page-analysis"> 实时异常分析页 </section>
    <section id="page-diagnosis"> 诊断详情页（八步分析） </section>
    <section id="page-report"> 综合报告页 </section>
    <section id="page-history"> 历史对比页 </section>
  </main>
  <footer> 底部操作栏 </footer>
</body>
```

### 4. 移动端适配 CSS
- `* { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }`
- `body` 设置 font-family（系统字体栈）、背景色 #F8F9FA、适配安全区域 padding
- 所有 `input` 设置 `font-size: 16px`（防止 iOS 自动缩放）
- 触控友好：按钮/可点击元素最小 44×44px 触控区域
- 响应式：max-width: 480px 居中（平板/桌面不过宽）

### 5. 页面切换 JS 框架
- 全局函数 `showPage(pageId)`：隐藏所有 section，显示指定 section，更新 header 步骤指示
- 页面切换添加淡入动画（CSS transition opacity 0.3s）
- 初始显示 `page-home`

### 6. 不需要实现任何业务逻辑，只需搭建好骨架与导航框架即可
