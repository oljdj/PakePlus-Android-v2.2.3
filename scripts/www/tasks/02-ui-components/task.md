# 子任务 02：通用 UI 组件库

## 执行目标
在 `index.html` 中实现所有页面复用的通用 UI 组件（纯 CSS 类 + JS 构造函数），包括：表单输入组、标签页、异常标注徽章、可折叠面板、步骤指示器、Toast 提示、底部操作栏。

## 依赖
- 子任务 01（骨架与 CSS 变量）已完成

## 详细 Prompt

在已有 `index.html` 骨架基础上，在 `<style>` 和 `<script>` 中追加以下通用 UI 组件的实现。

### 1. 表单输入组 `.form-group`
- 结构：`.form-group > label + .input-wrapper > (input/select + .unit + .status-badge)`
- `.form-group` 纵向排列，间距 var(--spacing-md)
- label 使用 --font-sm 加粗，--text-primary
- input/select 统一样式：全宽、高度 44px、圆角 var(--radius-md)、边框 1px solid #DADCE0、padding 0 12px、聚焦时边框色 --primary
- `.unit` 显示在输入框右侧（如 D、△、cpm、cm），用 --text-hint + --font-xs
- `.status-badge` 显示在输入框右下角的小圆点/图标，根据异常等级着色（正常绿/偏低黄↓/偏高黄↑/异常红/需关注蓝），初始隐藏

### 2. 标签页 `.tab-bar`
- 水平排列的 Tab 按钮，选中态有底部 2px --primary 指示条 + 文字变 --primary
- JS 函数 `initTabBar(container, tabs, onChange)`：tabs 为 [{id, label}]，onChange 回调返回选中的 tab id
- 适配移动端：Tab 可横向滚动（如果数量多），scroll-snap

### 3. 异常标注徽章 `.abnormal-badge`
- 5 种等级：normal（绿圆点 + "正常"）、low（黄圆点↓ + "偏低"）、high（黄圆点↑ + "偏高"）、critical（红圆点 + "显著异常"）、attention（蓝圆点 + "需关注"）
- 用 CSS 类 `.badge-normal` / `.badge-low` / `.badge-high` / `.badge-critical` / `.badge-attention` 控制颜色
- 行内显示，font-size: var(--font-xs)

### 4. 可折叠面板 `.collapsible`
- 面板头 `.collapsible-header`：点击切换展开/折叠，右侧有 ▶/▼ 箭头
- 面板体 `.collapsible-body`：展开时显示，折叠时隐藏，max-height 动画过渡
- JS 函数 `toggleCollapsible(el)`

### 5. 步骤指示器 `.stepper`
- 水平步骤条：圆形数字 + 文字标签，当前步骤高亮 --primary，已完成步骤打勾
- 用于诊断页八步分析流程展示

### 6. Toast 提示
- JS 函数 `showToast(message, type)`：type 为 success/warning/error/info
- 顶部弹出，2 秒后自动消失，圆角卡片 + 对应语义色图标

### 7. 底部操作栏 `.bottom-bar`
- 固定在底部（适配 safe-area-inset-bottom）
- 内含主操作按钮 `.btn-primary`（--primary 色满宽）和次操作按钮 `.btn-secondary`（描边样式）
- 按钮高度 44px，圆角 var(--radius-lg)

### 8. 通用工具函数
- `formatValue(val, unit)`: 格式化数值 + 单位
- `parseFusionRange(str)`: 解析融像范围 "模糊/破裂/恢复" 为 {blur, break, recovery}
- `clamp(val, min, max)`: 数值钳制

所有组件样式和 JS 均追加到 index.html 已有 `<style>` 和 `<script>` 块中，不创建新文件。
