# 先例研究与原创性决策（v1.19.0）

- 检索日期：2026-08-31
- 查询：`AI ecommerce product image generation`、`ecommerce product detail page design`、`product photography prompt workflow`
- 原始候选记录：源工作区保留 `reports/prior-art-candidates.json`；clean 发行包不包含该检索中间件。
- 结果：skills.sh 与 SkillsMP 均成功，合并 59 个候选 family，无目录缺证。
- v1.19 说明：本轮没有重新运行外部目录检索；以下候选和指标沿用此前真实记录。新增改造来自用户对五张主图、M1–M8 详情模块和三项交付的明确反馈，不伪称刷新外部证据。

## 指标语义

- `skills.sh installs` 是生态安装遥测，不是评分、正确性或设计质量；
- `SkillsMP repo stars` 是 GitHub 仓库星标，不是该 skill 的安装、评级或质量；
- 两种指标没有混算成“综合分”，只用作发现信号；所有采用项都重新阅读源文件并与本项目目标核对。

## 深读对象与决策

### 1. skills-101/superpowers · product-photography

- 发现信号：skills.sh 29.3K installs。
- **Keep**：商品摄影应覆盖主视觉、尺度、细节和生活方式等不同购买任务；光线必须具体。
- **Adapt**：在本 skill 中落成 G1–G5/GR 功能图库、尺度理解图、镜头与布光字段，而不是固定 7–9 张镜头清单。
- **Reject**：外部 inference.sh 工具绑定、固定序列、社交证明叠层和未经当前官方规则核验的平台尺寸。

### 2. nexu-io/open-design · ecommerce-image-workflow

- 发现信号：skills.sh 1.4K installs；SkillsMP 显示仓库 91,306 stars（仓库指标，不代表该 skill 质量）。
- **Keep**：产品身份锚、每条提示词重复保真约束、干净标签区、清单式交付。
- **Adapt**：落成 P/B/E/L/T/S/C 参考角色、授权/传输方式、产品与品牌/证据像素锁、每图生产包与三层 QA。
- **Reject**：所有任务固定 3 个槽位、任何策划都强制先有产品图、特定 Open Design 分发器依赖。

### 3. ajbeckliy/detail-flow · detail-flow

- 发现信号：skills.sh 31 installs。
- **Keep**：先蓝图后生成、两道确认门、先做少量代表切片、返修最小层级并保留已通过输出。
- **Adapt**：详情页落成独立设计大纲、全套逐屏执行确认、Anchor Checkpoint、首屏/场景族先行、局部编辑和问题层回滚；主图仍保持轻量策略门。
- **Reject**：固定 8 屏、强制 1:3 母版、固定 9:21 画幅；提示词-only 与用户明确 one-shot 不作机械停顿。

### 4. medusajs/medusa-agent-skills · storefront-best-practices

- 发现信号：skills.sh 3.6K installs。
- **Keep**：购买决策需要价格/变体/库存/细节/信任等真实信息，且移动端优先。
- **Adapt**：转换成购买疑问地图、D 动态字段、模块准入和移动端验收，不引入 storefront 代码。
- **Reject**：Medusa 专属实现与没有底层实验数据的“必然提升转化”表达。

### 补充阅读：motiful/product-shots · product-shots-detail-page

- 发现信号：skills.sh 42 installs。
- 发现其 Amazon 模块与转化提升陈述过于刚性或证据不足。本项目只保留“商品图覆盖不同信息任务”的一般启发，明确拒绝固定 Amazon 模块、无来源提升数字和跨平台通用化。

### 本轮定向深读：本地 `ppt-image-director`

- 来源：用户明确指定的本地 skill `/Users/jade01/.codex/skills/ppt-image-director/`；它不是目录排行榜候选，因此没有 installs、stars 或 rating 语义。本轮完整读取其 SKILL、visual director、inspiration search、image prompt、style config、generation pipeline 与机械 brief 拆分脚本。
- **Keep**：先建立全篇大纲与视觉系统，再把每页空间说明写到可直接生图的生产深度；同时保留场景族、一页一图、全局常量/单页差量、可见文字隔离和确认后逐字渲染。
- **Adapt**：PPT 原流程中的大纲与逐页方案在本 skill 中被有意拆成真实两阶段：电商大纲先锁购买决策、模块、屏序、逐屏文案素材、视觉概念、证据与承接；确认后才一次写完全部逐屏豆包执行稿。再加入 SKU/渠道/移动端、Visual System Contract、长页 screen map + L/T、P/B/E/L/T/S/C、style/layout 独立权重、provider 点名、完整 typography tuple、局部改字和逻辑 manifest。
- **Reject**：固定 14 张 Pinterest 候选、默认 90 强跟随、用户点名 IP 后严格模仿、整套固定视觉比例、把文字限制为少量短标题或交付空白底图、Logo/二维码/证书靠生成式“原样重画”、相邻页绝不复用融合方式、整套强制单渠道、最终只等用户点名问题。
- **Where**：`references/product-research-and-copy.md`、`main-image-set.md`、`detail-page-design.md`、`detail-page-prompt-spec.md`、`production-pipeline.md`、`project-packet-and-prompt-compiler.md` 与入口文件的 v1.19 Research/Outline/Execution/Anchor、字体执行和 QA 规则。

## 用户样例的 Keep / Adapt / Reject / Invent

- **Keep**：每张图先明确画面、图内文案和设计指引；一张图承担一个转化任务；主图与详情模块按购买问题组织。
- **Adapt**：用户明确要 5 张且未另定顺序时，封面/痛点/差异/场景/CTA 成为 `FIVE_CONVERSION_ARC_V1` 默认；它不冒充平台通则。痛点使用中性相关性，比较加证据门，CTA/价格服从渠道与 D 范围。详情页将 M1–M8 作为选配模块库，再映射成可拆合 Dxx 屏。
- **Reject**：用户未指定五张时仍无条件固定；为凑五张编造普通食品症状痛点、竞品、资质或动态价格；强制所有 M1–M8 或固定 8 屏；FAQ 为凑 6 问而填充；末屏免责声明补救前屏。
- **Invent**：五张的 count mode/默认角色/解冲突记录，M 模块与 D 屏实例双层标识，以及主图、Stage A 模块卡、Stage B 执行卡统一为三个用户可见一级项；继承 Research Gate、SP-ID 准入、完整 typography tuple、豆包直出与局部改字。

## 设计优势、验证与假设

- **设计优势（有机制依据）**：显式处理包装漂移、长页/参考内容泄漏、风格与版式串线、重复构图、伪科学装饰、可见文字错漏和动态价格过期；返修能落到单图、单文字区或单依赖族。
- **已验证**：包结构与触发规则可由本地脚本验证；行为夹具的结构与覆盖范围已静态检查，实际行为结果仍是 `missing evidence`；官方来源支持参考角色化、小步编辑、渠道差异、证据和动态字段边界。
- **尚未验证的假设**：该流程会提高 CTR/CVR、降低退款或一定获得“更高级”的专业盲评分；需要真实模型输出、设计师评审和店铺数据验证，不能把结构优化冒充转化实证。
