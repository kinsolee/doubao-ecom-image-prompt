# 电商视觉生产流水线

本文件是两种模式共用的生产主干。目标不是让提示词“更长”，而是让每一次生成都有可靠输入、明确画面任务、可控产品身份、可复用视觉系统、完整图内文字与可验证验收。视觉灵感、长页参考与资产角色读取 [visual-reference-and-style-system.md](visual-reference-and-style-system.md)；多图项目清单与五层提示词编译读取 [project-packet-and-prompt-compiler.md](project-packet-and-prompt-compiler.md)。

## 一、统一状态机

```text
A 任务与渠道定义
→ B0 输入/资产/OCR观察与身份审计〔必要时：同 SKU/规则/买家调研 → 卖点信息大纲 → Research Gate〕
→ B 确认事实范围、资产角色与风险
→ C 视觉基因/灵感、平台×类目规则与模块依据
→ D 主图/详情页模块候选 + 视觉方向
→ E 主图策略卡 / 详情页整套设计大纲
→ Gate 1：确认主图策略或详情页大纲
→ F Visual System Contract + 全部逐图/逐屏详细设计执行稿
→ Gate 2：统一确认详细执行包
→ G 首图/首屏与必要场景族锚 → Anchor Checkpoint
→ H 项目清单、五层提示词编译、豆包完整图文生成与局部修订
→ I OCR/目视逐字校对、拼接、事实/视觉/渠道验收
```

执行规则：

- `STANDARD` 的详情页先在 Gate 1 确认整套设计大纲，再一次完成所有逐屏详细设计执行稿并在 Gate 2 统一确认；之后生成首屏/必要场景族锚并做 Anchor Checkpoint，才批量。主图套系在 Gate 1 确认策略卡、Gate 2 确认整套执行包；单张图在 Gate 1 合并确认策略与执行，不增加批量门或锚点门。
- `FAST` 在用户明确“直接做、无需确认”时跳过对话停顿，但仍依次形成并版本化大纲/策略、详细执行稿、Visual System、产品锁、锚定、任务清单与内部检查。
- `PROMPT_ONLY` 按“Part A 大纲/策略 → Part B 全部详细执行稿”交付，不制造实际生图确认门；仍输出完整 Visual System、参考协议、分层提示词和验收。
- **稀疏输入例外**：详情页只给产品图，或只有身份/零散规格，且不存在完整、已批准、证据合格的本 SKU 卖点集时，先输出调研卖点信息大纲并等待用户确认。Research Gate 是身份范围与卖点选择门，FAST 不能内部代批，PROMPT_ONLY 也不能用未确认主张编译 listing-grade prompt。
- 身份/SKU 无法可靠解析时，只问一个能避免搜错商品的最小问题；只是缺逐页文案但已有可信事实时不阻塞，由流程代拟。
- 主图和详情页一起做时，先完成共同 Research Gate（如触发）、主图策略与详情设计大纲，再完成两端详细执行；Gate 2 后生成共享首图/首屏锚，检查通过才扩展。锚反馈若改变 Visual System 或文案，回退受影响执行确认。

## 二、A–C：先把“能画什么”锁清楚

### A. 任务与渠道卡

```text
交付范围：主图/副图/详情页/组合；是否实际生图
交互节奏：STANDARD / FAST / PROMPT_ONLY
目标渠道：平台、站点/地区、类目、端（搜索缩略图/商品页/广告位）
商品对象：品牌、完整品名、SKU/变体、售卖单位、在售内容物
目标用户与购买任务：谁在什么场景下做什么选择
发布时间：用于核验时效规则
动态字段：价格、优惠、赠品、库存、物流、服务承诺、活动期限
研究门状态：researchRequired / researchAvailability / researchOutlineVersion / researchGateStatus / completeApprovedEligibleSet / claimReadyCount
```

### B0. 稀疏输入与 Research Gate

若 `delivery includes detail page`、`input is product images or identity/scattered specs only`、`no complete approved evidence-eligible selling-point set` 同时成立。单一规格不豁免；未核 OCR/包装营销语不计 claim-ready：

`completeApprovedEligibleSet` 的判定读取准确 SKU/地区/版次、支撑自适应最小有效详情页所需的核心 SP、每项 `ELIGIBLE + ADOPTED` 与 F/必要 E，以及关键规格/限制/储运适配覆盖；不是 `claimReadyCount > 0`。任一缺口成立即为 false。

