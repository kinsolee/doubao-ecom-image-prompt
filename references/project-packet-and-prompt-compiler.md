# 项目包、生成清单与五层提示词编译

用于多图套系、完整详情页、跨渠道适配、实际生图、断点续跑或需要多轮返修的项目。普通单图分析和少量 prompt-only 任务不必制造完整文件树，但仍使用同一套字段和编译逻辑。

本协议借鉴“全局常量、锚定图、单页差量、机械任务清单”的生产思想，并针对电商增加 SKU、事实、动态字段、资产处理路线、完整可见文字和商业 QA。它是**逻辑交付协议**：宿主不允许文件写入时，以等价结构化内容交付；只有运行环境支持且用户要求持久化时才落盘。

## 一、三种交互节奏

| 模式 | 何时使用 | 对话停留 | 不能跳过 |
|---|---|---|---|
| `STANDARD` | 默认的实际多图生产 | Gate 1 确认主图策略/详情页大纲；Gate 2 统一确认全部详细执行稿；Anchor Checkpoint 检查首图/首屏 | 事实、规则、产品锁、两阶段设计、锚定、完整可见文字、三层 QA |
| `FAST` | 用户明确“直接做、无需确认、快速完成” | 不暂停；内部依次完成大纲/策略、全部执行稿与锚定自检，只在最终交付 | 与 STANDARD 相同，仅省略对话等待 |
| `PROMPT_ONLY` | 用户只要方案、文案、提示词或生产包 | 按“大纲/策略 → 全部详细执行稿”交付，不经过实际生图 Gate | 完整大纲/策略、视觉系统、参考协议、五层提示词、图文设计和验收 |

模式只改变正常交互节奏，不改变事实标准和生产深度。不要为了“快”把 FAST 变成无计划批量抽奖；也不要用生成 Gate 阻塞资料完整的 PROMPT_ONLY。详情页只给产品图，或只有身份/零散规格，且没有完整、已批准、证据合格的本 SKU 卖点集时，Research Gate 作为身份范围与卖点选择门，FAST 不能内部代批，PROMPT_ONLY 也必须等待用户确认。

## 二、单一来源与稳定 ID

稀疏输入经 Research Gate 确认的`调研卖点信息大纲`是可用卖点选择源；经 Gate 1 确认的主图策略或详情页大纲是内容/叙事源；经 Gate 2 确认的逐图/逐屏执行稿是生成源；视觉系统契约是设计源；事实账本与验证记录才是主张证据源。各自版本化，不靠聊天历史猜测。

`approval-log` 只记录用户批准了哪些卖点选择、文案或设计；`verification-log` 记录事实怎样被验证。批准不会改变 `F/K/Q/E/R/D/U` 状态。只有用户明确以第一方身份补充低风险事实且给出 SKU/地区/版本/条件时，才新建 `F(USER_CONFIRMED)`；高风险主张仍需合格证据。PROMPT_ONLY 记录 `delivered`，FAST 记录 `internal_fast`，但二者都不能伪造 Research Gate 用户确认。

- 商品图库图号使用稳定 ID，如 `G01`、`G02`；详情屏使用 `D01`、`D02`；锚点使用 `A-族名-01`；
- 局部返修不改图号，产生 `G01-v2` 等版本；删除的 ID 保留 tombstone，后续图不重排；
- 每张图绑定明确的 `Q/F/K/E/R/D/U`，不能只存一段自然语言；
- 已确认策略、视觉系统或事实发生变化时增加版本，记录修改人/时间/原因和受影响任务；
- 不覆盖已通过成图；新版本通过后再把旧版标 `superseded`。

## 三、轻量项目包

可写文件时建议采用下列逻辑结构；不可写文件时按同样章节输出：

