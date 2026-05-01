# 子任务 03：数据输入模块

## 执行目标
实现信息录入页（`page-input`）的完整交互，包括：基础信息输入、调节功能检查输入（Tab 1）、集合功能检查输入（Tab 2）、三级视功能检查输入（Tab 3）、AC/A 辅助计算联动、AMP 期望值自动计算。

## 依赖
- 子任务 01（骨架）
- 子任务 02（UI 组件库）

## 详细 Prompt

在 `index.html` 中实现 `page-input` 页面的完整数据录入功能。

### 1. 全局数据对象
在 JS 中创建全局对象 `gExamData`，结构如下：
```js
gExamData = {
  basic: { age: null, pd: null, nearDistance: 0.4 },
  accommodation: {
    nra: null, pra: null, bcc: null,
    ampRight: null, ampLeft: null,
    sensitivityRight: null, sensitivityLeft: null, sensitivityBinocular: null,
    flipperRightType: 'none', flipperLeftType: 'none', flipperBinocularType: 'none'
  },
  convergence: {
    farPhoria: null, nearPhoria: null,
    acMethod: 'calculated', acValue: null, adValue: null,
    npc: null,
    farBI: { blur: null, break: null, recovery: null },
    farBO: { blur: null, break: null, recovery: null },
    nearBI: { blur: null, break: null, recovery: null },
    nearBO: { blur: null, break: null, recovery: null }
  },
  binocular: {
    worthFourDot: '4', stereoAcuity: null
  }
}
```
所有输入框双向绑定到此对象（input 事件实时更新）。

### 2. 顶部基础信息区
- 年龄（数字输入，必填，范围 3-80）
- 瞳距 PD（数字输入，必填，单位 cm，范围 4-8，步进 0.1）
- 检查距离（数字输入，默认 0.4m，范围 0.25-0.5，步进 0.05）
- 年龄变化时自动计算并显示 AMP 期望值：`15 - 0.25 × age`（Hofstetter 公式），显示为提示文本

### 3. Tab 1：调节功能检查
使用 `.tab-bar` 组件，Tab 内容：
- NRA（数字输入，单位 D，步进 0.25）
- PRA（数字输入，单位 D，步进 0.25）
- BCC（数字输入，单位 D，步进 0.25）
- 右眼 AMP / 左眼 AMP（数字输入，单位 D，步进 0.25）
- 右眼灵敏度 / 左眼灵敏度 / 双眼灵敏度（数字输入，单位 cpm，整数）
- 右眼/左眼/双眼翻转拍通过困难类型（select 单选：无/负片困难/正片困难/双向困难）

### 4. Tab 2：集合功能检查
- 远距离眼位 / 近距离眼位（数字输入，单位 △，正=内隐斜/负=外隐斜）
- AC/A 测试方法（select：计算法/梯度法），切换方法时联动下方正常值显示
- AC/A 值（数字输入，单位 △/D）
- AD/A 值（数字输入，可选）
- NPC（数字输入，单位 cm，步进 0.5）
- **AC/A 辅助计算联动**：
  - 选计算法时，显示公式 `AC/A = PD + M × (近眼位 - 远眼位)` 并提供"自动计算"按钮，点击后填入计算结果（用户可覆盖）
  - 选梯度法时，显示公式 `AC/A = (初始隐斜量 - 最终隐斜量) / 附加度数`，提供 3 个辅助输入框 + 自动计算按钮
- 远/近距离融像范围 BI 和 BO：每项 3 个输入框（模糊/破裂/恢复），单位 △，用 `parseFusionRange` 解析

### 5. Tab 3：三级视功能检查
- Worth-4 点（select：4点/3点/2点/5点）
- 立体视（数字输入，单位 ″）

### 6. 底部操作栏
- "下一步：异常分析" 主按钮，点击后调用 `showPage('page-analysis')` 并触发异常标注计算
- 所有必填项校验：未填时 Toast 提示，已填则切换页面

### 7. 样式要求
- 每个 Tab 内容区可滚动（max-height 限制，overflow-y: auto）
- 输入组之间用分割线或卡片间隔
- 融像范围的 3 个输入框横向排列为一组
