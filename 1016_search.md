# 1016_search — 人形机器人 VLM/LLM 调用技能调研

核验日期：**2026-10-16**；检索窗口：**2025-10-16—2026-10-16 首次公开**。已先与本地 `papers.md` 去重；本文件独立记录，不修改已读库。依据仓库 [paper-review](/Users/xcp/Desktop/chagpt/wu-research/.agents/skills/paper-review/SKILL.md) 逐篇核对原文、调用接口、训练与复用、执行反馈和发布材料。工作区为 `main`，基准 HEAD `87180da0f0e7db2e98c494c6c0b70b881fa2683a`；原有未提交修改，因此未自动 pull、提交或推送。

**结论：新核验的主匹配论文8篇，尚未发现完整上层—技能—人形执行链达到“sim+deployment”或“可以sim”的新增论文。** HANDOFF的底层控制包达到sim+deployment，但上层agent缺失。新增可用仿真材料以BiGym 2.0最完整，不过论文主实验是评测前VLM开发程序；仓库另有在线控制入口，效果需单独核验；`isaac-g1-hierarchical`有在线技能流程，但首次公开日期未独立确认，且抓取／开抽屉采用简化状态绑定。不能把这两项无条件称为已开放的HomeBody替代品。

共保留**17个标准条目**：8篇主匹配（含导航／协作搬运两类窄任务及1篇撤稿）、1项日期待核的工程候选、5项边界扩展、1项仅子模块验证的架构、2项待核实线索。**全部仅核验材料，均未实际执行模型、仿真或真机；本次“已实际跑通”数量为0。** 作者论文中的运行结果与本次验证严格区分。“完全没开”限于核验时未找到本文系统实现，不代表基础模型闭源，也不是永久状态判断。

## 范围、检索与排序口径

检索结合arXiv版本历史与HTML/PDF正文、HRI/期刊正式出版记录、官方项目页、作者主页、GitHub实际文件及历史、Hugging Face模型／数据实体，并沿HomeBody、Thea及新候选引用继续追查。关键词覆盖humanoid/G1/H1与VLM/LLM、hierarchical、skill/tool calling、task planning、loco-manipulation、recovery、code generation、navigation等组合；检索列表只用于发现，纳入依据来自原文／官方材料。首发采用最早可核实公开记录，会议录用、期刊卷期及arXiv修订不重置时间窗；无法确认者独列，不伪装为已确认首发。

主匹配要求上层理解任务与观测，并通过有语义或任务空间含义的接口调用可执行技能／规划器／控制器，且该链有humanoid实验；允许共享一个目标条件策略，不要求每技能独立网络。纯人体角色、四足带臂、轮式双臂、空间问答及无技能调用层的端到端动作模型不纳入。离线VLM程序开发、仅语义调参及仅Monitor验证分别标出，供路线比较，不混进主匹配计数。

开源情况只写四项：**sim复现、sim重新训练、直接部署、缺少材料**。可用范围用短括号标明；尚未发布实现的记“完全没开源”，信息不足的记“未核实”。以上均为2026-10-16材料检查结果，全部未实际运行。

## 对比与复现优先级

