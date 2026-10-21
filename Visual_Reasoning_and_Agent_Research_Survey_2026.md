# Visual Reasoning 与 Agentic Visual Reasoning：技术脉络、研究现状与潜在切入点

**Research Landscape & Discussion Notes | 2026 年 10 月 21 日**  
**用途：**与教授讨论研究方向及小规模可执行课题。本文是文献调研和问题梳理，不是研究 Proposal，也不宣称下列候选问题已具备确定的新颖性。  
**范围：**Vision-Language Models (VLMs)、Visual Reasoning、视觉工具使用、GUI / Video / Embodied Agents；重点关注不需要训练大型基础模型的研究范式。

---

## Executive Summary（两分钟版）

**核心认识：Visual Reasoning 研究“如何根据视觉证据得出正确结论”；Agent 进一步研究“为了完成任务，应该获取什么信息、采取什么行动，并如何根据反馈调整”。** 二者结合之后，模型不再只看一次图片并给出答案，而可能主动放大图片、检索视频帧、调用几何工具、维护对象状态，甚至操作 GUI 或物理环境。

2025–2026 年，研究趋势已不只是“更大的 VLM + 更长的 Chain-of-Thought”。几篇代表性工作揭示了更加具体的瓶颈：

1. **感知不正确，增加推理未必有用。** ICML 2026 的 *From Seeing to Thinking* 研究如何分离视觉感知与推理训练；CVPR 2026 的 *VisRes Bench* 则展示了模型在基础视觉抽象和组合关系上的不足。[R3, R4]
2. **有了视觉工具，不等于有效使用视觉工具。** ICLR 2026 的 *DeepEyes* 通过 RL 训练主动视觉观察；ICML 2026 的 *FOVEA* 研究有限视觉带宽下如何选择证据；但另一篇 ICML 2026 工作指出，工具训练收益不一定来自真正学会以工具纠正原有错误。[R6–R8]
3. **答案正确率与任务完成率不是同一个指标。** VisualWebArena、OSWorld 把评估从静态图像问答推进到网页和桌面环境中可验证的行动。[R10, R11]

**对小团队而言，机会更可能来自受控实验与针对性机制，而不是训练新的通用 VLM / Agent。** 值得继续讨论的研究问题包括：视觉错误到底发生在哪个阶段；额外工具证据什么时候真正有用；视觉信息如何影响下游行动；以及在视频身份理解中，有限计算资源是否应该依据具体问题分配。不过其中部分问题已有非常直接的相关工作，必须先做机制级 novelty audit 和 falsification pilot。

---

## 1. Visual Reasoning 是什么？

### 1.1 从识别到推理

可以用一张包含桌子、杯子和书的图像理解三个层次：

| 层次 | 示例问题 | 需要的能力 |
|---|---|---|
| **Visual Perception** | “图中有哪些物体？” | 检测物体、颜色、文字、位置 |
| **Visual Understanding / Grounding** | “杯子在书的哪一边？” | 把语言中的对象与视觉区域对应起来，理解关系 |
| **Visual Reasoning** | “如果杯子与书交换位置，哪个物体将离桌沿更近？” | 组合视觉关系、假设性变换、空间比较 |

这些层次不是彼此独立的网络模块；复杂的 Visual Reasoning 依赖准确 Perception，并可能在推理中反复访问视觉信息。

主要任务包括：**空间／几何推理**（深度、位置、透视、旋转）、**视觉数学与图表推理**、**组合与抽象规则推理**、**视频中的时序和对象身份推理**，以及**物理／因果推断**。它们共享“答案必须依赖图像证据”的要求，但任务机制不同，不能仅凭一个统一的 VQA 准确率判断模型是否具备全部能力。

### 1.2 一个 VLM 如何处理图像？

经典的 encoder–projector–LLM 架构可以写成：

```text
Image / Video  -->  Vision encoder (e.g., ViT)  -->  Visual tokens
                                                         |
Question  ------------------------------->  Multimodal alignment / fusion
                                                         |
                                                   LLM / Transformer
                                                         |
                                            Reasoning tokens / Answer
```

