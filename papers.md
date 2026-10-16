# Papers — 已读论文与研究记忆

## ASB–GAC — Humanoid Whole-Body Manipulation via Active Spatial Brain and Generalizable Action Cerebellum（简称 ASB–GAC）

**重点关注**：True

**开源情况**：完全没开[2026-10-15]（指本文系统实现；官方目前仅网页与演示公开，未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-05-20（所读 v2：2026-07-07，另核查正式版补充材料）；一作（共同）：Zhizhao Liang、Yi-Lin Wei、Xuhang Chen；通讯：Wei-Shi Zheng；单位：中山大学计算机学院、鹏城实验室（按所读 arXiv v2）；是否中稿：是，[ECCV 2026](https://link.springer.com/chapter/10.1007/978-3-032-37422-6_8)；[项目主页](https://leungchaos.github.io/Humanoid-Whole-Body-Manipulation-via-Active-Spatial-Brain-and-Generalizable-Action-Cerebellum/)，[arxiv](https://arxiv.org/abs/2605.21133)。

**核心问题**：作者针对人形机器人在复杂空间中看不全、目标不可达、任务数据昂贵的问题，组合主动观察与任务规划、几何规划、基础模型抓取和下肢控制。

**系统拆解**：按[原文 §3／图2](https://arxiv.org/html/2605.21133v2#S3)的层级，一级组件为 Active Spatial Brain（ASB）和 Generalizable Action Cerebellum（GAC）。输入语言、RGB-D、本体状态；ASB 调用 GAC 并接收反馈，最终输出相机动作、灵巧手姿和上下肢关节目标，由 PD 执行。下表将 GAC 的两个子模块分行说明。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| ASB：Active Spatial Brain | 三个模块：Active Spatial Perception、Memory Bank、Adaptive Task Planning | 观察与历史 → 调整视角、分解／重规划任务 → 调用 GAC。图像记忆只保留最近一次基座移动以来的图像与相机位姿，文本保留全程命令和反馈。 |
| GAC → Humanoid Lower-Body Locomotion | 两个 agent：Spatial Navigation、End-Effector Reachable Space Solver；下接 RL locomotion policy | RGB-D／目标位置 → 栅格与 A* 路径、可达性位置优化 → 速度／转向／高度指令 → RL 策略输出下肢关节动作。 |
| GAC → Humanoid Upper-Body Dexterous Manipulation | 两个 agent：Grasp Pose Generation、Post-grasp Trajectory Generation | 观察／指令 → 合成人手物交互图＋物体重建 → 手／物姿态估计、对齐及灵巧手重定向；随后关键点＋推／拉／放／旋转原语 → 末端轨迹 → IK 输出关节命令。 |

**训练、复用与新增**：与上表逐项对应。[正式版补充 §2–3](https://media.springernature.com/original/springer-static/esm/chp%3A10.1007%2F978-3-032-37422-6_8/MediaObjects/700396_1_En_8_MOESM1_ESM.pdf)将框架称为 training-free（作者主张）；不能据此把底层策略的预训练也理解为不存在。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| ASB | prompt 调用现成 VLM，未报告另行训练；补充表2将规划模型记为 Gem-fl.，未展开完整型号。模型预训练数据未列。 | 主动视角控制、移动条件图像记忆和基于执行反馈的重规划。 |
| GAC → 下肢 | 复用 A*／RDP，采用 HOMIE 下肢策略；作者从头训练、微调还是直接加载权重仍未披露。[原 HOMIE](https://homietele.github.io/)的 RL 经验来自 Isaac Gym 在线仿真、无需 MoCap，本文是否完全照搬训练配置未知。肩部偏移模型 Φ(z)由 MuJoCo 估计。 | 可达性求解器调整站位与高度，并接入下肢控制。 |
| GAC → 上肢 | 补充明确默认用 Nano Banana 2 生成抓握图，Hyper3D 生成物体 mesh；HaMeR 和 MegaPose 分别估计合成图中的手姿与物姿，再按 EasyHOI 方法优化深度轴，最后重定向到灵巧手。抓后轨迹模型记为 Qwen-3.5，未给后缀／checkpoint；未报告这些模型在本文中微调。图像与 mesh 是推理／逐实例优化输入，非新增动作训练集。 | 将生成、重建、姿态估计和重定向接成抓握流程；用语义关键点、参数化原语与 IK 生成抓后轨迹。 |

**关键 Q&A（2026-10-15）**：Q：作者自己训练了什么，是从头还是微调，数据从哪来？A：作者称框架 training-free，主要调用现成模型；下肢采用经过 RL 训练的 HOMIE 路线策略，具体权重初始化仍未披露。§4.3 每设置至少20条 Easy 轨迹用于 TrajBooster／Ψ₀ 对照微调。补充材料补全了此前未确定的部分模型名称；训练经历与作者亲自重训仍须区分。

## VIRAL — Visual Sim-to-Real at Scale for Humanoid Loco-Manipulation

**重点关注**：True

**开源情况**：可以sim[2026-10-15]（[官方代码](https://github.com/NVlabs/GR00T-VisualSim2Real)已提供训练／仿真评估、权重、示范与资产，需 Git LFS、Isaac Sim 5.1／Isaac Lab；真机仅提供 ONNX 导出，未找到部署入口；未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2025-11-19（所读 v2：2025-11-27，正文及附录，另核查正式版与公开代码）；一作（共同）：Tairan He、Zi Wang、Haoru Xue、Qingwei Ben；通讯：未明确标注，Linxi Fan／Yuke Zhu 标注为项目负责人；单位：NVIDIA、CMU、UC Berkeley、CUHK；是否中稿：是，[CVPR 2026](https://openaccess.thecvf.com/content/CVPR2026/html/He_VIRAL_Visual_Sim-to-Real_at_Scale_for_Humanoid_Loco-Manipulation_CVPR_2026_paper.html)；[项目主页](https://viral-humanoid.github.io/)，[arxiv](https://arxiv.org/abs/2511.15200)。

**核心问题**：用大规模仿真学习人形机器人的视觉移动操作，在不使用真机任务示范训练／微调策略的条件下，将行走、放置、抓取和转身迁移到实机。

**系统拆解**：按[原文 §2／图2](https://arxiv.org/html/2511.15200v2#S2)，训练分 Teacher → Student 两阶段，二者都以预训练 WBC 为执行基础；部署时由 Student 接收 RGB＋本体状态并调用 WBC，Teacher 不参与实机决策。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Privileged RL Teacher | 任务层策略 MLP；Delta Action Space；Reference State Initialization（RSI） | 仿真完整状态，含任务阶段、物体位姿与接触信息 → 任务决策 → 移动／转向、手臂及手指的增量指令。 |
| Vision-based Student | 视觉编码器＋融合本体状态的策略头 | 机载 RGB＋可实测本体状态 → 预测 Teacher 的动作 → 相同指令接口；无需实机物体真值状态。 |
| Whole-Body Control（WBC） | 预训练 HOMIE；扩展手指指令接口 | 累积后的移动／上身指令＋本体状态 → 下身运动控制与上身、手指目标 → 关节控制执行。 |

[实机 §3](https://arxiv.org/html/2511.15200v2#S3)：G1 在两桌间重复放物、抓新物、转身，59 次尝试成功 54 次（约91.5%），周期20.2秒；不能把“54 cycles”写成54次无失败。泛化测试限于同类任务的位置、外观与物体变化；策略推理在外部 RTX 4090 工作站运行。

**训练、复用与新增**：与上表对应。任务训练依赖仿真数据；零真机策略微调仍包含手指系统辨识与相机标定，所采真机测量用于校准仿真。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Privileged RL Teacher | 作者用 PPO 训练任务策略，经验来自在线仿真；另采集200条仿真遥操作示范，用于抽取中间状态、重置机器人与场景（RSI），不是把200条示范直接当最终学生的全部训练集。论文未明确首轮参数初始化；[公开配置](https://github.com/NVlabs/GR00T-VisualSim2Real/blob/92bf0863d4a9b6ee29849736152b7769bd45c49c/gr00t/rl/config/exp/loco_manip/walk_stand_place_grasp_turn_homie.yaml)默认续训自有 Teacher checkpoint。 | 在 WBC 接口上组合阶段奖励、增量动作和示范状态重置，降低长任务探索难度。 |
| Vision-based Student | 作者以在线 DAgger＋BC 蒸馏，Teacher／Student 各控制一半仿真环境，由 Teacher 给动作标签。论文比较 DINOv3、ResNet等，未明确真机视觉型号与冻结策略；[公开默认配置](https://github.com/NVlabs/GR00T-VisualSim2Real/blob/92bf0863d4a9b6ee29849736152b7769bd45c49c/gr00t/rl/config/exp/loco_manip/wsdpt_student_for_teacher_v8q8.002_resnet_rgb_delay.yaml)为 ImageNet 预训练 ResNet18（参与微调）＋新建 MLP，不能与论文 DINOv3 实验混写。 | 混合教师／学生访问的状态，结合视觉域随机化、tiled rendering 与分布式蒸馏。 |
| Whole-Body Control（WBC） | 直接加载 HOMIE walk／stand 权重；[公开实现](https://github.com/NVlabs/GR00T-VisualSim2Real/blob/92bf0863d4a9b6ee29849736152b7769bd45c49c/gr00t/rl/trl/trainer/ppo_trainer_homie_api.py)在 opt_homie=False 时冻结参数，默认 Teacher／Student 配置均如此。底层已有仿真 RL 训练经历，本文任务训练不重训它。 | 将现成全身控制能力作为动作接口，并接入手指控制；手指动力学通过真机抓放测量做 SysID。 |

论文训练规模为 Teacher 16张、Student 64张 L40S；这描述作者实验配置，不是已证明的最低需求。可借鉴“任务策略＋现成 WBC”的接口与仿真数据生成方法；低算力重训能否达到原结果仍需实测。

## KPI — KPI: A Promptable Kernel for Physical Interaction on Humanoids

**重点关注**：False

**开源情况**：完全没开[2026-10-15]（指本文系统实现；[官方仓库](https://github.com/kpi-robot/kpi-robot.github.io)仅含论文、演示及网页源码，未提供控制器、agent、仿真或部署入口；未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-09-28（所读 v1，同日，正文 I–V 及补充 S1，另核查官网与网页仓库）；一作（共同）：Yikai Wang，未标共同一作；通讯：Yikai Wang、Lingfeng Sun；单位：Carnegie Mellon University、Autel US、Northeastern University；是否中稿：未核实（[作者主页](https://haozhang-thu.github.io/)列为 ICRA 2027 在投）；[项目主页](https://kpi-robot.github.io/)，[arxiv](https://arxiv.org/abs/2609.36151)。

**核心问题**：轨迹没有完整表达接触时的物理要求；作者让上层同时提供轨迹与交互契约，由快速控制环在线调整手臂阻抗、参考推进及前馈力。

**系统拆解**：输入语言、RGB-D、本体状态；agent 生成参考与 contract，KPI 控手臂、未修改的 SONIC 控腿腰。按[§III／图2](https://arxiv.org/html/2609.36151v1#S3)，保留 agent、kernel 与硬件实现的层级：

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Agentic Execution → Analyzer / Detector / Verifier | GPT-6 Astra 的角色调用 | 指令与观察 → Detector 在 RGB 定位、深度反投影；Analyzer 分阶段并编写参考与 contract；Verifier 检查运动学、碰撞及阶段结果 → 通过的计划或修订请求。 |
| Agentic Execution → Executor / motion cores | 共享参数化运动生成器、手指预设 | 已验证阶段 → 组合直线、圆弧、移动、身高等 cores → 名义参考送 KPI／SONIC，返回执行状态。 |
| KPI: Contract-Driven Physical Interaction | Interaction contract；有界在线优化 | 参考、实际末端姿态／速度、估计 wrench → 100 Hz 联合优化选定通道 → 刚度 Kp、阻尼 Kd、参考推进 ρ、前馈 wrench。 |
| Realization on a Humanoid Robot → Tunable task-space impedance / Damping decomposition / Sensorless wrench estimation | 笛卡尔阻抗；关节阻尼分解；准静态估力 | 参数与误差 → 500 Hz 手臂力矩；对角阻尼在电机驱动器、跨关节项在主机。关节力矩修正、扣重力、正则最小二乘 → 200 Hz 掌部 wrench 反馈。 |
| Realization on a Humanoid Robot → Whole-body control | 预训练 SONIC；遥操作旁路 GMR＋正运动学 | Agentic 用头／双腕三点目标，teleop 用 SMPL → SONIC 控腿腰；同源末端参考由 KPI 实现手臂接触响应。 |

[§IV／图4](https://arxiv.org/html/2609.36151v1#S4)：G1＋Dex3、机载 RGB-D、无腕力传感器；绞盘、开门通过、搬箱各5次，KPI 15/15 完成，SONIC 0/15、作者集成的 MCC* 1/15。图4指标是阶段进度，不能把基线20%进度当成功率。完整搬箱只用0.85 kg，1.85／3 kg 是另行静止抬举；写字为 scripted，抽屉及跑步／跪下搬箱为 teleop。样本小，未证明广泛场景泛化。

**训练、复用与新增**：本文没有重新训练任务策略或全身跟踪器；以下在线控制优化不等于学习训练。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Agentic Execution → Analyzer / Detector / Verifier | 调用 GPT-6 Astra，未报告微调或新增训练集，也未提供代码核实冻结；RGB-D／本体状态是推理输入。 | 角色分工、几何请求、contract 生成、执行前验证及阶段后修订。 |
| Agentic Execution → Executor / motion cores | 共享参数化生成器；未报告各 core 是单独训练的 RL policy，无任务专用示范训练。 | 组合同一动作空间完成不同任务。 |
| KPI: Contract-Driven Physical Interaction | 用当前误差／速度／估计力做局部有界优化，无离线训练集，也不识别环境刚度。 | Contract 将方向、响应要求、力区间、可调通道统一，闭环适应 gain／reference／feedforward。 |
| Realization on a Humanoid Robot → Tunable task-space impedance / Damping decomposition / Sensorless wrench estimation | 复用经典阻抗与 MCC 估力思路；本文未完整披露传动效率模型的标定数据／系数，不能代入 MCC 原文细节。 | 在 G1 现有关节接口实现任务空间阻抗，将局部阻尼下沉驱动器以减小延迟。 |
| Realization on a Humanoid Robot → Whole-body control | 直接复用未修改、未重训的 SONIC，遥操作复用 GMR；具体 SONIC checkpoint 未披露。 | 分出手臂命令，同时保留 SONIC 的腿腰跟踪与实际关节状态反馈。 |

**关键 Q&A（2026-10-15）**：Q：promptable 具体指什么？A：语言经 agent 生成结构化契约 σᵢ=(方向／子空间，tracking／compliant／constrained，力区间，可调通道)，并非学习出的 prompt token；三类响应均可附加力区间。上层决定阶段目标，KPI 每10 ms适应控制参数（[§III-A](https://arxiv.org/html/2609.36151v1#S3.SS1)）。Q：力区间是硬保证吗？A：不是；它约束局部预测，可达性受控制器边界限制。估力是准静态近似，冲击和强惯性交互不在所用局部模型范围，正文“预期误差小于5 N”也不是独立真值测量结果（[补充 S1](https://arxiv.org/html/2609.36151v1#S1a)）。

## ZEST — ZEST: Zero-shot Embodied Skill Transfer for Athletic Robot Control

**重点关注**：False

**开源情况**：完全没开[2026-10-15]（指本文系统实现；本次官方检索未找到系统代码、策略权重及训练／仿真／部署入口；[G1 数据记录](https://doi.org/10.5281/zenodo.21135719)已发布，元数据列有参考轨迹、指标与实机日志，Atlas 数据不含；本次未直接读取数据包，未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-01-30（所读 v1，同日，正文及附录 S1–S6／表S1–S7，对照 PDF；正式版仅核验出版信息）；一作（共同）：Jean-Pierre Sleiman，未标共同一作，前五名作者为按贡献排序的 core contributors；通讯：Farbod Farshidian；单位：RAI Institute、Boston Dynamics；是否中稿：是，[Science Robotics 2026](https://doi.org/10.1126/scirobotics.aec7695)，2026-07-17 接收、2026-08-12 出版（[出版商登记](https://api.crossref.org/works/10.1126/scirobotics.aec7695)）；未找到独立项目主页，[机构介绍](https://rai-inst.com/resources/in-the-press/zest-zero-shot-embodied-skill-transfer-for-athletic-robot-control/)，[arxiv](https://arxiv.org/abs/2602.00401)。

**核心问题**：用统一、单阶段的运动模仿 RL 配方，将动捕、单目视频和关键帧动画变为可直接迁移真机的动态、多接触控制策略，减少逐动作奖励与接触标注工程。

**系统拆解**：总体为参考数据 → 仿真训练 → zero-shot deployment（[PDF 图3／p.9](https://arxiv.org/pdf/2602.00401v1#page=9)，HTML 图号偏移）。部署输入当前本体观测、上一步动作和下一时刻参考，输出关节残差，经参考叠加及 PD 落到执行器；没有已实现的语言规划器或在线场景感知闭环。以下名称与层级依照原文 Materials and Methods：

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Systems and Modeling | PLA 的 Locally Projected／Dynamic Armature／Nominal Armature Model；Spot 执行器模型 | 硬件与闭链参数 → 等效转子惯量近似、功率／损耗建模 → 仿真模型及 PD 增益依据。 |
| Reference Motion Pipeline | MoCap／ViCap／Keyframe Animation；Kinematic Retargeting | 动捕、视频或动画 → 视频以 MegaSaM＋TRAM 重建；人类动作做时空优化重定向 → 目标机器人参考，无需接触标签。 |
| Training Setup and MDP Formulation → Actor and Critic Observations / Actions and Joint-Level Control | 非对称 Actor–Critic；残差动作；joint-level PD | Actor：IMU、关节位置／速度、上一步动作＋下一参考状态 → 残差 a → q_cmd=参考q＋Σa → PD；Critic 另见仿真真值／接触等，仅训练用。 |
| Training Setup and MDP Formulation → Adaptive Reference State Initialization | RSI、轨迹 bins、EMA failure、categorical sampler | rollout 失败／跟踪统计 → 难片段更常被选为初始状态，保留采样下限 → 训练更均衡。 |
| Training Setup and MDP Formulation → Assistive Wrench Automatic Curriculum | 基于模型的虚拟基座力／力矩 | 基座误差、名义动力学、bin 难度 → 提供训练辅助 → 随进步退至零；不部署真机。 |
| Hardware Deployment Pipeline | ONNX、配置元数据、MuJoCo 验证、真机控制 | checkpoint＋配置 → 导出／登记、仿真验证 → 直接部署 Actor＋参考＋PD。 |

[RESULTS／表1及 DISCUSSION](https://arxiv.org/html/2602.00401v1)：Atlas、G1、Spot 展示匍匐、滚翻、舞蹈、爬箱、后空翻等；实机每项用目标技能专用策略，多技能策略仅在仿真分析。表1只统计成功 trials 的误差，姿态误差不含 yaw，不能推出全部技能高成功率。G1 上／下箱各5/5是0.75 m箱高及接近名义初始位姿；部署无箱位姿感知，分布外存在失败。

**训练、复用与新增**：策略训练全部在仿真，但零真机微调不等于完全没有使用真机建模数据。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Systems and Modeling | 解析建模／参数辨识，非神经训练；静态参数来自厂商，Spot 效率参数用实机动态日志优化辨识；部分功率限制细节因专有信息未披露。 | 闭链惯量逐级近似及 Spot 动力模型，兼顾仿真速度与迁移精度。 |
| Reference Motion Pipeline | 自采 Xsens／Vicon 动捕、手机视频、目标机器人动画为离线参考；MegaSaM／TRAM 复用，未披露再训练／冻结／checkpoint；重定向为逐轨迹优化。多技能消融库15段。 | 组合相机／场景与人体重建，时空重定向同时处理尺度、时间和非穿透约束。 |
| Training Setup and MDP Formulation → Actor and Critic Observations / Actions and Joint-Level Control | Isaac Lab 单阶段 PPO、4096并行环境，以在线仿真经验训练 MLP；未报告已有策略 checkpoint 复用或教师蒸馏，初始化细节未披露。每个实机策略约单张L4、10小时；参考提供跟踪目标，非直接 BC。 | 通用跟踪／正则／存活奖励、适度域随机化、特权 Critic、残差动作；PD 按名义惯量定增益，仿真／真机一致（表S3–S7）。 |
| Training Setup and MDP Formulation → Adaptive Reference State Initialization | 不另训网络；利用 rollout 统计更新采样分布。 | 依据失败程度增采难动作／难阶段，减少长序列与多技能训练的薄弱片段。 |
| Training Setup and MDP Formulation → Assistive Wrench Automatic Curriculum | 解析辅助控制与同一难度统计联动，无额外学习策略；最终辅助为零。 | 自动退除支持，让大幅基座旋转等难动作更易探索和收敛。 |
| Hardware Deployment Pipeline | 导出后直接推理，无真机策略微调；Critic 与虚拟辅助不部署。 | 保持策略、配置、仿真验证及发布一致，降低人工部署差错。 |

**关键 Q&A（2026-10-15）**：Q：zero-shot 是否表示新动作／新机器人不用训练？A：不是；它指仿真策略直接迁移真机。跨机器人复用配方与结构，并非同一权重；实机仍逐技能训练，作者明确未测多技能策略对训练集外动作的泛化（DISCUSSION）。Q：与 skill library 的关系？A：可借鉴技能生产流程，但尚未证明通用、可组合的运动内核；视频在离线阶段生成参考，实机不靠视觉适应任意场景。长历史／未来窗口的负面消融也限于该网络、超参数和预算，不能泛化为历史信息无用。

## RelateAnything — RelateAnything: Real-Time Open-Vocabulary Relation Prediction From Any Inputs

**重点关注**：False

**开源情况**：未核实[2026-10-15]（机器人仿真／真机接入未核实：本文是图像关系感知组件，现有分级不宜直接标为可以sim。已核实[官方代码](https://github.com/Maelic/RelateAnything)、[预训练权重及 ONNX](https://huggingface.co/maelic/relsgg-vits16plus/tree/main)、[RA-4M 数据](https://huggingface.co/datasets/maelic/RA-4M/tree/main)、训练配置／入口和图像／摄像头推理入口；未运行。自重训需准备数据并取得 DINOv3 初始权重访问）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-09-11（v1，同日；阅读正文方法、评测定义、§6主要实验、推理开销与局限及附录 A–C 训练／数据部分，结合官方结果页核对，检查源码与模型／数据文件）；一作：Maëlic Neau（单作者）；通讯：Maëlic Neau；单位：Independent Researcher；是否中稿：未核实，官方资料目前按 arXiv 预印本引用；[项目主页](https://maelic.github.io/RelateAnythingProject/)，[arxiv](https://arxiv.org/abs/2609.12552v1)。用户提供的全小写 Demo 路径返回 404，[官方大小写路径](https://maelic.github.io/RelateAnythingProject/demo/)可访问。

**核心问题**：把关系预测从固定谓词分类、检测器类别标签和单数据集标注习惯中解耦：输入图像、任意来源的区域和用户指定关系词，输出有置信度的“主体—关系—客体”三元组。其价值是可插拔的关系感知接口；接到人形机器人前还需独立的空间记忆、动作规划、执行与验证机制，此为本项目的应用判断。

**系统拆解**：整体输入为单帧图像＋框（可选 mask）＋谓词字符串，输出区域索引间的关系分数／关系图。检测器／分割器在模型之外；物体类别名不进入前向网络，类别仅在训练损失中使用。系统不输出关节命令，也没有本文自带的机器人执行器链路。[§3、Fig.2](https://arxiv.org/html/2609.12552v1#S3)，[官方架构](https://github.com/Maelic/RelateAnything/blob/main/docs/architecture.md)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| RelateAnything → Visual path | DINOv3；多层特征融合；SoftSpatialPool | 图像与区域 → 三层 patch 特征归一化融合，框坐标查询整幅特征图 → 主体、客体、union 和 contact 区域特征 |
| RelateAnything → Pair sampling | 几何预筛；learned relatedness | N(N−1) 个有向区域对 → 几何阶段保留 400 对、相关性阶段保留 128 对 → 候选对及 pair logit |
| RelateAnything → Pair representation | 几何编码；RelationTransformer；joint self-attention；DeformableRelRead | 四种区域特征＋19维几何量 → 形成512维表示，结合其他关系对和场景、在四个锚点周围稀疏读取图像 → 上下文化的关系对特征 |
| RelateAnything → Text path | PredicateTextStudent；固定 embedding bank | 用户谓词字符串 → 仅在换词表时编码为512维单位向量 → 可替换的谓词矩阵；逐帧无需运行语言模型 |
| RelateAnything → Two branches and a predicate-conditioned gate | spatial／semantic 两支；谓词门控；VocabHead | 对特征分别生成空间／语义查询，与文本求余弦相似度；仅由谓词 embedding 决定两支权重 → 结合 pair logit 并校准得到分数，可分别解码空间／语义图 |

**训练、复用与新增**：本文需要训练。主关系模型以 RA-4M、raw Visual Genome 和 HICO-DET train 混合监督训练；论文配方按图像采样占比约72.7%／6.3%／21.0%，不能把 HICO 的“5%关系占比”误读为只用了5%训练图像。RA-4M 是借用 MegaSG 图像／框、由 Gemma-4-26B-A4B-it 三阶段流程（semantic、grow、spatial）生成关系，再按确定性几何规则处理；论文报告约47.4万图像、428万关系、10,102谓词，最终训练词表19,103词包含其他语料的词汇，词汇加入不等于使用其图像关系训练。当前发布包与论文统计存在版本差异。标准配方12 epoch、4×A100、每卡32图像，主干学习率5e-5、头4e-4。几何核验只检查可约束的关系，不能证明所有动作语义正确。[§4、附录A/C](https://arxiv.org/html/2609.12552v1#S4)，[训练说明](https://github.com/Maelic/RelateAnything/blob/main/docs/training.md)，[数据生成说明](https://github.com/Maelic/RelateAnything/blob/main/datagen/README.md)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| RelateAnything → Visual path | 复用 `facebook/dinov3-vits16plus-pretrain-lvd1689m` 初始化并全量微调；pooling／融合随关系模型训练，主干没有冻结 | 用整图特征和坐标查询读取区域，避免依赖物体类别；多层融合及对象—文本辅助对齐 |
| RelateAnything → Pair sampling | 几何预筛和相关性头均训练；正例来自已标注关系对，未标注对作降权负例；物体类别只用于估计损失权重 | 限定计算预算，并把 pair 相关性与谓词身份分开建模；相关性也会学到“哪些对容易被标注”的偏差 |
| RelateAnything → Pair representation | 新增关系模块随主模型训练；几何特征计算本身不是训练；deformable read 的残差门初始为0 | 显式保留主体／客体／union／contact与几何信息，并读取框外上下文 |
| RelateAnything → Text path | 本文训练6层学生，蒸馏冻结的 dino.txt 教师；公开训练历史记录 CLIP token embedding 投影初始化；训练完成后学生和词向量用于固定目标，不随关系头训练更新。数据为关系短语、同义／反义词对和空间组合短语 | 通过反义排斥、同义保持与邻域保持等目标分开 above/below 等反义谓词；不是直接拿现成CLIP文本空间打分。[学生实现](https://github.com/Maelic/RelateAnything/blob/main/relsgg/text/student.py)、[训练历史](https://huggingface.co/datasets/maelic/OV-SGG-Bench/blob/main/text_student_v2_512/student.history.json) |
| RelateAnything → Two branches and a predicate-conditioned gate | 新增视觉投影与门控随模型训练，门控无单独标签；batch-local InfoNCE 将同义词组作正例，并对可能同时成立的未标注谓词降权，另有方向交换、sigmoid、背景抑制等辅助项；文本矩阵不经学习投影 | 让几何与语义证据按谓词自动混合，允许替换词表；将遗漏标注视为未知，而非一律负例。[损失与分数实现](https://github.com/Maelic/RelateAnything/blob/main/docs/architecture.md) |

主要证据：用真值框、每有向对限一个谓词的跨数据集评测，VG150／PSG／IndoorVG 的 mR@50 为28.2／30.6／29.5，对照 OvSGTR 为10.4／8.8／12.8；这些是关系预测结果，不是机器人任务成功率。[官方结果与评测条件](https://github.com/Maelic/RelateAnything/blob/main/docs/results.md)。SpatialSense 宏AUC为0.690，但分类准确率62.5%低于原基准仅框基线68.8%，作者明确认为这尚不足证明视觉空间推理能力（不同指标不能混比，[§6.6](https://arxiv.org/html/2609.12552v1#S6.SS6)）。主表跨数据集不等于全部关系概念未见；已发布模型的 HICO 结果包含HICO训练份额，真正零样本对照另外训练。作者也承认缺少 RA-4M 人工抽检精度、若干结论仅单随机种子；图生成的假阳性和检测漏检仍限制应用。建议作为关系感知候选先测机器人视角下的方向、接触与遮挡变化，不据此认定已有可靠三维世界模型。[§10](https://arxiv.org/html/2609.12552v1#S10)。

## Spatial-TTT — Spatial-TTT: Streaming Visual-based Spatial Intelligence with Test-Time Training

**重点关注**：False

**开源情况**：缺少关键组件[2026-10-15]（已公开模型实现、训练／VSI-Bench 评测入口、nano 权重及部分数据；完整论文模型、全量空间 QA 和稠密场景描述数据仍列为待发布，97k 数据仓当前仅含 JSONL，视频需另备；属于视频空间问答模型，不据此判定机器人仿真可用；未运行；[官方代码](https://github.com/THU-SI/Spatial-TTT)，[nano 权重](https://huggingface.co/THU-SI/Spatial-TTT-nano/tree/main)）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-03-12（所读 v1，2026-03-12；正文、图2、实验表1–5、PDF 附录 A–B／算法1，并核对公开代码与资源）；一作（共同）：Fangfu Liu、Diankun Wu、Jiawei Chi；通讯：Yongming Rao、Yueqi Duan；单位：清华大学、腾讯混元、NTU（按原文简称）；是否中稿：是，[ECCV 2026 官方页面](https://eccv.ecva.net/virtual/2026/poster/5426)；[项目主页](https://liuff19.github.io/Spatial-TTT/)，[arxiv](https://arxiv.org/abs/2603.12255)。用户提供的项目路径大小写已按官方链接修正。

**核心问题**：长视频中的空间证据分散在不同视角和时刻；作者在预训练 VLM 中加入随当前序列更新的快权重记忆，并通过稠密场景描述训练它保留空间信息，再用于视频空间问答。

**系统拆解**：输入 RGB 视频／多视角图像和语言问题，经视觉编码、混合解码器输出文本答案；没有 motor skill、执行器或机器人控制闭环。3∶1 指每四个解码器注意力层中三个改为 TTT 层，一个保留全注意力；TTT 层内部的 SWA 与 TTT 分支并行，不是多个独立 agent。[§3.2／图2](https://arxiv.org/html/2603.12255v1#S3.S2)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Vision Transformer | Qwen3-VL 视觉编码与跨模态映射 | 视频帧 → 视觉 token；问题经 tokenizer，二者送入解码器。 |
| Hybrid TTT Architecture → Self-Attention Anchor Layers | 25% 层保留标准因果 self-attention，及原 FFN | 图文 token → 全上下文注意力 → 语义／推理特征。 |
| Hybrid TTT Architecture → TTT Layers → SWA | Sliding Window Attention，与 TTT 分支共享 Q/K/V 投影 | 当前与窗口内历史 token → 局部因果注意力 → 当前块内的时空交互特征。 |
| Hybrid TTT Architecture → TTT Layers → TTT Branch | Spatial-Predictive Mechanism；SwiGLU-MLP fast weights；Muon 更新 | 视觉 Q/K/V 按时间×高×宽重排、逐通道 3×3×3 卷积 → 用先前快权重读取当前 chunk，再由该 chunk 的 K/V 自监督更新记忆 → 与 SWA 输出融合；块长 2648。3D 卷积作用于视频 token 网格，不直接构造世界坐标地图。[算法1](https://arxiv.org/pdf/2603.12255v1#page=17) |

**训练、复用与新增**：这是“离线监督训练＋测试序列内自监督快权重更新”，不是免训练。离线从 Qwen3-VL-2B-Instruct 初始化：阶段1约16k条 SceneVerse 场景图衍生的稠密描述（ScanNet 3.6k、ARKitScenes 12.5k），32帧，前两轮把 SWA 窗口从5600退火到2648；阶段2用空间 QA 继续微调，64→128帧、窗口固定2648。§4.1／附录A.1给出约3M QA（2.5M公开的 VSI-590K、VLM-3R、InternSpatial、ViCA，0.5M依 ScanNet 场景几何与语义标注生成）；§3.4却写2M，规模存在原文不一致。几何、OBB和房间面积用于构造监督，不是推理时输入或在线 RL 经验。[§3.3–4.1](https://arxiv.org/html/2603.12255v1#S3.S3)、[附录A.1](https://arxiv.org/pdf/2603.12255v1#page=16)

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Vision Transformer | 复用 Qwen3-VL-2B-Instruct；公开97k训练脚本冻结 vision 和 merger，论文完整两阶段未逐项明确冻结清单，不能直接等同。[公开配置](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/spatial_ttt_train.sh#L78) | 沿用视觉输入前端，新增重点在解码器和训练监督。 |
| Hybrid TTT Architecture → Self-Attention Anchor Layers | 继承预训练语言模型参数；论文 backbone 学习率1e−6，公开脚本训练整个 language model 与 lm_head，非仅训练 TTT。使用上述描述／QA监督；推理不更新这些慢权重。 | 以3∶1保留 anchor，避免完全替换注意力破坏语义能力。 |
| Hybrid TTT Architecture → TTT Layers → SWA | 复用并训练原注意力 Q/K/V；共享前述监督，SWA 本身不是另一份独立模型／数据集。推理仅维护滑窗 KV，不做此分支权重优化。 | 大块 TTT 下补充块内因果交互；窗口退火渐进转移跨块记忆责任。 |
| Hybrid TTT Architecture → TTT Layers → TTT Branch | 新增快权重初值、更新／门控参数、Q/K scale-shift与卷积共同离线学习，新增参数学习率1e−5；代码快权重为随机初始化后训练，卷积 Dirac 初始化。测试时复制学好的快权重初值，以当前输入 K/V 为自监督信号执行块级 Muon 更新，无需真实答案，也不重训全部 VLM；每次公开 generate 调用重新建立状态。[初始化](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/models/causal_swa_lact.py#L327)、[推理状态](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/models/spatial_ttt.py#L183) | 在 LaCT／TTT 思路上加入空间预测卷积，并用稠密描述和渐进训练学习“应记住什么”；fast weights 是非线性 MLP 隐式记忆。 |

“空间线性记忆”不应理解为显式3D地图或无限视频恒定内存：记忆是 SwiGLU-MLP 参数。TTT 层有固定滑窗和待更新缓存，但25% anchor仍访问完整上下文，公开实现保留其 DynamicCache；因此“整个模型严格线性／恒定显存”不成立。表5在352×480、1024帧时报告11.9GB／799.4TFLOPs，Qwen3-VL-2B为21.2GB／1403.1TFLOPs，证明该范围内更省资源，不能外推无限流式。[§3.2、§3.4、表5](https://arxiv.org/html/2603.12255v1#S4.S4)、[anchor缓存代码](https://github.com/THU-SI/Spatial-TTT/blob/e2e33a62b6f92c33b7e24ff042be9737d05c5bdb/qwen-vl-finetune/models/spatial_ttt.py#L208)

作者报告 VSI-Bench 宏平均64.4（原Qwen3-VL-2B为50.3）、MindCube-Tiny准确率76.2；VSI-SUPER按1fps测10–120分钟，120分钟Recall准确率30.0、Count为38.4，并非长期可靠记忆已解决；部分基线OOM被计0，不能当作正常推理错误。VSI提升同时包含额外监督数据效应，消融移除3D卷积／稠密数据／anchor分别降至62.1／61.3／53.9。对本课题可作为空间记忆候选，尚无机器人任务闭环收益证据，也不能把nano公开权重视为复现了论文全量模型结果。[表1–4](https://arxiv.org/html/2603.12255v1#S4)

## Weave — Learning Whole-Body Dexterous Loco-Manipulation from Human–Object Interactions

**重点关注**：False

**开源情况**：可以sim[2026-10-15]（[官方仓库](https://github.com/xiaohu-art/Weave)已提供 Isaac Sim 5.1／IsaacLab 环境、G1＋Inspire 资产及训练／评估／推理入口，[数据](https://huggingface.co/datasets/appolyn/Weave)含参考轨迹与物体资产；[v0.2](https://github.com/xiaohu-art/Weave/releases/tag/v0.2)实际含 `model_31400.pt` 及训练配置，但这是在 D_test 训练的模型，不能据此复现主实验的未见序列泛化；未找到真机部署入口；未运行）。

**基本信息**：阅读日期 2026-10-15；arxiv上传：2026-09-15（v1，同日；已读正文方法、实验与限制，未见独立训练附录，另核验代码与权重附配置）；一作：Liu Cao，共同一作未披露；通讯：未披露；单位：清华大学交叉信息研究院、清华大学人工智能学院、大连理工大学、香港中文大学，作者的 † 表示部分工作在 CocoMatrix 完成，不是共同一作标记；是否中稿：未核实；[项目主页](https://xiaohu-art.github.io/Weave/)，[arxiv](https://arxiv.org/abs/2609.16683v1)。

**核心问题**：人体动作重定向后，手指“碰到物体”仍不等于能稳定搬运；作者把接触和物体几何同时纳入参考构建与统一 RL 策略，让人形机器人协调行走、平衡和多指抓取。

**系统拆解**：两级流程为 Interaction-Preserving Reference Construction（下称“参考构建”）→ Contact-Rich Skill Acquisition via Reference Tracking（下称“参考跟踪技能学习”）。离线输入是人体 SMPL-X 动作、物体轨迹和几何；执行时输入本体状态、真值物体状态／几何与短时域参考，单一策略输出 29 个身体＋12 个手指驱动关节的位置目标，交给 PD 执行，未接入视觉语言高层。[§IV、图2](https://arxiv.org/html/2609.16683v1#S4)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| 参考构建 → Whole-body retargeting | GMR-style whole-body IK | 人体／物体动作 → 关节与身体对齐、地面约束及平滑惩罚 → 初始机器人交互轨迹 |
| 参考构建 → Contact-aware hand refinement | 接触吸引、force-closure、穿透与正则项 | 初始轨迹＋手指接触 → 固定骨盆和腿，联合优化双臂与手指 → 更可抓稳的交互参考 |
| 参考构建 → Approach motion completion | Kimodo、轨迹拼接 | 交互起始姿态／朝向约束 → 生成不同接近方向的行走前缀 → 接近、抓取、搬运的完整参考 |
| 参考构建 → Reference annotations | 距物体表面的三值接触标签 | 完整参考 → 标注应接触／中性／应分离 → RL 初始化、短时域指令与奖励依据 |
| 参考跟踪技能学习 | 分组编码器、SimBaV2 非对称 actor–critic、PPO；输出接 PD | 本体状态＋物体真值位姿、指尖表面向量、接触标记、BPS-SDF＋短时域参考 → 统一策略 → 身体和手指位置目标；50 Hz 控制、200 Hz 仿真，欠驱动手的其余关节按 mimic 耦合 |

主实验在 9 个已见物体上，训练序列成功率 92.5%，相同物体的未见交互序列为 65.0%；不是未见物体或真机泛化。另一实验的 95.3% 对 91.5% 是**都在 D_test 训练**后的统一策略与九个单物体策略比较，训练总预算分别为 27k 与 9×3k，不能替代前述 65.0%。当前评估代码将未触发失败且到达片段终点计为成功。[§V-B/C](https://arxiv.org/html/2609.16683v1#S5.SS2)，[eval.py](https://github.com/xiaohu-art/Weave/blob/d839eb298358b71aedd0407c7ba9bbb05c14e05e/scripts/rsl_rl/eval.py#L121)

**训练、复用与新增**：

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| 参考构建 → Whole-body retargeting | 使用 OMOMO 人物交互数据；逐轨迹 IK 优化，不是训练神经策略 | 调整骨盆水平约束与地面高度处理，生成符合机器人构型的参考 |
| 参考构建 → Contact-aware hand refinement | 对交互参考逐实例优化，不是 RL；沿用已有接触优化／抓取质量思想 | 在人体到机器人转移时同时约束手指接触、抗扰抓握质量和穿透 |
| 参考构建 → Approach motion completion | 复用 Kimodo；本文所用 checkpoint、是否微调与冻结未披露，不能把 Kimodo 原始训练当作本文训练 | 补齐数据缺失的接近阶段；训练／测试每条原始交互分别生成 3／5 个方向变体 |
| 参考构建 → Reference annotations | 几何距离规则，无学习参数 | 将正负接触监督随参考交给统一策略 |
| 参考跟踪技能学习 | 作者训练统一策略；公开训练入口新建网络，checkpoint 附配置为 `resume: false`，未见预训练控制器微调或冻结子网设置。PPO 在线仿真经验＋9物体离线参考：D_train 7,869条／19.56h，D_test 1,605条／3.67h。SimBaV2 二维权重用 Muon，其余参数用 AdamW；采用参考初始化、提前终止及动力学随机化 | 将机器人／物体跟踪与手指对向抓取、接触匹配奖励联合优化，并在多个物体和动作间共享同一策略；PD 与手指 mimic 耦合直接使用配置 |

参考与执行轨迹须分开：[当前数据卡](https://huggingface.co/datasets/appolyn/Weave)列参考 9,474 条／23.2h、仿真 rollout 8,699 条／21.2h；不能把摘要“约23小时”写成已发布真机数据。复现需显式核对 split：当前 [`train.yaml`](https://github.com/xiaohu-art/Weave/blob/d839eb298358b71aedd0407c7ba9bbb05c14e05e/configs/track/train.yaml)默认读取 D_test、4096 环境，[作者答复](https://github.com/xiaohu-art/Weave/issues/2#issuecomment-5869091444)称实验使用 8×16384 并行环境，GPU 型号与耗时未披露。作者另提供[初版重定向脚本](https://github.com/xiaohu-art/OMOMO2Isaac/blob/force/scripts/humanoid_retarget.py)，完整 Kimodo 接近动作生成链未核实。对于本课题，可将其视为低层交互技能和仿真数据来源候选；感知、参考选择／组合及真机接触迁移仍需另外实现，尚不能视为完整 VLA 或自主 skill library 系统。[§VI](https://arxiv.org/html/2609.16683v1#S6)

## Ego-VCP — Ego-Vision World Model for Humanoid Contact Planning

**重点关注**：False

**开源情况**：可以sim[2026-10-16]（已核实[官方仓库](https://github.com/HybridRobotics/Ego-VCP)提供 Isaac Lab 环境与 G1 资产、低层 policy.pt／onnx、三任务及联合 world_model.pt、数据采集／世界模型训练／MPC 仿真入口；[离线数据](https://huggingface.co/datasets/Hang917/EgoVCP_Dataset/tree/main)含 ball／wall／tunnel 的实际 npz 文件。未找到完整真机通信、相机与部署入口；论文真机演示不等于部署代码公开；未运行）。

**基本信息**：阅读日期 2026-10-16；arxiv上传：2025-10-13（所读最新 v2：2026-03-08；阅读完整方法、实验与实现代码，8页论文无独立训练附录）；一作：Hang Liu，未标共同一作；通讯：论文未明确标注；单位：University of Michigan, Ann Arbor、University of California, Berkeley、The Chinese University of Hong Kong（按v2署名）；是否中稿：是，ICRA 2026，已核[官方会议程序 WeI1I.268](https://ras.papercept.net/conferences/conferences/ICRA26/program/ICRA26_ContentListWeb_4.html)；[项目主页](https://ego-vcp.github.io/)，[arxiv](https://arxiv.org/abs/2510.11682v2)。

**核心问题**：在接触稀疏、视觉部分可观测的条件下，用随机交互离线数据训练潜空间世界模型，再在线比较候选动作后果，决定扶墙、挡球或降低身体通过拱门。它提供了真正“预测候选动作后果再选动作”的控制例子，但尚不是语言驱动的通用技能库／任务harness。

**系统拆解**：当前64×48自我视角深度＋本体感知 → 推断潜状态 → 在世界模型中滚动模拟动作序列并评分 → MPC只执行所选序列首个动作，再接收真实反馈重规划。规划动作是双手末端目标与体高，低层策略将其落实为关节目标；**真机底盘速度由人操作，不在本文高层规划动作中**。[§III、Fig.2–3、§IV-E](https://arxiv.org/html/2510.11682v2#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Low-Level Controller | 本体感知策略；PPO；关节目标跟踪 | 角速度、重力投影、关节状态及速度／末端位置／体高命令 → 控制策略 → 关节位置目标，由底层执行器跟踪 |
| Ego-Vision Humanoid World Model → observation encoder／decoder | 图像CNN、本体MLP；随机潜变量 z | 深度、本体状态及动态记忆 h → 后验编码 → z；训练时由(h,z)重建观测，促使潜状态保留任务信息 |
| Ego-Vision Humanoid World Model → recurrent dynamics／latent prior | GRU/RNN；预测先验 | 上一步(h,z,a) → 更新动态记忆h；未来无图像时从h预测z → 供候选轨迹在潜空间展开 |
| Ego-Vision Humanoid World Model → surrogate value／termination heads | Q(h,z,a)；D(h,z) | 当前或预测潜状态＋候选动作 → 估计回报与失败概率 → MPC评分；预测终止概率超过0.9时后续价值置零 |
| Value-Guided Sampling MPC | Cross-Entropy Method（CEM）；滚动时域执行 | 候选动作序列 → 世界模型预测、平均多个时刻Q值、CEM筛选 → 当前动作；论文默认1024条候选、4步时域，真机25Hz |

**训练、复用与新增**：“无示范”指世界模型数据由已经训练好的低层控制器执行平滑随机高层命令产生，不是机器人无需训练。三类仿真环境采集(o,a,r,d)，归一化动作空间用增量随机游走（步幅0.32）；世界模型训练离线完成，低层PPO仍需在线仿真经验。其灵感来自 Dreamer／JEPA，但没有复用两者预训练权重。[§III-A–C](https://arxiv.org/html/2510.11682v2#S3)、[采集与训练入口](https://github.com/HybridRobotics/Ego-VCP/tree/main/ego_vcp/scripts)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Low-Level Controller | 本文说明仿真PPO训练，非遥操作示范；公开训练入口默认不恢复checkpoint。采集／规划阶段直接加载发布的低层policy，不随世界模型更新；原始论文低层完整训练成本未量化 | 将可追踪的速度、双手目标与体高作为高层接口，避免世界模型直接搜索全部关节控制 |
| Ego-Vision Humanoid World Model → observation encoder／decoder | 新建CNN／MLP并训练，公开train_wm.py从新模型开始，未加载视觉基础模型；离线深度和本体序列以观测重建NLL监督；训练参数统一优化，无冻结预训练视觉塔 | 同时压缩视觉和自身状态，用任务相关潜变量避免规划时逐像素生成未来图像 |
| Ego-Vision Humanoid World Model → recurrent dynamics／latent prior | 与编码器联合从头训练；离线状态动作序列监督；带stop-gradient的双向KL使预测先验与观测后验一致。代码配置h=256、z=32；stop-gradient不是整个模块永久冻结 | 用记忆h处理部分可观测性，未来由先验滚动而不依赖不存在的真实观测 |
| Ego-Vision Humanoid World Model → surrogate value／termination heads | 与世界模型训练；终止标签作BCE，Q回归离线轨迹的Monte Carlo回报，非专家动作标签，也不是在线TD策略优化；代码用折扣0.99并截断终止后回报 | 学到较密集的价值评分，替代直接累加难预测的稀疏接触奖励；Q只是近似启发式，作者不声称其等于最优Q |
| Value-Guided Sampling MPC | 无单独可训练策略权重；推理时固定世界模型，在每个控制周期优化候选序列。CEM是在线搜索，不是模型微调；当前发布配置默认2000候选，与论文1024应区分 | 对多时刻Q取均值以缓解高方差估计，结合短时域和反馈重规划；该目标与标准最优控制目标一般不等价 |

实验边界：定量对比与消融均在仿真，三个任务以各自reward报告，统计跨3种随机种子与10次独立试验；不能把reward换算成成功率。§IV-A报告0.5M离线转移可完成三任务，但没有把低层PPO训练成本计入该高层数据优势，也未比较SAC等off-policy方法。Table I中4步对挡球／过拱最好，TD-MPC在扶墙reward上反而更高；所以结论是这组视觉接触任务的综合取舍，并非所有任务胜出。G1＋D435i真机展示扶墙、挡未见箱子、过拱与任务衔接，未给统一真机成功率；长期空间地图、任意目标指定及自主速度规划仍需另补。[§IV、Table I](https://arxiv.org/html/2510.11682v2#S4)。

## WAP — World Action Planner: Generalizable Robot Decision-Making with Action-Conditioned World Models

**重点关注**：False

**开源情况**：缺少关键组件[2026-10-16]（[仓库](https://github.com/XiangchengZhang/world-action-planner)已开世界模型训练／推理、WebSocket 服务、LIBERO 手工动作想象 demo；[权重](https://huggingface.co/XiangchengZhang/world-action-planner/tree/main)含仿真世界模型、diffusion policy 与逆动力学模型。未见完整 VLM 提案／评价／排序、全局—局部搜索及整套评测／真机部署入口，真机适配权重未见；训练数据路径须自行配置；未运行）。

**基本信息**：阅读日期 2026-10-16；arXiv 首次上传 2026-07-30，所读最新 v2（2026-10-02）；读正文方法／实验、附录 C–E 的实现／训练／实验关键段落，理论结论与假设已读、证明未逐行复核。一作（共同）：Xiangcheng Zhang、Runhan Huang；通讯：未披露；单位：Harvard University；是否中稿：未核实（原文标 Preprint）；[项目主页](https://worldactionplanner.github.io/)，[arXiv](https://arxiv.org/abs/2607.27599)。

**核心问题**：VLM 能提出操作目标，但难预判碰撞和局部策略能否接手；作者把低层动作转成机器人骨架图，以动作条件视频世界模型想象执行，再用 VLM 修正和筛选目标姿态。

**系统拆解**：输入语言任务、多视角图像／历史和机器人状态；VLM 提出目标→控制器生成动作→世界模型预测视频→全局修正与局部搜索→真实执行、观察、在子任务起点重规划。输出机器人动作，不是仅生成视频。下表按正文 §3 与附录 C 的组件组织；Optional Policy 和 Robot Controller 均属于 World Action Planner，不能把它们合称一个端到端策略。[原文 §3、算法1、附录C](https://arxiv.org/html/2607.27599v2)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Pose-Image Conditioning for Action-Controlled Robot World Modeling | 机器人骨架渲染、视频 VAE、DiT | 当前状态＋候选动作→仅用机器人模型计算未来关节并渲染骨架；骨架与历史图像条件化视频模型→预测环境交互视频。无未来真值关节／物体状态输入，骨架并非准确物理预测。 |
| World Action Planner → Action Proposals | VLM agent | 指令＋多视角观察→MOVE／ROTATE／GRASP／RELEASE；多视角像素点三角化成 3D 目标，依赖相机标定。 |
| World Action Planner → Optional Policy | Diffusion Transformer | 合适的初始状态＋任务→局部操作动作；候选目标后继续想象该策略，以检查能否接手。 |
| World Action Planner → Robot Controller | 逆动力学／底层控制器 | 当前与目标末端姿态→动作块；真机按 10 mm 路点调用机器人控制器。 |
| World Action Planner → Global Trajectory Optimization | VLM 反馈优化 | 候选动作的想象视频→判断碰撞等问题、提出目标位置方向修正→新动作。 |
| World Action Planner → Local Search and Refinement | GridSearch、VLM 排序 | 目标位置邻域及偏航角 ±45° 候选→分别想象并排序→选中动作执行；搜索的是目标空间。 |

**训练、复用与新增**：世界模型、可选操作策略和仿真控制器有独立训练；推理搜索不等于训练。新任务／真机实验另有世界模型适配，不能读成整套零样本。训练依据为附录 C、D.1–D.2；代码冻结情况来自公开 `world_model/algorithms/wan/wan_t2v.py`。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Pose-Image Conditioning for Action-Controlled Robot World Modeling | 基于 Wan-T2V-1.3B（代码 Wan2.1-T2V-1.3B）训练 DiT；代码冻结 VAE／文本编码器。LIBERO-90 基础训练每任务 100 轨迹＝40 示范＋40 σ=.08 噪声重放＋20 σ=.16 重放，10k 步、batch64、16 H100。新布局以原布局示范及额外 10 条噪声轨迹适配；两项新任务用 50 条 VLM 探索轨迹训练 10 epochs；真机用 10 条失败探索、100 步适配。 | 用机器人模型而非场景模拟器把动作映射成姿态图；多视角拼图预测，flow matching 仅监督视频 token。新任务不需要专家示范，不等于不需要数据。 |
| World Action Planner → Action Proposals | 直接调用 Gemini 3.0 Flash；未报告本文微调。推理时输入观察、指令与提示模板。 | VLM 生成可执行目标／原语，作为后续想象搜索的起点。 |
| World Action Planner → Optional Policy | 作者训练 12 层、8 头、512 维 diffusion transformer，初始化／冻结范围未披露；LIBERO-90 每任务 30 条专家示范，预测 40 动作、执行 10。新布局用原布局每任务 5 条示范继续训练；无此策略的新任务／真机不能归入同一训练设置。 | 将局部操作能力作为可选先验，世界模型检验后续策略 rollout。 |
| World Action Planner → Robot Controller | 仿真用 UNet diffusion inverse dynamics，以完全随机轨迹训练，无任务信息；规模／初始化未披露。真机复用 Franka 底层控制，无神经控制器训练。 | 目标姿态与低层动作分离；真机抓放还含手写升降／开合原语。 |
| World Action Planner → Global Trajectory Optimization | 无单独训练模块；复用 VLM 与世界模型，以推理反馈逐次修改候选。 | 从想象轨迹诊断问题，作方向性目标修正。 |
| World Action Planner → Local Search and Refinement | 无单独训练模块；邻域枚举、世界模型 rollout、VLM 排序均为推理；原文披露搜索尺度及提示。 | 低维目标网格搜索，可叠加可选策略 rollout。 |

作者表3报告组合任务／新布局／新任务／真机成功率 72.0／82.0／78.0／65.0%，对应 5／6／2／2 个所选任务，不能当成全套基准或跨本体部署成绩。真机是 Franka Panda、双外部相机、绿色与紫色易拉罐各 20 次，世界模型只在绿色罐探索数据适配；不是人形全身控制。不同机器人上的额外结果主要验证世界模型预测。A100 上预测 20 帧约 17 秒，全局优化约 30 秒、局部搜索约 2–3 分钟；真机只用全局优化，不能据此推定实时闭环性能。对本课题的启发是“用预测检验技能起点／后果”；是否能支撑人形控制及规划时延仍待验证。[表3、附录D.2/E](https://arxiv.org/html/2607.27599v2)

## Thea — Towards the Harness of Embodied Agents

**重点关注**：False

**开源情况**：缺少关键组件[2026-10-16]（[仓库](https://github.com/EIT-HAI/Thea)已给 harness、模型接口、评估与仿真适配代码，但[仿真说明](https://github.com/EIT-HAI/Thea/blob/main/simulation/README.md)要求自行提供 primitive policy、安装基准及实现 factory；[真机模板](https://github.com/EIT-HAI/Thea/blob/main/examples/port-template/factory.py)仍留 SDK、感知、策略等实现空位。未见论文策略权重及其训练数据／入口；不能据终端模式或适配器判定机器人系统可直接复现；未运行）。

**基本信息**：阅读日期2026-10-16；arXiv首次上传2026-08-03，所读为同日v1，当前仅此版；已读正文方法、实验与附录工具注册、概念对照和执行日志，核查官方源码。首页“Technical Report, July 2026”不是上传日期。一作Qi Wang；共同一作、通讯未披露，†明确表示Core contributors，不等同共同一作；单位Eastern Institute of Technology, Ningbo（宁波东方理工大学）；是否中稿未核实。[项目主页](https://eit-hai.github.io/thea/index.html)，[arXiv](https://arxiv.org/abs/2608.11246)。

**核心问题**：怎样让通用模型可靠调度已有机器人能力？Thea把运动能力包装为工具，再用持久场景图与独立动作评估形成“观察—调用—验证—恢复”循环；新增重点是运行框架，并非新的全身控制策略。

**系统拆解**：输入用户任务、当前图像／状态和机器人能力，输出串行工具调用，由工具后的SDK、规划器或学习策略驱动执行器；动作后重新观察并评估。以下保留[§3–4](https://arxiv.org/html/2608.11246v1)的章节层级。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| System Architecture → Agentic Loop | 单步VLM决策、执行、反馈 | 当前上下文 → 每轮至多一次工具调用 → 执行后重决策；无调用则结束任务。 |
| System Architecture → Context Engineering | Resident／Refreshed／Accumulated | 常驻知识、最新观测、执行历史 → 分类组装与压缩 → 模型上下文。 |
| System Architecture → Tool Protocol | MCP schema、BuiltinTool、pre/post hooks | 名称和参数 → 安全检查及后端执行 → 结构化结果；POST_CONDITION仅供评估器。 |
| System Architecture → Skills | SKILL.md、load_skill | 元数据常驻，正文按需加载 → 提供操作建议／约定；本身不执行动作。 |
| System Architecture → Memory | Task Notes、MEMORY.md、tool_experience | 任务记录与客观反馈 → 结束时整理候选经验 → 跨任务文本记忆。 |
| System Architecture → Safety | clearance、执行约束、失败预算 | 新鲜四向净空及动作 → 拒绝／限制底盘移动，超预算求助 → 约束执行。 |
| System Architecture → User Interaction | query_user、notify_user | 不确定性／进度 → 阻塞询问或非阻塞通知 → 用户反馈。 |
| Bridging the Gaps → Scene Graph as Context | Persistent World State、引用查询 | 感知与已确认执行结果 → 更新稳定物体ID、粗3D框及on/inside/holding等关系 → 可查询世界状态。 |
| Bridging the Gaps → Evaluation as Exit Codes | 独立Evaluator、分段停止机制 | 当前同步观测＋动作后置条件 → 判断process/success/failure及原因 → 继续、停止或恢复。 |
| Bridging the Gaps → Embodiment Profile | Operational Envelope、Perception Configuration、Base-Relative Positions | 身体尺寸、可达域、相机／夹爪配置 → 常驻描述 → 为决策提供具身约束。 |

**训练、复用与新增**：模型推理、文档更新与策略训练须分别理解；本文没有训练新的高层决策模型，但不能因此称整个机器人系统“无需训练”。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| System Architecture → Agentic Loop | 实验复用GPT-5.5，reasoning effort=high；未报告本文微调，预训练数据不属于本文数据。 | 用执行事实逐轮校正决策。 |
| System Architecture → Context Engineering | 无参数训练；消费实时观测、工具结果及已有文档。 | 区分更新频率，避免历史图像替代当前状态。 |
| System Architecture → Tool Protocol | 复用SysNav、Astribot SDK及CaP-X／ACT／π0.5。实验端到端策略每任务200条示范训练或微调；ACT、π0.5也是Thea工具背后的同一策略，不能全部算作仅基线数据。基础checkpoint、冻结范围及完整训练流程未披露。 | 统一异构能力接口，并将动作成功与函数返回分开。 |
| System Architecture → Skills | 人工编写／维护的SKILL.md和资源；没有梯度训练。 | 渐进加载程序性知识，区别于工具契约和底层motor policy。 |
| System Architecture → Memory | LLM依据执行历史整理文本；候选经验经过置信度门槛，非更新模型权重。 | 把任务笔记与长期经验分开，限制未经证实内容进入记忆。 |
| System Architecture → Safety | 几何量、规则与执行预算；非RL训练。 | 在模型之外强制检查；不是经认证的物理安全系统。 |
| System Architecture → User Interaction | 消息通道，无学习数据；实验使用飞书／Lark。 | 将人类补充信息接回闭环。 |
| Bridging the Gaps → Scene Graph as Context | 复用Astribot SDK／SysNav感知；具体感知checkpoint及冻结情况未披露。在线图更新不是模型训练。 | 用稳定引用保持对象连续性；动作引起的变化需验证后写入。 |
| Bridging the Gaps → Evaluation as Exit Codes | 复用Qwen3.7-Plus，未报告本文微调；90个检查点用于评估，非训练集。 | 不读决策理由／自报成功，仅依客观证据判断；连续策略分段检查，程序式策略结束后检查。 |
| Bridging the Gaps → Embodiment Profile | 静态人工配置，无参数训练。 | 将机器人配置外置；换文档仍须实现对应硬件和策略适配。 |

[§5、图8–9](https://arxiv.org/html/2608.11246v1)主任务定量实验在轮式双臂Astribot S1：单物抓取、多物整理、导航取放分别每方法20／20／15次；另展示Cobot Magic、G1，不等于完成同规模跨人形平台评测。评估器在三平台各类均衡的90个检查点上准确率93.3%，但6.7%的失败与13.3%的进行中状态被误判成功；这个数是动作状态分类准确率，不是任务成功率。独立评估提供可研究接口，也仍是闭环失效来源；场景图只表达粗状态，不能直接证明可抓取或成功。对当前人形方向，可复用的是上下文／工具／验证边界，底层灵巧操作、行走控制及可靠感知仍须另行解决。

## WholeBodyWAM（WBC-Grounded版）— WholeBodyWAM: Generalizing Pre-trained World-Action Priors to Humanoid Loco-Manipulation via WBC-Grounded Coordination

**重点关注**：False

**开源情况**：完全没开[2026-10-16]（范围为本文2609.16644系统：[官方项目页](https://wholebodywam.github.io/)提供方法与演示，未找到官方UWBC／CASA实现、WholeBodyWAM权重、15K训练数据、训练配置或仿真／真机部署入口；所复用DreamZero和WBC的开放材料不能替代本文系统；未运行）。

**基本信息**：阅读日期 2026-10-16；arxiv上传：2026-09-15（所读最新v1，同日；阅读8页全文的方法、训练、实验与消融，原文未附独立训练附录；核对PDF作者页与官网）；一作：Zhuo Li（标Project Lead）；等贡献：Yiming Yao、Jim Tan（两人标∗，不扩写成前三人共同一作）；通讯：Fei Chen；单位：The Chinese University of Hong Kong、The University of Hong Kong、Peking University、Φ-Institute；是否中稿：未核实；[项目主页](https://wholebodywam.github.io/)，[arxiv](https://arxiv.org/abs/2609.16644v1)。同月另有同名论文，引用需保留本篇副标题及arXiv ID。

**核心问题**：把已有world-action模型的手臂操作知识迁到人形全身动作：保留操作分支，新增统一WBC命令表示，并在手臂当前方向可操作性下降时增强操作意图对身体调整的影响。对本课题更直接的启发是“上层动作表达如何可靠落到底层控制接口”，不是现成的技能库调度或多方案仿真验收harness。

**系统拆解**：语言＋视觉历史＋本体状态 → 共享Diffusion Transformer联合生成未来视觉、操作动作和UWBC命令 → 操作分支给手臂／手指目标，UWBC按所选控制器的能力映射成命令 → 低层WBC执行。论文的概率分解只是依赖关系，**不是先生成视频、再做动作推理；也未报告显式采样多个未来并由价值函数选优的MPC过程**。[§III、Fig.2–3](https://arxiv.org/html/2609.16644v1#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| WholeBodyWAM → context encoding | T5 text encoder；Wan video VAE；state encoder | 指令、视觉历史、本体状态 → 文本特征、视频latent和状态token → 提供DiT条件 |
| WholeBodyWAM → shared DiT／Structured Action Factorization（SAF） | visual、manipulation、UWBC三类token；joint flow matching | 条件与带噪未来block → 联合去噪 → 未来视觉v、手臂关节／手指目标m、全身命令u；保留原有视觉—操作路径 |
| WholeBodyWAM → shared DiT → Coordination-Aware Self-Attention（CASA） | task-directional manipulability；门控attention bias | 当前关节构型、相邻观测末端位移 → 方向可操作性与离线voxel参考比较 → 在同一预测block内增强manipulation→UWBC注意力 |
| Unified Whole-Body Controller Interface（UWBC） | 56维语义槽；能力注册与mask | 46维共享字段（16维任务空间＋15维下肢／腰位置＋15维速度）及10维控制器残差 → 按物理语义、单位与支持能力启用／屏蔽 → 具体WBC可读命令 |
| Downstream WBC | SONIC、AMO、GEAR WBC | 操作目标和已映射命令 → 控制器按自身接口执行全身运动 → G1动作；不能把这三种控制器当作同时运行的三个agent |

**训练、复用与新增**：以DreamZero的预训练14B WAM初始化，在SIMPLE中用PICO 4 Ultra与腕部追踪器采集15K全身示范，转换为SMPL并通过GMR重定向到G1，再依槽位物理语义计算UWBC监督、时间对齐。LoRA后训练rank=4、α=4、50,000步、学习率1e-5、每GPU batch=1；之后还需任务微调，主仿真实验每任务100条示范，另测50／100／200／300条。推理16步二阶UniPC去噪。以上是本文训练，不应把引用WAM／WBC的原始训练配方直接归给本文；完整冻结清单、新模块初始化及真机微调数据量未报告。[§IV-A/B](https://arxiv.org/html/2609.16644v1#S4)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| WholeBodyWAM → context encoding | 复用预训练T5、Wan VAE；具体型号／checkpoint和是否冻结未明确。轻量state encoder随系统提供本体条件，初始化细节未报告；输入来自上述示范和运行时观测 | 让同一个条件上下文服务视觉、操作与全身意图，而不是独立生成互不协调的动作 |
| WholeBodyWAM → shared DiT／Structured Action Factorization（SAF） | 复用DreamZero14B先验，以LoRA后训练及下游示范微调；三模态加权flow-matching监督。保留先验不等于完全不更新；新UWBC投影／头的精确参数训练范围未披露 | 保留已有操作表示，同时新增显式可寻址的全身命令流；避免将所有异质命令简单拼成单一动作向量 |
| WholeBodyWAM → shared DiT → Coordination-Aware Self-Attention（CASA） | gate由运动学、关节速度限制和离线采样可操作性地图计算，不是另训的RL策略；注意力所在DiT随上述训练适应该偏置。末端位移估计任务方向，离线地图计算不是学习示范 | 当手臂沿当前方向的能力相对下降，选择性增加UWBC query对同block manipulation key的注意力，双臂取较大需求 |
| Unified Whole-Body Controller Interface（UWBC） | 槽位含义／单位与能力mask是预定义接口；训练监督由SMPL→GMR后的机器人参考动作推导，生成这些命令的网络随DiT训练 | 共享物理语义，保留控制器特定的10个残差槽；未支持字段屏蔽，不声称一个命令向量无适配驱动所有WBC |
| Downstream WBC | 复用已有SONIC／AMO／GEAR WBC，本文未明确给出它们的重训或checkpoint。跨WBC实验对每种控制器分别做任务微调，不是固定单模型直接替换控制器 | UWBC把上层意图映射到已有执行接口，降低控制器语义差异带来的适配负担 |

实验边界：SIMPLE六任务、三个扰动等级、SONIC执行时平均成功率91.9%，DreamZero为65.0%、使用同一15K后训练数据的DreamZero-PT为73.6%；去CASA／UWBC／SAF分别降到86.9%／84.7%／80.8%。跨三WBC分别微调后平均89.2%，成功率方差10.5 pp²，对Cosmos-3的35.0 pp²降低70%，不能写成零样本可迁移性。G1＋两只BrainCo Revo 2手＋头部及双腕三台D435i RGB相机的八任务真机实验，每方法／任务20次：ID／OOD成功率81.3%／68.8%，DreamZero为57.5%／40.0%；OOD任务进度0.82对0.59，即摘要的+0.23。真实对照仅DreamZero，缺DreamZero-PT等同数据真机对照；不宜将全部收益单独归因CASA或一般开放世界能力。[Table II–III、Fig.4、7、§IV](https://arxiv.org/html/2609.16644v1#S4)。

## WAA — World Action Agent: Harnessing VLMs for Robot Manipulation via World Action Rehearsal

**重点关注**：False

**开源情况**：完全没开[2026-10-16]（检查原文、arXiv、[通讯作者论文页](https://www.yingcong.me/publication/2026-zhang-world-action-agent/)及可定位公开资源，未找到本文官方系统代码、演化技能库、小模型适配权重、轨迹数据或仿真／部署入口；论文给出接口、提示词与训练配置，不能代替实现；未运行）。

**基本信息**：阅读日期2026-10-16；arXiv首次上传2026-09-24，所读为同日v1，当前仅此版；已读方法、实验及附录视图、工具、提示词、微调和失败分析。共同一作Yehang Zhang、Haojian Huang；通讯Ying-Cong Chen，Zexi Li的†表示项目负责人；单位HKUST(GZ)、CUHK、Knowin AI；是否中稿未核实，arXiv标注Working in progress；未提供独立项目主页。[arXiv](https://arxiv.org/abs/2609.29964)。

**核心问题**：怎样让VLM把视觉中的空间判断变成可修正动作？WAA构建带几何定位、虚拟动作预览与图内微调的工作空间，并把示范提炼为图文技能；执行靠运动规划和控制接口，技能库本身不是motor policy。

**系统拆解**：输入任务、场景点云／重建几何、机器人状态，形成Canvas给Main Agent；模型调用查询、提案或执行工具，动作后刷新画面。按[§3、附录C–D](https://arxiv.org/html/2609.29964v1)保留工作空间与学习路径；这里的rehearsal是目标姿态、IK及轨迹可行性预览，没有学习式接触动力学或未来视频预测。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Main Agent | VLM、query/proposal/execution tools | 任务＋当前Canvas＋控制上下文 → 选择工具和参数 → 空间查询、待执行提案或物理动作。 |
| Visual Action Workspace → Interaction-centered Canvas | Global view、两路Contact views | 场景几何＋机器人FK＋活动区域 → 优化可见性、紧凑性和视图互补性 → 可定位到3D的多视图。 |
| Visual Action Workspace → Action rehearsal | Main／Imagination Agent、IK、cuRobo | 目标点／抓取候选 → 构造姿态、规划并叠加虚拟机器人 → 可编辑提案；调用execute_action才执行。 |
| Visual Action Workspace → Closed-loop execution and in-view correction | 图内拖动、局部旋转、夹爪控制 | 已规划动作及新观测 → 执行大动作，再把Contact view像素偏差映射为有限末端位移 → 更新实态。 |
| Learning and Evolution through the Harness → Multimodal manipulation skills | 技能文本、关键状态图、Skill Agent | 当前问题＋技能 → 单独查看历史参考图并判定适用性 → 建议与不确定性，无机器人控制权限。 |
| Learning and Evolution through the Harness → Skill evolution from demonstrations and teaching | Learner、Editor、Reviewer | 示范视频或人工接管过程 → 提取证据、编辑并审核 → 新增／修订／淘汰图文技能。 |
| Learning and Evolution through the Harness → Learning to pilot | 小VLM主代理 | Canvas、任务、控制上下文及技能建议 → 模仿教师工具调用 → 替换Main Agent。 |

**训练、复用与新增**：主实验的技能演化是非参数学习；另有独立的小模型LoRA实验。两者训练数据来源、泛化口径不同。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Main Agent | 默认直接复用Gemini 3.7 Flash，主实验不微调；执行时子agent和grounding也用该模型。输入当前图像／几何是推理观测。 | 将VLM决策落到有限工具接口；终止自评与环境成功判据分离。 |
| Visual Action Workspace → Interaction-centered Canvas | 几何视角优化不训练。比较仿真场景点云、四视角RGB-D＋FK手模型、8张RGB经VGGT重建；VGGT具体checkpoint／冻结情况未交代，未报告本文微调。全部实验输入来自仿真。 | 围绕当前交互生成互补视图，将图像点击关联到空间锚点。 |
| Visual Action Workspace → Action rehearsal | 复用IK与cuRobo；VLM在虚拟提案中迭代，不以试动作训练控制策略。 | 将意图与执行分开，先看规划反馈和姿态投影再决定执行；不预测抓取后的物体动力学。 |
| Visual Action Workspace → Closed-loop execution and in-view correction | 数值规划／低层控制器，无本文motor-policy训练；局部拖动／旋转可直接执行。 | 保留视觉闭环的厘米级修正；规划可行不等同任务成功。 |
| Learning and Evolution through the Harness → Multimodal manipulation skills | 技能存适用条件、程序、参考状态图和可见成功条件；查询不更新模型参数。 | Main Agent拿完整文本，Skill Agent隔离历史图像，随当前观测变化使旧建议失效。 |
| Learning and Evolution through the Harness → Skill evolution from demonstrations and teaching | Codex GPT-5.5按API生成种子；再取LIBERO-90每个源任务一条专家轨迹，用GPT-5.5的Learner／Editor／Reviewer演化；评测前冻结库，不用LIBERO-Pro／robosuite失败继续调库。炉灶单示范案例还用对齐末端朝向，不能泛称纯视频。 | 将视频／人工指导转成可审查的程序性知识，不更新模型权重，不直接生成新motor policy。 |
| Learning and Evolution through the Harness → Learning to pilot | LIBERO-Pro六split中112条成功Gemini轨迹、1774次主代理调用；Qwen3.5-9B基座LoRA(rank8, α16)，冻结视觉编码器及多模态投影器，21.6M可训练参数。仅目标工具调用计损失，10epoch、lr5e-5；4×H20约9.5h，无独立验证集，取末轮adapter。 | 用harness记录训练小模型操作同一接口；只替换主代理，子agent和grounding仍依赖Gemini。见[附录E、表7](https://arxiv.org/html/2609.29964v1)。 |

[§4、表1](https://arxiv.org/html/2609.29964v1)LIBERO-Pro共六split、每split60次，完整方法平均75.6%基于仿真场景点云；RGB-D为71.1%，VGGT为68.9%；同框架无技能28.9%、种子技能43.3%。技能来自LIBERO-90且评测冻结，不能把“小模型也未用LIBERO-Pro训练”作为结论。Show-Harness同Gemini骨干由作者重跑，每split30次；其余基线引用原论文成绩，数据与评测条件并非全统一。robosuite三任务只检验这套操作接口的迁移，未做人形／真机验证。小模型“域外”1.7%→43.3%指LIBERO-Pro内未见seed，不是新物体、新任务或新机器人泛化；此时云端子agent仍在。[§4.3、附录E–F](https://arxiv.org/html/2609.29964v1)也显示视角误判、目标身份丢失、错误宣告成功；Reviewer审核技能知识，并非Thea式独立逐动作成功评估器。对本项目，值得验证的是空间交互接口和可审查技能记忆，不能据此推导全身稳定性、接触安全或实时性能。

## RoboHarness — Memory-Driven Orchestration of Heterogeneous Robot Policies for Long-Horizon Planning

**重点关注**：False

**开源情况**：完全没开[2026-10-16]（在本论文[官方主页](https://www.robo-harness.com/)及原文范围未见系统代码、Memory Bridge／策略卡实现、自建 500 任务定义、记忆库、真机适配权重与运行／部署入口；底层模型的公开代码不等于框架开源；未运行）。

**基本信息**：阅读日期 2026-10-16；arXiv 首次上传 2026-07-20，所读最新 v3（2026-09-29）；读正文方法／实验及附录 B–D 的训练、真机、技能实现与评测设置。一作（共同）：Jinbang Huang、Zhiyuan Li；通讯：未披露；单位按 v3：Huawei Noah’s Ark Lab、University of British Columbia、University of Toronto、McGill University、Department of Foundation Model, 2012 Labs。是否中稿：未独立核实；arXiv 作者备注称获 ECCV 2026 Agent in the World Workshop 最佳论文，[官方 workshop 页](https://interactive-intelligence-lab.github.io/eccv26-aiw-page/)未查到对应获奖名单，不能据此记为 ECCV 主会录用。[项目主页](https://www.robo-harness.com/)，[arXiv](https://arxiv.org/abs/2607.18060)；注意不是同名编号 2603.24060。

**核心问题**：异构策略擅长的任务和可接手状态不同，顺序调用容易失败；作者用 coding agent、策略能力描述、成功经验记忆与 Memory Bridge 路由和衔接策略，并依据执行反馈修改编排。

**系统拆解**：输入任务、图像、状态／物体姿态、策略卡与历史；agent 分解子任务→查询兼容性和记忆→必要时运动规划桥接→调用保留原接口的策略→检查结果、更新记忆与编排。输出经各策略及控制器执行的机器人动作；不是将各策略重训成统一动作模型。[§4、图1、附录C](https://arxiv.org/html/2607.18060v3)

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Coding Agent | Codex / GPT-5.5 | 任务＋观察＋策略描述／反馈→分解、选择与调用技能、重规划；可检查代码与修改编排。 |
| Understanding Skills | 五类 Assessment | 姿态稳定性、视觉／语义相似度、图像质量、状态—策略兼容性→给 agent 的选择证据。 |
| Memory Skills → Memory Management | 策略专属链式轨迹、分层检索 | 成功轨迹保存观察、指令、末端／关节／夹爪状态及特征；先语义后视觉检索相关锚点。 |
| Memory Skills → Memory Bridge | 局部排序模型、目标筛选、运动规划 | 同子任务／夹爪状态下扩展锚点邻域→拟合进度排序→筛选靠近记忆且可达的目标状态→运动桥接、再检查并切换策略。 |
| Heterogeneous Policies as Agentic Skills → π0.5 | VLA | 语言＋原生观察／状态→动作；承担部分操作与真机抽屉交互。 |
| Heterogeneous Policies as Agentic Skills → Faster-WAM | 世界—动作模型 | 原生任务／视觉输入→视频预测和动作输出。 |
| Heterogeneous Policies as Agentic Skills → RL-post-trained OpenVLA-OFT | GRPO 后训练 VLA | 原生语言／观察→操作动作。 |
| Heterogeneous Policies as Agentic Skills → TAMP | PDDLStream、FastForward、OMPL/RRT | 目标＋对象姿态／状态→符号任务与运动计划；含物体姿态、VLM 视觉控制、记忆辅助三种模式。 |
| Evolution Skills | Policy Adaptation、Harness Refinement、Metadata Update | 成败记录→适配抓取／逻辑表示，修正编排代码／参数及策略描述→后续回合继续使用。 |

**训练、复用与新增**：没有统一联合训练不等于无训练；复用预训练模型、本文策略微调、桥接时拟合及跨回合编排适配须分开。以下按附录 B–C 核验。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Coding Agent | 直接调用 GPT-5.5，未报告本文权重微调；输入策略卡、观察与运行记录。 | 以代码 agent 编排异构能力，支持检查与修改运行逻辑。 |
| Understanding Skills | 明确冻结 SigLIP2-base-patch16-224、DINOv2-base、bge-large-en-v1.5；姿态均值／方差、曝光／Laplacian 锐度等为统计规则。兼容性复用记忆与桥接结果。 | 汇集任务／视觉／状态证据；不是训练好的成功概率判别器。 |
| Memory Skills → Memory Management | 从各策略相应数据集的成功轨迹初始化，并追加成功执行；存储／检索无新神经训练。失败也记入供 evolution 使用的执行历史，但不等同于成功轨迹库。 | 分策略记忆与时间链接，保留检索锚点前后上下文。 |
| Memory Skills → Memory Bridge | 每次从检索邻域拟合线性 pairwise ranking SVM；以相对时间顺序为排序标签、归一化机器人状态为输入；是在线拟合。 | 得分只表示局部相对进度；加近邻距离、运动可行性与阈值约束，再权衡运动成本选目标。 |
| Heterogeneous Policies as Agentic Skills → π0.5 | LIBERO 系列复用官方 pi05_libero；RMBench／BEHAVIOR 作者按任务数据训练，30k 步，未完整披露数据规模／冻结范围；真机从 pi05_libero 微调开／关抽屉各 50 条人工示范。 | 将学习操作与 TAMP 结构化执行互补。 |
| Heterogeneous Policies as Agentic Skills → Faster-WAM | 作者按原公开流程在 LIBERO 训练；本文未充分披露基础 checkpoint、数据规模与冻结范围。 | 以原生接口接入策略库，不代表四种策略均用于所有实验。 |
| Heterogeneous Policies as Agentic Skills → RL-post-trained OpenVLA-OFT | 复用 RLinf/RLinf-OpenVLAOFT-GRPO-LIBERO-90；GRPO 后训练是所用模型的来源，未见本文作者重训证据。 | 接入另一种可独立选择的操作策略。 |
| Heterogeneous Policies as Agentic Skills → TAMP | 作者实现规划器组合；几何／符号求解不是策略训练。仿真用模拟器物体姿态并加噪，真机用 ArUco 位姿；记忆模式可执行检索片段前 40%。 | 三种执行模式及记忆辅助；真机能力依赖标记感知与平台 API。 |
| Evolution Skills | SIMPACT 适配抓取，PDDLLM 学习逻辑表示；GPT-5.5 根据失败改编排／元数据及离散参数。仿真启用三项，经验和修改只在同一任务跨评测回合保留。 | 利用执行反馈修正系统，不能当成静态、空记忆的零样本评测。 |

v3 报告 LIBERO-Plus 加权成功率 91.9%、LoHo 五任务 96.0%；自建 LIBERO-Complex 500 实例 86.8%，去掉 Memory Bridge 降至 60.8%。BEHAVIOR-1K 只测六项任务，不代表全部千项。主页仍有旧版 93.2／95.2 等数字，本条以 v3 为准。真机是 UR5e＋双 RealSense D435＋ArUco：五种结构与四种干扰各 15 次，共 135 次，使用 π0.5＋TAMP，不能泛化为四策略／人形真机验证。[§5、表1、图3–5、附录B](https://arxiv.org/html/2607.18060v3)

Memory Bridge 不是分布密度估计或成功概率，也不是“进到记忆邻域就安全”的保证；关节／末端状态相近仍可能面对不同物体布局，桥接后还须结合视觉、任务状态检查。与本课题直接相关的是技能接口、可接手状态和恢复机制；异构策略能否在我们的本体上共享状态表示、获得可靠位姿及及时完成 agent 调用，仍是未验证条件。

## HomeBody（工程项目）— A Humanoid That Explores, Remembers, and Acts on Its Own

**重点关注**：False

**开源情况**：可以sim[2026-10-16]（已核实[官方仓库](https://github.com/Stanford-TML/homebody)提供MuJoCo环境、G1／Dex3与厨房资产、AMO及adapter权重、navigate／pick／place技能、VLM与无VLM脚本入口；限已提供场景与技能，未运行。REAL2SIM和REAL在当前README仍标Planned，日期分别为10月16日和23日，较本次PDF所写12日和19日延后；对应模块仍抛出NotReleased，不能据计划日期归为已发布）。

**基本信息**：阅读日期2026-10-16；arxiv上传：未提供，目前按工程项目而非已核实论文归档；首位署名：Gio Huh，未声明共同一作；通讯：未标注，C. Karen Liu与Guy Tevet的†表示共同指导；单位：Caltech、Stanford University；是否中稿：未核实。阅读范围为[项目主页](https://tml.stanford.edu/homebody/)的方法、FAQ、架构图及官方代码／资产，源码核验版本为[c09fcf3](https://github.com/Stanford-TML/homebody/tree/c09fcf39468e7bab43d744458a53ef1b1265cd6b)；网页Paper按钮未提供论文链接，不据演示补写训练或评测结论。

**核心问题**：如何让人形机器人利用环境记忆、通用VLM和可执行技能，在家庭场景里持续完成移动与抓放。核心价值是把观察、任务选择、技能参数、局部重试和失败反馈接成闭环；不是新训练一个端到端操作策略，也没有展示对多组候选动作的未来物理后果统一评分选优。

**系统拆解**：官网完整流程为Explore采集→Real2Sim重建→任务执行：先建立有尺度的环境与语义记忆，再让VLM根据当前观察和历史结果选择技能及目标，技能内部进行几何规划、执行与验证，失败回传VLM。**当前公开仿真只覆盖后半段**：加载已有厨房、初始语义与地图，再循环观察→决策→技能→结果。它使用MuJoCo；官网Real2Sim构建Isaac Sim数字孪生，不应混为同一个执行后端。[架构与FAQ](https://tml.stanford.edu/homebody/)、[运行循环](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/src/homebody/session/session.py)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Spatial context and memory | Explore；Real2Sim；地图与历史观察 | 官网：iPhone／D435i／LiDAR／关节记录→SLAM几何、VLM辅助重建、ICP对齐与关键帧记忆→空间上下文；公开版加载src_kitchen的地图、语义与初始场景描述 |
| High-level task | GPT Astra；session与技能调用反馈 | 指令、当前视图／手部状态、空间上下文及近期结果→VLM选择技能与参数，或声明done→Runner调用技能并记录结果供下轮决策 |
| Skill bank → Navigate | prepare／execute；占据图、A*、到达检查 | 地图平面目标及朝向→膨胀障碍、搜索和跟踪路径→到达／超时／碰撞等结果，失败可由高层改目标 |
| Skill bank → Pick／Place | 几何抓取／放置、手臂轨迹、局部恢复 | 像素目标及手或放置点→定位、候选抓取、站位和轨迹规划、闭手抬升／释放验证→结构化成功或失败；官网另演示开抽屉／从抽屉取物，当前公共registry只含navigate、pick、place |
| Physical execution → AMO／robot backend | AMO下身控制；上肢IK／伺服；传感与动作接口 | 速度、体姿与上肢目标→预训练AMO和关节控制→G1动作及新观测；控制循环上肢／手250Hz、AMO 50Hz；当前实际提供MuJoCo backend |

**训练、复用与新增**：作者“不需环境专门训练／新增策略学习”的声明限定于本系统的组装与部署，所用VLM、感知网络与AMO都有先前训练经历。当前代码未提供新运动策略训练流程，不能称为所有模块从零训练或未经训练。几何建图、轨迹搜索、视觉伺服与重试属于估计／规划／执行，不等于网络训练。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Spatial context and memory | 官网用采集的多传感器记录建图、对齐和重建，不是遥操作技能示范；数据量未披露。Astra复用现成模型，未报告微调；公开版只给已有场景资产，完整采集重建模块未发布 | 将地图、对象语义、历史视角与机器人定位提供给任务执行；现有代码不能验证重新探索陌生住宅到建模的全流程 |
| High-level task | 复用托管VLM，当前默认gpt-6-astra，未报告本文微调；指令、图像和执行历史是推理上下文，不是监督训练集 | 以统一技能参数和结果组织循环；Runner核对参数、传感时间戳、执行epoch与声明的错误码，并支持中止，便于界定失败发生在哪一层 |
| Skill bank → Navigate | A*与跟踪／重试规则，无独立学习导航策略或训练数据；使用已有地图与在线位姿 | 将可走性规划、执行和到达验证封装为可调用接口；不能把地图目标自动等同于物理可达并成功完成 |
| Skill bank → Pick／Place | 公开仿真用渲染深度／分割及几何计算，无单独训练的抓放policy。官网真机描述复用SAM、Fast-FoundationStereo、SAM2.1／SAMURAI等感知组件，未报告本文微调或数据量；这些真实感知误差不在当前仿真输入中复现 | 用抓取姿态、站位调整、轨迹／IK、接触及抬升验证和有限次恢复组成技能；参数／失败接口有参考价值，但不是任意学习策略无需适配即可替换 |
| Physical execution → AMO／robot backend | 直接复用AMO控制权重、adapter和归一化统计；THIRD_PARTY_NOTICES确认与上游发布文件一致，没有本文重训证据。Holosoma运行代码经适配；上游训练配方不归作本文新训练 | 以统一backend连接观测、全身动作与物理状态；公开真机backend尚为NotReleased占位，不能推断可直接部署到用户平台 |

核验边界：仿真backend直接返回渲染真值深度、分割及由模拟器直接读取的机器人／相机位姿，初始语义也由场景资产提供；不能据此认为已解决真机遮挡、定位漂移或开放词汇目标识别。技能有局部物理验证，但高层done被代码明确记为model_claim，任务物理评分另行计算，不能称为通用可靠的任务完成判定。官网真机视频与FAQ展示整链路并承认模型延迟、硬件问题，目前未见论文式统一成功率评测。依据：[仿真观测实现](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/src/homebody/backends/mujoco.py)、[技能契约](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/src/homebody/skills/contract.py)、[权重来源](https://github.com/Stanford-TML/homebody/blob/c09fcf39468e7bab43d744458a53ef1b1265cd6b/THIRD_PARTY_NOTICES.md)。
