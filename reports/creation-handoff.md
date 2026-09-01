# Creation Handoff — moyuxl-ecom-image-prompt 1.19.0

## 1. 结果

- Skill：`moyuxl-ecom-image-prompt` 1.19.0，owner：摸鱼小李，maturity：production。
- 路径：`/Users/jade01/Desktop/电商skill/ecom-image-prompt/`。
- 工作：把参考方法或稀疏产品资料转成事实可追溯、产品/品牌/证据保真、视觉系统一致，并能在豆包逐屏直接生成完整画面与文字的主图和详情页生产包；仅有产品图时先研究并确认卖点信息。
- 本轮状态：本地源文件已更新；未发布、提交或推送。

## 2. 本轮定向学习

用户明确指定本地 `ppt-image-director`。本轮完整读取其 SKILL、visual director、inspiration search、image prompt、style config、generation pipeline 与 `extract_briefs.py`，只借生产机制，不复制 PPT 专用媒介规则。

- 学到并迁移：先锁全篇叙事/视觉系统，再把逐页空间说明写到可直接生图的生产深度；同时保留整图分析/单页版式锚、场景族、全局常量 + 单页差量、可见文字隔离和任务 manifest。
- 电商化改造：把 PPT 的大纲/逐页方案真正拆成两个阶段，前者变成购买决策、模块、屏序、文案素材和视觉系统大纲，后者变成逐屏豆包执行稿；再叠加 SKU 事实、移动端、P/B/E/L/T/S/C、style/layout 独立权重、动态字段、完整文字排版和商业 QA。
- 明确拒绝：固定 14 张 Pinterest、默认 90 强跟随、第三方 IP 严格模仿、固定 70/30 图文比、只允许少量短标题或交付无字底图、Logo/二维码/证书靠 AI 重画、相邻页绝不复用版式、整套强制单渠道和仅用户点名式 QA。

已有目录研究仍保留：

- `skills-101/superpowers:product-photography`（29.3K skills.sh installs）：摄影任务与具体布光；拒绝固定镜头序列。
- `nexu-io/open-design:ecommerce-image-workflow`（1.4K installs；SkillsMP 91,306 是仓库 stars）：产品身份锚；拒绝固定 3 槽和工具绑定。
- `ajbeckliy/detail-flow:detail-flow`（31 installs）：两道门、代表切片、最小层返修；拒绝固定 8 屏/画幅。
- `medusajs/medusa-agent-skills:storefront-best-practices`（3.6K installs）：购买决策与移动优先；拒绝专属 storefront 实现和无证转化断言。

本轮没有重新跑外部目录检索。新改造来自用户对五张主图、M1–M8 详情模块和三项交付协议的明确反馈，只在已有官方证据边界上重组流程，不伪称刷新了外部证据。

## 3. v1.19.0 核心改造

1. 主图数量改为条件默认：用户明确要 5 张且未另定顺序时，使用 `FIVE_CONVERSION_ARC_V1`：G1 封面点击、G2 痛点共鸣、G3 差异卖点、G4 场景价值、G5 CTA 收口。未指定数量时才自适应。
2. 五张数量与不安全表达解耦：遇平台/类目/证据冲突，保留 5 个槽位，记录 `defaultRole/resolvedRole/adaptationReason`，用背标、细节、规格、实收、储运或用法补位，不编痛点、竞品、资质、价格或 CTA。每张可见文案不超过 5 行，相邻限定语也计入且不得为压行数删除。
3. 详情页公开协议改为 M1–M8 选配模块库：痛点、核心优势、配方/工艺、场景、品牌/资质、规格/对照、FAQ、购买/法务。模块以 `moduleInstanceId` 实例化为 Dxx 屏，可拆可合，不等于固定 8 屏。
4. M7 仍可选；入选时围绕真实疑问约 6 题（默认 5–7），末题必须是产品属性/用途边界 + 必要声明。M5/M8 仍按证据和渠道选配。
5. Stage A 模块卡、主图策略卡与 Stage B 逐屏卡都只显示“画面 / 图内文案 / 设计指引”三个一级项；证据链、状态和生产参数嵌入设计指引。

### 继承的 v1.18.0 基座