```text
project-packet/
├── project.json                    # 渠道、SKU、交付范围、模式、状态
├── facts-ledger.json               # F/K/Q/E/R/D/U 与来源/版本
├── research-selling-points.json    # 稀疏输入的 SP 候选、状态、来源和用户选择
├── claim-set.json                  # claimId、风险、批准原文、允许变换、限定与适用范围
├── main-image-strategy.json        # 主图任务的 Gate 1 策略；不适用则省略
├── detail-outline.json             # 详情页 Gate 1 大纲；不适用则省略
├── style-system.json               # 视觉系统契约与版本
├── screen-execution/               # Gate 2 的逐图/逐屏详细执行稿与完整 prompt
├── approval-log.json               # 卖点选择/文案/设计批准；不等于事实验证
├── verification-log.json           # OCR/SKU/来源/证据验证与冲突解决
├── assets/
│   ├── product/                    # P
│   ├── brand/                      # B
│   ├── evidence/                   # E
│   ├── continuity/                 # C
│   ├── style/                      # S；仅有权进入项目的资产
│   ├── layout/                     # L/T 裁片、线框与信息层级规格
│   └── sources-analysis-only/      # SRC 长页/竞品/情绪板，不进入模型输入
├── briefs/                         # 每图 IMAGE_DELTA 与编译后完整 prompt
├── manifests/
│   ├── anchors.json                # 必要锚点任务
│   └── images.json                 # 其余独立任务
├── outputs/
│   ├── final/                      # 豆包生成的完整图文成片
│   └── revisions/                  # 局部改字或重生后的版本
└── qa/
    ├── machine.json
    ├── design.json
    └── commercial.json
```

临时候选和搜索缓存不进入正式项目包；只有选定资产、必要来源记录和衍生关系进入。

## 四、资产记录：角色与处理路线显式化

不能只写一个路径数组。每项资产至少记录：

```json
{
  "assetId": "L03",
  "role": "L",
  "assetKind": "full-screen-layout-crop",
  "owner": "user-or-source-owner",
  "authorization": "owned | licensed | analysis-only | unknown",
  "source": "assets/layout/editorial-03.png",
  "derivedFrom": "SRC01",
  "derivation": {
    "screenType": "editorial",
    "fullScreen": true,
    "cropValidated": true,
    "nativeResolution": true
  },
  "transferMode": "MODEL_REFERENCE",
  "allowedDimensions": ["title-band", "grid", "safe-area", "whitespace"],
  "forbiddenDimensions": ["product", "copy", "brand", "color", "decoration"],
  "weight": 65,
  "integrityCheck": ["no-source-copy", "no-logo", "no-example-text"]
}
```

允许的 `transferMode`：`ANALYSIS_ONLY / MODEL_REFERENCE / LOCKED_LAYER / PROMPT_SPEC`。完整规则见 [visual-reference-and-style-system.md](visual-reference-and-style-system.md)。`B/E` 默认是 LOCKED_LAYER；`T` 作为信息层级、字级、卡片和表格骨架编译进 `PROMPT_SPEC`，最终文字仍由豆包直接生成；整条长参考母本只能 ANALYSIS_ONLY。

## 五、提示词五层编译

逐图源卡先分层，最终再编译为一条自足提示词：

```text
provider adapter
+ GLOBAL_CONSTANTS
+ FAMILY_ANCHOR_RULES
+ IMAGE_DELTA
+ VISIBLE_TEXT_BLOCK
+ ACCEPTANCE_CRITERIA
= COMPILED_PROMPT
```

分层的目的不是减少最终提示词信息，而是保证多图共用规则逐字一致、单图只承载自己的变化，避免每页手工改写导致漂移或冲突。

### 1. `GLOBAL_CONSTANTS`

项目级稳定内容，版本变化会影响多个任务：

- 渠道/端/画布身份和平台 `R` 的稳定约束；
- SKU、产品身份不变量和所选产品保真路线；
- B/E 的原像素处理要求；
- Visual System Contract 的品牌正/反面气质、色彩角色、字体、图形、摄影、布光、材质和固定母题；
- 全局网格、安全区、字级层级、移动端与缩略图标准；
- 参考权利与第三方排除；
- 全局可见文字格式、AI 记录和固定禁止项。

不要放入：本图专属文案、局部道具、某一屏的镜头、会变化的 `D` 字段、未核验的 U、整张第三方参考内容。全局常量不能用模糊的“保持一致”；必须写出实际规则。

### 2. `FAMILY_ANCHOR_RULES`

按真实视觉差异建立，常见维度是环境、光型、镜头、材质关系和连续母题，而不是页面标题。内容包括：

- 族名、锚点 ID 与已通过锚图版本；
- 本族允许的场景、主光、色温、镜头范围、背景材质、产品尺度和上下过渡；
- C 的允许/禁止控制维度；
- 与其他族需要保持一致的品牌和产品元素；
- 本族最常见的漂移风险。