1. 从产品图登记 OCR/可见包装观察、不可读区域与置信度；观察不是 `F`，需目视复核并匹配品牌、SKU/条码、地区与包装版次；
2. 按 [product-research-and-copy.md](product-research-and-copy.md) 调研同 SKU 官方事实、权威类目/监管知识、买家疑问、证据入口和当前规则；
3. 先交付`调研卖点信息大纲 vX`：身份/范围、来源、已核事实、Q 地图、带 SP-ID 的候选卖点、消费者意义、证据/限定、`evidenceStatus` 四态、`selectionStatus`、Top 1–3、冲突和确认清单；
4. 用户明确采用/修改/不采用并确认 SKU/范围后，先做 claim-ready 检查：只有至少一个 `ELIGIBLE + ADOPTED` SP 才能进入含产品主张的详情页视觉设计；若没有，则等待用户补证据，或由用户明确缩小为无产品主张的身份概念范围。确认只选择信息，不能把 K/Q/竞品/U 变成事实；高风险 SP 仍需 E。

资料完整或已有用户批准且证据合格的卖点集时跳过本门，避免无意义多一次确认。联网失败时交付证据受限版，不伪造来源或静默选择相似 SKU。

### B. 资产清单与参考角色

每张输入资产同时分配**语义角色**和**传输方式**，禁止只写“参考图 1/2”或只列路径：

| 代码 | 角色 | 控制维度 | 默认传输 | 不允许 |
|---|---|---|---|---|
| `P` | Product identity 产品身份 | 几何、包装、标签、颜色、配件、液位、数量 | 按 fidelity route 使用 `LOCKED_LAYER` 或 `MODEL_REFERENCE` | 被风格/版式覆盖或改版 |
| `B` | Brand integrity 品牌完整性 | logo、字标、二维码、授权图形、监管标识 | `LOCKED_LAYER` | 由 AI 自由重画、改色、变形、补字 |
| `E` | Evidence 证据资产 | 真实证书、报告、工厂、产地、标签照片 | `LOCKED_LAYER` 或 `ANALYSIS_ONLY` | 由 AI 重画、补编号或伪造 |
| `L` | Layout 构图锚 | 大区块比例、产品位置、标题带、安全区、留白 | 授权清楚时 `MODEL_REFERENCE`，否则 `PROMPT_SPEC` | 复制第三方产品、文案、配色和专属画面 |
| `T` | Type/component 信息骨架 | 字级、对齐、卡片、表格容器和信息密度 | `PROMPT_SPEC`，编译进豆包图文 prompt | 抄原文、商标字体或参考图中的精确数据 |
| `S` | Style 风格锚 | 色盘关系、材质气质、布光、摄影后期 | 授权清楚时 `MODEL_REFERENCE`，否则 `PROMPT_SPEC` | 把参考产品、版式、文案、数据带入成品 |
| `C` | Continuity 连续性锚 | 已通过成图、场景族光色/母题、上下边缘 | `MODEL_REFERENCE` 或 `PROMPT_SPEC` | 覆盖 P/B/E/F/R 或锁死每屏构图 |

整条详情页、模板拼贴和情绪板母本使用 `SRCxx`，固定 `ANALYSIS_ONLY`；它不是模型参考角色。按语义屏边界提炼 L/T，只有自有或明确授权的完整单屏裁片才可像素直通，第三方未知权利资产只转成文字规则或去内容的抽象骨架。

冲突按维度解决：产品由 P；品牌由 B；证据由 E；主张与限制由 F/R；空间由主图策略或详情页大纲 + 当前执行稿 + L/T；风格由 Visual System + S/C。任何维度都不得反压 P/B/E/F/R。styleWeight 与 layoutWeight 独立；只支持一张参考图的模型在关系编辑路线优先 P，把 L/S 转成文字。多参考图由 provider adapter 重排并连续编号，提示词逐张写 assetId、作用和禁区，不靠数组顺序猜角色。

产品图缺失时可以完成研究、大纲/策略、文案、母版和提示词；实际图只能标为“外观未锁定的概念稿”，不能称为上架定稿。

### C. 平台×类目规则预检

不把尺寸、张数或“第五张必须如何”写成平台永久通则。但用户明确要 5 张且未自定顺序时，`FIVE_CONVERSION_ARC_V1` 是本 skill 的设计默认，不是对平台规则的声称。每次项目仍用目标平台当前官方后台/规则页核验：