- **Vision encoder：** 将图像切分为 patches、提取视觉特征；visual token 通常不是一一对应的“物体标签”。
- **Multimodal alignment：** 用 projector 或更深层融合结构，让视觉特征与语言模型协同工作。
- **Reasoning：** 模型结合问题和视觉证据预测输出；文字 CoT 只是部分模型采用的一种可见形式，并不保证每一步在视觉上正确。

**LLaVA（NeurIPS 2023）**是理解这一基本架构的好起点；**Qwen3-VL 技术报告（2025，预印本）**代表了更现代的多层视觉特征融合、时空位置表示和长上下文视频处理路线。[R1, R2]

### 1.3 Visual Reasoning 为什么困难？

一次错误回答至少可能源于四个不同环节：

1. **Perception error：** 看错一个小字、漏掉物体、错误估计位置。
2. **Grounding / representation error：** 看到了对象，但把“它”和问题中指代的对象对错了。
3. **Reasoning error：** 相关视觉关系已正确取得，但变换、计数或逻辑推导出错。
4. **Evidence-use error：** 工具或中间步骤提供了正确证据，模型却没有根据它修改结论。

这四类原因要求不同的对照实验。**不能因为模型输出长篇推理仍然答错，就认定失败必然发生在语言推理环节。** *From Seeing to Thinking* 和 *VisRes Bench* 的价值，正是在拆解这种常被混淆的能力差异。[R3, R4]

---

## 2. Visual Reasoning 当前有哪些主要技术路线？

| 技术路线 | 主要思路 | 典型贡献形式 | 需要留意的问题 |
|---|---|---|---|
| **Better representations** | 更细的视觉 token、多尺度特征、空间结构 | 编码／对齐模块、表示训练 | 图像信息是否真的保留下来？ |
| **Reasoning post-training** | 用 SFT、RL 改进多步视觉回答 | 数据、训练目标、奖励机制 | 提升来自视觉感知还是语言推理？ |
| **Visual CoT / intermediate images** | 画图、辅助线、局部标注等中间视觉步骤 | 中间表示或生成机制 | 人工提供的中间图有效 ≠ 模型能自主产生正确图 |
| **Tool-augmented reasoning** | 调用 Crop、Zoom、Depth、OCR、Detector、计算器 | 外部工具 + VLM | 工具输出是否可靠、是否被真正使用？ |
| **Structured external state** | 用 Scene Graph、tracks、event timeline 保存对象及关系 | 外部记忆或表示接口 | 维护成本、错误累积、是否优于内部隐表示？ |

特别值得关注的三篇论文：

- **MIRA（CVPR 2026）**：聚焦需要中间视觉草图或图示的任务。给模型提供有效的中间视觉线索可以明显帮助推理，但这不能直接证明模型有能力独立绘制这些线索。[R5]
- **Scene Graph Thinking（ICML 2026）**：通过图结构和训练信号改善关系推理。这意味着“引入显式 Scene Graph”并非一个未经探索的新想法。[R15]
- **Don't Show Pixels, Show Cues（CVPR 2026）**：将外部深度、光流等视觉工具结果转为结构化、语言可读的提示；说明“信息如何进入推理模型”与“是否拥有信息”同样重要。[R16]

---

## 3. Agent 增加了什么？与 Visual Reasoning 为什么结合？

### 3.1 从单次回答到观察—行动闭环

普通 VLM 问答通常近似为：

\[
\hat y=f_\theta(I,q)
\]

加入工具与环境后，系统需要反复决定下一步动作：

```text
Task / Goal
    ↓
Observe image, video, or screen
    ↓
Understand → Reason → Select next action
    ↓
Tool call / Click / Camera move / Robot action
    ↓
Observe new evidence or changed environment
    └──────────────────────────────↺
    ↓
Verify outcome → Finish or revise
```

这可以理解为 **Observe → Reason → Act → Re-observe**。经典的 **ReAct（ICLR 2023）**将 reasoning 与 action 交替组织，是理解现代 LLM Agent 的重要起点。[R9]

必须注意：**Agent 并不等于一种新的基础模型。** 它既可以是冻结 VLM 加少量工具调用，也可以是通过交互轨迹训练的策略；也不是任何固定的“先做 A 再做 B”脚本都能称为具备动态决策能力的 Agent。Anthropic 的工程综述清楚地区分了预定义 Workflow 与模型动态决定工具和执行路径的 Agent。[R12]

### 3.2 两种 Action：获取信息 vs. 改变世界

