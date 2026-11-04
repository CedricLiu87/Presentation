# Research Directions Update
**From Explicit Physical Representation to Decision-Making and Generalizable Robot Learning**  
**汇报性质：前期调研与候选研究方向（2026 年 11 月）**

## 1. 研究背景：从 PHYWM 出发

在 *Where Explicit Physics Pays Off in Manipulation World Models*（PHYWM）中，我们研究了在固定数据、模型与训练预算下，向 manipulation world model 显式提供物理因子，与只输入当前观察或交互历史相比，如何影响预测和闭环控制。论文使用 PointWorld 与 DINO-WM，研究 mobility constraints、摩擦与重心等信息。

其中一个重要发现是：传统 prediction error 与控制收益并不总是一致。在多个 mobility 场景中，显式估计因子只带来约 5% 的目标运动预测误差改善，却让对锁死目标的无效推动减少约 37%。论文还发现，当历史已经能够辨认物理约束、估计的因子不准确，或规划成本难以利用模型中已编码的物理信息时，显式因子的收益可能消失。尤其是 DINO-WM 的 factor-flip 实验显示，物理因子能够改变预测，但未必改变规划器最终行为。

这些结果提示一个值得进一步研究的现象：**物理知识影响模型预测，不等于它真正帮助机器人选择更好的动作。** 但原论文并未系统刻画物理因子如何影响各候选动作的价值比较，也没有完整隔离信息在预测、成本计算、动作搜索和闭环执行各环节的作用。

## 2. 当前最具体的候选 Idea：Physical-Factor-Guided Action Selection

**暂定标题：** *Beyond Stopping: How Can Physical Factors Improve Action Selection in World-Model Planning?*

**研究问题：** 在哪些物理和任务条件下，显式物理信息不仅帮助 world model 识别“某个动作不能做或不应继续”，而且能够改变多个**可执行动作之间的相对价值**，帮助机器人主动选择更优操作策略，最终改善任务成功率、效率或安全性？

这是相对于 PHYWM 已有结果的一个自然推进。原论文中最清晰的效益之一，是识别锁死/受限对象并减少无效推动。下一步希望研究更广泛的**正向动作选择**，例如同一物体在不同摩擦、重心或接触条件下，是否应选择不同的推动高度、方向、接触位置、力度或操作方式。关键不在于展示物理参数会改变运动轨迹，而在于证明这些差异足以改变任务中最优或更优的动作，并且 world model/planner 能够利用它们。

**一个待验证的假设：** Physical Factor 对决策的价值取决于三层条件：①该因子确实改变候选动作的真实相对收益；②这种差异不能被已有观察历史或简单反馈轻易消除；③世界模型和规划成本能保留并利用这种差异，找到实际可执行的更优动作。三个条件缺一时，即使预测准确率提高，也可能没有闭环收益。

**计划中的最小研究路径（尚未开始新的模型训练）：**

1. 在原生物理、动作权限和任务评价得到验证的场景中，预先定义具有物理意义、可真实执行的候选动作；不为了得到正结果事后筛选。
2. 通过可靠的仿真对照估计不同物理条件下各动作的真实回报，先问“物理参数是否真的改变值得采取的动作？”；不能保证严格同状态回放时，改用明确标注的独立随机复位设计。
3. 冻结同一 world model 与候选动作，比较无因子、估计因子和 Oracle 因子下的预测、planner cost、动作排序以及最终选择，单独区分软注入和硬约束投影。
4. 将因子条件规划与强反馈、鲁棒控制、解析物理方法对比，并报告成功率、失败率、执行路径和代价；如果强简单基线已经足够，接受这一结果。
5. 仅当观察到可信、可复现的剩余能力缺口后，才考虑具体的新方法或架构。

**已做过的后续探索：** 在 ManiSkill tipping 中完成 960 回合独立随机复位实验。降低接触高度的统一策略成功率为 76.25%，留一布局的物理条件选择策略为 75.0%；尚未证明依据物理条件选择动作优于强简单控制。随后在 RoboDyna 的 Catch Ramp Ball 验证了公平的物理抓取接口，完成 32 回合小规模 state-access 预测信息 Pilot：强反馈、简单解析预测、未来落点 Oracle 均为 8/8 成功；由于一条冻结参考轨迹前缀检查未通过，正式结论为 INCONCLUSIVE / No-Go。该 Oracle 是**未来球落点**，不是物理因子，因此不能据此判断物理因子本身无用。两组探索共同促使我们强调：必须先证明有可执行、存在真实价值差异的动作选择问题，再研究 world model 是否能够兑现信息价值。

