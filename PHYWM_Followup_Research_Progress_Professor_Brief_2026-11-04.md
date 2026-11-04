# PHYWM 后续研究：阶段性调研、实验验证与选题判断

> **研究进展汇报（供 Prof. Yanhai Xiong 讨论）**  
> **日期：2026 年 11 月 4 日**  
> **研究主线：** 从 *Where Explicit Physics Pays Off in Manipulation World Models*（PHYWM）出发，探索具有独立科研价值的后续问题；同步考察 A / B / C 三个研究领域。  
> **当前状态：** 文献审查、已完成实验的机制诊断与 Novelty Check；**尚未确立可以直接立项的新算法或独立论文贡献**。

## 0. Executive Summary

PHYWM 的核心发现是：**对 World Model 预测有帮助的物理信息，不一定能提高机器人的实际决策能力；其价值取决于观测历史、估计可靠性、模型如何使用信息，以及下游规划能否利用预测变化。**

围绕这一发现，近期开展了两种相互补充的探索：

1. **直接延续 PHYWM。** 从其 Tipping、DINO-WM、History、自回归规划等负面或反直觉现象出发，检查是否存在尚未解决、足以支撑独立研究的机制问题。
2. **能力问题驱动的 A/B/C 调研。** 分别研究 **A. World Models for Efficient Robot Adaptation**、**B. Reliable Learning from Imagined Experience**、**C. Physics-Generalizable Robot Learning**，寻找现有强模型在真实操作任务中反复失败的场景，而不是预先规定新方法必须使用 explicit physical factors。

**截至目前的主要结论：**

- **观察到了真实能力缺口，但尚未识别出确定的新机制。** 物理变化下的策略鲁棒性、World Model 的失败预测可靠性、以及经验/物理知识的迁移均有强外部证据；但各领域已有不少直接竞争方法。
- **已有内部实验提供了重要的否定性约束。** Tipping 960 回合比较没有验证按物理参数选择策略优于强统一控制策略；Catch Ramp Ball 32 回合 pilot 中，强 reactive 和简单解析方法已达到与受限 Oracle 相同的观测成功率。不能为了保留假设而改动阈值或重选任务。
- **较有潜力的下一步不是直接训练新 World Model，而是先核查两类尚未完全厘清的失败：** (P1) 动作失败预测在隐藏物理条件变化下是否失真；(P2) 已有物理知识或失败修复能否跨任务减少真实适应成本。
- **研究决策：** A/B/C 全部保留；P1/P2 进入进一步 Novelty/Feasibility Gate；**暂不启动新架构训练或大规模 benchmark 开发**。

**证据分类：** **[I]** = 本组 PHYWM 原稿和现有实验记录；**[E]** = 外部论文作者报告（已核对公开论文/项目页，但未独立复现）；**[H]** = 尚待验证的科学假设。下面严格区分三者。

---

## 1. 从 PHYWM 原论文出发：哪些结果值得延续？ [I]

原稿：*Where Explicit Physics Pays Off in Manipulation World Models*，参见内部 `PHYWM.pdf`。

原论文比较 Implicit、History、History + Estimated Factor、History + Oracle Factor，在 ManiSkill 中使用 PointWorld 与 DINO-WM，覆盖 articulated mobility、clutter 和 tipping 等六类场景，控制训练与优化预算，并同时评估预测和闭环 MPC。