| 工作／可核实首发 | sim复现 | sim重新训练 | 直接部署 | 缺少材料 | 人形验证范围 |
|---|---|---|---|---|---|
| [BiGym 2.0](https://arxiv.org/abs/2610.07594) · 2026-10-06 | 可以 | 可以（ACT） | 不可以 | 缺少部署代码 | **主实验为程序开发扩展**；MuJoCo动态G1，20任务资源、9任务主要量化，无真机 |
| [isaac-g1-hierarchical](https://github.com/mturan33/isaac-g1-hierarchical) · 首次公开未核实 | 可以（简化抓取） | 未核实 | 不可以 | 缺少部署代码 | **工程候选**；Isaac Lab G1在线VLM技能；磁吸抓取／抽屉状态写入 |
| [HANDOFF](https://arxiv.org/abs/2606.06493) · 2026-06-04 | 可以（WBC） | 缺少部分训练数据 | 可以（G1 WBC） | 缺少上层规划与技能调用代码 | 主匹配；G1仿真2500任务，真机多阶段操作展示 |
| [EgoActor](https://arxiv.org/abs/2602.04515) · 2026-02-04 | 不可以 | 不可以 | 不可以 | 缺少技能模型、训练数据、仿真与部署代码 | 主匹配；G1移动取放／交互；蹲起仅Habitat |
| [HoloAgent-0](https://arxiv.org/abs/2606.23565) · 2026-06-22 | 不可以 | 不可以 | 不可以 | 缺少技能模型、集成代码及部署配置 | 主匹配；G1/R1导航与交互展示；复杂操作另用轮式平台 |
| [HuGo](https://arxiv.org/abs/2609.30594) · 2026-09-24 | 不可以 | 不可以 | 不可以 | 缺少WBC模型、仿真环境、训练与部署代码 | **离线扩展**；G1仿真5任务，真机按钮／举箱 |
| [Cybo-Waiter](https://arxiv.org/abs/2603.10675) · 2026-03-11 | 不可以 | 不可以 | 不可以 | 完全没开源 | 主匹配；G1五单任务＋三长任务，各10次真机 |
| [C2C](https://arxiv.org/abs/2603.03768) · 2026-03-04 | 不可以 | 不可以 | 不可以 | 完全没开源 | **协作搬运窄匹配**；G1＋真人三任务各5次，另多载体演示 |
| [HumanoidVLN](https://arxiv.org/abs/2608.12860) · 2026-08-13 | 不可以 | 不可以 | 不可以 | 完全没开源 | **导航窄匹配**；四种人形sim，G1真机20集 |
| [KINO](https://arxiv.org/abs/2609.18869) · 2026-09-16 | 不可以 | 不可以 | 不可以 | 完全没开源 | 主匹配；G1仿真与搬箱／提桶真机，依赖动捕 |
| [InterEvolve](https://arxiv.org/abs/2610.02196) · 2026-10-01 | 不可以 | 不可以 | 不可以 | 完全没开源 | **离线扩展**；G1 sim；真机踢／推箱回放sim latent |
| [SafeHumanoid](https://arxiv.org/abs/2511.23300) · 2025-11-28 | 不可以 | 不可以 | 不可以 | 完全没开源 | **调参边界**；G1上身六类演示，既有轨迹 |
| [HumanoidVLM](https://arxiv.org/abs/2601.14874) · 2026-01-21 | 不可以 | 不可以 | 不可以 | 完全没开源 | **调参边界**；G1桌面五组接触任务 |
| [HODAgent](https://arxiv.org/abs/2608.17584) · 2026-08-18 | 不可以 | 不可以 | 不可以 | 完全没开源 | 主匹配但**已撤稿**；G1导航／社交，非抓取搬运 |
| [FRAMES](https://arxiv.org/abs/2609.22538) · 2026-09-18 | 不可以 | 不可以 | 不可以 | 完全没开源 | **仅子模块验证**；MuJoCo G1遥操作技能上的Monitor |
| [MetaWorld](https://arxiv.org/abs/2601.17507) · 2026-01-24 | 未核实 | 未核实 | 未核实 | 代码仓库无法访问 | 论文称VLM→专家→TD-MPC2，HumanoidBench |
| [LG-HSS](https://doi.org/10.1016/j.robot.2026.105736) · 首次公开待核 | 未核实 | 未核实 | 未核实 | 开源材料待核实 | 摘要／章节预览证实MCP技能与G1真机 |

**最值得动手的顺序**：若目标是立即开展有接触物理的G1仿真比较，优先BiGym 2.0的公开环境／WBC／示范／评估入口，再自行接在线技能层；它的公开运行范围材料较齐，但未证明所有论文基线可一键复训。若目标是把控制接口推进到真机，先复现HANDOFF的WBC，接受上层agent与部分训练数据仍需补齐。若只想研究高层VLM模型行为，EgoActor权重适合离线观测→动作测试，不能当完整机器人复现。工程G1适合调接口、技能状态机与重规划，不适合作为接触抓取成功率依据。**严格在线技能调度的现成开放仿真起点，已读HomeBody仍优先；本轮未找到可直接取代它的新增完整包。**

## 与已读库去重及更新检查

[现有记录](/Users/xcp/Desktop/chagpt/wu-research/papers.md)已包含ASB–GAC、VIRAL、KPI、ZEST、RelateAnything、Spatial-TTT、Weave、Ego-VCP、WAP、Thea、WholeBodyWAM、WAA、RoboHarness、HomeBody，本次不重复建立条目。与主题最相关的四项复查结果如下；**未发现足以升级分类的重要发布更新**，故只记录检查结论，不重抄原文笔记。

| 已读工作 | 2026-10-16实际发布检查 | 对本次结论的影响 |
|---|---|---|
| [HomeBody](https://github.com/Stanford-TML/homebody/tree/c09fcf39468e7bab43d744458a53ef1b1265cd6b) | HEAD仍c09fcf3；SIM已发布，REAL2SIM/REAL正文仍Coming soon，计划日期不能当发布证据 | 保持可以sim；机体为G1/Dex3，sim使用真值深度／分割／位姿，任务完成含模型自报；未运行 |
| [Thea](https://github.com/EIT-HAI/Thea/blob/b9c47830155194e53be801eefe8aea6a26a97366/examples/port-template/factory.py) | HEAD b9c4783；factory仍NotImplementedError，[simulation](https://github.com/EIT-HAI/Thea/blob/b9c47830155194e53be801eefe8aea6a26a97366/simulation/README.md)需自接primitive | 保持缺少关键组件；G1定性与Astribot主定量不能混写；未运行 |
| [ASB–GAC](https://leungchaos.github.io/Humanoid-Whole-Body-Manipulation-via-Active-Spatial-Brain-and-Generalizable-Action-Cerebellum/) | 官网Code仍disabled、链接为# | 保持完全没开；既有ECCV正式版信息已在papers.md |
| [KPI](https://github.com/kpi-robot/kpi-robot.github.io) | 仓库仍为网页、视频与assets，没有控制系统代码 | 保持完全没开；不把网页GitHub仓库算系统开源 |

VIRAL等已读控制／动作模型可成为底层技能来源，但缺少本次要求的上层语言观测推理与显式调用，不能仅因人形、开源或高成功率再次列为新匹配。

## BiGym 2.0 — Benchmarking Learned and Agent-Developed Policies for Humanoid Household Manipulation（扩展匹配：开发阶段VLM＋仿真基准）

**重点关注**：False

**开源情况**：[2026-10-16] sim复现：可以；sim重新训练：可以（ACT）；直接部署：不可以，缺少部署代码。[代码](https://github.com/swirl-uk/BiGym2)

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-10-06（所读v2，2026-10-08；已读主文与附录接口、训练、评估、agent开发协议并核查源码）；共同一作Zexi Zhang、Zecheng Zhu；通讯未披露；单位Imperial College London；是否中稿未核实；[项目主页](https://bigym2.github.io/)，[arXiv](https://arxiv.org/abs/2610.07594)。

**核心问题**：将家务操作任务放到真实受步态扰动的G1执行栈上，公平比较学习策略与VLM开发的代码策略。论文的coding-agent主实验研究**开发阶段用LLM生成、试跑并修改程序，正式评测时仅运行定稿程序**。开发期间可以与模拟器交互；“无需LLM”只指主实验的最终评测。仓库另有inloop在线控制入口，不能把主实验协议外推为整个项目不支持在线agent。

**系统拆解**：开发时，VLM读取任务、单条三视角示范视频及API，在预算内试跑/改`policy.py`；执行时，程序读机载三相机＋仿真本体状态（含pelvis世界位姿GT）→上肢关节／夹爪／身体命令→冻结GR00T-WBC运行腿腰→MuJoCo接触动力学。严格版无IK，工具增强版提供IK、标定与像素射线工具。依据[§III/V、附录G](https://arxiv.org/html/2610.07594v2#S3)、[工具接口源码](https://github.com/swirl-uk/BiGym2/blob/main/bigym/loco/agent/templates/api_tools.md)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Agentic policy development | Codex/Claude harness、bounded interaction、policy.py | 任务＋单示范＋API→写代码、通过rollout诊断→冻结程序；没有预建任务技能库。 |
| Agent-developed program → Strict / Tool-augmented interface | NumPy/OpenCV等；工具版IK/相机模型 | 每步三张84×84机载图像、本体状态→程序计算20维（可扩展21维）命令；工具版可调用几何IK，并不提供物体真值。 |
| Loco-manipulation execution → GR00T-WBC | frozen Walk/Balance policies | 基座速度、高度等＋本体状态→腿腰关节目标，和直接给出的手臂、夹爪目标共同执行。 |
| BiGym 2.0 → Environment / demonstrations / evaluation | MuJoCo、VR+Mink、controller state restore | 20场景与原生示范→统一环境；重置机器人和controller状态→固定种子、成功保持1秒、完整trace。 |

**训练、复用与新增**：研究主线包含VLA/IL/RL基线，但这些不应混为“VLM技能调度”的训练。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Agentic policy development | GPT-6 Astra/Claude Opus5.5现成模型，未报告微调；每任务单条示范视频＋101k环境step开发预算，三次独立session；不允许训练网络、拟合回归或构建示范查表。 | 冷启动代码开发协议；冻结后100个隐藏种子测试，10次重复确认确定性。 |
| Agent-developed program → Strict / Tool-augmented interface | 无参数学习；视觉处理、IK、几何工具是代码／规划，不是技能网络。 | 控制相同观测与动作范围，量化经典机器人工具带来的增益。 |
| Loco-manipulation execution → GR00T-WBC | 直接复用冻结NVIDIA GR00T-WBC两套公开ONNX；无本文重训。 | 在采集、训练、reset、评估中使用同一动态执行栈。 |
| BiGym 2.0 → Environment / demonstrations / evaluation | 本文用VR+Mink收集20任务×60=1200条示范；5400条agent评估episode已发布。另论文ACT/DP、CQN-AS/DrQ-v2+/DEAS及π0.5分别学习／微调，是比较基线而非agent训练数据。 | 保留全身物理与controller内部状态，支持可复现回放；任务适配步行G1而非原BiGym悬浮基座。 |

**关键 Q&A（2026-10-16）**：Q：是否给GPT提示与示范视频，让它操作机器人？A：主实验给模型任务说明、API文档和同一次人类遥操作示范的头部／双腕三路同步视频。模型在101k环境step预算内写`policy.py`、试跑、看观测／失败结果并改代码；之后冻结程序，在100个隐藏seed上评测。程序由50Hz的`Policy.act(obs, tools)`接口输出手臂关节、夹爪与身体指令，GR00T-WBC处理行走和平衡；评测中不请求GPT，也不再读取示范视频。这里“离线”意指正式评测前的开发，不是禁止仿真交互或离线RL；代码冻结也不意味着所有程序开环：reaching会用相机像素反馈，drawer closing有开环关节序列，反馈使用取决于生成代码（[§III-E／V-B／附录G](https://arxiv.org/html/2610.07594v2#S5)）。Q：仓库是否也能让模型在线控制？A：有独立[`inloop` CLI](https://github.com/swirl-uk/BiGym2/blob/main/bigym/loco/agent/cli.py)，每episode启动模型session，按[`PROMPT_INLOOP.md`](https://github.com/swirl-uk/BiGym2/blob/main/bigym/loco/agent/templates/PROMPT_INLOOP.md)逐条调用`look/move_hand/walk/gripper`；这更接近在线工具控制，但不能将论文冻结程序的成绩当作该模式的评测结果，本次也未运行。原“不能算在线技能harness已解决”表述过宽，修正为“论文主实验未验证运行时VLM持续选技能／重规划”；公开在线入口仍需单独检验任务验证、恢复及效果。Q：未见初态与GT／感知的边界？A：开发可在60个示范seed上试跑，正式评测在100个隐藏seed上按同一任务分布采样；不等于只见p1后去任意p2或新场景。17任务随机：一般物体位置扰动1–5cm、yaw依任务变化，reaching目标在标称点附近盒内采样，抽屉任务还改变机器人位姿／开度；关洗碗机和两项壁柜为固定初态。主实验不给物体／球的GT坐标，由三路RGB感知；但本体向量含直接的仿真关节状态、pelvis世界坐标x/y/z/yaw及速度，机器人定位不由视觉SLAM估计。因此是“视觉物体感知＋仿真本体真值”，不能笼统称所有输入都由传感器估计。工具增强版另提供手腕／相机位姿和IK，仍不给物体GT；成功评分则由环境真值判定，与策略观察分开（[附录A、B-C/Table A2、G-D](https://arxiv.org/html/2610.07594v2#A2.SS3)，[observation源码](https://github.com/swirl-uk/BiGym2/blob/b03e66c8297b5e86b08e946f48b68895a6399d32/bigym/loco/agent/envtools.py#L258)）。仓库额外privileged tier可给部分任务物体状态，不能混入主实验的images tier。Q：人形验证与失败恢复边界？A：仅MuJoCo动态G1，20任务资源，主要量化比较为9任务；无真机验证。开发agent可读执行结果再改代码，但冻结程序不会调用LLM在线反思；跌倒只记录，通常继续到预算耗尽，因此不是提供通用倒地恢复。相对于本次需求，它最有价值的是材料完整的仿真底座与统一评估接口，论文主实验评估的是程序开发能力；仓库额外online模式的完整执行与恢复能力尚未在本轮实际验证。

## isaac-g1-hierarchical（工程候选）— Hierarchical VLM-RL Pick-and-Place and Drawer Opening for the Unitree G1 Humanoid

**重点关注**：False

**开源情况**：[2026-10-16] sim复现：可以（简化抓取）；sim重新训练：未核实；直接部署：不可以，缺少部署代码。[代码](https://github.com/mturan33/isaac-g1-hierarchical)

**基本信息**：阅读日期2026-10-16；无arXiv论文，工程作者Mehmet Turan Yardımcı（[CITATION.cff](https://github.com/mturan33/isaac-g1-hierarchical/blob/b8e36bb741d33c281e3442779b27a6f3c4c1c871/CITATION.cff)）；共同一作／通讯／单位未核实；是否中稿未核实。最早可核实Git历史为[2026-02-24初始提交](https://github.com/mturan33/isaac-g1-hierarchical/commit/590b00c)，2026-04已有VLM仿真日志；**Git作者时间不证明当日已公开，首次对公众开放日期未独立核实，暂收工程候选，不计严格首发确认数**。所读为当前代码、README、训练脚本与日志；项目主页即官方仓库。

**核心问题**：将语言任务和模拟器状态转换成G1技能序列，以本地大模型、RL行走／伸臂和脚本技能搭建可修改的分层执行流程。

**系统拆解**：任务＋真值语义地图、可选相机→Qwen3-VL JSON计划→注册技能→行走／手臂策略和手指控制→Isaac Lab关节目标；后台周期性读新状态、技能结果并决定继续／改计划／结束。[规划器](https://github.com/mturan33/isaac-g1-hierarchical/blob/b8e36bb741d33c281e3442779b27a6f3c4c1c871/planning/vlm_planner.py)、[执行器](https://github.com/mturan33/isaac-g1-hierarchical/blob/b8e36bb741d33c281e3442779b27a6f3c4c1c871/planning/skill_executor.py)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Planning → SemanticMap | 模拟器物体／机器人状态 | 位姿、物体类别、表面和抽屉→结构化JSON；不是已解决的真实视觉建图 |
| Planning → OllamaVLMPlanner / ClosedLoopController | JSON校验、SimplePlanner兜底、后台重规划 | 指令＋状态＋可选图像／完成记录→约10秒一次continue/replan/done→技能计划 |
| SkillExecutor | pre_reach/walk_to/reach/grasp/lift/lower/place/pull等 | 技能参数→Pure Pursuit、PID、状态机与策略调用→成功／失败、原因；部分成功由距离／计时／绑定状态决定 |
| Low-level → locomotion / arm / finger | 两个RL checkpoint、规则手指控制 | 速度、末端目标、本体状态→loco 66→15／arm 39→7网络和手指目标→50 Hz关节执行 |
| Environment → interaction | magnetic attach、物体／抽屉状态写入 | 靠近目标→绑定物体至手并更新物体／抽屉位置→简化搬运结果；非完整接触力学抓握 |

**训练、复用与新增**：README首段的“三策略”不能代替源码核验；当前实际提供两份运动权重，手指并无独立学习权重。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Planning → SemanticMap | 无训练；读取sim真值 | 将场景对象转成模型可读符号 |
| Planning → OllamaVLMPlanner / ClosedLoopController | 直接调用Qwen3-VL:4b，未报告微调；相机非必需 | 计划解析、允许技能校验、规则fallback、历史反馈 |
| SkillExecutor | 几何／规则程序，无独立任务示范训练 | 将RL行走/伸臂与脚本抓放、倒退拉抽屉组合 |
| Low-level → locomotion / arm / finger | 配套ULC的PPO流程：Stage2 loco在Stage1上做抗扰微调；Stage2 arm冻结loco再学伸臂，在线Isaac Lab经验；两文件对应2026-04作者训练run。当前目录的train_loco.sh是另一stock基线，不能当作这两权重的训练入口 | 暴露独立速度／末端目标；手指用规则而非第三个已验证RL网络 |
| Environment → interaction | 模拟器资产、绑定规则，无学习 | 以简化交互支持高层流程开发；这些规则须在接触真实性实验中替换 |

**关键 Q&A（2026-10-16）**：Q：实际验证到哪里？A：[作者VLM日志](https://github.com/mturan33/isaac-g1-hierarchical/blob/b8e36bb741d33c281e3442779b27a6f3c4c1c871/results/vlm_runs/10.0of10_20260402_221515.json)确有8步完成、1/1站立的G1仿真，但其10/10是自定义流程分数；日志也写明距离物体0.219 m时已绑定，不能解释成精确抓握。当前README的批量成绩主要是规则模式，不能充作VLM泛化率。代码的抽屉fallback允许0.60–0.70 m距离绑定，pull会直接更新抽屉状态；因此更适合接口和规划调试，不能优先于接触验证充分的控制基座。失败结果可供后台重规划，但没有独立任务完成评估器或充分恢复统计；Linux路径作者也未测试。

## HANDOFF — Humanoid Agentic Task-Space Whole-Body Control via Distilled Complementary Teachers

**重点关注**：False

**开源情况**：[2026-10-16] sim复现：可以（WBC）；sim重新训练：缺少部分训练数据；直接部署：可以（G1 WBC）；缺少上层规划与技能调用代码。[代码](https://github.com/lzyang2000/HANDOFF)

**基本信息**：阅读日期 2026-10-16；arxiv上传：2026-06-04（阅读 v4，2026-09-29，方法、实验、参考与官方代码）；一作：Lizhi Yang，共同一作未明确标示；通讯：未明确标示；单位：Caltech、IHMC；是否中稿：未核实；[项目主页](https://lzyang2000.github.io/HANDOFF/)，[arxiv](https://arxiv.org/abs/2606.06493)。

**核心问题**：把语言／视觉任务分解接到紧凑的 10 维身体指令，再由一套蒸馏全身策略协调行走、双臂和跌倒恢复，避免要求上层生成完整关节轨迹。

**系统拆解**：输入语言与 RGB-D，输出 G1 29 关节目标。这里的技能选择发生在参数化动作工具层，部署时不切换三个教师；[§III、图5](https://arxiv.org/html/2606.06493v4#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Agentic planner → high-level reasoner | 正则匹配，LLM fallback | 指令 → 原子任务序列；不是每次分解都调用 LLM |
| Agentic planner → VLM／waypoint tracker／skill selector | RGB-D投影、路径跟踪、技能选择、腕部修正 | 图像与任务 → pelvis-frame 目标 → 基座速度、躯干高度、双腕位置组成 10-D command；近目标重新视觉查询 |
| HANDOFF → complementary teachers（训练路径） | WBC／locomotion／AMP recovery | 同一仿真状态与各自参考 → 按上下肢及恢复状态提供蒸馏监督 |
| HANDOFF → MoE student | context-conditioned action-sliced KL + soft MoE | 10-D command＋本体历史 → 单一 29-DoF 策略动作，50 Hz；不运行时硬切教师 |
| Hardware execution | 关节跟踪、夹爪、限幅／CoP过滤 | 关节目标 → 500 Hz 电机执行；离散夹爪命令在 10-D 位置接口之外 |

**训练、复用与新增**：对应[§III-A–G](https://arxiv.org/html/2606.06493v4#S3)，无任务特定微调只适用于将训练好的控制器接到后续任务。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Agentic planner → high-level reasoner | 复用 LLM；未报告型号、权重更新或任务微调 | 正则优先、LLM兜底的任务拆解 |
| Agentic planner → VLM／waypoint tracker／skill selector | 本地VLM及ChatGPT API，具体checkpoint未披露；投影、跟踪和修正是几何／规则计算 | 将语义目标落到固定10-D接口；近距离再查询 |
| HANDOFF → complementary teachers（训练路径） | 本文训练三个PPO教师；WBC用Bones/SEED重定向动作并CoP修正，locomotion用在线仿真奖励与手臂扰动，AMP用1个跌倒起身＋17个行走／转向片段；不是直接拿三个现成策略拼接 | 分别补足姿态协调、速度跟踪、倒地状态覆盖 |
| HANDOFF → MoE student | PPO＋教师KL蒸馏；4096并行环境，本体历史和随机10-D命令，域随机化；教师作为蒸馏监督；学生初始checkpoint及教师是否冻结未明确披露，不补作推断 | 按动作切片、速度和恢复状态混合监督；单一学生策略 |
| Hardware execution | 不训练；需G1 29-DoF、ZED-M、Jetson Thor、Dex1-1及ROS2／SDK适配 | 500 Hz跟踪与夹爪/腕部修正、保护边界 |

**关键 Q&A（2026-10-16）**：Q：人形验证及恢复有多充分？A：作者有 G1 自然语言多阶段搬运／下蹲／双手操作真机展示；定量 2,500 次五任务成绩属于 MuJoCo（每任务500次、注入实测视觉定位误差，抓取为12 cm夹爪范围内的运动学抓取），不能当真实接触成功率。另500个配对倒地仿真中完整方法89.6%恢复，成功恢复者100%续走；真机没有同规模重复试验。工具会视觉再查询，底层可起身续走，但未给出通用任务级异常恢复评测。[§IV-C/D、表III](https://arxiv.org/html/2606.06493v4#S4)。Q：能否直接复现整条agent链？A：目前不能；底层sim/部署材料可作为基座，完整上层需补实现，SEED训练材料也需另备，全部未实际跑通。

## EgoActor — EgoActor: Grounding Task Planning into Spatial-aware Egocentric Actions for Humanoid Robots via Visual-Language Models

**重点关注**：False

**开源情况**：[2026-10-16] sim复现：不可以；sim重新训练：不可以；直接部署：不可以。缺少技能模型、训练数据、仿真与部署代码。[代码](https://github.com/BAAI-Agents/RoboNoid/tree/main/papers/EgoActor)

**基本信息**：阅读日期 2026-10-16；arxiv上传：2026-02-04（v1，同日；正文、附录 X–XI 与实验表，另查代码／权重）。一作：Yu Bai，未标共同一作；通讯：Xinlong Wang、Börje F. Karlsson；单位：北京智源人工智能研究院；是否中稿：未核实；[项目主页](https://baai-agents.github.io/RoboNoid/EgoActor/)，[arxiv](https://arxiv.org/abs/2602.04515)。HF 8B 仓库创建于2025-10-22，不能据此认定公开日；可核实的论文首发在窗口内。

**核心问题**：把任务与第一视角历史观测转换为可路由的语言动作，协调人形的移动、主动观察与操作技能。

**系统拆解**：[原文 III／IV-D](https://arxiv.org/html/2602.04515v1#S3)中输入为任务、RGB历史和动作历史；输出先经过显式语言动作接口再交下游控制器，故符合分层技能调用。真机是 G1＋Dex3-1＋自制两轴头＋D455，验证接近取放、找人交互及窄通道；下蹲／起立仅在 Habitat-Sim 验证，不能说已实现真机可变高度全身操作；讲话／询问成功只检查输出文本。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| EgoActor | Qwen3-VL；Structured Language Actions（SLA）与 Natural Language Actions（NLA） | 指令＋10张历史图＋最近3组观测动作 → 预测移动距离／转角／视角或操作指令。 |
| Action Parsing | SLA参数解析、NLA关键词路由 | SLA→速度／角度；问候→预设动作；操作NLA→GROOT；公开样例只打印解析结果，不下发电机。 |
| Downstream Skills | Unitree walking policy、GROOT-N1.5、预设交互动作 | 行走指令→厂商行走控制；操作文本＋视觉／状态→上肢动作；执行后新图像返回高层。 |

**训练、复用与新增**：与上表对应；[IV-A–C／XI](https://arxiv.org/html/2602.04515v1#S4)给出训练来源，[公开示例](https://github.com/BAAI-Agents/RoboNoid/blob/main/papers/EgoActor/sample_inference.py)可核实模型服务和解析接口。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| EgoActor | 基于Qwen3-VL 4B/8B，对所有线性层施加LoRA，混合数据训练1 epoch；来源包括EgoTaskQA、398条本地视频、Habitat轨迹、VLN-CE、MindCube、GQA和规划问答；不是从头训练VLM。 | 统一SLA/NLA与多帧动作历史监督。 |
| Action Parsing | 规则，无训练；公开`utils.py`实现动作解析。 | 将语义输出落到已有技能接口。 |
| Downstream Skills | 行走复用Unitree官方策略并人工标定；GROOT-N1.5全参数微调40k步、约700条桌面操作样本；预设动作无学习。 | 语言操控与移动对齐，新增两轴头用于主动观察。 |

**关键 Q&A（2026-10-16）**：Q：闭环及失败恢复到哪一级？A：历史图像／动作支持重新观察与调整，论文展示避障和主动后退观察；未给独立技能成功判定器、系统化抓取重试或跌倒恢复机制，不能等同HomeBody式恢复编排。论文表III有真机移动取放验证，但公开材料目前只能确认高层模型推理，不是机器人全栈材料齐全。

## HoloAgent-0 — A Unified Embodied Agent Framework with 3D Spatial Memory

**重点关注**：False

**开源情况**：[2026-10-16] sim复现：不可以；sim重新训练：不可以；直接部署：不可以。缺少技能模型、集成代码及部署配置。[代码](https://github.com/HorizonRobotics/HoloAgent)

**基本信息**：阅读日期 2026-10-16；arxiv上传：2026-06-22（v1，方法与实验全文、官方代码）；一作（共同）：Xiaolin Zhou、Liu Liu、Tingyang Xiao、Wei Feng；通讯：Liu Liu；单位：Horizon Robotics、D-Robotics；是否中稿：未核实；[项目主页](https://horizonrobotics.github.io/robot_lab/holoagent/)，[arxiv](https://arxiv.org/abs/2606.23565)。

**核心问题**：把空间记忆、导航、VLA操作和人形运动包成有类型、进度与失败反馈的技能，使LLM规划能在真实机器人上持续修正。

**系统拆解**：语言/语音＋传感器 → AgentOS检索记忆并生成skill graph → ROS2技能调用 → 各本体执行器；状态流返回监控、记忆与重规划。[§2–4、图1](https://arxiv.org/html/2606.23565v1#S2)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Embodied AgentOS | 规划、调度、监控/验证 | 请求＋空间/历史证据 → 带先决条件、依赖和恢复关系的技能图 → 调用、重试、澄清或重规划 |
| Skill Layer → interaction/perception | listen/speak、detect/localize/verify | 语音/视觉 → 交互与目标证据 → 置信度、事件、记忆更新 |
| Skill Layer → HoloNavi | HMSG快匹配、VLM慢验证、frontier exploration | move_to/find → 候选视点、旋转复查或探索 → 已验证导航目标与控制后端 |
| Skill Layer → HoloBrain | VLA manipulation backend | pick/place/open/handover/push/fold＋视觉 → 手臂/夹爪动作；失败与低置信度回传 |
| Skill Layer → HoloMotion | motion-tracking／velocity-tracking | 命名参考动作或速度目标 → 全身关节控制；反馈平衡、接触与恢复状态 |
| Memory Layer | 几何/语义/HMSG＋时序轨迹 | LiDAR/IMU/视觉或多相机、SAM2/SigLIP → 持续3D实例、floor-room-view-object层级及执行记忆 |

**训练、复用与新增**：本文重点是系统集成，没有报告训练一个新的总策略；不能把被引用基础模型的全部预训练当作本文新增训练。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Embodied AgentOS | 复用LLM；正文未给统一型号/微调记录；技能调用与ROS2反馈是推理时数据 | 有状态技能图、资源调度、结果验证与恢复依赖 |
| Skill Layer → interaction/perception | 复用ASR/TTS及视觉组件；本文未给独立训练 | 类型化接口及可监控状态 |
| Skill Layer → HoloNavi | 复用FSR-VLN的层级检索/慢验证；语言和当前视图为推理输入 | 将快慢导航置入AgentOS、失败复看与主动探索 |
| Skill Layer → HoloBrain | 复用引用的HoloBrain VLA；本文没有额外操作数据训练方案 | 将局部VLA暴露为可中断、有反馈的操作技能 |
| Skill Layer → HoloMotion | 复用引用的HoloMotion全身模型；未核实本文另训/微调 | 参考跟踪、速度与恢复接口接入AgentOS |
| Memory Layer | 复用FAST-LIVO类几何后端或GeoFlow-SLAM++、SAM2/SigLIP；在线建图/关联不是权重训练 | 持久实例、HMSG和时序记忆随新观测/技能结果更新 |

**关键 Q&A（2026-10-16）**：Q：哪些成绩真正属于人形？A：Unitree G1和R1有导航、交互、全身动作与跨机器人协调真机展示；长程移动操作用轮式双臂平台，不能说G1完成折衣。HM3D-ObjNav的82.6% SR／42.8% SPL抽象了底层执行，主要验证导航推理；真实公寓导航97.70% Top-1／98.90% Top-5是候选目标到达指标，正文未逐本体拆分，不能当整套人形操作成功率。全栈证据主要是定性轨迹。[§5、图7、表1–2](https://arxiv.org/html/2606.23565v1#S5)。失败可触发换候选、旋转复查、探索、重定位或重规划；文中定义完整，但尚缺统一全栈重复统计。

## HuGo — LLMs as Whole-Body Policy Code Designers for Humanoid Loco-Manipulation（扩展匹配：离线策略代码生成）

**重点关注**：False

**开源情况**：[2026-10-16] sim复现：不可以；sim重新训练：不可以；直接部署：不可以。缺少WBC模型、仿真环境、训练与部署代码。[代码](https://github.com/labicon/HuGo)

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-09-24（所读v1；已读全文方法、仿真、真机与限制，核查仓库实际文件）；一作Seoyeon Choi，共同一作、通讯未披露；单位UC Berkeley、ShanghaiTech、Google DeepMind、Nomagic，Markus Wulfmeier部分工作在DeepMind完成；是否中稿未核实，主页标Preprint；[项目主页](https://iconlab.negarmehr.com/HuGo/)，[arXiv](https://arxiv.org/abs/2609.30594)。

**核心问题**：让LLM直接编写并迭代高层闭环控制程序，在冻结WBC之上适配新任务，免去每任务重新设计奖励／收集示范。**它不在线调用预设任务技能库**；LLM在开发阶段读任务和rollout观测，执行时是生成代码调用低层控制器，因此按用户允许“控制器”作为技能的宽口径单列。

**系统拆解**：任务、机器人及观测／动作接口说明→LLM产出`policy_code(observation, policy_state)`→程序逐步输出速度、两手目标与身体高度→冻结低层policy→关节目标；rollout视频和数值轨迹交给LLM诊断并局部改代码。依据[§III–IV](https://arxiv.org/html/2609.30594v1#S3)、[公开程序示例](https://github.com/labicon/HuGo/blob/7a2a6e4952d428bfebc7f11a1f100991adf495a7/docs/static/simulation/push_button_2/policy_code.txt)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| High-level policy generation → Zero-shot Policy Generation | LLM、固定任务/接口prompt | 任务、尺寸、坐标系、观测和命令约束→生成带内部状态的高层代码；无任务技能库、示范程序或参考轨迹输入。 |
| High-level policy generation → Rollout Evaluation and Feedback | analysis function；数值摘要＋关键帧 | 稀疏轨迹与当前代码→LLM生成分析函数，处理完整轨迹→≤200词摘要、任务关键帧＋均匀补帧→LLM失败诊断。 |
| High-level policy generation → Iterative Policy Code Refinement / Best-of-N | 局部diff、depth/round、候选比较 | 诊断→局部改代码→重跑；仅成功率提高才接受，保留当前深度失败尝试；多次独立生成选优。 |
| Closed-loop high-level policy | 生成的状态机／几何控制程序 | 在线数值观测＋内部状态→10维速度／朝向、双手位置、身体高度目标；执行时不逐步查询LLM。 |
| Frozen low-level policy | rl_isaac 或SONIC 3-Point VR | 任务目标＋本体状态→跟踪全身命令→关节位置；各后端命令坐标系与高度能力不同。 |

**训练、复用与新增**：training-free仅适用于高层代码生成／修改，不适用于已训练的WBC；论文没有把每任务成功率比较当作RL梯度更新。依据[§V-A、VI](https://arxiv.org/html/2609.30594v1#S5.SS1)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| High-level policy generation → Zero-shot Policy Generation | 复用gpt-5.4，未报告微调；任务与接口文档为上下文。 | 无示范任务逻辑的高层程序生成。 |
| High-level policy generation → Rollout Evaluation and Feedback | 同一现成LLM处理本文仿真／真机rollout，不更新参数；每rollout最多10张图，失败样本优先。 | 由模型自行生成数值分析函数、选择事件帧，结合视觉发现接触／平衡问题。 |
| High-level policy generation → Iterative Policy Code Refinement / Best-of-N | 每候选100个固定仿真初态用于开发，最终另用100个held-out；每任务独立生成30个程序、对2000个随机10子集报告best-of-10；最多7 depths、每depth3 rounds。 | 代码搜索与经验比较，不是策略网络训练；失败修改在当前depth内保留。 |
| Closed-loop high-level policy | 无梯度训练，参数及阶段逻辑来自生成／改写；在线任务物体位姿被假定可测。 | 用状态反馈调整接近、对齐、接触与保持阶段。 |
| Frozen low-level policy | 本文训练`rl_isaac`通用命令跟踪器，奖励参考已有工作，初始化／完整训练材料未披露；另直接复用SONIC 3-Point VR、无修改；它既有大规模人体动作训练不能误算为HuGo任务示范。 | 对不同低层接口生成适配代码；生成后两种WBC均冻结。 |

**关键 Q&A（2026-10-16）**：Q：验证范围与恢复是什么？A：IsaacSim G1有低门、窄门、按钮、推箱、举箱五任务（SONIC不能边走边改高度，未测低门），另对HDMI两任务比较；真机仅按钮／举箱，SONIC后端、外部动捕观测，各任务3个程序各15次。最好程序分别93.3%／66.7%，但其他程序会明显退化，不能只报最佳值。举箱真机还用了文中额外几何hint。最差按钮程序从20%经过真机代码修订到90%，新候选各10次，这属于**跨rollout代码改进**，不是在线自主恢复成功率。生成代码能做阶段闭环，但论文指出LLM未可靠定位举箱碰撞成因、低层保持接触力不足；不能将“无任务示范”解读成无人工任务描述、无感知前提或无需WBC训练。

## Cybo-Waiter — A Physical Agentic Framework for Humanoid Whole-Body Locomotion–Manipulation

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-03-11（所读v1，方法§III、实验§IV、JSON示例与引用）；一作（共同）：Peng Ren、Haoyang Ge；通讯：Pei Chi、Kai Chen；单位：北京航空航天大学、中关村学院、天津大学、中国科学技术大学、中关村人工智能研究院、DeepCybo；是否中稿：未核实；未发现独立项目主页，[arxiv](https://arxiv.org/abs/2603.10675)。

**核心问题**：把VLM计划编译为带前后置条件的技能程序，以多对象3D几何验证确定失败原因，再做局部恢复与重规划。

**系统拆解**：语言＋RGB-D／本体状态→typed JSON子任务→对象3D grounding与谓词监控→RL步态或上肢MPC→G1关节执行；监控器返回条件级诊断。依据[§III／图1](https://arxiv.org/html/2603.10675v1#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Structured VLM Task Decomposition | typed JSON、preconditions/success_conditions/failure_handlers | 指令与观察→动作原语、对象、执行参数、连续满足帧数、超时及重试预算 |
| Conditional Segmentation and Geometric Coordinate Estimation | SAM3、RGB-D反投影、PCA／时间滤波 | 目标／目的地／关系对象→mask、质心、范围、置信度→工作空间对象关系 |
| Supervisor | 几何谓词、stable_frames、可选VLM语义复核 | 多对象状态→in_progress/done/blocked/failed＋失败条件／误差；VLM与几何冲突时先重观测 |
| Feedback-Driven Replanning | 四类恢复操作、预算管理 | 诊断→重观察、重grounding、改技能参数或修改剩余任务；保留已完成子任务 |
| Integrated Execution → locomotion | gait-conditioned RL skill | gait_id＋速度／转向→单策略12动作片段映射7种移动原语→下身命令 |
| Integrated Execution → manipulation / coordinator | 上肢MPC、独立下身稳定、轻量切换 | 目标几何→约束末端轨迹→手臂；不可达时先移动，误差／条件违规时暂停操作 |

**训练、复用与新增**：training-light只描述系统集成，底层确有作者的RL训练；[§III-E](https://arxiv.org/html/2603.10675v1#S3.SS5)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Structured VLM Task Decomposition | 复用VLM，具体型号／checkpoint／冻结范围未披露；未报告本文微调 | 显式JSON技能契约 |
| Conditional Segmentation and Geometric Coordinate Estimation | 复用SAM3；未报告微调；相机数据是推理输入 | 同时grounding目标及所有谓词引用对象 |
| Supervisor | 几何／时序规则；可选VLM未报告另训 | 几何优先，连续帧稳定与条件级诊断 |
| Feedback-Driven Replanning | 规则恢复＋VLM重规划；非策略梯度训练 | 先局部纠正，再在预算耗尽后改剩余计划 |
| Integrated Execution → locomotion | 作者在仿真训练单一gait-conditioned RL策略；12段行走／侧移／跑／转向片段＋域随机化。原文未充分披露优化器、初始化、片段来源规模及权重 | 把12 clips映射成7种agent可选移动原语 |
| Integrated Execution → manipulation / coordinator | MPC、几何与模式切换不属于训练；未披露完整下身稳定／手臂动力学实现 | 在满足几何与稳定条件时切换移动／操作 |

**关键 Q&A（2026-10-16）**：Q：人形验证与恢复证据？A：G1＋Dex3-1＋头部RGB-D，桌外RTX5080处理感知／规划，机载控运动。五项Being-0对齐任务各10次，本文成功9/8/10/9/8次；另整理桌面、分类、取饮料递人各10次，完整系统7/8/9次，去Supervisor为5/6/7次。[§IV、表I–II](https://arxiv.org/html/2603.10675v1#S4)。这是小样本室内真机验证，表I未清楚交代Being-0数字是否同机重跑；无恢复模块独立大规模成功率。原文未给VLM型号和完整控制细节，适合借鉴验证接口，现阶段不能直接整套复现。

## C2C — Cognition to Control: Multi-Agent Learning for Human-Humanoid Collaborative Transport

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-03-04（v1）；主要阅读v2（2026-09-17）全文／PDF首页；注意arXiv摘要页仍保留旧题名，v2 PDF题名变成*Physical Contracts for Plug-Compatible Human–Humanoid Collaboration*；一作／共一：Hao Zhang、Yisen Li（PDF星号）；通讯：Ding Zhao、H. Eric Tseng、Hao E. Zhang（首页脚注）；单位：University of Texas at Arlington、Carnegie Mellon University；是否中稿：未确认录用，作者主页列ICRA 2027 Submitted；[arxiv与版本历史](https://arxiv.org/abs/2603.03768)、[v2 PDF](https://arxiv.org/pdf/2603.03768v2)。

**核心问题**：把共享载荷路径、伙伴状态、协调动作和动力学执行分成接口稳定的层级，让仿真伙伴可由真人替换，且更换MARL算法时不必改下游执行器。属于“VLM调用／参数化可执行搬运控制器”的窄匹配：未展示多个异构语义技能的选择序列，不能与HomeBody通用技能调度同等描述。

**系统拆解**：[v2 §III-A–C](https://arxiv.org/html/2603.03768v2#S3)：任务目标＋结构化鸟瞰观测→VLM锚点路径→几何验证→MARL任务空间残差→WBC→关节力矩。视觉输入是由状态构建的64×64任务图（载荷、目标、障碍、伙伴、编号锚点），不是原始RGB自主感知；真人伙伴状态依赖MoCap。仿真为Isaac Lab九场景（定向推物、狭窄搬运、长物搬运各三场景），每种配置5种子×100次；[§IV-C／表V](https://arxiv.org/html/2603.03768v2#S4.SS3)G1＋真人三类任务各5次，PPO+PCGrad成功率100%／100%／80%；另有两台G1＋一人转向与穿越隔断演示，无该三载体真机统计。不是户外、任意新物体或无外部测量验证。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Cognition：VLM＋验证器 | Qwen2.5-VL-7B-Instruct；两视角提案＋碰撞／净空验证 | 目标／鸟瞰状态图→仅选图上编号锚点→共同载荷坐标中的可行路径；0.5Hz＋事件触发。 |
| Coordination：伙伴状态适配＋MARL技能 | CTDE；210维观测；MLP或hard-tree actor | 路径、自己／伙伴／载荷／接触／距离观测→有界11维残差→基座速度、CoM高度、躯干俯仰、双腕目标；2Hz。 |
| Execution：Whole-body control | 逆动力学QP、关节反馈、饱和 | 11维任务指令→接触／摩擦／关节／力矩／净空约束优化→关节力矩；100Hz。 |

**训练、复用与新增**：对应[v2 §III／表III](https://arxiv.org/html/2603.03768v2#S3)；以下均为论文报告，代码缺失，不能当作实现检查。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Cognition：VLM＋验证器 | 直接复用冻结Qwen2.5-VL-7B-Instruct，无任务微调；锚点采样与几何检查是算法。 | 将两个视角提案约束为同一物体路径，剥离不合法语言输出。 |
| Coordination：伙伴状态适配＋MARL技能 | Isaac Lab两可学习载体、集中critic／分散actor，20亿action steps；比较HAPPO、HATRPO、PPO+PCGrad；MLP为256/256/128，奖励为沿路径进展、倾斜与掉落；部署冻结actor，无在线学习。 | 同一物理状态接口可替换仿真伙伴或MoCap真人；硬树和三载体扩展保持动作接口。 |
| Execution：Whole-body control | 解析QP与反馈控制，无神经网络训练；文中给出约束和停车规则，但求解器配置／机器人模型资产未公开核实。 | 将学习限制于有界任务空间残差，动力学可行性由WBC承担。 |

**关键 Q&A（2026-10-16）**：Q：失败恢复是否明确？A：路径／ID非法先重询一次，再走融合占据图的确定性最短路；仍无路则停止。锚点到达、路径失效、目标改变或偏离超过0.35m触发认知重规划；QP不可行或净空低于0.15m时停基座、保持最后可行姿态。失抓／掉落在实验中直接终止，没有重新抓取恢复。它证明了受控搬运任务的VLM→技能→执行器闭环；不能推导任意语言任务库，也没有材料齐全或本次跑通的证据。

## HumanoidVLN — A Physics-Grounded Simulator and Benchmark for Vision-Language Navigation Across Diverse Humanoid Embodiments

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期 2026-10-16；arxiv上传：2026-08-13（v1，方法、实验和官网）；一作：Quan-Dung Pham；共同贡献标注对应作者尚未完全核实；通讯：未明确；单位：VinMotion、University of Southern California；是否中稿：未核实（官网写Humanoids 2026 under review）；[项目主页](https://humanoid-vln.github.io/)，[arxiv](https://arxiv.org/abs/2608.12860)。

**核心问题**：让视觉语言导航动作通过真实双足动力学执行，并比较不同人形形态、导航模型与路径控制器的组合，而非把机器人沿路线直接传送。

**系统拆解**：这里是导航专用技能链，范围比HomeBody窄，但满足“VLM理解指令/观测→可执行控制器”；不是通用操作技能库。[§III、VI-A](https://arxiv.org/html/2608.12860v1#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| VLN models | NaVILA、StreamVLN、JanusVLN或DualVLN | 指令＋第一视角 → 前进/左右转/停止（前三者）或连续速度（DualVLN） |
| Hierarchical control → PD/MPC tracker | 离散动作配PD，连续输出配MPC | 导航动作/路径 → 速度与朝向参考 |
| Hierarchical control → RL locomotion | 每本体的低层策略 | 参考＋本体状态 → 双足关节力矩；物理运动产生新观测 |
| Dataset construction（离线） | 轨迹生成、Generator/Reviewer/Paraphraser＋人工核验 | 可通行场景与执行视频 → 933条参考episode，每条细粒度指令＋3种粗粒度表达 |

**训练、复用与新增**：对应[§III–VI](https://arxiv.org/html/2608.12860v1#S3)，离线指令生成agent不等同于在线机器人planner。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| VLN models | 四模型公开checkpoint零样本评估；没有在HumanoidVLN上训练/微调 | 统一观测/动作适配和跨本体比较 |
| Hierarchical control → PD/MPC tracker | 经典控制；不是训练模块 | 按离散/连续动作空间匹配控制接口 |
| Hierarchical control → RL locomotion | 各本体RL策略，本文未充分披露其具体初始化、训练集/奖励、权重来源与重训范围 | 将导航落到各形态的接触动力学、关节约束 |
| Dataset construction（离线） | 使用GRScenes/SAGE场景及3DGS重建；多agent生成后人工纠正，仅作评测指令，非本次VLN训练集 | 可通行性筛选、执行相机扰动与四种指令风格 |

**关键 Q&A（2026-10-16）**：Q：验证边界与失败恢复？A：Isaac Sim评估G1、H1及两个内部人形，四模型×933集；另DualVLN/G1在茶水间和studio两场景共20集实机配对，验证导航误差相关性（r=0.935、平均绝对差0.68 m），没有验证操作或广泛跨场景迁移。上层根据新图像继续导航，但一旦达到跌倒判据便终止，无自动起身恢复；论文报告的是episode级Fall Rate，官网“step fraction”表述不精确，按原文采用。[§VI-A/D](https://arxiv.org/html/2608.12860v1#S6)。

## KINO — A Keyframe Interface for VLM Planning and Whole-Body Control in Humanoid Loco-Manipulation

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-09-16（所读v1，唯一公开版；已读全文方法、实验、失败恢复与限制）；一作Sitong Chen，共同一作、通讯未披露；单位ETH Zürich；是否中稿未核实；未发现独立项目主页，[arXiv](https://arxiv.org/abs/2609.18869)。

**核心问题**：以带语义标签的全身关键帧作为VLM和RL控制的接口，使语言任务能转成可执行的双手搬箱／单手提桶及移动目标。

**系统拆解**：指令＋第三视角图像＋执行状态→VLM选预定义关键帧→按场景物体位姿／尺寸重定向、必要时插入行走关键帧→关键帧条件策略输出G1的29维关节位置→PD执行。它调用的是共享策略的语义目标，不是每个skill一个独立网络。依据[§III、图2/5](https://arxiv.org/html/2609.18869v1#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| VLM-Based Planner | Planner context；Constrained keyframe output；event-driven loop | 任务、图像、前一目标成败、根／物体位姿及目标误差→选择keyframe、object_to_manipulate、task_complete三个JSON字段→新目标或结束；机器人／物体静止且留足执行时间后才重问VLM。 |
| Robot Keyframes → Keyframe Library | 全身／物体姿态＋人工语义标签 | 参考动作代表帧→手工组织与标注→approach、crouch、lift、carry等可选目标。 |
| Scene Alignment → Keyframe Retargeting | 物体归一化坐标变换；damped least-squares IK | 模板姿态＋当前物体位姿、尺寸→缩放／旋转手和根位置并解IK，约束脚高度→场景适配关键帧。 |
| Scene Alignment → Path Planning | tangent-aligned circular arc／Dubins path | 远端行走目标→限曲率路径，按转角≤45°、位移≤2m选择中间目标→行走关键帧序列；不是通用障碍物避碰规划的证据。 |
| Keyframe-Conditioned Policy | PPO、非对称actor-critic、三组critic、PD | 最近10帧本体状态、物体状态、前动作、目标帧→共享全身策略→29关节位置目标→G1 PD控制。 |

**训练、复用与新增**：高层无需本文机器人数据微调，不代表低层无需训练。依据[§III-A/B、§IV](https://arxiv.org/html/2609.18869v1#S3.SS1)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| VLM-Based Planner | 复用Qwen3.6:27B，在单RTX4090运行；未报告微调或冻结参数细节。图像／执行历史为推理输入。 | 约束式关键帧输出与执行反馈驱动的事件循环。 |
| Robot Keyframes → Keyframe Library | 52条AMASS行走、105条OmniRetarget双手搬箱、120条内部采集单手提桶片段；训练采样按身体／物体加速度显著性筛选，滤去周期性腿部运动；在线库仍人工构建标注。 | 让训练目标分布覆盖抓／放等语义阶段，减小VLM稀疏目标与训练目标的失配。 |
| Scene Alignment → Keyframe Retargeting | 几何变换及数值IK，无梯度训练；在线需物体姿态和尺寸。 | 保留机器人—物体相对关系并适配尺度。 |
| Scene Alignment → Path Planning | 几何路径生成，无训练数据。 | 将超出参考数据范围的目的地转成控制器可跟随的中间目标。 |
| Keyframe-Conditioned Policy | 本文训练PPO；是否随机初始化未披露。参考动作跟踪、到时关键帧目标奖励及正则分组critic；物体初始位姿增广、动力学／噪声／延迟／推力随机化。critic使用参考状态等特权信息；不能把部署策略称作直接复用现成tracker。 | 用显著性关键帧训练目标条件全身策略。 |

**关键 Q&A（2026-10-16）**：Q：人形验证和失败恢复有多充分？A：[§IV](https://arxiv.org/html/2609.18869v1#S4)有MuJoCo G1的搬箱／提桶闭环和真机两类展示；真机物体位姿来自外部动捕，上层用第三视角相机，不是纯机载自主。44%→92%为100次、改变起始/目标距离的采样消融结果，非已报告的真机成功率；成功阈值为物体与根位置误差<0.3m。VLM利用图像及goal error识别失败、重发前一关键帧，图9给定性恢复示例；未给系统化恢复成功率。Approach失败后局部评估会重置到物体附近，后续统计也有跳过阶段，不能把阶段表每列当同一100次完整无重置试验。Q：复现价值？A：很适合研究语义技能目标与可执行控制器之间的接口，但目前缺整套实现、权重与内部桶动作数据，不是立即可跑候选。

## InterEvolve — Test-Time Evolution of Reward Programs for Humanoid Loco-Manipulation（扩展：离线搜索奖励程序）

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-10-01（所读v2，2026-10-04；主文、全部附录、特别是训练A、程序搜索C和真机D.8）；共同一作Zhuo Lin、Sirui Xu；通讯未披露（Yu-Xiong Wang、Liang-Yan Gui标记共同指导，不等于已核实通讯）；单位University of Illinois Urbana-Champaign；是否中稿未核实；[项目主页](https://sirui-xu.github.io/InterEvolve/)，[arXiv](https://arxiv.org/abs/2610.02196)。

**核心问题**：在不为新任务重训控制器的情况下，让LLM依据任务、场景和执行反馈，迭代分阶段奖励程序，再通过冻结FB行为模型调用已有全身能力。它的高层是在模拟里离线搜索可执行策略，真机执行时不再调用LLM，故不混入HomeBody式在线闭环主榜。

**系统拆解**：任务＋结构化场景→固定验证准则→LLM改奖励程序、CMA-ES调常数→阶段奖励在冻结状态库上推断latent→object-aware FB策略→29关节目标；模拟rollout返回阶段/准则反馈继续改程序。真机使用仿真预先生成的latent序列，控制器仍根据当前本体/物体估计闭环。依据[原文§3、附录A–D.8](https://arxiv.org/pdf/2610.02196v2)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Task specification → Verifier | LLM-assisted parser、固定外部准则C | 任务文本＋场景物体/障碍条件→一次性生成任务/物理约束→对完整rollout打分；搜索者不能改评分标准。 |
| Evolving reward programs at test time | DeepSeek-V4-Flash outer loop、CMA-ES inner loop、validator、skill library | 任务/场景＋当前最优程序＋各阶段/准则执行反馈＋已验证程序库→LLM改阶段/奖励项，CMA-ES调有界常数→多场景复核后接受改进。 |
| Reward programs as editable task strategies | 阶段奖励r、完成条件g、常数θ、共享运行上下文 | 当前阶段/接触/物体状态/时间→判断阶段进度、更新目标与奖励→当前阶段奖励；可用等待/超时/回退逻辑表达恢复。 |
| Executing reward programs without retraining | object-aware FB motor model、fixed reward-inference bank | 当前奖励在固定bank计算加权backward embedding→latent z；冻结actor读z＋本体/物体特征→29关节位置；模拟每控制步重新算z。Inspire手另用标量抓取命令映射手指。 |
| Transfer and physical deployment | FastSAM+CLIP、FoundationPose、LiDAR/接触/里程计融合、latent replay | RGB-D识别箱体并跟踪6D位姿，与其他传感融合→actor所需物体特征；仿真latent序列按步回放＋当前状态→50Hz关节目标；物体估计无效则暂停。 |

**训练、复用与新增**：无重训仅指**新任务程序搜索期间**；底层object-aware模型在本文确实训练过。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Task specification → Verifier | 解析不报告参数训练；场景、任务文本与固定评价约束是推理输入。 | 将奖励程序与独立验证器分离，避免搜索通过放松评价“进步”。 |
| Evolving reward programs at test time | DeepSeek-V4-Flash API权重固定、上下文学习；CMA-ES是逐任务数值搜索。程序库约28KB、8任务，存程序、条件和验证结果；不是训练集。 | 双层结构/常数搜索、候选代码检查、多初态确认；改进须超过重复评估噪声。 |
| Reward programs as editable task strategies | 不训练网络；程序结构由LLM改、常数由CMA-ES优化。 | 将长任务转成可解释阶段、奖励及完成条件，允许程序级修复和组合。 |
| Executing reward programs without retraining | 复用BFM-Zero的body-only actor/forward/backward分支并冻结，新增object residual分支以FB目标、示范判别器和辅助稳定性目标训练；OMOMO/GRAB经改造OmniRetarget映射到G1。正文报告四类OMOMO箱体每类留50clips、剩余3866训练clips。bank训练后从多任务/多物体replay采样并缓存，任务搜索时冻结。 | 给body prior加物体状态与身体到物体表面的几何特征；奖励到latent的即时推断接口使程序可调用底层能力。 |
| Transfer and physical deployment | 复用FastSAM、CLIP、FoundationPose，未报告本文微调；真机控制权重固定。 | 带时间戳位姿、多源滤波；仿真latent回放与反馈控制结合，不是真机在线reward搜索。 |

**关键 Q&A（2026-10-16）**：Q：人形验证、反馈与恢复究竟覆盖哪里？A：Isaac Lab G1（橡胶手为主定量、Inspire手为定性）验证8类任务、3类组合与参考跟踪；8任务宏平均成功率86.5%针对模拟中演化后的奖励程序，不是真机成功率。G1真机Fig.4仅展示踢箱、推箱，未见系统性硬件试验次数；附录D.8明确回放模拟产生的latent，当前状态进入冻结控制器。LLM的失败修复发生在模拟episode之间；原文限制明确搜索需要GPU小时，不能物理执行中实时重规划。项目页额外“连续踢箱”示范明确使用mocap，不能统称所有演示仅机载感知。另v2 Table14跨物体程序适配标注**by hand**，项目页“evolve into same skills”的展示不能替代自动适配证据。

## SafeHumanoid — VLM-RAG-driven Control of Upper Body Impedance for Humanoid Robot（边界扩展：控制参数调度）

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期2026-10-16；arxiv上传：2025-11-28（所读v1全文）；一作Yara Mahmoud，共同一作／通讯未标明；单位Skoltech；是否中稿：是，HRI 2026 Companion Proceedings，正式标题为SafeHumanoid: VLM-RAG-Driven Impedance Control of Humanoid Robot（[出版登记](https://api.crossref.org/works/10.1145/3776734.3794539)，2026-03-16）；无独立项目主页，[arxiv](https://arxiv.org/abs/2511.23300)。正式版仅核验出版元数据，方法结论以所读预印本为准。

**核心问题**：根据任务、物体脆弱性与人的出现，为现有上身动作选择刚度、阻尼和速度。它接近“VLM→控制器”接口，但主要是既有轨迹的语义调参，没有证实任务级技能启动／选择，因此不计入主匹配。

**系统拆解**：RGB＋固定任务prompt→Molmo结构化语义→相似场景检索→14上肢关节的28个增益与速度→IK／阻抗控制。**末端6DoF目标为预设**，不能解释成视觉自主抓取规划。[§3–5](https://arxiv.org/html/2511.23300v1#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Communication Architecture | 机载／外部TCP链路 | 1–2Hz图像→服务端推理→29值payload；机载50Hz使用最近有效值，断连用保守配置 |
| Perception Module (VLM) | Molmo-7B BnB 4-bit、固定JSON schema | 当前图像＋任务prompt→任务、物体、人员、工作区、置信度等语义 |
| Safety Parameter Generation (RAG) | all-MiniLM-L6-v2、FAISS exact NN、格式化 | 语义→384维embedding→16场景最邻近条目→Kp/Kd/速度；低置信度／近似并列回退 |
| Robot Control Module | 锁定非手臂关节的IK、RNEA、joint impedance | 预置末端目标＋检索增益→关节参考／重力前馈→G1上肢PD执行 |

**训练、复用与新增**：新增数据用于经验参数查表，而非训练新动作网络。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Communication Architecture | 网络协议与控制规则，无训练 | 慢语义环与本地快控制分开 |
| Perception Module (VLM) | 复用Molmo，未报告微调；固定prompt不是权重训练 | 用固定语义字段衔接检索 |
| Safety Parameter Generation (RAG) | 复用MiniLM／FAISS；G1先导实验记录图像、关节、力矩与增益，人工筛出16条、34列CSV，覆盖pick/handover/other；非LLM生成新参数 | 检索经人工验证参数，含保守fallback |
| Robot Control Module | IK/RNEA/PD计算，无神经训练；增益来自先导实测 | 将语义配置落到已有位置接口及重力补偿 |

**关键 Q&A（2026-10-16）**：Q：验证和恢复边界？A：G1实机上身擦拭、取物／递物、倒液等六类演示，腿腰在IK中锁定；成功定义主要是增益／速度按语义正确调节，并无完整长任务成功率或统计安全保证。最坏语义链延迟约1.4秒；提供检索／通信fallback，未提供任务失败后自主重试。论文引用安全标准，不能由此认定系统获得安全认证或已实测证明接触力上界。[§5–7、表1](https://arxiv.org/html/2511.23300v1#S5)。

## HumanoidVLM — Vision-Language-Guided Impedance Control for Contact-Rich Humanoid Manipulation（边界扩展：控制参数调度）

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-01-21（所读v1全文）；一作Yara Mahmoud，共同一作／通讯未标明；单位Skoltech；是否中稿：是，HRI 2026 Companion Proceedings（arXiv注明LBR；[出版登记](https://api.crossref.org/works/10.1145/3776734.3794528)，2026-03-16）；无独立项目主页，[arxiv](https://arxiv.org/abs/2601.14874)。正式版仅核验出版元数据。

**核心问题**：把视觉推断的任务与物体类别映射为任务空间阻抗和夹爪配置，是可解释的语义控制接口，并非自主分解多步任务的技能harness。

**系统拆解**：第一视角RGB→Molmo逐项yes/no任务推断→两阶段FAISS检索→Cartesian impedance与夹爪角→IK→G1位置控制器。参考轨迹来源、启动方式未披露，不能认定VLM自主产生目标；不计入主匹配。[§3、图1](https://arxiv.org/html/2601.14874v1#S3)。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| VLM–RAG System | Molmo、MiniLM、impedance/gripper databases | 图像→任务标签→先检索阻抗，再结合条目与标签检索夹爪→K/D/角度 |
| Robot Control → End-effector Impedance Control | 虚拟质量–弹簧–阻尼、IK | 目标参考＋检索平移增益→虚拟末端轨迹→关节目标；质量与旋转阻抗固定 |
| Robot Control → Gripper Configuration | 单DoF open/close与角度配置 | 对象/任务对应角→左右夹爪目标→内置位置控制执行 |

**训练、复用与新增**：与SafeHumanoid为相同研究线的另一篇论文，不是arXiv修订版；增益空间、数据结构与评估不同，分别记录。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| VLM–RAG System | 复用Molmo-7B-O BnB 4-bit、all-MiniLM-L6-v2，未报告微调；两个手工实测JSON库各覆盖9项任务／抓握条目。14测试图像用于检索评估，非模型训练集 | 从关节增益改为任务空间平移增益，并联合检索夹爪 |
| Robot Control → End-effector Impedance Control | 无神经训练；各任务实测多组K/D选择配置 | 无腕力传感器条件下的虚拟阻抗与IK执行 |
| Robot Control → Gripper Configuration | 角度经重复实机试验选择，非学习策略 | 对刚性／柔软／脆弱对象匹配单DoF夹爪角 |

**关键 Q&A（2026-10-16）**：Q：93%意味着什么？A：14张图里13张正确检索阻抗及夹爪配置，不是13/14次自主任务成功。G1＋双单DoF夹爪实机覆盖曲面跟随、按摩球施压、双手放瓶／蛋、叉刺、抓举五组桌面任务，主要报告z向误差和由模型计算的虚拟力；虚拟力不是传感器测得的真实接触力。局部位置反馈已闭环，物理力／触觉反馈、自主失败恢复仍未实现或未报告。[§4–5、表1](https://arxiv.org/html/2601.14874v1#S4)。

## HODAgent — Towards On-Demand, Responsive Humanoids for Physical World Human Interaction（已撤稿）

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期 2026-10-16；arxiv上传：2026-08-18（读取v1正文及附录；最新v2为2026-08-20撤稿通知）；一作：Wang Warren Chen；前五人Wang Warren Chen、Jiahao Zhang、Zhenjiang Li、Mingxu Wang、Lei Yi标“Core contributors”，不直接改称共同一作；通讯：Jie Chen；单位：正文未列完整机构，通讯邮箱为xiaopeng.com；是否中稿：未核实，当前已撤稿；项目主页未提供；[arxiv](https://arxiv.org/abs/2608.17584)，[可读v1](https://arxiv.org/html/2608.17584v1)。

**核心问题**：用半双工agent管理移动过程中插入的新请求、保留已完成任务进度，并用执行结果复核完成状态。

**系统拆解**：任务/语音＋当前观测→System-2四模块→System-1有限时域运动块→System-0 SONIC→电机；按[附录B](https://arxiv.org/html/2608.17584v1#A2)定义，SONIC内部运动生成器也属于System-0，不能把GR00T桥误写成新训练VLA。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| System-2 → Env-Interactor | Omni模型与事件路由 | 语音/视觉/任务状态 → 回答、忽略、启动或打断Planner |
| System-2 → Planner | decomposition–planning–review | 目标/进度/反馈 → 短工具序列 → 复核、续做或纠错 |
| System-2 → Executor | 异步dispatch、状态监控与streaming inference | 工具调用 → 平台adapter；适当时提前规划下一段，执行结果回传 |
| System-2 → hierarchical Memory | task/subtask/action三级记忆 | 请求、进度、观测和结果 → 保留已完成/未完成义务及后续上下文 |
| System-1 | motion-command planner | 距离/转角或命名动作 → 有限时域方向/朝向/速度块或参考动作及入场/退场 |
| System-0 | SONIC运动生成与反馈控制 | 运动块 → 全身参考 → 50 Hz控制器的29关节目标 → G1 500 Hz位置/阻尼命令 |

**训练、复用与新增**：本文不训练新底层策略或通用VLA，重点是harness；[§3、§5、附录B](https://arxiv.org/html/2608.17584v1#S5)。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| System-2 → Env-Interactor | 真机复用Qwen3-Omni；没有报告微调 | 连续接收输入与任务打断路由 |
| System-2 → Planner | 真机复用Qwen3.6-27B；仿真另比较35B-A3B；无本文权重训练报告 | 短程规划及基于证据的完成复核 |
| System-2 → Executor | 规则/调度，非学习 | 共享接口、动作生命周期和规划/运动重叠 |
| System-2 → hierarchical Memory | 执行日志与在线状态，不是离线训练数据 | 分层保留义务与进度 |
| System-1 | 参数化运动编译与配置参考；非RL训练 | 分块、时长/速度限幅、动作前后稳定过渡；仿真另用校准补偿 |
| System-0 | 复用SONIC/GR00T连接栈；本文未披露重训；命名动作由反馈跟踪 | 平台适配、控制模式互斥、停止/恢复与状态反馈 |

**关键 Q&A（2026-10-16）**：Q：人形验证和失败恢复能推出什么？A：v1报告Isaac Sim 164个服务/多目标导航case（22场景），以及G1真机50次原子、50次组合、30次完整任务，分别92%、72%、63.3%通过；完整任务是找穿指定颜色衣服的人击掌、接近挥手、绕障问候，不是抓取搬运验证。真机无语义地图，仅当前视图与机载传感器。它能中断、重规划、复核与重获稳定站立，但不保证硬实时停止：案例中停止意图推理期间又启动一次0.3 m前进。上述均为撤稿v1作者报告，不能作稳定性已被独立验证的结论。[§4–5、表3、图7](https://arxiv.org/html/2608.17584v1#S5)。

## FRAMES — Failure Recovery And Monitoring of Embodied Skills for Humanoid Loco-Manipulation（架构匹配，仅Monitor人形验证）

**重点关注**：False

**开源情况**：完全没开源[2026-10-16]。

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-09-18（所读v1；已读方法§II、实验§III、结论及技能接口表）；一作Ajay Vikram Periasami，共同一作未披露；通讯未明确指定；单位Duke University，Department of Computer Science及Department of Mechanical Engineering and Materials Science；是否中稿未核实；独立项目主页未找到；[arXiv](https://arxiv.org/abs/2609.22538)，[原文](https://arxiv.org/html/2609.22538v1)。

**核心问题**：为现有全身控制器增加“规划—技能执行—过程监控—失败恢复—经验复用”监督层，及时识别人形操作中的抓取、接触和语义目标错误。架构符合VLM/LLM调用技能，但本文仅独立验证Monitor，完整规划／恢复链仍待验证，单列为弱验证匹配。

**系统拆解**：设计链为任务＋观测／历史→Planner选择技能及参数、前后置条件→Memory辅助参数化→Skill Executor生成根／末端目标→冻结CEER输出G1 29关节命令；Monitor从技能开始与当前的图像／状态判断进度，失败则停止并交Recovery选择下一技能。`move_to/turn/approach_object/move_hands/grasp_object/release_object`等是中层可执行API；`perceive_object`提供几何观测。以下是论文**设计**，不能把各模块都写成已端到端验证（[§II-B–G、Fig.2/Table I](https://arxiv.org/html/2609.22538v1#S2)）。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Planner Agent | GPT-5.4-mini；skill selection＋focused parameterization | 原始任务、当前观测、执行历史、技能库→选下一子任务/API、初始参数及preconditions/success/failure contract；收到同技能记忆后只修改schema许可参数，技能、目标物和contract保持固定。 |
| Skill Executor → CEER Controller | 参数技能映射G、预训练全身策略 | 技能参数＋当前状态→约10Hz根和末端目标；CEER读目标＋本体状态→50Hz 29-DoF全身关节命令；完成／中止时返回status/message/measurements。 |
| Monitor Agent | GPT-5.4-mini；skill-local temporal context | 开始与当前机器人视角、俯视、侧视共六图＋物体位移、基座变化、距离与接触变化＋任务contract→约1Hz输出in_progress/success/failure及证据；失败中断技能，成功须在技能结束后复核。 |
| Recovery Agent | GPT-5.4-mini；共享技能库 | 失败动作、执行结果、Monitor解释、状态变化、当前观测和历史→重试调参、纠正接近／手部运动、释放不稳接触或补观察；恢复动作再次走同一执行／监控链。 |
| Memory Module | persistent semantic memory bank、同技能检索、结果一致性过滤 | 结束／中断任务的已完成物理子任务→保存参数、局部／整任务结果、原因和经验；仅检索局部与整任务结果一致记录，成功任务只保留各技能最后一次成功；辅助Planner/Recovery参数化。 |
| Language-Conditioned Geometric Grounding | perceive_object、SAM3、深度反投影与坐标变换 | 物体文本＋同步RGB-D→分割、重建可见表面→可见中心/尺度、深度、置信度及手物距离，供接近／抓取参数使用。 |

**训练、复用与新增**：本文新增监督层与接口组合，未报告新任务策略训练；不能将引用CEER的既有训练当作FRAMES重新训练。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Planner Agent | 直接调用GPT-5.4-mini；未报告微调或规划训练集，不能据此推断更具体训练设置。 | 一次决策选一个参数技能，并给执行contract；记忆只允许修改参数。 |
| Skill Executor → CEER Controller | 原文明示预训练CEER未经修改、低层策略固定；本文未重训。§III-A复述该控制器既有训练使用Isaac Lab、16384并行环境、4×L40s、8小时；具体初始化／损失／数据仍指向CEER原文，不在此补猜。 | 将导航、操作API映射到CEER统一根／末端目标接口，并返回结构化执行结果。 |
| Monitor Agent | 复用GPT-5.4-mini，未报告微调。100次模拟执行是**独立评估样本**，不是训练集：5类各10失败＋10匹配成功，微调机器人姿态／接近／操作参数。 | 保留技能开始基线，比较累计变化；把物理完成与语义成功分开，支持执行中止。 |
| Recovery Agent | 复用GPT-5.4-mini，未报告微调、恢复训练数据或恢复成功率。 | 将失败证据显式加入决策，复用同一API与监控机制。 |
| Memory Module | 文本记录及检索，不是参数训练；记忆规模、长期学习收益未量化。 | 以子任务／整任务结果一致性筛选经验；避免把最终失败中的暂时局部“成功”当正例。 |
| Language-Conditioned Geometric Grounding | 复用SAM3，未报告本文微调；RGB-D、标定和机器人位姿是推理观测，几何处理无学习训练。 | 提供planner-callable的语言指向物体几何接口，输出可见表面而非完整物体模型。 |

**关键 Q&A（2026-10-16）**：Q：人形与失败恢复验证到哪一步？A：仅MuJoCo 29-DoF Unitree G1，无真机验证。§III-B明确100次试验的技能序列由人遥操作，**Planner和Recovery未参与**；五类为抓取失败、掉物、接近碰撞、抓错物和放错位置。Monitor检测48/50失败、接受46/50成功，总准确率94%；这是分类准确率，不是长任务或自动恢复成功率。两次失败抓取被误判仍在执行，三次安全接近被误判碰撞，一次正确放置被误判失败，反映视觉接触判断边界；抓错物判断不额外提供物体名称／位姿字段，而依据指令与图像识别被举起的物体。Memory与SAM3接口写入架构，但本评估不能证明其端到端收益；结论明确完整规划—恢复闭环仍需评估（[§III-B/C、Table II、§IV](https://arxiv.org/html/2609.22538v1#S3)）。

## MetaWorld — MetaWorld: Skill Transfer and Composition in a Hierarchical World Model for Grounding High-Level Instructions（待核实，建议不进入已核验复现候选排名）

**重点关注**：False

**开源情况**：[2026-10-16] sim复现／sim重新训练／直接部署：均未核实（代码仓库无法访问）。

**基本信息**：阅读日期2026-10-16；arxiv上传：2026-01-24（v1，同日；正文完整方法和实验，PDF首页）；一作：Yutong Shen，共同一作未标；通讯：Tongtong Feng（PDF星号）；单位：北京工业大学、复旦大学、清华大学；是否中稿：未核实，arXiv自述投稿ICLR 2026 World Model Workshop不构成录用证明；独立项目主页未提供；[arxiv](https://arxiv.org/abs/2601.17507)。

**核心问题**：以VLM给出的专家组合先验辅助TD-MPC2控制与在线适应，缓解高层语义和人形控制间的映射问题。

**系统拆解**：[原文Fig.2／§3](https://arxiv.org/html/2601.17507v1#S3)写明VLM输入任务及环境语义，生成专家权重，状态选择器修正权重，以专家动作作为物理层参考。仅HumanoidBench仿真，表1报告Stand/Walk/Run/Door回报；无真机。原文正文对reach/run列表有不一致，不能自行补全实验任务。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Semantic planning layer | VLM＋输出解析 | 任务T／环境E→提示生成归一化专家权重；具体VLM与视觉输入接口未披露。 |
| Skill transfer layer | Dynamic expert selection、motion prior fusion | 当前状态→状态相关专家概率，与VLM权重融合→参考专家动作。 |
| Physical execution layer | TD-MPC2 latent dynamics、MPC | 状态／参考动作→潜空间预测与滚动优化→仿真人形动作。 |

**训练、复用与新增**：对应§3–4；代码不可读，以下为论文描述而非实现核验。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Semantic planning layer | Prompt使用VLM，未报告微调；具体模型／版本未知。 | 专家权重接口。 |
| Skill transfer layer | 作者称AMASS模仿／轨迹跟踪奖励训练运动专家；部分技能继承TD-MPC2，具体checkpoint未知。 | 语义先验与状态专家选择融合。 |
| Physical execution layer | 基于TD-MPC2的在线仿真学习，初始化／冻结细节未披露；MPC本身是优化。 | TD损失外加入参考动作约束。 |

**关键 Q&A（2026-10-16）**：Q：能视为类似Thea的已核验VLM技能执行栈吗？A：仅能确认论文声称的VLM→专家→物理控制层级与人形仿真结果。环境E的表示、运行时VLM重调用时机和失败恢复没有充分说明；§2.3闭环重规划写的是“during training”。因此保留为待核实线索，不给可复现推荐；不能把数学形式自动解释成在线VLM观察反馈循环。

## LG-HSS — Language-Guided Hybrid Skill Scheduling for embodied manipulation（待核实线索，暂不纳入已确认时间窗及复现排名）

**重点关注**：False

**开源情况**：[2026-10-16] sim复现／sim重新训练／直接部署：均未核实。

**基本信息**：阅读日期2026-10-16；arxiv上传：未发现；首次公开日期未核实：[Crossref记录](https://api.crossref.org/works/10.1016/j.robot.2026.105736)仅有2026-09-09 DOI记录创建时间，`published-online`缺失，不能拿DOI创建日当论文首次公开日；期刊卷206标2026-12。第一作者Xiaxi Si，共一Yuao Guo（出版社二人均标1，脚注说明equal contribution）；通讯及单位未核实；出版社可确认发表于*Robotics and Autonomous Systems*、article105736；[出版社原始页面](https://www.sciencedirect.com/science/article/pii/S0921889026004070)、[DOI](https://doi.org/10.1016/j.robot.2026.105736)。Elsevier公开API返回书目信息，`openaccess=0`，未返回正文。

**核心问题**：以明确的混合技能接口连接语言任务拆解、几何规划与接触控制，并由视觉语义反馈进行局部重规划；公开摘要和引言显示高度符合本次架构标准，但全文、首发日和开源实现仍需补证。

**系统拆解**：[出版社摘要／Introduction／section previews](https://www.sciencedirect.com/science/article/pii/S0921889026004070)支持语言＋观测→typed MCP技能调用→几何／学习控制器→G1执行→VLM反馈。验证为真实Unitree G1＋灵巧手，章节预览说明7DoF手臂与头部RealSense D435i，涉及长序列／接触操作；尚不能确认实际任务清单、下肢参与、试次数和成功率，不能把“G1真机”扩写成全身移动操作。

| 一级组件／子模块 | 原文内部结构 | 输入 → 处理 → 输出 |
|---|---|---|
| Language-based task planning | 章节预览称DeepSeek-V3.2 | 指令／语义状态→任务拆解与技能决策→带类型的MCP调用及参数；确切schema待全文。 |
| Hybrid skill scheduling | planning-based geometric primitives＋learning-based contact controllers | 技能与参数→几何运动或接触控制→机器人指令；传到底层关节／执行器的接口尚未核实。 |
| Vision-language monitoring | 章节预览称GPT-4视觉语义反馈 | 执行观测→结构化语义状态→监控、中途调整与局部重规划；判定规则／重试预算未知。 |

**训练、复用与新增**：限出版社可读内容，不能补造实现细节。

| 对应结构 | 训练、复用与数据 | 作者在本文中的设计 |
|---|---|---|
| Language-based task planning | 声称无需任务专用高层重训练；是否有提示优化／训练数据未知。 | 用typed MCP限制技能调用与参数化。 |
| Hybrid skill scheduling | 章节预览称学习控制器由PPO训练；具体环境、奖励、数据量、模型与checkpoint未核实；几何原语的planner未核实。 | 在同一技能先验内组合几何规划与接触学习控制。 |
| Vision-language monitoring | 复用VLM推理的角色可确认；具体微调与视觉状态格式未知。 | 用执行后的语义反馈触发局部重规划。 |

**关键 Q&A（2026-10-16）**：Q：为何不进入主榜？A：架构与人形真机证据已达到值得保留的候选程度，但首次online日期、全文方法和实际可下载材料仍不充分。不能因为期刊卷期写12月就断言尚未公开，也不能因9月已登记DOI就确认首次公开日在时间窗；目前应独立列待核实，后续优先补原文与作者仓库。

## 筛除与边界记录

以下保留最容易误纳的线索及原始证据，避免后续按新标题、会议年份或演示重新重复检索；并非穷尽性文献目录。

| 工作 | 不列主匹配的依据 |
|---|---|
| [Being-0](https://arxiv.org/abs/2503.12533)、[Hierarchical Vision-Language Planning](https://arxiv.org/abs/2506.22827)、[HEAD](https://arxiv.org/abs/2508.03068)、[DualTHOR](https://arxiv.org/abs/2506.16012)、[Proprio-MLLM](https://arxiv.org/abs/2510.07882) | 首发分别2025-03-16、06-28、08-05、06-19、10-09，均早于窗口；架构相关不能覆盖日期条件。 |
| [ROS-LLM](https://arxiv.org/abs/2406.19741) | 首次预印本为2024-06，不能按2026期刊版当新增。 |
| [BiBo](https://arxiv.org/html/2511.00041v1#A4.SS6) | v1为2025-10-28；物理人类角色、CLoSD/PHC与IsaacGym，未证实机器人本体。不能将人体avatar当humanoid robot；更早OpenReview日期未核实，未据此作时间排除。 |
| [CoINS/InterNav-VLM](https://arxiv.org/html/2601.03956v1#S9)、[FALCON](https://arxiv.org/html/2512.04381v1) | 实际为四足带臂；FALCON明确Go2＋Airbot。前者人形列未来工作。 |
| [Cortex](https://arxiv.org/html/2607.05377v1)、[RoboQuest](https://arxiv.org/html/2610.10388v1#S3.SS1)、[EmbodiedSkills](https://arxiv.org/html/2609.01281v2#S6) | 分别为ARX ACONE双臂／LIBERO-RoboTwin、Franka Panda＋移动底盘、MetaWorld/CLIPort/CALVIN；未有人形验证。EmbodiedSkills v2换标题不是第二篇新工作。 |
| [Skill2Real](https://arxiv.org/html/2610.02788v1#S4.SS3)、[PhysEvo](https://arxiv.org/html/2610.08995v1#S5)、[COAP/Embodied Turing Machines](https://arxiv.org/html/2610.12369v1#S4.SS1)、[RoboBridge](https://arxiv.org/html/2610.02717v1#S4) | 新近工具／代码／技能工作，但原文分别UR5e、PiPER/RoboDojo、RoboDojo双臂、LIBERO-PRO与桌面单臂；未有人形实验。 |
| [LongAct](https://arxiv.org/html/2605.14504v1)、[EmbodiSkill](https://www.microsoft.com/en-us/research/publication/embodiskill-skill-aware-reflection-for-self-evolving-embodied-agents/)、[APIVOT](https://arxiv.org/html/2607.08024v1) | 抽象embodiment、ALFWorld/EmbodiedBench或KitchenWorlds交互，未验证物理人形执行。 |
| [Humanoid Hanoi](https://arxiv.org/html/2602.13850v1#S3)、[CLAW](https://arxiv.org/abs/2604.11251)、[Locomotion-Grounded Humanoid Soccer](https://arxiv.org/abs/2609.38852) | 有人形技能但缺所需VLM/LLM任务与观测理解；分别递归/PDDL、键盘/timeline/模板、运动足球技能。 |
| [WholeBodyVLA](https://arxiv.org/abs/2512.11047)、[HumanoidArena](https://arxiv.org/html/2606.17833v1)、[FRoM-W1](https://arxiv.org/abs/2601.12799) | 前两者是动作模型／连续参考到tracker，未有显式语义技能调用；后者文本→运动→RL跟踪，缺环境观测推理。WholeBodyVLA不是已读WholeBodyWAM。 |
| [POT-VLA](https://arxiv.org/html/2607.18016v2#S3) | **G1真机71/80有证据**，不能以无真机排除。边界在§3.1上层可为VLM或人工任务文件，§3.2同一VLA直接产动作chunk；未证明评测用了独立VLM技能调用层，暂不列严格匹配。 |
| [MetaWorld-X](https://arxiv.org/html/2603.08572v1#S3) | VLM在训练／离线提供先验，运行时缓存语义＋IRM选择专家，未证明VLM读当前观测调用；与MetaWorld不是同一篇。 |
| [HumanoidTTT](https://arxiv.org/html/2610.00198v1)、[CHOREO](https://arxiv.org/html/2609.22274v1) | 前者缓存ID／规则门控／DDQN，无VLM任务观测决策；后者语言计划预先固定，未披露LLM场景观测，在线重规划／恢复关闭。首发分别9月18、9月11，不能从编号猜月份。 |
| [Claude plays robotics](https://www.anthropic.com/research/claude-plays-robotics) | G1实验是代码控制／RL平衡；高层调预训练移动技能的详细实验是Go2四足，操作是Franka。不能拼接不同本体结果，推导已验证G1语义技能栈。所链接embody仓库检查404。 |
| [OmniSim](https://github.com/omnilink-tech/omnisim/blob/main/docs/developer/rl-current-state.md)、[SkillBlenderLC](https://ajay-vikram.github.io/projects/skillblenderlc.html) | OmniSim有MCP与G1，但[技能说明](https://github.com/omnilink-tech/omnisim/blob/main/docs/developer/skill-library.md)含辅助承重、合成序列，未确认LLM观测→G1技能任务验证；SkillBlenderLC为H1仿真奖励／技能合成，完整refine loop仍方案，未证实在线观测调用。均不以工具列表或视频补足缺口。 |
| [OBiMan](https://github.com/Zixin-Tang/OBiMan) | 已读源码TaskSimulator只改符号字典，非动力学sim；全文物理人形执行未核实。Git历史早于窗口，但提交日不当公开日；不能按IROS2025会议时间直接纳入。 |
| [REAL](https://arxiv.org/html/2607.13653v2)、[Vision-language guided planning and control for humanoid whole-body manipulation](https://doi.org/10.1049/icp.2026.1882) | 待补证线索：REAL的移动双臂平台人形身份未确认；后一篇仅核到IET出版元数据（Ruixuan Jiao/Bo Zhou/Fang Fang，2026-05-01），正文与调用结构未读到，均不凭标题纳入。 |

## 复现判定的实际边界

BiGym补充核验：论文主实验的定稿程序与仓库额外inloop在线工具控制为两条协议；前者成绩不能替代后者效果。本轮修正此前“项目仅离线”的过宽表述，主匹配论文计数仍依其已报告实验范围。代码核查覆盖文件内容与加载路径，而非只看README；权重检查覆盖真实ONNX/PyTorch文件或HF模型分片与配置，资产检查覆盖机器人场景文件或可访问资产包及内部路径。大型权重未全部下载，未安装／启动完整模拟器，未调用模型API做任务，也未接机器人，因此没有任何“本次跑通”结论。对于仅给基础模型地址、Code coming soon、网页仓库、符号dry-run或演示视频的项目，均未据此升级。

若继续复现，应按所选范围建立独立运行记录：先验证资产／权重加载和单技能闭环，再验证VLM实际调用、任务完成判据和受控失败恢复，最后统计多初态结果；目前的排序是材料与论文证据排序，不是实测性能排名。本次只写入`1016_search.md`，不改`papers.md`、`BLUEPRINT.md`或已有关注标记。