```text
| R-ID | 平台/站点 | 类目 | 图位 | 当前规则 | 来源/后台路径 | 日期 | 状态 |
```

规则类型同时记录：`R-LISTING`（商品必填/普通商品页）、`R-AD`（广告/营销投放）、`R-SCENE`（搜索卡、直播卡等特定场景），不得把广告规则直接冒充普通商品必填规则。至少核验：张数与顺序、比例/像素/文件限制、首图背景和文字限制、必拍角度/标签/资质槽位、AI 内容声明、广告与普通商品图差异。公开资料不足时标 `PENDING`，让用户以上架后台实时校验为准，不能拿博客经验冒充官方规则。

### C2. 视觉基因与参考研究

产品事实检索与视觉灵感检索分开。没有明确视觉方向时，从商品身份、购买任务、货架差异、品牌正/反面气质、材料、使用场景、渠道图位和风险提取视觉基因；查询必须带 `ecommerce detail page / product photography / editorial layout / brand identity / information design` 等落点词。候选数量按项目复杂度自适应，记录来源、日期、授权状态、可借机制与泄漏风险；不固定 Pinterest、候选数量或默认强跟随。完整流程见 [visual-reference-and-style-system.md](visual-reference-and-style-system.md)。

## 三、事实、证据与动态字段

统一 ID：

- `Fxx`：已核验的本 SKU 产品事实；
- `Kxx`：权威类目通识，只能解释，不证明本产品；
- `Qxx`：买家疑问/研究洞察；
- `Exx`：真实证据资产及有效范围；
- `Rxx`：目标平台当前规则；
- `Dxx`：会变化的商业字段，如价格、活动、赠品、库存、物流；
- `Uxx`：待确认、冲突或仅为第三方宣称的信息。
- `SPxx`：调研卖点候选；本品 `ELIGIBLE` SP 必须引用合格 F，高风险主张还必须绑定范围/时效匹配的有效 E；`CATEGORY_ONLY` 可引用 K 且只能以类目/材料为主语；Q 只决定问题与顺序。

`Dxx` 在 SKU、地区、活动期限、适用条件和更新责任明确后，和其它已确认文字一样直接编译进豆包成图；变化时优先局部改字或只重生对应 job。无法确认范围就不生成该字段。免责声明不能修复无事实依据或客观虚假的大字主张；限定条件必须紧邻对应主张、清晰可读。

## 四、D–E：模块选择、大纲与主图策略卡

每个候选图/屏只有同时满足以下条件才进入方案：

1. 回答一个高优先级购买疑问；
2. 本品专属卖点有已验证 `F`；HIGH_RISK 主张强制有效且 scope 匹配的 `E`，不能只靠用户确认；`K` 仅类目教育，`Q` 只决定问题/顺序；或只是无事实暗示的真实场景；
3. 符合 `R` 与品类风险边界；
4. 不与前后图重复；
5. 视觉上能在目标端清楚表达。

状态使用 `INCLUDE / MERGE / TEXT_ONLY / OMIT / PENDING`。用户未指定主图数量时不凑 5 图；明确要 5 张时保留 G1–G5 数量，用真实的背标、细节、规格、尺度、实收、储运或使用步骤替换不安全角色，不造痛点、竞品、资质、价格或 CTA。详情页不为凑固定屏数制造空洞页面。

主图数量与角色在内部记录 `countMode / templateId / imageSlot / defaultRole / resolvedRole / adaptationReason`。`USER_EXPLICIT_5` 使用 `FIVE_CONVERSION_ARC_V1`；只有平台、类目、证据或用户自定顺序才允许改角色或顺序，并在 `adaptationReason` 解释。第五张可做购买收口，但价格、赠品、物流和仿平台按钮仍服从 D/R 边界。

详情页先从 M1–M8 选配，再以 `moduleId / moduleInstanceId / Dxx` 实例化；同一 M2 可拆成多屏，多个模块也可合并为一屏，因此 M1–M8 不等于固定 8 屏。M5/M8 为可选；M7 本身也可省，一旦入选则围绕真实未解疑问约 6 题（默认 5–7），末题必须是产品属性/用途边界 + 必要声明，不凑问题。

主图在 Gate 1 给用户看的每张策略卡固定先输出三项：