1. 新增条件 Stage 0：`详情页交付 + 只给产品图或只有身份/零散规格 + 无完整、已批准、证据合格卖点集 → 身份/OCR审计 → 整体调研 → 调研卖点信息大纲 → Research Gate 用户确认`。单一规格不豁免；“完整”须锁定 SKU/版次、足以支撑最小有效详情页、全部产品主张 `ELIGIBLE+ADOPTED` 且 F/必要 E 与关键限制齐全。
2. 卖点大纲与设计大纲分层：前者回答“哪些信息能讲”，包含来源、事实、Q、SP-ID 候选、消费者意义、证据/限定、四态准入、Top 1–3、未知冲突和确认清单；后者才回答屏数、屏序、Visual System 和逐屏设计。
3. Research Gate 高于交互节奏：FAST 不能内部代批，PROMPT_ONLY 也不能用未确认主张生成 listing-grade prompt；确认后仍做 claim-ready check，无 `ELIGIBLE+ADOPTED` SP 就补证或由用户明确缩为无主张身份概念范围。离线只交付证据受限大纲。
4. 用户确认不等于事实验证；OCR/包装观察、K、Q、竞品、搜索摘要和 U 不会因点头晋升 F。新增 Claim Eligibility Gate，VISIBLE_TEXT 必须追溯到准入 SP/claim/F；功效、比较、认证、专利、产地、检测数字、销量等高风险主张还必须绑定范围与时效匹配的 E。
5. 项目包新增 `research-selling-points.json`、`claim-set.json` 与独立 `verification-log.json`；状态链增加 sparse/identity/research/selling-point approval/claim-ready，并规定 SKU/SP 变化使对应设计、prompt 和成图失效。
6. 保留 v1.17 的`设计大纲 → 全套逐屏执行 → Anchor`、完整 typography tuple、豆包直接文字生成、P/B/E/L/T/S/C、Visual System、五层编译和局部返修。

## 4. Keep / Adapt / Reject / Invent

- **Keep**：事实账本、稀疏输入 Research Gate、产品 fidelity routes、逐屏生成、OCR/目视校对和三层 QA。
- **Adapt**：五张是用户明确数量下的默认转化叙事，而不是全平台永久规则；M1–M8 是可选信息家族，再映射为可拆合屏。
- **Reject**：未指定仍强凑五张、不看证据生成痛点/竞品/CTA、全商品强制 M1–M8、FAQ 凑题、第四个用户可见内部绑定栏。
- **Invent**：`USER_EXPLICIT_5 + FIVE_CONVERSION_ARC_V1 + defaultRole/resolvedRole/adaptationReason`、M 模块/D 屏实例双层标识，以及三项公开协议与内部追溯的嵌套结构。

## 5. 优势与证据等级

- **[design advantage]** 参考的“风格、版式、品牌、产品、证据、连续性”不再混成一张图或一个权重；长页内容泄漏有显式阻断点。
- **[design advantage]** 全局系统与单屏差量分离，最终 prompt 又保持自足，减少多图手工复述漂移并方便单图返修。
- **[design advantage]** 每段文字连同框位、硬换行、字体类别/气质、字重、相对字号、HEX、对齐、行距、字距、对比和产品关系直接进入豆包 prompt；不再让模型自行猜排版，也不交付空白卡片。
- **[design advantage]** logo/二维码/证书继续由 B/E 真实资产约束；这是事实保真规则，不是文字能力降级。
- **[validated advantage]** 触发烟测 48/48；JSON/YAML 语法通过；行为 fixture 扩展到 26 个。
- **[hypothesis / missing evidence]** 尚未完成同一豆包模型的 v1.18/v1.19 盲测、专业设计师评分、真实断点续跑或店铺 CTR/CVR 数据，不能声称已验证提升审美、事实准确率或转化。

## 6. 验证与限制

- `validate_skill.py`：`ok: true`，0 failures、0 warnings；SKILL 已回到 production context budget 内。
- `trigger_eval.py`：48/48，0 false positive，0 false negative。
- `export_skill_ir.py`：已同步到 1.19.0，intent/workflow/resources/gates 已更新。
- `evals/behavior_cases.json`：26 个静态设计契约；没有 provider runner，不能冒充 provider-backed evidence。
- `release_check.py --phase local --run-tests`：包校验、版本/报告一致性和 secret scan 通过；因该目录不是 Git 仓库、没有 `tests/`，git/branch/unit-test 门为环境性 block；provider/human output evidence 仍缺失。
- clean 桌面包与 ZIP 通过结构、单入口、JSON/YAML、链接和归档完整性检查；没有 provider/human output evidence。本轮未请求发布。
- 权限：只读网络研究可选；不声明账号、付费、发布、文件写入或 subprocess 权限。

## 7. 下一轮建议

用同一 SKU、同一产品 P、同一豆包模型做 v1.18/v1.19 对照，重点盲评五张叙事完整度、三项卡可执行性、M1–M8 选择质量、产品保真、中文准确和局部改字稳定性。