简单套系可以只有一个族并单阶段生产；复杂套系才增加锚点。不要固定 2–4 族或每族都做多张草案。

### 3. `IMAGE_DELTA`

只写该图相对项目常量与族规则的变化：

- 图位、唯一购买问题和已准入的 `SP/Q/F/K/E/R/D`；
- composition family、integration method、layout fingerprint；
- 本图产品位置/尺度/状态、前中后景、动作、道具和接触关系；
- L/T 的局部空间骨架；
- 镜头、焦点、景深、光影和材质的本图差量；
- 本图所有已确认可见文字及完整信息区；
- 与相邻图的承接与区别；
- 针对性禁项。

评审字段保持紧凑，生产深度集中在 IMAGE_DELTA 和最终 COMPILED_PROMPT。不要把“设计意图”写成大段，却把真正的产品、空间、光、材质和文字位置缩成一句。

### 4. Claim Eligibility Gate

每个产品主张先建 `claimId`，至少记录：`semanticType`、`sellingPointId`（稀疏输入的 PRODUCT_CLAIM 必填）、`factIds[]/evidenceIds[]/knowledgeIds[]/dynamicIds[]/ruleIds[]`、`riskClass`、`evidenceRequirement`、`approvedExactCopy`、`allowedTransforms`、`prohibitedTransforms`、`qualifierTextIds[]`、适用 SKU/地区/期限。编译前逐条检查：

- SP 同时满足 `evidenceStatus=ELIGIBLE` 与 `selectionStatus=ADOPTED` 才可作本品主张；`CATEGORY_ONLY+ADOPTED` 只作教育；`MODIFIED` 必须重审；
- 本品陈述必须有状态和 scope 都匹配的 `F`；功效、比较、认证、专利、产地、检测数字、销量等 `HIGH_RISK + REQUIRED` 必须另有有效且 scope 匹配的 `E`，`F(USER_CONFIRMED)` 单独不够；
- `K` 只以类目/材料为主语做教育，`Q` 只决定问题与顺序，`E` 不能脱离字段级 `F` 单独证明文案；
- `D` 必须有 SKU、地区、期限、条件与更新责任；
- `U/CONFLICT`、未批准/待核/禁用 `SP`、竞品/评论/搜索摘要不得进入可见文案；
- VISIBLE_TEXT 的 `exactString` 必须等于 `approvedExactCopy` 或只做 allowedTransforms；加强结论、用户 MODIFIED 文案都重新审核；必需限定由 `qualifierTextIds` 同屏邻接；
- 没有合格 claim 的文字块应删除、降为准确身份/类目教育或保持 PENDING，不阻塞无关屏继续。

`semanticType` 仅用：`PRODUCT_CLAIM / OBJECTIVE_FACT / CATEGORY_EDUCATION / DYNAMIC / NAV_LABEL / LEGAL`。PRODUCT_CLAIM 强制 claim/SP/F；OBJECTIVE_FACT 强制 F；CATEGORY_EDUCATION 强制 K 且主语不得指向本品；DYNAMIC 强制 D；NAV_LABEL 可无 claim；LEGAL 绑定适用 R/F。这样非主张标签不被过度建 claim，产品主张也不能借“sourceId 可选”逃逸。

canonical 字段的**键均须存在**，不代表每种语义都伪造非空来源；不适用标量写 `null`，不适用集合写 `[]`。非空矩阵：PRODUCT_CLAIM 要 `claimId + sellingPointId + factIds`（高风险另须 `evidenceIds`）；OBJECTIVE_FACT 要 `factIds`；CATEGORY_EDUCATION 要 `knowledgeIds`；DYNAMIC 要 `dynamicIds`；NAV_LABEL 不强制来源；LEGAL 至少要适用的 `ruleIds` 或 `factIds`。其余来源字段按真实依赖填写。

### 5. `VISIBLE_TEXT_BLOCK`

每条可见文字独立记录，不能把布局说明混在可见字符串里，也不能把全部排版参数压缩进一个 `style` 字段：