```text
图/屏名：
画面：具体主体、环境、动作、构图与第一落点
图内文案：逐字文案、换行与位置；本屏所有已确认文字都进入 VISIBLE_TEXT_BLOCK 并直接生成
设计指引：任务、层级、情绪、证据表达、移动端与风险处理；内部嵌套 Q/F/K/E/R/D、状态/待确认项、可见文字清单、countMode/templateId/defaultRole/resolvedRole/adaptationReason 或 moduleId/moduleInstanceId，以及 compositionFamily / integrationMethod / layoutFingerprint 和 L/S/T 授权状态
```

用户可见卡严格只有上述三个一级项，内部绑定不单列第四项；也不能只给三句泛化描述就直接生图。

详情页不在 Gate 1 直接倾倒全部逐屏长 prompt，而是先交付整套设计大纲：项目/事实边界、上游 `selling-point-outline@vX`（如触发）、核心设计命题、Visual System 预设、购买决策叙事、模块取舍、屏序、每屏唯一信息和逐字文案素材、视觉概念、构图/场景族、疏密、参考分工、上下承接与待确认项。设计阶段不得自行新增未准入 SP。STANDARD 只确认这份大纲；通过后才进入全部逐屏详细执行。完整格式见 [detail-page-design.md](detail-page-design.md)。

## 五、F–H：Visual System、详细执行、锚定与模型编译

### 1. Visual System Contract（可版本化 Style Bible）

锁定：品牌气质与反面边界、核心视觉母题、色盘角色与来源、字体/字级、网格与安全区、圆角/描边/图标、摄影与镜头、布光、材质、产品尺度、构图族、密度节奏、场景族、上下衔接、参考资产、styleWeight/layoutWeight、渠道适配和固定禁止项。视觉观察得到的 HEX 只能标“设计近似值”，正式品牌规范才标“精确值”。主图缩略图和详情页移动端分别检查，不能只在大画布上“看着精致”。结构见 [visual-reference-and-style-system.md](visual-reference-and-style-system.md)。

Visual System 生成项目级 `GLOBAL_CONSTANTS`；后续图不手工改写这些规则。每张最终提示词仍把常量与本图差量编译成自足文本，方便复制到不同工具。

### 2. 先完成全部逐图/逐屏详细执行稿

Gate 1 通过后，基于确认版本锁定 Visual System，并一次写完所有入选图/屏的详细执行稿，不逐页询问。每屏仍以“画面 / 图内文案 / 设计指引”三个一级项交付；其中 `COMPILED_PROMPT` 必须包含完整画布、产品、空间、镜头、光影、色彩、材质、图形、每个文字块的精确排版、连续性、禁止项与验收，不能依赖“同上”。STANDARD 在全部执行稿完成后只进行一次 Gate 2；PROMPT_ONLY 交付到这里即完成；FAST 内部自审后继续。

### 3. 先锚定再批量

- 主图套系：先做首图；必要时再做一个信息型或场景型锚点。
- 详情页：先做首屏，再为差异最大的场景族按需增加锚点；简单套系可以只有一个场景族。
- 用户提供整条参考详情页时，母本只做分析；先建立 screen map，再按目标屏型提炼 L/T。只有授权清楚的完整单屏裁片可传模型，且每个目标屏通常只用一个主 L。
- 只有存在真实待决变量时，关键锚点通常出 1–4 张低/中成本构图草案，每张只改变一个主变量；构图已严格指定时可直接做 1 张。
- Gate 2 已用于确认详细执行包。实际生成的首图/首屏与必要场景族锚使用 `Anchor Checkpoint`：检查产品是否像、层级是否对、视觉方向和真实文字是否成立。没通过就修锚点、执行稿或 Visual System，不带病批量。

### 4. 先选择产品保真路线

不能把“上传一张产品参考图并写严格锁定”称为像素锁。按画面关系选择：

| 路线 | 适用 | 产品保真口径 | 典型处理 |
|---|---|---|---|
| `PIXEL_LOCK_COMPOSITE` | 产品姿态基本不变，重点换背景/台面/外部道具 | 真实产品像素保留 | 抠图/透明产品层 + 确定性位置/裁切 + 生成背景 + 接触阴影/反射融合 |
| `RELATIONAL_GENERATIVE_EDIT` | 手握、倾倒、穿戴、遮挡、复杂反射或透视变化 | 高输入保真的**目标**，不是像素不变保证 | 生成式编辑 + 包装差异/OCR/人审；必要时恢复或重贴标签/logo |
| `CONCEPT_ONLY` | 没有用户产品图 | 外观未锁定 | 只交付概念稿，不声称上架定稿 |

