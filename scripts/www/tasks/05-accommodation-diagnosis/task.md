# 子任务 05：调节功能异常诊断引擎

## 执行目标
实现四种调节功能异常类型的诊断推理逻辑：调节不足、调节过度、调节不能持久、调节灵敏度异常。每种类型输出必要条件与充分不必要条件的满足情况及最终诊断结论。

## 依赖
- 子任务 03（gExamData）
- 子任务 04（analyzeAbnormalities 结果）

## 详细 Prompt

在 `index.html` 中实现调节功能诊断引擎。

### 1. 全局诊断结果对象
在 `gExamData` 中追加 `diagnosis` 字段（如果尚未存在），结构：
```js
gExamData.diagnosis = {
  accommodation: {
    insufficiency: { result: false, necessaryConditions: [], sufficientConditions: [] },
    excess: { result: false, necessaryConditions: [], sufficientConditions: [] },
    illSustained: { result: false, necessaryConditions: [], sufficientConditions: [] },
    infacility: { result: false, necessaryConditions: [], sufficientConditions: [] }
  },
  // 集合和三级视功能诊断在后续子任务填充
}
```

每个 condition 对象结构：
```js
{
  label: 'AMP 显著降低',         // 条件描述
  met: true,                     // 是否满足
  detail: '左眼AMP=5.0D < 7.0D'  // 满足/不满足的具体数值说明
}
```

### 2. 诊断函数
创建 `diagnoseAccommodation(data)` 函数，返回上述 accommodation 对象。

#### A. 调节不足（Accommodative Insufficiency）
- **必要条件**：双眼 AMP 中较低者 < (15 - 0.25 × age) - 2.00D
- **充分不必要条件**（需至少满足 2 项）：
  1. PRA 降低：PRA < 2.50D（取绝对值判断，注意 PRA 为负值输入时如 -1.00D 表示偏低）
  2. BCC 滞后：BCC > +0.75D
  3. 任一眼翻转拍负片困难
  4. 近距离 BO 模糊值低于正常范围（< 12）
- **诊断规则**：必要条件满足 + 至少 2 项充分不必要条件满足 → 诊断调节不足

#### B. 调节过度（Accommodative Excess）
- **必要条件**：BCC 超前（BCC < +0.25D）
- **充分不必要条件**（需至少满足 1 项）：
  1. AMP 正常：双眼 AMP 均 ≥ (15 - 0.25 × age)
  2. 任一眼翻转拍正片困难
  3. NRA 降低：NRA < +2.00D
  4. 内隐斜：近眼位 > 0 或远眼位 > +1
- **诊断规则**：必要条件满足 + 至少 1 项充分不必要条件满足 → 诊断调节过度

#### C. 调节不能持久（Ill-sustained Accommodation）
- **必要条件**：AMP 正常（≥期望值-2）但提示需用户确认"首测正常但重复测量下降"特征（设 `needsUserConfirm: true`）
- **充分不必要条件**（需至少满足 2 项）：
  1. BCC 滞后：BCC > +0.75D
  2. 任一眼翻转拍负片困难
  3. PRA 偏低：PRA < 2.50D
  4. 近 BO 模糊值偏低（< 12）
- **诊断规则**：必要条件满足（需用户确认耐力特征）+ 至少 2 项充分不必要条件 → 提示可能调节不能持久

#### D. 调节灵敏度异常（Accommodative Infacility）
- **必要条件**：单眼灵敏度 < 12 cpm 或双眼灵敏度 < 8 cpm
- **充分不必要条件**（需至少满足 1 项）：
  1. AMP 正常：双眼 AMP 均 ≥ (15 - 0.25 × age)
  2. BCC 正常：+0.25D ~ +0.75D
  3. NRA 与 PRA 均降低：NRA < +2.00D 且 |PRA| < 2.50D
  4. 翻转拍双向困难
- **诊断规则**：必要条件满足 + AMP/BCC 正常 + 至少 1 项充分不必要条件 → 诊断调节灵敏度异常
- **鉴别**：仅双眼灵敏度异常而单眼正常 → 提示聚散功能问题而非单纯调节灵敏度异常

### 3. 调节不能持久用户确认
在诊断流程中，若调节不能持久的必要条件初步满足，在 `page-diagnosis` 中显示一个确认弹窗/面板：
"调节幅度首测正常，但重复测量时是否逐渐下降？" → 用户确认后才标记 `result: true`

### 4. 辅助函数
- `getAmpExpected(age)`: 返回 15 - 0.25 × age
- `isAmpNormal(ampValue, age)`: ampValue ≥ getAmpExpected(age) - 2
- `isAmpSignificantlyLow(ampValue, age)`: ampValue < getAmpExpected(age) - 2