**Epistemic action（为获取信息而行动）**：放大图像、查看另一视频帧、旋转摄像头、查看遮挡背后。目的在于减少不确定性。

**Instrumental action（为完成任务而行动）**：点击 Submit、拖动界面元素、机器人抓取物体。目的在于改变世界／推进任务。

一个成熟的视觉 Agent 可能反复交替使用它们，例如先放大按钮辨认，再点击，然后观察是否真的提交成功。

### 3.3 三种有代表性的交叉领域

| 交叉方向 | Action 是什么？ | 最终成功标准 | 实验难度（对小团队） |
|---|---|---|---|
| **Agentic Visual Reasoning** | Crop、Zoom、区域检索、调用视觉工具 | 问答正确率与视觉工具成本 | 相对可控，可使用冻结 VLM |
| **GUI / Computer-Use Agent** | 点击、输入、滚动、软件操作 | 环境验证的任务完成率 | 环境状态与交互复现较重 |
| **Video / Embodied Agent** | 时序证据检索、跟踪、导航或物理控制 | 视频事件理解、目标跟踪或实际动作成功 | 视频工具较可控；真实机器人较重 |

领域代表：**DeepEyes（ICLR 2026）**利用 RL 训练主动视觉感知，**FOVEA（ICML 2026）**使用 training-free 的顺序信息获取思想，在受限的视觉带宽下选择查看哪些区域。[R6, R7] 另一方面，**VisualWebArena（ACL 2024）**与 **OSWorld（2024）**把评估推进到真实网站与桌面软件执行。[R10, R11]

### 3.4 Tool Use 为什么不一定有效？

这里有一个非常重要、也容易被忽视的区别：

> **Tool available → Tool called → Useful evidence obtained → Evidence correctly integrated → Better decision**
>
> 上述每一个箭头都可能失败。

例如模型可以准确调用 Crop，但裁到了错误区域；即使 Crop 正确，也可能忽视返回的数字；甚至原本回答正确，加入工具之后反而答错。

- **What Does Vision Tool-Use RL Really Learn?（ICML 2026）**指出，工具使用 RL 的整体收益不一定意味着模型真正学会了利用 Crop 修正原有错误；在其分析设置中，模型内部能力变化和减少工具干扰占了重要部分。[R8]
- **The Illusion of Visual Tool-Use（2026 年 arXiv 预印本）**通过替换工具观察等因果干预，区分“调用了却没看”和“看了却没有合理安排后续操作”。[R17]

这类论文的重要意义是：**比起报告多调用几次工具之后准确率增加了多少，更重要的是证明哪一次工具观察通过什么途径改变了最终答案。**

---

## 4. 2026 年的研究与产业格局：哪些问题已经成熟，哪些还困难？

从能力与落地角度，可以粗略划分为四层：

1. **基础多模态模型：** 主要由拥有大量算力和数据的团队推进。核心竞争在视觉编码、训练数据、Post-training、视频／长上下文及通用性能。
2. **推理与工具机制：** 高校和小团队可以基于开放权重研究模型如何使用视觉证据、中间表示或工具，通常不需要从零预训练。
3. **Digital Agents：** 通过网页、桌面系统和软件 API 完成任务，关注 GUI grounding、规划、执行验证、异常恢复与安全性。
4. **Embodied / Physical AI：** 在机器人、自动驾驶等物理环境中连接感知、空间理解、规划、动作与反馈，通常对仿真／数据／硬件要求较高。

Stanford **2026 AI Index** 汇总的 OSWorld 计算机操作任务成绩已从早期约 12% 进步到 2025 年约 66.3%，但离全面可靠仍有差距；不能用某个 benchmark 上的成功率直接外推生产环境表现。[R13] Google DeepMind 的机器人研究则体现了 Embodied Reasoning 与 Visual-Language-Action 模型的产业推进方向。[R14]

**对资源有限的团队，重要的不是与行业模型规模竞争，而是在受控场景下解释一个现有强模型仍然存在的具体失效机制。** 例如，为什么额外视觉证据没有改善答案、为什么正确感知没有带来正确动作、为什么记录所有视频状态与记录任务所需状态可能不一样。

---

## 5. 有哪些潜在研究方向？（候选，不是已确认的 gap）

