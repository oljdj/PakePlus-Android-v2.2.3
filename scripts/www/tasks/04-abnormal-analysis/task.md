# 子任务 04：异常项自动标注模块

## 执行目标
实现实时异常标注计算引擎与 `page-analysis` 页面展示。根据 `gExamData` 与 Morgan 正常参考值对照，自动标注每项检查的异常状态，并渲染到异常分析页。

## 依赖
- 子任务 02（UI 组件：abnormal-badge）
- 子任务 03（gExamData 数据对象）

## 详细 Prompt

在 `index.html` 中实现异常标注引擎与展示页。

### 1. 正常值配置表（JS 常量对象）
创建 `NORMAL_VALUES` 常量，包含所有检查项的正常值范围（基于 Morgan 标准参考值）：

```js
const NORMAL_VALUES = {
  nra: { min: 2.00, max: 2.50, unit: 'D' },
  pra: { min: 2.50, max: null, unit: 'D' },       // ≥2.50
  bcc: { min: 0.25, max: 0.75, unit: 'D' },
  // AMP 为动态计算，min = 15 - 0.25 * age - 2（低于期望值2D为显著异常）
  sensitivityMonocular: { min: 12, max: null, unit: 'cpm' },
  sensitivityBinocular: { min: 8, max: null, unit: 'cpm' },
  farPhoria: { min: -3, max: 1, unit: '△' },
  nearPhoria: { min: -6, max: 0, unit: '△' },
  npc: { min: 5, max: 7, unit: 'cm' },
  acCalculated: { min: 5, max: 7, unit: '△/D' },
  acGradient: { min: 3, max: 5, unit: '△/D' },
  stereoAcuity: { min: null, max: 60, unit: '″' },
  // 融像范围正常值
  farBI: { blur: null, breakMin: 4, breakMax: 10, recoveryMin: 2, recoveryMax: 6 },
  farBO: { blurMin: 5, blurMax: 13, breakMin: 11, breakMax: 27, recoveryMin: 6, recoveryMax: 14 },
  nearBI: { blurMin: 9, blurMax: 17, breakMin: 17, breakMax: 25, recoveryMin: 8, recoveryMax: 18 },
  nearBO: { blurMin: 12, blurMax: 22, breakMin: 15, breakMax: 27, recoveryMin: 4, recoveryMax: 18 }
}
```

### 2. 异常判定函数
创建函数 `analyzeAbnormalities(data)` 返回异常项数组，每项结构：
```js
{
  key: 'nra',              // 检查项标识
  label: 'NRA（负相对调节）', // 中文标签
  value: 1.50,             // 实际值
  normalRange: '+2.00~+2.50D', // 正常范围文本
  level: 'low',            // normal / low / high / critical / attention
  description: '低于正常值下限' // 异常描述
}
```

**判定逻辑**（严格按照 PRD 4.2 节）：

| 检查项 | 判定条件 | 异常等级与类型 |
|--------|---------|--------------|
| NRA | > +2.50D | high（偏高：调节痉挛/近视过矫） |
| NRA | < +2.00D | low（偏低：调节过度表现） |
| PRA | < -2.50D（即 < 2.50 取绝对值判断） | low（偏低：调节不足表现） |
| BCC | < +0.25D | low（调节超前） |
| BCC | > +0.75D | high（调节滞后） |
| AMP单眼 | < (15-0.25×age) - 2 | critical（调节幅度显著降低） |
| AMP单眼 | ≥ (15-0.25×age) - 2 且 < (15-0.25×age) | attention（正常但越高越好） |
| 单眼灵敏度 | < 12 cpm | low（调节灵敏度下降） |
| 双眼灵敏度 | < 8 cpm | low（调节灵敏度下降） |
| 远眼位 | > +1 | high（远内隐斜） |
| 远眼位 | < -3 | low（远外隐斜过大） |
| 近眼位 | > 0 | high（近内隐斜） |
| 近眼位 | < -6 | low（近外隐斜过大） |
| AC/A(计算法) | < 5 | low |
| AC/A(计算法) | > 7 | high |
| AC/A(梯度法) | < 3 | low |
| AC/A(梯度法) | > 5 | high |
| NPC | < 5 | high（集合过度倾向） |
| NPC | > 10 | low（集合不足倾向） |
| NPC | 7-10 | attention（需关注） |
| Worth-4点 | ≠ 4点 | critical |
| 立体视 | > 60″ | critical |
| 立体视 | 40-60″ | attention |

### 3. 实时联动
- 在 `page-input` 中每个输入框的 `input` 事件中，调用 `analyzeAbnormalities` 并更新对应 `.status-badge` 的显示
- 切换到 `page-analysis` 页面时，渲染完整的异常项汇总列表

### 4. page-analysis 页面布局
- 顶部：基础信息摘要（年龄、PD、检查距离）
- 主体：按模块分组的异常项列表
  - 调节功能异常项（绿/黄/红/蓝色徽章标注）
  - 集合功能异常项（同上）
  - 三级视功能异常项（同上）
- 每项显示：检查项名称 + 实际值 + 正常值范围 + 异常徽章
- 底部操作栏："下一步：诊断推理" 按钮 → `showPage('page-diagnosis')`
- 正常项也显示（绿色徽章），异常项排在前面

### 5. 输入页同步反馈
在 `page-input` 中，每个输入框旁边的 `.status-badge` 实时更新为对应异常等级的小圆点（不显示文字，只在 analysis 页展开详情）