| 原论文现象 | 核心数字 / 实验 | 留下的科学问题与限制 |
|---|---|---|
| **小预测收益，大行为变化** | 四个存在 mobility 歧义的场景：estimated factor 的 moving-point error 约降低 **5%**；locked 无效推行程 **385→241 mm（−37%）**，limited **307→239 mm（−22%）** | 改善预测的“哪里”比平均预测误差更重要；但强力停止规则等简单基线仍必须比较 |
| **Prediction metric 与 control 不一致** | Phantom target motion 与 wasted push 的相关系数 **r=0.81**，普通 moving-point error 为 **r=0.62** | 提供有价值的行为诊断，但“decision-relevant error 比 MSE 更有用”并非新的一般理论 |
| **History 已经足够时，额外估计未必有益** | Sliding window locked wasted push：History **146 mm**、Estimated **152 mm**、Oracle **2 mm** | 信息冗余、估计误差、硬投影及规划敏感度尚不能混成单一机制 |
| **Oracle 有效，Estimator 反而可能有害** | Tipping topple：History **38.7%**、Estimated **42.6%**、Oracle **24.9%**；estimated friction 与真实值 **r≈0.08** | 估计质量不足是强解释；不能立即推出新不确定性模块会提升控制 |
| **预测响应 Factor，但 Planner 不受益** | DINO-WM 在 locked clutter 中，History **378 mm**、Estimated **390 mm**、Oracle **386 mm**；正确 factor 会改变预测特征 | 可能存在 model–planning-cost interface mismatch，但仅修改 image cost 有较高 incremental 风险 |
| **History 甚至降低成功率** | Storage free success：Implicit **63%**、History **48%** | 模型预测再喂回 history 可能产生 distribution shift；需要严格固定候选动作和历史更新策略来确认机制 |

**关键因果限制：** Estimated Factor 不只是一个独立的 factor token；它涉及额外估计信息（部分来自模拟器分割和交互力）及结构性 output projection。原论文的闭环差异不应直接被归因为“表示学习自动理解了物理”。

**可继承的核心思想：** 不只问模型是否拥有物理知识，而要问**知识是否可获取、是否被准确用于预测、是否能改变真实决策，以及是否超越简单控制与解析方法**。

---

## 2. 已开展的研究工作与实际实验 [I]

### 2.1 研究思路的演进

| 阶段 | 主要工作 | 阶段发现 / 研究决策 |
|---|---|---|
| **初步 PHYWM follow-up** | 审查 Factor Injection、Action Ranking、DINO planner、Tipping 估计失败；比较预测误差与真实决策收益 | 单纯增加 ranking / regret 分析或新的 cost module，难以保证独立性；先重新审查能力问题和相关理论 |
| **A/B/C 系统文献调研** | 精读/对比 adaptation、imagined RL、physics generalization 的代表性论文；重新核对论文中的失败与强 baseline | 保留 A/B/C，不强行合并成一个 physics-aware WM；将 novelty 和 capability gap 置于模型设计之前 |
| **Round / Gate 系列机制验证** | 对 Tipping 隐藏摩擦/COM 的动作差异、任务可行性、恢复状态和强策略进行受控检验 | 有些条件下存在描述性策略差异，但未证明普遍、可泛化的物理条件选择收益；严格同态 replay 未获可靠验证 |
| **P2d Catch Ramp Ball pilot（11 月 3–4 日）** | 对固定放置、强 reactive、简单解析预测、独立球未来位置 Oracle 做冻结控制和独立 reset | 强简单方法已经解决观测到的有限任务；没有形成学习型 World Model 的独立成功率机会 |
| **Failure-Driven Audit（11 月 4 日）** | 从 Warp RL、JEPA-TTT、WorldFailBench、AMD/CureWM 等**已报告的真实失败**出发审查，而非构造新的困难任务 | 找到了剩余性能缺口，但尚未证明因果原因是缺失 World Model 或显式物理知识 |
| **Novelty / Repair Gate（11 月 4 日）** | 对 Failure Boundary、Counterfactual Failure、Physical Diagnosis、Decision Sufficiency 做直接竞争检查 | P1/P2 有可能继续深入；简单 Failure Detector、factor adapter、event verifier 等路线不值得直接启动 |

### 2.2 Tipping / Gate5–6：960 回合对照（已执行）

原始报告：内部 `REPORT.md`，2026-11-02。

- **预先固定设计：** 20 布局 × 6 个摩擦/COM 组合 × 4 策略 × 2 重复，共 **960 个随机独立复位回合**，每策略 240 回合；不是严格同一初始完整物理状态的反事实实验。
- 主成功率：低接触退后—重新接触策略 **76.2%**；`push_4mm` **65.4%**；`push_8mm` **65.0%**；`hold` **0%**。
- 留一布局（LOLO）选择：根据物理条件选策略 **75.0%**，无需物理信息的统一低接触策略 **76.2%**；差约 **−1.25 个百分点**，95% CI **[−5.42, +6.67]**。
- **结论：Gate6 No-Go。** 未达到预注册的物理条件选择增益标准；对不同摩擦/COM 条件的描述性优势不能当成已被证实的稳定策略反转。严格 replay 在此前诊断中亦未可靠成立。