```json
[
  {
    "textId": "TXT-D03-01",
    "semanticType": "OBJECTIVE_FACT",
    "exactString": "配料：苹果汁",
    "renderedLines": ["配料：苹果汁"],
    "forcedLineBreaks": false,
    "route": "MODEL_TEXT",
    "claimId": null,
    "sellingPointId": null,
    "factIds": ["F02"],
    "evidenceIds": [],
    "knowledgeIds": [],
    "dynamicIds": [],
    "ruleIds": [],
    "qualifierTextIds": [],
    "region": "左上安全区",
    "box": {"xPct": 7, "yPct": 8, "wPct": 46, "hPct": 16},
    "width": "46% canvas",
    "maxLines": 2,
    "fontFamilyOrCategory": "modern geometric Chinese sans-serif",
    "fontMood": "clean, fresh, editorial",
    "fontWeight": 800,
    "relativeSize": "5.6% canvas height",
    "colorHex": "#D71920",
    "alignment": "left",
    "lineHeight": "1.05em",
    "letterSpacing": "0.01em",
    "contrastBackground": "high contrast on #FFF8F3",
    "relationToProduct": "right edge keeps 8% gap from product silhouette",
    "validation": ["OCR_EXACT", "VISUAL"]
  },
  {
    "textId": "TXT-D03-02",
    "semanticType": "OBJECTIVE_FACT",
    "exactString": "净含量280mL",
    "renderedLines": ["净含量280mL"],
    "forcedLineBreaks": false,
    "route": "MODEL_TEXT",
    "claimId": null,
    "sellingPointId": null,
    "factIds": ["F04"],
    "evidenceIds": ["E03"],
    "knowledgeIds": [],
    "dynamicIds": [],
    "ruleIds": [],
    "qualifierTextIds": [],
    "region": "下半屏信息板",
    "box": {"xPct": 8, "yPct": 58, "wPct": 84, "hPct": 7},
    "width": "84% canvas",
    "maxLines": 1,
    "fontFamilyOrCategory": "Chinese sans-serif",
    "fontMood": "precise, neutral",
    "fontWeight": 700,
    "relativeSize": "3.1% canvas height",
    "colorHex": "#232323",
    "alignment": "center",
    "lineHeight": "1.0em",
    "letterSpacing": "0em",
    "contrastBackground": "high contrast on #FFFFFF",
    "relationToProduct": "information board remains clear of product and shadow",
    "validation": ["OCR_EXACT", "VISUAL"]
  }
]
```

豆包工作流中，已确认的标题、副标题、卖点、标签、参数、表格、FAQ、当前价格和法务全部使用 `MODEL_TEXT`，它们的 `exactString` 全部进入模型可见文字白名单。每条 `MODEL_TEXT` 上述字段全部必填，最终 `COMPILED_PROMPT` 为每条逐项展开，不允许“同上”或样式继承。只有本屏明确不需要文字时使用 `NONE`。包装、logo、二维码和证据中已存在的文字由 P/B/E 参考保持，不另造内容。

模型可见字符串禁止混入：顶部/左侧/中央等位置词，标题区/卡片/模块等结构词，字号/HEX/渐变/阴影等设计词，步骤 1/要点 A 等无事实占位词。除非这些词本来就是用户确认的真实文案。生成后出现任何未列白名单的文字、数字、logo 或指令词都失败。

### 6. `ACCEPTANCE_CRITERIA`

重复最重要的 P0 可观察条件，而不是再堆风格形容词：

- 产品几何、包装、标签方向、颜色、数量、状态；
- B/E 完整性与二维码/编号/OCR；
- 第一落点与手机端/缩略图可读；
- 主张对应 F/E/R，U 不出现；
- 关键物理关系、接触阴影、反射/折射；
- 全部可见文字的分区 OCR/逐字校对；
- L/S/T/C 无内容泄漏；
- 本图画幅、安全区和衔接。

## 六、完整任务 manifest

manifest 是运行时真相，不依赖参考图数组的隐含顺序：

