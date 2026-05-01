# 子任务 06：集合功能异常诊断引擎（八步分析法）

## 执行目标
实现集合功能异常的八步分析法诊断逻辑，涵盖 8 种集合异常类型（集合不足、集合过度、散开不足、散开过度、基本型外隐斜、基本型内隐斜、融像性聚散异常、假性集合不足）的诊断推理。

## 依赖
- 子任务 03（gExamData）
- 子任务 05（调节诊断结果，用于假性集合不足判断）

## 详细 Prompt

在 `index.html` 中实现集合功能八步分析法诊断引擎。

### 1. 全局诊断结果扩展
```js
gExamData.diagnosis.convergence = {
  steps: [],          // 八步分析过程，每步的结果
  types: {
    convergenceInsufficiency: { result: false, necessaryConditions: [], sufficientConditions: [] },
    convergenceExcess: { result: false, necessaryConditions: [], sufficientConditions: [] },
    divergenceInsufficiency: { result: false, necessaryConditions: [], sufficientConditions: [] },
    divergenceExcess: { result: false, necessaryConditions: [], sufficientConditions: [] },
    basicExophoria: { result: false, necessaryConditions: [], sufficientConditions: [] },
    basicEsophoria: { result: false, necessaryConditions: [], sufficientConditions: [] },
    fusionalVergenceDysfunction: { result: false, necessaryConditions: [], sufficientConditions: [] },
    pseudoConvergenceInsufficiency: { result: false, necessaryConditions: [], sufficientConditions: [] }
  }
}
```

### 2. 八步分析法函数
创建 `diagnoseConvergence(data, accommodationDiagnosis)` 函数，按以下八步顺序推理，每步结果推入 `steps` 数组：

#### 第一步：看眼位
- 解析远眼位、近眼位的隐斜方向与量值
- 符号约定：正值 = 内隐斜(eso/BO)，负值 = 外隐斜(exo/BI)
- 输出：`{ step: 1, title: '看眼位', result: { farType, farValue, nearType, nearValue } }`

#### 第二步：初步判断异常眼位类型
根据 PRD 4.3.2 第二步的 6 种模式判定：
- 近眼位问题 → 集合不足/集合过度
- 远眼位问题 → 散开过度/散开不足
- 远近均异常 → 基本型外隐斜/基本型内隐斜
- 输出：`{ step: 2, title: '初步判断', result: { pattern, preliminaryType } }`

#### 第三步：看抵抗力（正负融像储备）
- 外隐斜：检查 BO 模糊值是否 ≥ 2 × 外隐斜量（Sheard 准则）
  - 远外隐斜 → 检查远 BO 模糊值
  - 近外隐斜 → 检查近 BO 模糊值
  - 远近外隐斜 → 检查远近 BO 模糊值
- 内隐斜：检查 BI 恢复值是否 ≥ 内隐斜量（1:1 准则）
  - 远内隐斜 → 检查远 BI 恢复值
  - 近内隐斜 → 检查近 BI 恢复值
  - 远近内隐斜 → 检查远近 BI 恢复值
- 输出：`{ step: 3, title: '看抵抗力', result: { criterion, satisfied, details } }`
- 不够时标注 `warning: '融像储备不足，可能出现视疲劳/复视'`

#### 第四步：看 AC/A 值
- 判定 AC/A 状态：高 / 低 / 正常
- 结合第二步的初步判断和 AC/A 状态，细化诊断：
  - 高AC/A + 近眼位异常 → 集合过度
  - 高AC/A + 远眼位异常 → 散开过度
  - 低AC/A + 近眼位异常 → 集合不足
  - 低AC/A + 远眼位异常 → 散开不足
  - 正常AC/A + 远近均异常 → 基本型
  - 正常AC/A + 远近均正常 → 无显著聚散异常
- 输出：`{ step: 4, title: '看AC/A值', result: { acStatus, refinedType } }`