以下问题以**失败现象 → 机制假设 → 可检验差异 → 直接相关工作**的方式组织。重点是先搞清楚是否真的存在尚未解决的问题，而不是提前发明算法。

### Direction A｜Perception–Reasoning–Decision 的错误归因

**问题：** 一个 VLM/Agent 失败时，到底是没看到信息、看到却没推理对，还是已经推理对却没有采取正确行动？

**研究价值：** 很多系统只报告最终 Accuracy / Success Rate，却没有分离模型各阶段的真正作用。如果某项方法主要减少了观察错误，却宣称提高了推理能力，就可能解释错了收益来源。

**小规模切入：** 设计 matched intervention：保持同一模型和题目，分别提供原图、经核实的正确关系描述、正确中间状态或经过验证的行动目标；分析最终答案与实际动作。可使用现成公开模型与受控数据，先不训练。

**最直接重叠：** *From Seeing to Thinking* 已拆分训练能力，*The Illusion of Visual Tool-Use* 已做证据因果审计。因此，“分离 perception 和 reasoning”本身不新；新工作必须有明确且未被覆盖的失败环节，例如**视觉理解正确却无法转化为行动**，并与现有诊断严格比较。[R3, R17]

**阶段性评价：** 适合机制型研究，但 novelty 需要精确限定；中等风险，实验相对可控。

### Direction B｜Gap 1：Estimated Geometry 的可靠性与空间决策

**直观问题：** VLM 直接看视频和 3D Geometry Tool 算距离，两者如果给出不同答案，究竟能否在不知道真实答案的情况下选出更可靠的一方？

**已有工作：** *Seeing Is Not Measuring*（2026 年预印本）已经使用 3D 检测与几何求解器改善 metric spatial reasoning，也表明真实几何检测误差会严重限制收益。这意味着“几何工具也会错”不是新发现。[R18]

**真正关键的前提：** 两种方法必须存在数量可观的**逐样本互补正确**，且这种互补性必须能够由测试时实际可获取的信号预测。如果 VLM 答对的题几乎全被 Geometry 也答对，那么任何二选一 Router 的改进空间都很小。

**为什么目前存疑：** 对距离测量这类问题，专门的 3D 几何方法可能结构性优于直接 VLM；“两种系统不同”并不推出“错误可以互补”。因此不建议现在就设计复杂 Router。

**已有验证进度（来自项目 Codex 审计摘要，尚未独立读取所有本地报告）：**

- ReVSI closest-object 候选子集为 **132 题 / 55 个场景**；注意任务比较的是参考物体与候选物体之间的 **3D surface-to-surface distance**，而非到摄像机的深度。
- Stage 0.5 在预选 **3 个场景**取得标定文件；**48 帧中 39 帧**通过对齐检查，**9 帧仍为 INCONCLUSIVE**，不能解释为题级可用率。
- **真实 VLM–Geometry 配对预测：0 条。** 当前处于 **BLOCKED / INCONCLUSIVE**，完全不能判断互补假设成立或不成立。

**第一项真正有价值的实验：** 同题、同源观察下，统计 both correct、VLM-only correct、Geometry-only correct、both wrong；计算完美选择器的上界。只有上界有意义，才进一步考虑可部署的选择规则。还必须检查现有 Geometry Tool 是否借用了 GT 最佳视角，避免额外 Oracle 信息混入比较。

**阶段性评价：** 科学问题可检验，但互补性前提较弱；宜以证伪型 pilot 为先，而非先做方法。

### Direction C｜Gap 3：Visual Tool 的证据真的被使用了吗？

**直观问题：** Agent 使用 Crop、OCR、Depth 等工具后答错，是工具本身错了，还是工具正确但 Agent 没有使用返回的信息？

**必须区分两种机制：**

- **Evidence-use failure：** 工具证据正确，但没有对结论产生恰当影响；
- **Evidence-reliability failure：** 工具证据不正确，却被 Agent 当成可靠事实。

**最直接重叠：** *The Illusion of Visual Tool-Use* 已研究因果证据使用；*RSure-Agent*（2026 年 10 月 arXiv 预印本）已在遥感工具 Agent 中提出验证、补充证据与拒绝机制。这意味着“判断工具是否可靠”或“增加 Verifier”已是相当拥挤的方向。[R17, R19]