**对选题的影响：** 不能仅凭“物理参数会改变动力学”就认为“知道参数必然比强统一控制策略更有价值”。应先证明存在实质、可泛化的残余决策收益。

### 2.3 P2d Catch Ramp Ball：32 回合 pilot（已执行）

原始报告：内部 `EXECUTION_REPORT.md`，2026-11-03—04。

| 控制器 | Native success | 备注 |
|---|---:|---|
| C0：固定目标放置 + feedback | **0/8** | 其中 1 回合未通过预先冻结的上游参考前缀一致性检验 |
| C1：当前/历史状态 Reactive | **8/8** | 使用有权限的当前/历史状态与反馈控制 |
| C2：简单解析轨迹预测 | **8/8** | 使用运动历史和已知几何 |
| C3：独立 ball-only futureXY Oracle | **8/8** | 仅提供独立测定的球未来 XY；并非全知最优策略 |

- C1/C2/C3 均观察到实际球杯接触与停止后保持的**代理证据**，但不是严格证明任意任务上的物理捕获。
- 原冻结预注册整体判定 **INCONCLUSIVE**（包含一项前缀完整性未通过）；样本每组仅 8 个，有明显天花板效应。
- **实际意义：** 尚无证据表明额外未来信息在该设置中提供强 reactive / 解析方法无法达到的成功率收益；不启动 P2e 或新大型模型训练。

### 2.4 本轮“新完成”的工作边界

最近的 A/B/C、Failure-Driven 和 Novelty 文档主要是**论文审读、结果复算、对照设计与风险核查**；**没有**在这些文档撰写过程中额外完成 Warp RL、JEPA-TTT、WorldFailBench 或 CureWM 的独立训练复现。公开论文中的数字均属 **[E] 作者报告**，不能描述为我们的实验发现。

---

## 3. A / B / C 三条研究路线：读了什么、说明什么？ [E]

### A. World Models for Efficient Robot Adaptation

**关注问题：** 机器人遇到新的任务或动力学变化时，如何在有限真实交互下快速恢复能力，并保留可重复利用的知识？

