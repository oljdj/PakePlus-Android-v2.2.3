# 子任务 07：三级视功能诊断与诊断详情页渲染

## 执行目标
1. 实现三级视功能诊断逻辑（Worth-4 点与立体视判定）
2. 实现 `page-diagnosis` 诊断详情页的完整渲染，包括调节诊断、集合八步分析过程、三级视功能诊断的可折叠展示

## 依赖
- 子任务 05（调节诊断结果）
- 子任务 06（集合诊断结果与八步过程）
- 子任务 02（UI 组件：collapsible、stepper、abnormal-badge）

## 详细 Prompt

### 1. 三级视功能诊断函数
创建 `diagnoseBinocular(data)` 函数：

```js
gExamData.diagnosis.binocular = {
  worthFourDot: { value: '4', result: 'normal', description: '正常双眼单视' },
  stereoAcuity: { value: 40, result: 'normal', description: '正常立体视' }
}
```

判定逻辑：
| 检查项 | 结果 | 诊断 | 描述 |
|--------|------|------|------|
| Worth-4点 | 4点 | normal | 正常双眼单视 |
| Worth-4点 | 3点 | abnormal | 单眼抑制 |
| Worth-4点 | 2点 | abnormal | 单眼抑制（另一眼） |
| Worth-4点 | 5点 | critical | 复视（双眼视破坏） |
| 立体视 | ≤60″ | normal | 正常立体视 |
| 立体视 | > 60″ | abnormal | 立体视异常 |
| 立体视 | 无法测出(0) | critical | 无立体视 |

### 2. page-diagnosis 页面结构

页面分为三大可折叠区域，使用 `.collapsible` 组件：

#### 区域一：调节功能诊断
- 标题：`调节功能诊断` + 诊断结果摘要标签（如"调节不足"红色标签）
- 展开后显示 4 种调节异常类型，每种：
  - 类型名称 + 诊断结果（✓确诊 / ✗排除）
  - 必要条件列表：每项显示条件描述 + 满足状态（✓绿色 / ✗红色）+ 数值详情
  - 充分不必要条件列表：每项显示条件描述 + 满足状态 + 数值详情
  - 必要条件用红色边框标注（`.necessary-condition`），充分不必要条件用蓝色边框标注（`.sufficient-condition`）

#### 区域二：集合功能诊断（八步分析法）
- 顶部使用 `.stepper` 组件显示 8 步流程指示器，当前步骤高亮
- 下方为 8 个可折叠面板（每步一个），默认展开当前步骤
- 每步面板：
  - 面板头：步骤编号 + 标题 + 判定摘要
  - 面板体：该步的详细判定数据（格式化展示）
  - 第三步（看抵抗力）特别展示 Sheard/1:1 准则的计算过程
  - 第八步（综合诊断）展示各类型的必要/充分条件满足情况（同调节诊断的展示方式）
- 点击 stepper 的步骤圆圈可跳转到对应步骤并展开

#### 区域三：三级视功能诊断
- Worth-4 点：显示结果 + 诊断描述
- 立体视：显示实际值 + 正常值 + 诊断描述
- 异常结果用红色标注

### 3. 诊断触发
- 进入 `page-diagnosis` 时，依次调用：
  1. `diagnoseAccommodation(gExamData)` → 更新 `gExamData.diagnosis.accommodation`
  2. `diagnoseConvergence(gExamData, gExamData.diagnosis.accommodation)` → 更新 `gExamData.diagnosis.convergence`
  3. `diagnoseBinocular(gExamData)` → 更新 `gExamData.diagnosis.binocular`
- 调用完毕后渲染页面

### 4. 条件高亮样式
- `.necessary-condition`：左侧 3px 红色边框，背景 #FFF5F5
- `.sufficient-condition`：左侧 3px 蓝色边框，背景 #F0F4FF
- 条件满足 ✓：绿色文字 + 加粗
- 条件不满足 ✗：灰色文字

### 5. 底部操作栏
- "下一步：综合报告" 按钮 → `showPage('page-report')`