**尚值得理解的问题：** 在什么具体错误条件下，现有验证策略仍然失败？例如当两种工具共享感知偏差时，再多一次同源验证可能无法提供独立信息。但这只是**机制假设**，不是已确认的新颖性。

**阶段性评价：** 领域重要但直接重叠强；适合先读竞争论文与失败案例，不急于提出新系统。

### Direction D｜Gap 5：Question-aware Tracking Budget Allocation

**直观问题：** 给定视频问答任务，昂贵的跟踪算力应该花在**最难关联身份的帧**，还是花在**最可能影响当前答案的对象/事件**？

例如“三杯藏球”：B/C 两杯交叉的 ID switch 可能影响整体 MOT 指标，但未必影响隐藏目标 A 的最终身份；A/B 交叉则可能直接影响答案。不过 B/C 的错误也可能进一步传播，因此不能无条件认定某次交叉与问题无关。

**最直接重叠：** *SAM-Deep-EIoU / Selective Mask Propagation*（2026 年预印本）已提出按身份关联不确定性选择性调用昂贵 VOS。因此，“高风险时启用昂贵 Tracker”不是新方法。[R20]

**还需验证的差异：** 传统 Tracking 目标和某道问答的损失可能不一样，但**优化目标不同 ≠ 实际最优策略一定不同**。需要比较 Question-aware 与现有 Assignment-uncertainty 策略在真实、相同 GPU 成本下能否产生有意义的不同调度。

**关键限制：** 不能事后利用真实身份轨迹识别“关键事件”却不计其信息成本。SMP demo 也没有直接提供公平 VideoQA 评价所需的标注，目前只有可行性审计，没有真实端到端比较。

**阶段性评价：** 工具与实验体量相对可控，机制清晰；与传统 MOT 交叉的 novelty 风险较高。

### Direction E｜Intermediate Visual State：什么时候需要显式外部记忆？

**直观问题：** 长视频或多步 GUI 操作中，模型应每次重新读取截图、依赖对话历史，还是维护物体轨迹／事件时间线／对象关系表？

**相关工作：** Scene Graph Thinking、TOC-Bench、已有 Video Tracking / Structured Memory 与视觉 Agent 已覆盖大量“使用结构化状态”的方法。[R15, R21]

**可以进一步考察的不是“加一个表格”，而是**外部状态的正确性、维护成本和问题相关性：当状态估计不可靠或任务已变化时，它是否反而造成持续性错误？这一方向与 Agent Memory 和 Temporal Reasoning 都有联系，但需要非常具体的设置才能避免重复已有工作。

---

## 6. 如何比较这些方向的科研规模？

下面是**研究实施层面**的粗略估计，不是已经核实的 GPU-hours，也不是录用概率。

| 方向 | 是否需要训练大模型 | 主要负担 | 目前最需要先证实什么 | 第一轮建议 |
|---|---|---|---|---|
| A. 错误归因 / decision interface | 通常不需要 | 构建可控干预、避免标签泄漏 | 现有分析没有覆盖的具体错误机制 | 值得广泛阅读 |
| B. Geometry reliability（Gap 1） | 不需要 | 3D 标定、真实几何推理配对 | VLM 与 Geometry 的逐题互补性 | **先完成 falsification pilot** |
| C. Tool evidence reliability（Gap 3） | 不需要 | 工具错误分类与强 verifier baseline | 现有因果审计／验证机制的未解决失败 | **先做 novelty audit** |
| D. Question-aware tracking（Gap 5） | 不需要 | 视频、Tracker、任务标注、GPU 计时 | 问题导向调度是否真的不同且有用 | 小样本受控比较 |
| E. Explicit visual memory | 不一定 | 记忆构建与长期评估 | 信息损失和错误积累是否无法被简单方案解释 | 条件成熟再推进 |

现阶段可以把 **“Frozen 4B–8B VLM + 公开工具 + 少量受控样本 + 强 baseline”** 当作探索规模的目标，但绝不能把它当作所有课题都能成功的保证。真正决定计算和工程成本的可能是长视频多轮调用、跨帧 3D 对齐、真实 GUI 重置以及完整的统计检验。

---

## 7. 一个更值得沿用的研究方法

我们希望采用的不是“从已有论文拼组件”的选题方式，而是：