**创新性边界：** 我们尚未证明这是一个未被解决的 gap。*What Must a World Model Distinguish for Planning?* 已讨论 decision sufficiency 与动作集合的关系，*AD-WM* 已研究 action-discriminative prediction 和规划表现。因此不能将“action ranking 比 MSE 重要”作为新贡献。潜在区别应来自**物理因子对动作优劣的具体作用、在 world-model→cost→search→control 中的机制性失效，以及跨不同物理条件的可检验规律**。这些都需要进一步文献和实验验证。

## 3. 三个并行的潜在长期兴趣方向

| 方向 | 关心的能力问题 | 已调研的代表工作与阶段认识 | 有待深入的切入点 |
|---|---|---|---|
| **A. World Models for Efficient Robot Adaptation** | 如何让机器人通过少量真实交互，快速适应新物体、新动力学或连续任务？ | **WorldAgen** 将测试时交互与世界模型更新结合；**MetaPusher** 研究在线动力学适应与重新规划；组合任务 continual-WM 工作研究知识复用与遗忘。说明“在线适应”本身不是空白。 | 新旧任务间哪些动力学知识可复用，何时需要更新，以及能否减少真实交互而不牺牲成功率。 |
| **B. Reliable Learning from Imagined Experience** | 在 world model 中进行规划或策略训练时，怎样避免利用模拟错误而学到现实中失败的动作？ | **AMD Learning to Fail** 研究失败被虚构成成功的现象；**CureWM** 用真实执行验证的反事实失败修复模型。既有研究已证明问题真实，也覆盖部分解决方法。我们曾审查 AMD 数据，但严格 action-fork 证据不足。 | 在策略不断优化、动作分布改变之后，哪些未被现有不确定性或失败修复方法发现的错误仍会系统性损害真实回报。 |
| **C. Physics-Generalizable Robot Learning** | 面对未见过的质量、摩擦、接触或重心条件，机器人怎样保持稳定的任务表现？ | **RoboTwin-Phys** 系统测量物理参数变化下的鲁棒性缺口；**PhyFilter** 显示物理先验与反馈能改善未知动力学泛化。单纯报告 OOD 下降或反馈有效不够新。 | 哪些真实物理变化超出强反馈/解析控制的处理能力，什么样的可迁移信息能帮助机器人跨物理条件泛化。 |

A/B/C 是**彼此独立**的潜在研究兴趣，不以 D 的成功为前提，也不因为 D 与 PHYWM 联系更紧密而被提前淘汰。

## 4. 目前结论与希望讨论的问题

我目前希望把 D 作为**可以向导师讨论、尚需验证的具体 candidate research idea**，把 A/B/C 作为并行探索的研究兴趣。D 的价值不是声称“物理信息一定改善成功率”，而是将其放进真正的动作选择过程，探索**哪些物理知识值得 world model 保留，如何影响选择更好动作，以及哪些环节阻碍这种收益实现**。

目前没有完成决定性的新方法验证，也不宣称已解决新颖性问题。下一步希望与导师讨论：一方面，这一方向能否与 PHYWM 明确区分，并且与已有 decision-aware world model 工作形成实质差异；另一方面，在 A/B/C 中，哪些能力问题更有机会形成独立而重要的研究课题。

### 主要参考文献

- PHYWM: *Where Explicit Physics Pays Off in Manipulation World Models*（已有论文；代码 https://github.com/IoET-y/wm-explicit-physics）
- Wei et al., *What Must a World Model Distinguish for Planning?* (2026): https://arxiv.org/abs/2609.33030
- Qiu et al., *AD-WM: Action-Discriminative World Models for Counterfactual MPC* (2026): https://arxiv.org/abs/2609.30264
- Wan et al., *WorldAgen* (AAAI 2026): https://ojs.aaai.org/index.php/AAAI/article/view/38925
- Lee et al., *MetaPusher* (2026): https://arxiv.org/abs/2609.21122
- Zhou et al., *Benchmarking World Models for Continual Learning on Compositional Tasks* (2026): https://arxiv.org/abs/2609.22055
- AMD, *Learning to Fail* (2026): https://www.amd.com/en/developer/resources/technical-articles/2026/rl-inside-world-models.html
- Xu et al., *World Models Dream of Success / CureWM* (2026): https://arxiv.org/abs/2610.09134
- Zhang et al., *RoboTwin-Phys* (2026): https://arxiv.org/abs/2609.26292
- Jia et al., *Physics Filtering Favors the Generalization of Robot Learning* (2026): https://www.nature.com/articles/s44182-026-00114-y