| 论文 | 主要方法/结果 | 对我们的启示及 Novelty 约束 |
|---|---|---|
| [MiDAS — *What Does It Take For A Generalist Robot Policy To Adapt Using Minimal Data?*](https://arxiv.org/abs/2608.11363) | 1/few demonstration + 在线 value/residual RL；部分物体位置 OOD 失败可被额外 curriculum 显著缓解 | “在线 RL 只能微调动作”并不成立；论文中的 object-swap 失败**不是物理参数变化** |
| [Warp RL — *Reshaping Base Policy Distributions for Dynamics Adaptation*](https://arxiv.org/abs/2606.31043) | 通过可逆条件分布变换超越 additive residual；质量 ×5 的 PushT 中 Warp-ES **67.4%**，未适应 **31.6%**，原动力学基准 **98.8%** | 动作分布重塑已经被做；67.4% 是真实残余，但不能把原动力学 98.8% 当成新动力学可恢复上界 |
| [JEPA-TTT — *Persistent Test-Time Training of Latent World Models*](https://arxiv.org/abs/2610.00722) | 在线自监督持续更新 latent dynamics predictor，在 8 个固定 dynamics shift 上改善规划；平均 best score 相对冻结模型提升 **153%** | “World Model 在线适应并纠正计划排序”已被覆盖；序列性变化和遗忘未测试，但“未测试”不等于“已证明失败” |
| [Benchmarking World Models for Continual Learning on Compositional Tasks](https://arxiv.org/abs/2609.22055) | 评估多任务连续迁移和遗忘；模块化方法在特定条件下有明显优势，但依赖使用所有任务训练的冻结 encoder | 简单 modular / continual WM 不新；训练时已知未来任务与未知未来任务必须区分 |
| [PhysMem — *Learning Physical Principles from Interaction with Test-Time Memory*](https://arxiv.org/abs/2602.20323) | 交互→假设→验证→物理原则记忆；一项 brick insertion 对照为 **76% vs 23%**（原则归纳 vs 经验检索） | “存储并跨任务复用物理原则”已是现有方向；要研究更具体的知识有效边界与交互成本 |
| [DALI — NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/hash/696996164c52a52b8a162e62574fc9f9-Abstract-Conference.html) | 动力学 context / 世界模型对未见动力学的适应与泛化 | Context-conditioned adaptation 具有直接先例，不能仅凭加入 latent dynamics belief 宣称新颖 |

**A 的阶段判断：** 研究动机充分，但“快速适应”“持久记忆”“迁移物理参数”“修正动作分布”都不是空白。需要找到强基线仍存在的特定条件，尤其关注**跨任务的适应成本与先前经验的实际复用收益**。

### B. Reliable Learning from Imagined Experience

**关注问题：** World Model 生成的想象轨迹在多大程度上真实反映成功、失败及动作条件差异？错误的虚拟经验是否让策略学到虚假的收益？

| 论文 | 主要方法/结果 | 对我们的启示及 Novelty 约束 |
|---|---|---|
| [World-Gymnast](https://arxiv.org/abs/2602.02454) | 在 learned world model 内训练/改进机器人策略 | “在想象环境里 RL”本身已经成立，不是新贡献 |
| [WoVR](https://arxiv.org/abs/2602.13977) / [WoVR+](https://arxiv.org/abs/2610.17922) | 使用短/关键帧初始化 rollout、policy–WM 协同更新及多模态状态等控制幻觉 | 单纯缩短 rollout、加 co-evolution 或改 rollout sampling 很可能 incremental |
| [HaWMPO](https://arxiv.org/abs/2609.09941) | 将 hallucination reliability 引入 policy optimization | “给错误 rollout 降权”已有直接竞争 |
| [AMD — *Learning to Fail*（2026-10-20）](https://www.amd.com/en/developer/resources/technical-articles/2026/rl-inside-world-models.html) | `adjust_bottle` 上原 WM 将 **82.5%** 真实失败描绘为成功；加入失败轨迹后约 **26.3%**；修正 PPO 配方在 5 seeds 下平均真实仿真增益 **+9.7 pp** | 说明修复结果能转化成真实学习收益，但动作细节决定成败的 fork 仍较难；必须区分 action conditioning 与长程 drift |
| [CureWM — *World Models Dream of Success*](https://arxiv.org/abs/2610.09134) | 从共享起点执行验证扰动动作，利用失败/成功反事实修复 checkpoint；484 个 held-out LIBERO failures 的过度乐观比例从约 **80%** 降到 **30%–43%** | Failure-data finetuning / counterfactual action pairs 已被做；跨任务与下游闭环收益的证据仍受测试设置限制 |
| [DreamTrue — *Action-Faithful Robot World Model with Counterfactual Post-Training*](https://arxiv.org/abs/2610.12468) | 几何标定、动作反事实训练及视频奖励，减少动作—物体交互缺陷 | Action fidelity 与反事实 post-training 直接竞争很强 |
| [World Action Verifier](https://world-action-verifier.github.io/) | 通过 forward plausibility / inverse action compatibility 验证生成未来 | 单独新增一个 verifier 很难构成强创新 |

**B 的阶段判断：** “World Model 把失败想象成成功”是可复现的真实问题，但已被 AMD、CureWM 等直接研究。潜在机会在于**动作—条件交互、预测修复的泛化，以及错误学习信号的真实后果**；前提是更可靠的因果与对照评估。

### C. Physics-Generalizable Robot Learning

**关注问题：** 强机器人策略为什么在未见物理条件下失效？这种失败来自隐藏动力学、感知、几何、策略表达、探索还是控制执行？

| 论文 | 主要方法/结果 | 对我们的启示及 Novelty 约束 |
|---|---|---|
| [RoboTwin-Phys — *Do WAMs and VLAs Understand the Physical World?*](https://arxiv.org/abs/2609.26292) | 50 个操作任务，13 类扰动；五模型 Official Random 平均 **86.99%**，Physical Random 平均 **38.48%**，差 **−48.51 pp** | 有显著综合鲁棒性缺口；但混合了质量/摩擦、尺寸/桌面、相机等变化，不能全归因于 hidden dynamics |
| [ForceTwin — *Physics-informed Digital Twins for Robotic Manipulation*](https://arxiv.org/abs/2609.21751) | 带力测量的人类交互辨识机构动力学，9 组任务平均完成率 **87%**；VLM prior **60%**、kinematics-only **57%** | “识别物理参数提高操控能力”已有强证据；但仍依赖额外交互与模型假设 |
| [PhysMem](https://arxiv.org/abs/2602.20323) | 在物理规律相近时复用知识，变化大时在线验证/更新更重要 | “发现旧规则不总能迁移”不是新颖现象 |
| [Warp RL](https://arxiv.org/abs/2606.31043) | 对受控 dynamics shift 调整策略分布 | 需要超越 model-free adaptation 等强基线，而不是只比较 frozen policy |
| [When Does Test-Time Physical Diagnosis Pay?](https://arxiv.org/abs/2609.22299) | 从可识别性、证据使用到决策价值/安全实现建立六个关卡；在未见机制上有证据被获取却未被用到的情况 | “能估计物理状态不代表有用”已有人系统研究；与 PHYWM 非常直接竞争 |

**C 的阶段判断：** 物理条件变化确实造成严重退化，但现有证据未隔离全部因果因素。第一项价值最高的工作是分离 `physics-only / geometry-only / camera-only / combined`，并用可执行性 reference 与强控制器确认不可恢复的真实缺口；不能直接把 benchmark reproduction 作为新论文贡献。

---

## 4. 最近的新线索：从“反复失败的 World Model”而非抽象 idea 出发 [E/H]

### 4.1 Failure-Driven Audit：哪些异常是作者实际观察到的？

| 公开研究 | 已被报告的失败或边界 | 本次审查后的判断 |
|---|---|---|
| [WorldFailBench — *Do World Models Understand Policy Failures?*](https://arxiv.org/abs/2610.33565) | 七个 WM 在 **Failure Reason / Failure Stage / Failure Boundary** 方面不可靠，模型可预测“失败”却错判失败发生在哪一阶段；场景微调可导致真实成功率骤变，而生成世界未跟随 | 失败是实证存在的；但该文 boundary 主要控制**位置、朝向和尺度**，不能当成 mass/friction OOD 证据 |
| [CureWM](https://arxiv.org/abs/2610.09134) | 给同一起点的成功与失败动作构造执行验证对，能显著修复“失败看起来成功” | 问题和一种主要修复方法都已被做；不能重复提出 counterfactual failure data 本身 |
| [DreamTrue](https://arxiv.org/abs/2610.12468) | Action following 和交互真实性受到标定及失败数据覆盖影响 | 简单 action-faithful post-training 和视频 verifier 竞争密集 |
| [JEPA-TTT](https://arxiv.org/abs/2610.00722) | 单次固定动力学偏移有明显适应收益，但序列性 dynamics shifts 没有评估 | 属于“未评估条件”，尚非新发现的失败；也要与 continual WM 比较 |
| [What Must a World Model Distinguish for Planning?](https://arxiv.org/abs/2609.33030) | 将机制、响应和决策充分性区分；搜索阶段与候选选择阶段需要的信息不同 | 仅增加 action-ranking regret 或 query-aware candidate analysis，独立创新性风险高 |

### 4.2 最新专项 Novelty Check：Action Fidelity vs Condition Fidelity

当前最有希望、但**尚未立项**的交叉问题是：

> **当一个 World Model 已经能在固定环境下区分好动作与坏动作时，物理条件改变后，它还能正确预测这些动作在何时失效吗？**

我们检查了两类直接竞争证据：

- **WorldFailBench** 主要比较环境条件改变后的 failure boundary，包括外观/几何等场景变化；
- **CureWM** 主要通过共享起点下的动作扰动/执行验证改进 action-conditioned failure fidelity。

**这两篇并未直接完成一个控制良好的 `Action × Hidden Physical Condition` 联合对照。** 然而，这只说明两篇直接论文没有做，并不说明整个学术界尚未做过。Active System Identification、Decision-Sufficient WM、Robust MPC、反事实训练和 physical diagnosis 都是必须继续排除的近邻研究。

研究中区分三种可量化现象：

1. **Action sensitivity：** 固定条件、改变动作，能否正确改变预测后果？
2. **Condition sensitivity：** 固定动作/策略、改变动力学，能否正确改变预测后果？
3. **Action × Condition interaction：** 物理条件改变是否反转动作相对优劣，World Model 能否预测且 planner 真正利用？

这些指标本身不是论文创新。只有发现现有强模型无法处理且真实影响决策/学习的系统性失效机制，才有开发方法的理由。

### 4.3 与 PHYWM 的直接联系

- PHYWM **locked/limited**：如果模型凭空预测进展，planner 会浪费动作；这是局部 action–outcome mismatch。
- PHYWM **tipping**：物理因子估计不可靠，导致 Oracle 信息有用而 Estimated 没法转化成安全收益。
- PHYWM **DINO-WM**：正确物理信息影响预测，但未可靠改变闭环行为。
- 新文献将同一大问题推广到 **failure stage、failure boundary、反事实失败、imagined RL**。这提供了自然的延续路径，但不意味着统一机制已经被证明。


### 4.4 同期扫描但暂不优先立项的场景

- **离屏后的持续状态演化。** [WRBench — *Current World Models Lack a Persistent State Core*](https://arxiv.org/abs/2606.20545) 在 **23 个视频世界模型、9,600 个生成视频**中观察到离屏变化与重新观察时状态不一致。这接近 object permanence / long-term state 的能力缺口，但目前证据主要针对**视频生成模型**，不能直接推导出具备状态反馈的机器人控制 World Model 同样失败；且 benchmark 已明确提出该问题。
- **接触模式突变与跨模态状态表示。** [ContactWorld](https://arxiv.org/abs/2606.13877) 分析 12 类接触丰富任务中的视觉/触觉表示与规划；[ACTS](https://arxiv.org/abs/2610.24285) 研究从人类交互学习 visuo-tactile dynamics 并向机器人迁移。由此可见，“加入触觉/力觉或接触状态预测”已不是新颖题目；只有发现现有强方法仍不能解释的**接触转换机制和闭环损失**才值得升级。

这些问题的调查方法借鉴了另一项研究 [*Ask About the Right Moment / Track, Then Bind*](https://arxiv.org/abs/2610.31471) 的思路：从**强模型反复犯的具体错误**出发，区分系统已有的局部能力与它们之间的错误接口，然后用精确消融检验机制。这里借鉴的是**找问题的方法**，并不是直接把 video identity binding 的结构迁移到物理控制。

---

## 5. 当前候选问题与可发表性风险（不是成熟 Proposal） [H]

| 代号 | 候选问题 | 与 PHYWM 的关系 | 最大竞争 / 主要风险 | 当前决定 |
|---|---|---|---|---|
| **P1** | **Physical-condition-dependent failure fidelity：** 已修复动作失败预测的 WM，在隐藏动力学变化时是否仍误判动作后果和失败边界？ | 从“物理信息能否影响决策”发展为“动作后果预测是否跨物理条件可靠” | CureWM、DreamTrue、WorldFailBench、physical diagnosis、decision sufficiency；严格同条件比较和新方法独立性尚未成立 | **继续 Novelty + Asset Gate** |
| **P2** | **跨任务物理知识迁移/适应成本：** 已在任务 1 中学到的动力学知识，何时能减少任务 2 的真实交互成本？ | 将 factor 从一次性注入变成可复用知识 | PhysMem、ForceTwin、DALI、continual WM、MiDAS；既有两个任务是否共享可识别的机制尚未知 | **保留，先做必要条件检查** |
| **B1** | **Imagined RL 的 action-dependent false success：** 想象训练是否因特定动作分叉错误而学习失败策略？ | 从 phantom motion 的单次规划错误走向 learning signal 错误 | AMD、WoVR、HaWMPO、CureWM；先前严格状态恢复失败 | **独立保留；无有效对照则 No-Go** |
| **P3** | 正确信息未进入 Planner（DINO-WM interface） | 原论文直接的负结果 | 图像代价分解、候选搜索和 decision sufficiency 都有强先例 | **诊断/可能补充实验，不直接作为一作主线** |
| **P4** | Oracle 有效但 estimated 信息有害；主动探测何时值得 | Tipping 直接线索 | System ID、PhysMem、When Does Physical Diagnosis Pay? | **作为其他问题的基线/成本审查** |

**注意：** Warp RL 在质量变化下的 67.4% 或 WorldFailBench 的失败，并不自动意味着有一个“只要用 WM 就能解决”的剩余空间；还需有新动力学下的强可行参考、独立简单控制器、相同动作权限与交互预算。

---

## 6. 下一阶段建议：先证明问题重要，再决定方法

建议将后续工作限定为两个小范围、可被否证的决策门槛，不新建大型模型：

### Gate A：P1 的 Novelty & Reproducibility Audit（优先）

1. **文献核对：** 逐项确认 WorldFailBench/CureWM/DreamTrue/physical diagnosis/decision-sufficiency 是否已有 `action × hidden dynamics × closed-loop outcome` 的直接对照；列出真实重叠与遗漏，不能把“该论文未做”误写为“无人做”。
2. **资源核对：** 确认 checkpoint、动作接口、可执行任务、真实 failure label、严格状态初始化/回放是否可获得。尤其不能假定跨项目 benchmark 的训练和评价直接兼容。
3. **残余能力验证：** 在可重复且可执行的 physics-only 条件下，先检查**任务是否有非零的可恢复差距**、同一动作集是否出现有意义的排序变化，以及强解析/反馈/鲁棒策略是否已解决问题。
4. **最低升级条件：** 不只是发现 WM 预测错误，还要证明这种错误导致可重复的决策或学习损失，并找到强 baseline 没有解决的机制。
5. **停止条件：** 公共资产无法支持公平比较；Oracle 条件没有可测决策收益；简单物理估计或 robust policy 已恢复性能；机制已被近邻工作直接解决——任一成立即暂停方法开发。

### Gate B：P2 的 Cross-Task Transfer Necessary-Condition Check（并行、轻量）

1. 不预设 push/grasp 必须迁移，先从现有 ManiSkill 资产中确认**同一隐藏机制确实对两个任务的行动选择有影响**。
2. 检查任务 1 的经验能否区分目标物理状态；任务 2 的最优或高质量动作是否依赖这些信息。
3. 对比无记忆、直接历史、解析辨识、强 recurrent/adaptive policy、Oracle，报告**成功率—新任务交互成本曲线**。
4. 若简单解析模型或鲁棒控制器已经完全复用信息，或源任务本身不可辨识，则不启动 learned-WM 方法。

### 与教授讨论的三个问题

1. **在目前可用的 PHYWM/ManiSkill 环境与计算资源下，哪些任务最能证明 learned World Model 相对简单物理/反馈模型存在独特、非人为构造的能力价值？**
2. **P1（动作 × 隐藏物理条件下的失败真实性）和 P2（物理知识跨任务迁移）中，哪条更有可能形成独立的机制贡献，而不是现有论文的增量改进？**
3. **是否有合适的公开 checkpoint / benchmark 或实验室硬件可以尽快做强基线审查？** 在没有公平比较之前，不宜立项模型训练。

---

## 7. 总结

目前最明确的进展是：

- 从 PHYWM 已验证的**预测—决策落差**出发，系统地探索了 A/B/C 三类能力问题；
- 在 Tipping 与 Catch Ramp Ball 中获得了有方法学价值、必须保留的**负结果与不确定结果**；
- 通过最新工作排除了若干过于直接的增量路线，例如“再加一个物理因子”“再加一个失败检测器”“在线更新 dynamics predictor”“简单做物理记忆”；
- 发现两条仍值得核查但没有被证明具有独立创新性的候选：**P1 failure fidelity under hidden dynamics shifts** 与 **P2 decision-useful cross-task physical knowledge transfer**。

**当前没有可以负责任地宣布为成熟论文选题的结果。** 建议继续保留 PHYWM 的科学思想，以失败驱动的 Novelty / Capability / Feasibility Gate 筛选后续方向；在排除强基线和复现风险之前，不提前决定网络架构或投放大量算力。

---

## 参考文献与研究资产

### 本组资料（内部；不要与外部独立复现混淆）

- **[I1]** `PHYWM.pdf` — *Where Explicit Physics Pays Off in Manipulation World Models*；实验设置、Table 1–3、Appendix。
- **[I2]** `REPORT.md` — 2026-11-02 Tipping Gate5 / Gate6，960 回合实验及留一布局统计。
- **[I3]** `EXECUTION_REPORT.md` — 2026-11-03—04 P2d Catch Ramp Ball，32 回合及完整验证状态。
- **[I4]** `PHYWM_ABC_Research_Audit_2026-11-04.md` — A/B/C 文献与研究门槛对照。
- **[I5]** `PHYWM_ABC_Phase2_Novelty_and_Transfer_Gate_2026-11-04.md` — PhysMem 等跨任务物理知识迁移 Novelty Gate。
- **[I6]** `PHYWM_Failure_Driven_Audit_Round3_2026-11-04.md` — Warp RL、JEPA-TTT、continual WM 的剩余失败与复现风险。
- **[I7]** `PHYWM_Followup_Novelty_Gate_2026-11-04.md` — WorldFailBench / CureWM 与 PHYWM 延续审查。

### 公开论文（含核心论文及主要直接竞争）

**基础理论与 PHYWM 直接相关：**

1. [What Must a World Model Distinguish for Planning? — arXiv:2609.33030](https://arxiv.org/abs/2609.33030)
2. [When Does Test-Time Physical Diagnosis Pay? — arXiv:2609.22299](https://arxiv.org/abs/2609.22299)

**A — Efficient Adaptation / Continual Learning：**

3. [MiDAS — arXiv:2608.11363](https://arxiv.org/abs/2608.11363)
4. [Warp RL — arXiv:2606.31043](https://arxiv.org/abs/2606.31043)
5. [JEPA-TTT — arXiv:2610.00722](https://arxiv.org/abs/2610.00722)
6. [Benchmarking World Models for Continual Learning on Compositional Tasks — arXiv:2609.22055](https://arxiv.org/abs/2609.22055)
7. [PhysMem — arXiv:2602.20323](https://arxiv.org/abs/2602.20323)
8. [DALI — NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/hash/696996164c52a52b8a162e62574fc9f9-Abstract-Conference.html)

**B — Reliable Imagination / Failure Fidelity：**

9. [World-Gymnast — arXiv:2602.02454](https://arxiv.org/abs/2602.02454)
10. [WoVR — arXiv:2602.13977](https://arxiv.org/abs/2602.13977)
11. [WoVR+ — arXiv:2610.17922](https://arxiv.org/abs/2610.17922)
12. [HaWMPO — arXiv:2609.09941](https://arxiv.org/abs/2609.09941)
13. [AMD, Learning to Fail (2026-10-20)](https://www.amd.com/en/developer/resources/technical-articles/2026/rl-inside-world-models.html)
14. [CureWM — arXiv:2610.09134](https://arxiv.org/abs/2610.09134)
15. [DreamTrue — arXiv:2610.12468](https://arxiv.org/abs/2610.12468)
16. [World Action Verifier — project](https://world-action-verifier.github.io/)

**C — Physical Generalization / Benchmarks：**

17. [RoboTwin-Phys — arXiv:2609.26292](https://arxiv.org/abs/2609.26292)
18. [RoboTwin-Phys official repository](https://github.com/yefeng00/RoboTwin-Phys)
19. [ForceTwin — arXiv:2609.21751](https://arxiv.org/abs/2609.21751)
20. [WorldFailBench — arXiv:2610.33565](https://arxiv.org/abs/2610.33565)
21. [WorldFailBench official repository](https://github.com/DAXIAORobotics/WorldFailBench)
22. [CureWM official repository](https://github.com/jiuyixu25/CureWM)
23. [WRBench — arXiv:2606.20545](https://arxiv.org/abs/2606.20545)
24. [ContactWorld — arXiv:2606.13877](https://arxiv.org/abs/2606.13877)
25. [ACTS — arXiv:2610.24285](https://arxiv.org/abs/2610.24285)
26. [Ask About the Right Moment / Track, Then Bind — arXiv:2610.31471](https://arxiv.org/abs/2610.31471)

**文献核查日期：** 2026-11-04。部分研究为预印本；涉及代码、权重和训练环境的可用性应在真正启动复现前再次检查。