```json
{
  "schemaVersion": "1.0",
  "projectId": "apple-juice-280ml-detail-cn",
  "jobId": "D03",
  "version": 2,
  "channel": "detail",
  "slot": "screen-03",
  "countMode": null,
  "templateId": null,
  "imageSlot": null,
  "defaultRole": null,
  "resolvedRole": null,
  "adaptationReason": null,
  "moduleId": "M3",
  "moduleInstanceId": "M3-01",
  "family": "studio-light",
  "status": "execution_approved",
  "researchAvailability": "online",
  "researchGateStatus": "approved",
  "completeApprovedEligibleSet": true,
  "claimReadyCount": 3,
  "sellingPointOutlineVersion": "selling-point-outline@v1",
  "factsLedgerVersion": "facts-ledger@v3",
  "verificationLogVersion": "verification-log@v2",
  "claimSetVersion": "claim-set@v2",
  "mainImageStrategyStatus": "not_applicable",
  "detailOutlineStatus": "approved",
  "outlineVersion": "detail-outline@v2",
  "executionVersion": "screen-execution/D03@v2",
  "approvalMode": "user",
  "generationStatus": "requested",
  "fidelityRoute": "RELATIONAL_GENERATIVE_EDIT",
  "dependsOn": ["A-studio-light-01@v1"],
  "assetBindings": [
    {"assetId": "P01", "role": "P", "transferMode": "MODEL_REFERENCE", "required": true},
    {"assetId": "B01", "role": "B", "transferMode": "LOCKED_LAYER", "required": true},
    {"assetId": "C01", "role": "C", "transferMode": "MODEL_REFERENCE", "required": false},
    {"assetId": "L03", "role": "L", "transferMode": "MODEL_REFERENCE", "required": false},
    {"assetId": "S01", "role": "S", "transferMode": "PROMPT_SPEC", "required": false}
  ],
  "promptLayers": {
    "globalConstants": "style-system@v2 + sku-invariants@v3",
    "familyAnchor": "studio-light@v1",
    "imageDelta": "briefs/D03-delta@v2",
    "visibleText": "TXT-D03@v2",
    "acceptance": "QA-D03@v2"
  },
  "facts": ["F02", "F04"],
  "sellingPoints": [],
  "claims": [],
  "evidence": ["E03"],
  "rules": ["R01"],
  "dynamicFields": [],
  "canvas": {"aspect": "adaptive-by-R01", "safeArea": "measured"},
  "outputs": {
    "final": "outputs/final/D03-v2.png",
    "revisions": "outputs/revisions/D03/"
  },
  "inputHash": "provider-neutral-hash",
  "qaStatus": "pending"
}
```

`assetBindings` 表达语义与处理路线；provider adapter 再生成实际 `modelReferences`、连续编号、提示词点名和 API 参数。不可把“路径排第一”当作 P 最高优先的唯一保证。

## 七、锚点任务与批量任务

- `anchors` 只包含真正需要先生成/确认的场景族代表；
- `images` 中其余任务通过 `dependsOn` 引用已通过锚点；
- 每个 listing-grade 图都必须能访问 P 或锁定 P 层，不能只靠第一张锚图记住产品；
- 锚图自身也可以使用授权的 S/L，但必须遵守逐张点名和权利门；
- 同一视觉家族尽量使用同一模型、版本、设置与锚定链；不同渠道任务可以共享 P/事实/视觉系统，但按各自 R、画幅、文字和 CTA 规则形成独立 job；
- 单图任务不制造空的 anchors manifest；简单小套系可单阶段并行。

Gate 2 通过代表全部详细执行稿已确认，不代表锚图已生成。Anchor Checkpoint 另行更新锚点状态和版本；FAST 模式用同样 QA 内部通过后自动继续；PROMPT_ONLY 只输出计划锚点与依赖，`generationStatus=not_requested`，不伪造已生成输出或用户批准。

## 八、豆包 adapter 编译检查

本 skill 默认在豆包中使用，文字能力不再做分级或降级判断。实际投喂前只核对当前豆包入口的参考图上限、输入保真、局部编辑、尺寸/比例、会话连续性和输出元数据。然后：

1. 按 fidelity route 决定 P 是 LOCKED_LAYER 还是 MODEL_REFERENCE；
2. 删除无权或无法直接传入的参考，把 L/S/T 转成 `PROMPT_SPEC`；
3. 仅一参考图时优先 P；P 已像素锁时按本图任务选择 C/L/S；
4. 对实际模型参考重新编号 1…N；
5. 在 prompt 中逐张写 assetId、角色、允许与禁止维度；数量必须 1:1；
6. 编译五层为自足中文提示词，检查重复、矛盾和长度预算；
7. 把本屏全部 `MODEL_TEXT` 逐字编译进豆包 prompt；P/B/E 参考仍不得被改成虚构内容。

提供商改变时只替换 adapter，不重写事实、策略与视觉系统。

## 九、断点续跑与影响范围