```text
Observed failure
    ↓
Find the specific failing operation
    ↓
Read the closest papers across neighboring fields
    ↓
State a falsifiable mechanism hypothesis
    ↓
Minimal intervention + matched baselines
    ↓
Go / No-Go (including valid negative findings)
```

判断一个候选是否值得成为论文，需要同时回答：

**（1）Problem：** 解决的究竟是哪一种真实失败？  
**（2）Method：** 与最接近的现有方法相比，有无本质机制差异？  
**（3）Setting：** 场景是否带来新的挑战，而非只换一个 benchmark？  
**（4）Evidence：** 不依赖 Oracle、额外预算或筛除困难样本，关键优势仍然成立吗？

只要某篇论文“看起来相似”，就直接放弃，可能错失有意义的差异；但只因为三元素中有一项不同就直接认为有 novelty，同样缺乏依据。**最重要的是剩余差异是否产生可验证的新知识。**

对 Gap 1，负结果可能说明 VLM 与几何工具没有可用的互补错误；对 Gap 5，负结果可能说明已有 MOT 不确定性已经足够预测下游风险。**这些都是应当如实保留的科学结论，不能通过更换评价协议来制造收益。**

---

## 8. 建议与教授讨论的三个问题

1. **更偏模型机制，还是更偏交互决策？** 如果希望在现有实验预算内快速得到严谨结果，冻结 VLM 的视觉证据使用机制分析，和 Video / GUI Agent 的小规模决策实验，哪类更适合团队现有能力？
2. **是否存在可复现且具有下游影响的 failure mode？** 对候选方向，应该优先拿到逐题错误表、动作轨迹和任务结果，而非先设计新模型。
3. **什么样的证据足以证明“一个小的结构改变”有价值？** 例如，在固定输入、模型和计算预算下，只改变证据形成的时机、表示方式或调度规则，并能解释效应何时消失。

**当前建议：** 把 Visual Reasoning × Agentic Decision-Making 作为较宽的研究兴趣；保留 Gap 1 与 Gap 5 的受控可行性验证，Gap 3 优先研究最强竞争工作的机制与失败案例。它们都是**研究候选，而非确定选题**。

---

## 参考资料与建议阅读顺序

> **状态说明：** “会议论文”指已有会议正式论文页；“预印本”指 arXiv 尚不宜当作已接收成果；“工程说明”用于解释系统与产业做法，而非同行评审论文。下列链接截至 **2026-10-21** 核对，文中只援引与当前论点直接相关的结论。

### 基础：VLM 与 Agent