#### 第五步：看调节与集合关系
- 集合不足 → 可能导致调节过度
- 集合过度 → 可能导致调节不足
- 输出：`{ step: 5, title: '看调节与集合关系', result: { relation } }`

#### 第六步：判断原发与继发
- 核心规则："相同性质找调节，不同性质找集合"
  - 调节不足 + 集合不足 → 调节原发，集合继发
  - 调节过度 + 集合过度 → 调节原发，集合继发
  - 集合不足 + 调节过度 → 集合原发，调节继发
  - 集合过度 + 调节不足 → 集合原发，调节继发
- 输出：`{ step: 6, title: '原发与继发', result: { primary, secondary, rule } }`

#### 第七步：看调节是否影响屈光度
- 调节不足 → 不影响屈光度
- 调节过度 → 需雾视/眼贴眼罩/反转拍/散瞳排除假性近视
- 输出：`{ step: 7, title: '调节与屈光度', result: { affectsRefraction, recommendation } }`

#### 第八步：综合诊断（由各类型的必要/充分条件判定决定，见下方）

### 3. 各类型详细诊断条件判定

在第八步中，对每种集合异常类型逐一判定：

#### A. 集合不足
- 必要条件：近眼位 < -6 且远眼位在 +1~-3 范围或轻微外隐斜；NPC > 7
- 充分不必要条件：AC/A 低；近 BO 模糊值降低（< 12）；NRA 降低（< +2.00D）；双眼翻转拍正片困难；BCC < +0.25D

#### B. 集合过度
- 必要条件：近眼位 > 0 且远眼位正常或轻微内隐斜；AC/A ≥ 7（计算法）
- 充分不必要条件：近 BI 恢复值降低；PRA 降低；双眼翻转拍负片困难；BCC > +0.75D；NPC < 5

#### C. 散开不足
- 必要条件：远眼位 > +1 且近眼位在 0~-6 范围；AC/A < 5（计算法）
- 充分不必要条件：远 BI 恢复值降低；PRA 偏低

#### D. 散开过度
- 必要条件：远眼位 < -3 且近眼位正常；AC/A 高（计算法）
- 充分不必要条件：远 BO 模糊值降低；NRA 偏低；双眼翻转拍正片困难

#### E. 基本型外隐斜
- 必要条件：|远眼位 - 近眼位| < 5 且均为外隐斜；AC/A 正常
- 充分不必要条件：远近 BO 模糊值均降低；NRA 偏低；双眼翻转拍正片困难；NPC > 10

#### F. 基本型内隐斜
- 必要条件：|远眼位 - 近眼位| < 5 且均为内隐斜；AC/A 正常
- 充分不必要条件：远近 BI 恢复值均降低；PRA 偏低；双眼翻转拍负片困难；NPC < 5

#### G. 融像性聚散异常
- 必要条件：远近隐斜均正常；AC/A 正常
- 充分不必要条件：远近 BO/BI 融像储备均降低；NRA < +2.00D 且 PRA 偏低；双眼翻转拍双向困难

#### H. 假性集合不足
- 必要条件：眼位表现类似集合不足（近高度外隐斜）；AMP 显著低于年龄预期
- 充分不必要条件：BCC > +0.75D；AC/A 低
- 鉴别核心：附加 +0.75D 后 NPC 明显改善 → 假性（提示用户确认）

### 4. Sheard 准则与 1:1 准则辅助计算
- `checkSheardCriterion(phoriaValue, boBlurValue)`: 返回 `{ satisfied, prismNeeded }`，其中 `prismNeeded = (2 * phoriaValue - boBlurValue) / 3`
- `checkOneToOneCriterion(phoriaValue, biRecoveryValue)`: 返回 `{ satisfied }`

### 5. 八步分析过程写入 steps 数组
第八步的输出：`{ step: 8, title: '综合诊断', result: { diagnosedTypes, summary } }`