状态分两支。只有 `SPARSE_RESEARCH_REQUIRED=true` 才走：`sparse_input_detected → identity_resolved → research_complete / research_outline_evidence_limited → selling_point_outline_draft → selling_point_outline_approved(user) → claim_ready_check`。资料本来完整时走：`existing_approved_set_validated → claim_ready_check`，不得重新生成或等待卖点大纲。两支随后共同进入：`outline_draft → outline_approved → execution_draft → execution_approved → anchor_generated → anchor_checked → queued → generated_complete → text_checked → qa_pass / qa_fail → superseded`。无论联网状态如何，只有存在至少一个 `ELIGIBLE + ADOPTED` 的 SP 才能从 `claim_ready_check` 进入含产品主张的 outline；若没有，转 `needs_user_evidence`，或由用户明确缩小并批准 `safe_identity_only_scope → outline_draft`。离线/部分联网时标记 `research_outline_evidence_limited`；清晰包装/用户资料若产生可验证 LOW_OBJECTIVE F，仍可形成 ELIGIBLE SP 并经用户 ADOPT 后继续。不得把用户批准、`NO_CLAIM_READY_RESULT` 或 evidence-limited 误记成有合格主张。

主图单独任务可用 `strategy_draft → strategy_approved` 替代 outline；**主图+详情页组合不能互相替代**，分别记录 `mainImageStrategyStatus` 与 `detailOutlineStatus`，两者按模式达到 approved/delivered 后才进入共同执行与共享锚。`approvalMode` 记录 `user / internal_fast / not_requested`，但 `selling_point_outline_approved` 只允许 `user`；PROMPT_ONLY 资料完整时用 `outline_delivered → execution_delivered`，稀疏输入停在卖点大纲待确认。

恢复项目时先校验项目版本、输入 hash、依赖锚点、资产缺失和已有输出，只续跑缺失或失效任务。不要无理由从研究和首图重新开始。

变更影响：

| 上游变化 | 默认重建范围 |
|---|---|
| `D` 价格/赠品/物流变化 | 使用豆包局部改字，无法稳定局部编辑时重生对应 job |
| 任何图内文案校对 | 只修正对应 `textId` 的文字区；不因一个错字重生整套 |
| 单张 L 或本图构图变化 | 只重生对应 job |
| 某场景族光色/锚点变化 | 重生该族依赖任务，其他族保持 |
| 某 F/E/R 变化 | 只使引用该 ID 的任务失效；全局身份/禁令例外 |
| SP 状态、产品身份/SKU/地区/版次变化 | 使对应 SP、claim、设计大纲和所有下游引用任务失效；重新经过必要确认 |
| Style System 全局变化 | 按变更维度和族依赖计算，不默认覆盖已通过产品层 |
| P 包装、SKU 或售卖单位变化 | 所有依赖该 P/SKU 的成片重新验收，通常大范围失效 |

每次修复记录“保留项、变化项、原因、受影响 ID、输出版本和 QA”。

## 十、何时才值得增加脚本

当前协议优先稳定结构化字段，不用正则解析自由 Markdown。至少经过多个真实项目并证明策略结构稳定、存在手抄漂移后，再考虑确定性工具。首个值得做的脚本应是 manifest validator，而不是绑定某个模型的生成器；它至少检查：

- listing-grade job 缺 P 或 P 路线不合法；
- 悬空/循环依赖、重复 output、未知 F/E/R/D；
- 稀疏输入缺卖点信息大纲/用户确认，或下游引用未准入 SP、无合格 F 的 claim；
- 每个 claim-bearing textId 的 claim/SP/F/E 必须存在于 manifest，manifest 的 claim 集与全部 claim-bearing textId 精确一致；其它非文案依赖另字段记录；
- 每个 `qualifierTextId` 必须指向同屏存在且实际进入 `MODEL_TEXT` 的文字；其 exactString 同样经过批准，框位与被限定主张邻接，并通过移动端可读性检查；
- factsLedger/verificationLog/claimSet 版本或 hash 与编译时不一致；
- `LONG_PAGE` 或 analysis-only 原图进入 model references；
- B/E 走自由生成、二维码缺扫码检查；
- 已确认可见文案没有全部进入 `MODEL_TEXT` 或只交付无字底图；
- D 当前值未绑定 SKU/地区/期限/条件就生成入图；
- 模型参考点名数量不一致；
- input hash、版本或 QA 状态缺失。

在没有真实脚本和运行证据前，只能把本协议称为设计契约，不能声称已机械验证或支持自动断点续跑。