- **[R1]** Liu et al. [*Visual Instruction Tuning (LLaVA)*](https://proceedings.neurips.cc/paper/2023/hash/6dcf277ea32ce3288914faf369fe6de0-Abstract-Conference.html). **NeurIPS 2023**. VLM 基本架构与视觉指令微调。
- **[R2]** Bai et al. [*Qwen3-VL Technical Report*](https://arxiv.org/abs/2511.21631). **2025 预印本**. 新一代开放权重多模态模型的系统设计。
- **[R3]** Wu et al. [*From Seeing to Thinking: Decoupling Perception and Reasoning Improves Post-Training of Vision-Language Models*](https://proceedings.mlr.press/v306/wu26f.html). **ICML 2026**. 分离感知与推理训练。
- **[R4]** Törtei et al. [*VisRes Bench: On Evaluating the Visual Reasoning Capabilities of VLMs*](https://openaccess.thecvf.com/content/CVPR2026/html/Tortei_VisRes_Bench_On_Evaluating_the_Visual_Reasoning_Capabilities_of_VLMs_CVPR_2026_paper.html). **CVPR 2026**. 受控视觉抽象与关系推理评估。
- **[R5]** Zhou et al. [*When Visualizing Is the First Step to Reasoning: MIRA*](https://openaccess.thecvf.com/content/CVPR2026/html/Zhou_When_Visualizing_is_the_First_Step_to_Reasoning_MIRA_a_CVPR_2026_paper.html). **CVPR 2026**. 中间视觉图示在推理中的价值。
- **[R9]** Yao et al. [*ReAct: Synergizing Reasoning and Acting in Language Models*](https://arxiv.org/abs/2210.03629). **ICLR 2023**. Reason–Act–Observe 的基础思路。
- **[R12]** Anthropic. [*Building Effective Agents*](https://www.anthropic.com/engineering/building-effective-agents). **2024 工程说明**. Agent 与 Workflow 的差异及工程成本。

### 核心交叉：视觉工具、主动观察与 Agent

- **[R6]** Zheng et al. [*DeepEyes: Incentivizing “Thinking with Images” via Reinforcement Learning*](https://proceedings.iclr.cc/paper_files/paper/2026/hash/cdb347d7516d52a9280ea9d6708911f6-Abstract-Conference.html). **ICLR 2026**. 通过 RL 学习主动视觉工具使用。
- **[R7]** Liu et al. [*The Perceptual Bandwidth Bottleneck in VLMs / FOVEA*](https://proceedings.mlr.press/v306/liu26an.html). **ICML 2026**. 有限视觉带宽下的训练免费证据获取。
- **[R8]** Ma et al. [*What Does Vision Tool-Use Reinforcement Learning Really Learn?*](https://proceedings.mlr.press/v306/ma26as.html). **ICML 2026**. 对工具训练增益的机制分析。
- **[R10]** Koh et al. [*VisualWebArena*](https://aclanthology.org/2024.acl-long.50/). **ACL 2024**. 视觉网页智能体任务。
- **[R11]** Xie et al. [*OSWorld*](https://os-world.github.io/). **2024 项目／评测环境**. 真实桌面计算机任务。
- **[R13]** Stanford HAI. [*2026 AI Index — Technical Performance*](https://hai.stanford.edu/ai-index/2026-ai-index-report/technical-performance). **2026 年行业与能力报告**。
- **[R14]** Google DeepMind. [*Gemini Robotics-ER 1.6*](https://deepmind.google/blog/gemini-robotics-er-1-6/). **2026 官方技术介绍**。Embodied Reasoning 的产业代表。
- **[R15]** Yang et al. [*Scene Graph Thinking*](https://proceedings.mlr.press/v306/yang26ck.html). **ICML 2026**. 结构化视觉关系与训练。
- **[R16]** Janjua et al. [*Don't Show Pixels, Show Cues: Perception Programs*](https://openaccess.thecvf.com/content/CVPR2026/html/Janjua_Dont_Show_Pixels_Show_Cues_Unlocking_Visual_Tool_Reasoning_in_CVPR_2026_paper.html). **CVPR 2026**. 将工具结果组织为结构化推理线索。

### 与候选研究最接近的工作

- **[R17]** Wang et al. [*The Illusion of Visual Tool-Use: A Causal Audit of Thinking with Images*](https://arxiv.org/abs/2608.06270). **2026 年 8 月预印本**. 工具观察因果贡献的诊断。
- **[R18]** Glantz & Grange. [*Seeing Is Not Measuring: Tool-Augmented Metric Spatial Reasoning for VLMs*](https://arxiv.org/abs/2609.29073). **2026 年 9 月预印本**. 显式几何工具与估计误差。
- **[R19]** Liu et al. [*RSure-Agent: Reliable Use of Tool Observations for Remote Sensing Agents*](https://arxiv.org/abs/2610.04836). **2026 年 10 月预印本**. 工具证据验证与错误传播。
- **[R20]** Holmberg. [*SAM-Deep-EIoU: Selective Mask Propagation for Multi-Object Tracking*](https://arxiv.org/abs/2606.13033). **2026 年预印本**. 有条件调用昂贵 VOS 维护 ID。
- **[R21]** Chen et al. [*TOC-Bench: A Temporal Object Consistency Benchmark for Video LLMs*](https://arxiv.org/abs/2605.09904). **2026 年预印本**. 视频时序对象身份和事件一致性。

**建议先读四篇：** R3（Perception vs. Reasoning）、R9（Agent 基本逻辑）、R6（主动视觉工具）、R8（工具收益与真实机制）。再根据与教授讨论的兴趣，进入 Gap 1 的 R18 或 Gap 5 的 R20。

---

*说明：本文件为独立整理的 literature-based research overview。候选方向与数据状态部分根据本次讨论及你转述的 Codex 审计整理；它们不能替代对原始实验报告、代码和逐样本结果的直接复核。全文未加入虚构实验结果，也未以未发表的研究草稿作为已发表证据。*