生产卡必须写明路线；只有第一条可以使用“锁定产品像素”。第二条使用“最大限度保持产品身份”，并接受局部关系生成会改动相邻像素的事实。

### 5. 豆包完整图文生产优先级

```text
背景/空间底图
→ 锁定的真实产品像素或高保真产品层
→ 与产品发生关系的道具、液体、阴影、反射、遮挡
→ 色彩与光影融合
→ 图标/信息结构
→ 全部已确认中文文字、表格、价格与法务文案在同一次豆包生图中完整渲染
```

能保留真实产品像素时，优先把它作为 P/B/E 参考交给豆包，让模型在完整图文成片中保持包装、logo和证据一致。复杂倾倒、反射或遮挡使用 `RELATIONAL_GENERATIVE_EDIT` 时，以 `P` 为最高优先级并逐项验收，不能同时承诺像素完全不变。

### 6. 项目清单与五层编译

多图项目使用稳定 job ID、事实/规则 ID、资产绑定、依赖、输出版本和 QA 状态；不靠聊天顺序或模糊路径列表。结构化 manifest 至少记录：`jobId/channel/slot/family/status/fidelityRoute/dependsOn/assetBindings/promptLayers/F-E-R-D/canvas/outputs/version/inputHash/qaStatus`；主图再记录 `countMode/templateId/imageSlot/defaultRole/resolvedRole/adaptationReason`，详情屏记录 `moduleId/moduleInstanceId`。运行环境不能写文件时，在交付中提供等价逻辑表；不要声称已落盘或已自动续跑。

生产源卡可覆盖 16 类信息：任务/画幅、参考角色与传输方式、艺术方向、网格、产品锁、前中后景、卖点映射、场景道具、镜头、布光、色彩、材质、图形、文字、衔接、质量/禁止项。它是检查表，不是必须逐项堆满的句式。

把源卡编译成模型提示词时：

- `P0`：任务、产品锁、主体关系、关键动作、硬性文字白名单、绝不能出现的错误；
- `P1`：镜头、布光、材质、色盘、版式和语义道具；
- `P2`：微小装饰、颗粒、次级气氛。

分层顺序固定为：`DOUBAO_ADAPTER + GLOBAL_CONSTANTS + FAMILY_ANCHOR_RULES + IMAGE_DELTA + VISIBLE_TEXT_BLOCK + ACCEPTANCE_CRITERIA`。GLOBAL 保持项目规则逐字一致；FAMILY 锁本族光色、材质、镜头和 C；DELTA 只写该图购买问题、构图、场景、文案与变化；VISIBLE_TEXT 收录本屏所有已确认的精确可见字符串，包括标题、副标题、卖点、标签、参数、表格、FAQ、当前价格和法务；ACCEPTANCE 重复文字、产品和事实的可观察 P0。

参考资产由 provider adapter 按能力与授权筛选后重新连续编号，提示词逐张写 assetId、角色、允许和禁止维度，条数必须与实际模型参考图 1:1。`assetBindings` 的语义和传输方式比数组位置优先。只有一参考图时关系编辑优先 P；像素锁路线的 P 已在 LOCKED_LAYER 时，模型参考可按本图需要选择 C/L/S。

只编译与本图有关的字段，互相冲突时删掉低优先级要求。源卡里的坐标、占比、Hex、角度和焦段要进一步标记：`生成引导值`（模型近似遵循）、`验收测量值`（生成后检查）或`锁定约束`（真实产品/品牌/证据的合成与裁切必须保持）。不能把提示词中的数值误当成工具级精确控制。质量按可执行性、无冲突和验收结果判断，不按 500–1000 字或形容词数量判断。详见 [project-packet-and-prompt-compiler.md](project-packet-and-prompt-compiler.md)。

### 7. 迭代纪律

- 草案阶段一次只变化构图、镜头、背景或色彩中的 1–2 项；
- 选定构图后，用豆包选中成图、同会话和参考图作为连续性锚，只改变声明的变量；
- 任何图内文字错误都优先用豆包局部改字，明确“只改这个文字区，产品、构图、光影和其他文字不变”；
- 任何局部编辑后重新做产品差异、OCR 与整体物理检查；
- 每轮重述“必须保留”清单，不能只说“其他不变”；
- 产品身份或整体构图错误才整图重做；
- 使用稳定 job ID 和版本：D 价格/赠品/物流变化用豆包局部改字或重生对应 job；单张 L/构图变化只重生对应 job；某族锚变化只级联该族；P/SKU 变化才触发大范围重新验收；
- 恢复项目时先校验输入版本、依赖、hash、缺失输出和 QA，只续跑缺失/失效任务，不覆盖已通过成片。

