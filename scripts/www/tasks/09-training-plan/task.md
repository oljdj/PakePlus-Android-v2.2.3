# 子任务 09：视觉训练方案输出

## 执行目标
在综合报告页中追加"视觉训练方案"卡片，根据诊断结果输出分阶段训练方案（调节异常训练方案 + 集合异常三阶段训练方案 + 训练通用原则）。

## 依赖
- 子任务 08（综合报告页已渲染）
- 子任务 05、06（诊断结果）

## 详细 Prompt

在 `page-report` 的"处理方法建议"卡片之后，追加"视觉训练方案"卡片。

### 1. 训练方案数据
在 JS 中创建训练方案对照表常量：

#### 调节异常训练方案
```js
const ACCOMMODATION_TRAINING = {
  insufficiency: {
    core: '提升调节幅度',
    phase1: { title: '第一阶段', tools: '推进训练 + 负镜阅读', goal: '建立调节刺激能力' },
    phase2: { title: '第二阶段', tools: '翻转拍 + 远近字母表', goal: '提升调节灵活度' },
    endpoint: '单眼+2.50/-6.00D通过'
  },
  excess: {
    core: '调节放松',
    phase1: { title: '第一阶段', tools: '正镜阅读 + 镜片排序', goal: '体验并学习放松调节' },
    phase2: { title: '第二阶段', tools: '翻转拍（侧重正镜）', goal: '提升正镜通过速度' },
    endpoint: '单眼≥11cpm，双眼≥8cpm'
  },
  illSustained: {
    core: '提升耐力',
    phase1: { title: '第一阶段', tools: '翻转拍（侧重负镜）+ 推进训练', goal: '克服负镜困难与耐力下降' },
    phase2: { title: '第二阶段', tools: '镜片排序进阶', goal: '精细化调节控制' },
    endpoint: '灵敏度达标且持久'
  },
  infacility: {
    core: '摆动训练',
    phase1: { title: '第一阶段', tools: '翻转拍双向 + 远近字母表', goal: '加快调节切换速度' },
    phase2: { title: '第二阶段', tools: '镜片阅读排序', goal: '巩固调节感知' },
    endpoint: '单眼≥12cpm，双眼≥8-10cpm'
  }
}
```

#### 集合异常训练方案（三阶段体系）
```js
const CONVERGENCE_TRAINING = {
  general: {
    phase1: { title: '第一阶段：感知+幅度训练', tools: '聚散球 → 红绿可变矢量图 → 镜片阅读+翻转拍' },
    phase2: { title: '第二阶段：跳跃性融像+灵敏度', tools: '裂隙尺 → 偏心环卡 → 救生圈卡' },
    phase3: { title: '第三阶段：功能整合+迁移', tools: '棱镜翻转拍 → 线珠旋转训练 → 电脑视觉训练' }
  },
  convergenceInsufficiency: {
    phase1Goal: '增加PFV至30▲+调节训练',
    phase2Goal: '增加跳跃融像+NFV',
    phase3Goal: '聚散-调节-眼球运动整合',
    endpoint: '镜片阅读通过+2.00/-6.00D，可变矢量图达到30▲'
  },
  convergenceExcess: {
    phase1Goal: '建立发散感知+调节训练',
    phase2Goal: '跳跃融像（裂隙尺12号集合+6号散开）',
    phase3Goal: '聚散转换训练',
    endpoint: '调节灵敏度达到12cpm，可变矢量图散开达到15▲'
  },
  divergenceExcess: {
    phase1Goal: '增加远距PFV+调节训练',
    phase2Goal: '改进远距PFV',
    phase3Goal: '远距聚散灵敏度',
    endpoint: '跳跃融像10次25▲BO和15▲BI/分钟'
  },
  basicExophoria: {
    phase1Goal: '自主集合+PFV+调节',
    phase2Goal: '中距离融像',
    phase3Goal: '远距离PFV',
    endpoint: '翻转拍配合聚散设备快速通过'
  },
  basicEsophoria: {
    phase1Goal: '增进近距NFV+调节',
    phase2Goal: '远距NFV',
    phase3Goal: '聚散灵敏度',
    endpoint: '翻转拍配合聚散设备快速通过'
  }
}
```

#### 训练通用原则
```js
const TRAINING_PRINCIPLES = [
  '先单眼后双眼（单眼只有调节没有集合）',
  '先练差的眼后练相对好的眼',
  '紧张、放松训练以放松结束',
  '远、近训练以远处结束',
  '每次累计训练时间不超过30分钟，每日1-2次',
  '先进行幅度训练，再进行灵活度训练',
  '训练设备先用简单的再用难的'
]
```

### 2. 渲染逻辑
创建 `renderTrainingPlan(diagnosis)` 函数，根据诊断结果渲染训练方案卡片：

#### 卡片标题："视觉训练方案"

#### 调节异常训练
- 如果确诊了调节异常，显示对应类型的训练核心 + 分阶段方案
- 每阶段用卡片嵌套展示：阶段标题 + 训练工具 + 训练目标
- 底部显示训练终点

#### 集合异常训练
- 如果确诊了集合异常，显示三阶段训练体系
- 先显示通用三阶段框架（每阶段的通用工具）
- 再显示该异常类型的三阶段具体目标
- 底部显示训练终点

#### 训练通用原则
- 始终显示，以提示框样式（蓝色背景框）列出 7 条原则

#### 无异常时
- 显示"视功能各项指标正常，暂无需视觉训练"（绿色）

### 3. 样式
- 训练阶段用时间线样式（左侧竖线 + 圆点 + 右侧内容）
- 训练工具用标签列表样式（圆角灰色背景的 inline 标签）
- 训练终点用高亮框（绿色边框 + 背景）