## 六、豆包文字直出

每条可见文字使用 [project-packet-and-prompt-compiler.md](project-packet-and-prompt-compiler.md) 的 canonical schema：`textId / semanticType / claimId / sellingPointId / factIds / evidenceIds / knowledgeIds / dynamicIds / ruleIds / qualifierTextIds / exactString / renderedLines / forcedLineBreaks / region / box / width / maxLines / fontFamilyOrCategory / fontMood / fontWeight / relativeSize / colorHex / alignment / lineHeight / letterSpacing / contrastBackground / relationToProduct / validation`。键均保留，不适用标量为 null、集合为 []；按 semanticType 强制相应来源，不得退化成单一 sourceId 或模糊 style。所有已确认文字进入 VISIBLE_TEXT_BLOCK，不设短字或组数上限。

| 内容 | 豆包生成要求 |
|---|---|
| 标题、副标题、卖点、场景标签 | 逐字直出；每块完整写强制分行、框位、字体类别/气质、字重、相对字号、HEX、对齐、行距、字距、对比和产品关系 |
| 参数、营养表、配料、尺码、FAQ、法务 | 完整直出；逐格/逐题使用同一完整文字 schema、清晰卡片/表格结构和可读字号，逐项校对 |
| 价格、优惠、赠品、物流 | 当前值与 SKU/地区/期限/条件齐全时直出，变更时局部改字或重生该图 |
| 包装原字、logo、二维码、证书/报告 | 按 P/B/E 真实参考保持原样，不编造新文字、编号或图形 |

豆包 prompt 必须为每个 textId 重复展开完整 schema，并明确“这些文字必须实际出现在成图中，不得只留空白位置”；禁止“其它同上、沿用标题样式、继续逐块列出”。顶部/左侧/中央、标题区/卡片/模块、字号/HEX/渐变/阴影等设计词只是摆放指令，不能泄漏为画面文字。生成后按 renderedLines 做 OCR 精确比对，并目视检查框位、行数、字体类别/字重/字号、颜色、对齐、行距、字距、对比和产品避让；失败只修对应文字区。

## 七、I：三层验收

### 机器/规则检查

- 文件规格、比例、安全区、AI 元数据/声明；区分生成服务提供者添加标识、平台核验/展示、发布用户主动声明和各主体不得恶意删除/隐匿标识，不默认要求商家手工同时制造所有显式/隐式标识；
- OCR 与文字白名单逐字比对；检查是否泄漏位置词、结构词、设计词或参考页示例文字；
- SKU 数量、标签朝向、logo/包装差异、重复部件；
- B/E 原像素完整性、二维码扫码、证书/报告/编号 OCR；
- manifest 的稳定 ID、依赖、版本、事实引用、输出冲突和 QA 状态；实际模型参考图与逐张点名 1:1；`LONG_PAGE`/analysis-only 原图不得进入 modelReferences；
- 色盘偏差、清晰度、背景边缘、上下拼接；
- 动态字段版本与有效期。

### 设计检查

- 第一落点、信息层级、缩略图/手机端可读；
- 产品保真、材质与光影物理一致、接触阴影和遮挡合理；
- Visual System 版本一致；styleWeight/layoutWeight 作用维度没有串线；L/S/T/C 未泄漏参考产品、文案、数据、配色或占位图；
- 多图统一而不重复，layoutFingerprint 与疏密节奏合理，场景与道具有语义；
- 每图确实回答对应购买疑问。

### 商业/事实/法务人工检查

- 标题、主图、详情、SKU 与直播/投放口径一致；
- 所有产品主张可追溯到 `F/E`，比较数据写明对象、条件、来源、范围和有效期；
- 无虚构工厂、实验室、专家、证书、专利、评价或销量；
- P/B/E 和参考来源的 owner、授权状态、使用范围可追溯；analysis-only 第三方像素未进入成图生产链；
- 限定条件清楚邻接；AI 生成记录、提示词、来源和元数据按平台/法规留存。

未通过只返修受影响层或单图；上游事实/Visual System 错误才回滚所有受影响页面。
