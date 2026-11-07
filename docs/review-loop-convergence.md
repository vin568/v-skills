# 评审循环为什么不收敛 —— 实证调研与 Skill 设计准则

> 起因:`test-review` skill 连续 review 几轮始终不收敛。待检验命题:"开放性是**领域固有**的,换任何
> skill 都一样"。本文用一手文献实证检验该命题,并给出 v-skills 里所有"评审/审计类 skill"的收敛设计准则。
> 结论:**命题一半成立、一半被推翻**;`test-review` 的不收敛主要是**过程设计缺失**,不是领域判决。

## 1. 结论(三层拆分)

| 层 | 判定 | 一句话 |
| --- | --- | --- |
| A. 理想层:"穷尽问题空间"的收敛点 | **成立**(定理级) | 不存在可靠又有效的可计算测试判据(Howden 1976);等价变异体/不可行路径不可判定;非缺陷类 findings 实证上永不枯竭 |
| B. 操作层:"工程化的收敛目标" | **被推翻** | 停止规则(stopping rule)是测试领域的一等公民,已被定义、量化并工业化为门禁;人工评审真缺陷收益 1-2 轮饱和;LLM 循环 2-4 轮饱和;判据分解可把评审一致性从 α≈0.09 提到 0.67 |
| C. 对 `test-review` 的归因 | **过程设计缺失** | 它不缺领域给的收敛点(领域有的是),它缺的是:**没有声明任何收敛目标**——无冻结判据清单、无 findings→结论映射、无轮次预算、无跨轮持久 ID,且把无界的意见类 findings(P3)与有界的真缺陷流一起计入产出 |

## 2. 证据链

### 2.1 理想完备判据不存在(命题 A 成立的部分)

- Goodenough & Gerhart (1975) 提出"理想"测试判据须同时 **reliable + valid**(满足即可保证无错)。
  随即被证明**不存在可计算实现**:"there is no computable criterion that satisfies the two
  requirements"(Howden 1976;且 reliable 与 valid 对给定程序互斥,Weyuker & Ostrand 1980)。
  出处:Zhu, Hall & May, *Software Unit Test Coverage and Adequacy*, ACM Computing Surveys
  29(4), 1997, §1.1(全文:https://www.cs.toronto.edu/~chechik/courses18/csc410/p366-zhu.pdf)。
- 连代理指标自己的 100% 都不可判定:不可行路径/死代码不可判定(Weyuker 1979,同上综述 §2/§4);
  等价变异体检测不可判定(Budd & Angluin 证明,转引自 Jia & Harman 2011,
  https://web.eecs.umich.edu/~weimerw/2018-481/readings/mutation-testing.pdf),实证中
  **10-40% 的变异体是等价的**,最好的自动检测器只能找出 **47.63%**。
- 工程师的一手证词:Facebook 变异测试试点中两位拒绝者给出的理由是 *"the infinite ways one
  could modify a piece of code"*(Beller et al., ICSE-SEIP 2021,
  https://ar5iv.labs.arxiv.org/html/2010.13464)。

### 2.2 操作层收敛目标:领域恰恰是范本(命题 B 被推翻的部分)

- **停止规则是学术定义**:adequacy criterion 的第一含义就是 stopping rule,Zhu et al. 1997
  Definition 1.1(C: P×S×T→{true,false});且任何量化测量 M 配阈值 r 即构成停止规则 M_r
  (Definition 1.2)。Weyuker (1986) 的公理化同样把 adequacy 当 stopping rule 使用。
- **工业门禁真实存在**:变异分数门禁(PIT `mutationThreshold` "fail the build",
  https://pitest.org/quickstart/maven/;Stryker `thresholds.break`,
  https://stryker-mutator.io/docs/stryker-js/configuration/);覆盖率分级(Google 官方博客
  2020:60% acceptable / 75% commendable / 90% exemplary,per-commit 99%,"gate deployments…
  should prevent the code from being checked in",
  https://testing.googleblog.com/2020/08/code-coverage-best-practices.html);DO-178B 对 Level A
  航电软件**强制** MC/DC。统计停止规则:Mills (1972) 缺陷播种捕获-再捕获估计;Dalal & Mallows
  (1988, *JASA* 83(403)) 给出**精确最优停止公式**及剩余缺陷数的概率保证。
- **官方标准把"停止"定义成干系人约定**:ISTQB 术语表 "exit criteria"——"agreed upon with the
  stakeholders for permitting a process to be officially completed",且 CTFL v4.0 §5.1.3 明言
  **"Running out of time or budget can also be viewed as valid exit criteria"**(干系人接受风险
  即可)。https://glossary.istqb.org/en_US/term/exit-criteria
- **但门禁≠穷尽**(这是 A 与 B 的衔接):覆盖率与测试有效性在控制套件规模后相关性趋零
  (Inozemtseva & Holmes, ICSE 2014, 31,000 套件,Kendall τ 最低 −0.35,
  https://cs.uwaterloo.ca/~rtholmes/papers/icse_2014_inozemtseva.pdf);Google 与 Facebook 都
  **拒绝**用绝对变异分数做门禁(Google:绝对分数"infeasibly expensive… not actionable",改为
  per-diff 存活变异体呈现,Petrović & Ivanković, ICSE-SEIP 2018,
  https://eecs481.org/readings/mutation-google.pdf);静态分析 Google 的做法是"清空存量→把检查
  升级为编译错误挡构建"——有限检查集+确定性输出=可执行的不动点(Sadowski et al., CACM 61(4),
  2018, DOI 10.1145/3188720;2009 年 Fixit 周也只修了存量告警的 16% 就收手:预算式收敛)。

### 2.3 人工评审:真缺陷流 1-2 轮饱和;意见流永不枯竭

- 评审会议(第二轮)边际真缺陷增益仅 **3-5%**(Votta 1993,AT&T 50 次检视);Porter–Votta–Basili
  (TSE 1995)实测会议增益 4.7%/3.1% 对会议损失 6.8%/7.7%,**净贡献≈0**。
- Porter, Siy, Toman & Votta(TSE 1997,Lucent 5ESS 真实项目 18 个月随机实验):**2 人组 = 4 人组**
  的缺陷检出;第二个独立小组**不比**一个小组多找到缺陷;轮间修复不提升第二轮检出。
- 工业实践跨文化收敛到 2 名评审者、1-2 轮(Rigby & Bird, ESEC/FSE 2013);评审者对同一区域的有用
  评论占比在第 3-4 次过顶(Czerwonka et al., ICSE-SEIP 2015,
  https://www.microsoft.com/en-us/research/wp-content/uploads/2015/05/PID3556473.pdf)。
- **饱和平台远低于 100%**:Fagan (1976) 约 2/3;PVB 1995 团队仅检出播种缺陷 24-55%;Kemerer &
  Paulk (2009) 50-67%;McIntosh et al. (2016):70-87% 的出缺陷组件当初是被 100% 评审过的。
- **决定性数据**(对本案最关键):Porter et al.(TOSEM 1998)——
  *"while the number of true defects being raised dropped as time went by, the total number of
  issues did not"*;全部提出的问题里只有 **18%** 是真缺陷,60% 是"软性维护意见"。
  新增独立评审者总能找到**新的**东西(N-fold inspection,Schneider–Martin–Tsai 1992)。
  → **把一切 finding 都计数的评审循环,人工评审员同样永远跑不完。** 这不是 LLM 特有,也不是
  skill 特有,是"计数口径"问题。

### 2.4 LLM 循环:没有"无限发现"模式;饱和或退化;不稳定集中在近平局区

- 无外部信号的自我批评:数学任务上 Self-Refine 的自我反馈 **94% 是 "everything looks good"**
  (Madaan et al., NeurIPS 2023);Huang et al.(ICLR 2024
  https://arxiv.org/abs/2310.01798)**所有** model×benchmark 组合的内在自校正都**降准**
  (如 GPT-3.5 CommonSenseQA 75.8→38.1→41.8,先崩后弹=震荡);CRITIC 无工具时 GSM8K **−1.8**;
  Kamoi et al.(TACL 2024)综述结论:*没有任何工作证明纯内在自反馈在通用任务上成功*,成功案例
  都有外部可靠信号(执行器/标准答案/检索)。
- 饱和速度:迭代自提升的"生成-验证差距"**2-3 轮内趋零,与模型规模无关**(Song et al. "Mind the
  Gap", https://arxiv.org/abs/2412.02674);Self-Refine 自己的逐轮边际增益 11.3→6.4→3.0。
- Judge 的两个世界:同题重复裁决高度稳定(一致性 0.85-0.97+,Shi et al. 2025;
  GPT-4 vs 人类 85% ≈ 人类互相 81%,Zheng et al. NeurIPS 2023),**但在质量接近的对上,微扰即翻转
  46-83%**(Wang et al. 2023 换序冲突率;Shi et al. 2025:不一致度随质量差缩小呈抛物线上升)。
  → 评审循环的后期恰好整日活在近平局区:那儿的裁决近似抛硬币,**每轮重判=每轮"新发现"**。
- 工业部署全都靠**收窄**而非多轮收敛:Google AutoCommenter 有用率 54% 平台期→压制 17-22 个
  不可行动类别后 >80%(https://arxiv.org/abs/2405.13565);Meta 接受率 13%→19.7%(门控+微调,
  无门控版本反而让评审变慢,https://arxiv.org/abs/2507.13499);Mozilla/Ubisoft 原始接受率仅
  7-8%(https://arxiv.org/abs/2411.07091);Atlassian 限定范围+质量门控后 38.7%
  (https://arxiv.org/abs/2601.01129)。

### 2.5 收敛可工程化,且效应量已知

- 判据分解为二值清单:12 个 LLM 评审器间一致性 Krippendorff's α 从 **0.09→0.48**,大模型组
  **0.05→0.67**(≈人类评审员互相的 κ≈0.7);二值化本身无效,是**分解**起作用(CheckEval,
  https://arxiv.org/abs/2403.18771)。
- 判据**具体度**同时预测 AI 自一致性与 AI-人类一致性(QWK 0.731→0.903,按判据可操作性排序)
  (AlAmir 2026, https://www.frontiersin.org/journals/education/articles/10.3389/feduc.2026.1861960/full)。
- 人类评分者:75 项研究的元综述——analytic、topic-specific 的 rubric + 示例/培训提升评分信度
  (Jönsson & Svingby 2007, DOI 10.1016/j.edurev.2007.05.002)。
- 聚合协议:多次采样多数投票提升与人类一致率;强行温度 0 反而降低准确率;无辅助手段时所有 judge
  的自一致性 α<0.8(Haldar & Hockenmaier, EMNLP Findings 2025,
  https://aclanthology.org/2025.findings-emnlp.1361.pdf)。
- 逐条评级本质上多解(rating indeterminacy)时,修法是换测量模型(响应集/升级),不是接受发散
  (Guerdan et al., NeurIPS 2025, https://arxiv.org/abs/2503.05965)。

## 3. 反向证据(不支持上述结论的部分,如实列出)

- 强 judge 在区分度好的样本上达到并超过人类互相的一致率(85% vs 81%,Zheng 2023)——LLM 评审
  不是天然不可用。
- judge 的同题重复稳定性很高(0.85-1.00,Shi 2025)——**run-to-run 噪声不是不收敛主因**;
  主因是"每轮重派锚点"与"近平局区重判"。
- Self-Refine 在偏好类任务上平均 +20%——循环有价值,失败模式集中在**验证型**任务。
- 领域内部对"stop"的共识本身承认:任何门禁都不证明没有更多问题(§2.2 最后一组引用)。

## 4. Skill 设计准则(每条溯源)

任何"对开放性制品做多轮评审/审计"的 skill(`test-review`、`architecture-fit-review`、后续新增):

1. **开头声明机器可检查的停止条件**。有限冻结清单 over 有限范围,或量化测量+阈值(Zhu Def 1.2
   的 M_r)。收敛的定义是"对冻结清单的复审复现相同裁决且无新增高严重度项",**不是**"没有更多
   问题"。证据:§2.2;内部正面样本:`autopilot-grilling` 的"frontier 为空即完成"。
2. **判据清单跨轮冻结,finding 用持久 ID**。禁止每轮重新派生 INV/规则集——每轮换锚点=每轮
   "新发现"。证据:§2.4 近平局翻转 + §2.5 分解判据的稳定性;内部正面样本:`autopilot-grilling`
   的 Q#/A# 永不重编号 + 决策日志。
3. **把定性维度分解成具体的、近二值的判据**,越具体越稳。证据:§2.5 全节(CheckEval、
   concreteness gradient、Jönsson & Svingby)。
4. **预算轮次(2-4 轮),并写明预算耗尽时的语义**(降级为"推荐答案+待复核",如
   `autopilot-grilling` 的 `⚠️ Needs review`)。证据:§2.3 人工 1-2 轮、§2.4 LLM 2-4 轮饱和;
   ISTQB:预算耗尽是合法退出(干系人知情)。
5. **收敛口径只计"真缺陷/高危守卫"流,意见流记录但不阻塞**。真缺陷 vs 意见要显式分类;
   P3/风格类永不计入收敛判定。证据:§2.3(18% 真缺陷;意见流不枯竭)。
6. **近平局裁决升级给人,不要机器重判**。区分度好的交给 skill,质量接近的(该不该删/合/保)
   抛硬币不如问人。证据:§2.4 两世界;§2.5 响应集/升级协议。
7. **高严重度断言(P0/P1)必须有外部 oracle 实证**(物理变异探针、执行、真实契约),否则标注
   "假设"不计入收敛。证据:Huang 的 oracle 行、CRITIC 有工具才转正、Reflexion 靠单元测试。
8. **用聚合(多样本投票)稳定裁决,而不是低温单次**。证据:Rating Roulette。

## 5. 对 `test-review` 的落地情况(已于 2026-11-07 按下列准则改造)

| 准则 | 落地机制(改造前状态) |
| --- | --- |
| 1 停止条件 | (原无)新增 "Convergence Definition" 小节 + 输出首节 `收敛状态`:上一轮缺陷流全部 fixed-verified/wontfix-escalated + 冻结清单零新增 P0/P1 + 仅剩意见流,并显式声明"收敛≠没有更多问题" |
| 2 冻结清单/持久 ID | (原 INV 每轮重编)新增 `<git-dir>/test-review/<scope-id>.md` 台账:INV 冻结编号、`F-#` 跨轮持久、后续轮只做验证关闭,新增 INV 须记 `INV-D#` delta 并给理由 |
| 3 判据分解 | (原五维度定性整评)新增 "Binary Probes":五维度各拆 2-3 条近二值判据(yes/no/UNVERIFIED + 证据指针)入台账 |
| 4 轮次预算 | (原无)默认 2 轮深度 + 1 轮验证;耗尽→`⚠️ 待人工复核` 并停 |
| 5 双流分计 | (原 P3 与 P0 同表)Step 4 拆缺陷流(P0/P1/P2,阻塞收敛)/意见流(P3/MERGE/DELETE,永不阻塞);输出两张表 |
| 6 近平局升级 | (原 MERGE/DELETE/KEEP 每轮重判)新增 "Verdict Stickiness":首轮裁决粘滞,仅新 oracle 证据可推翻;存疑项一次性打包升级给用户 |
| 7 P0 需 oracle | (原物理探针可选)P0/P1 必须物理实证或显然可静态判定,否则标 `[待实证假设]`(H)不计收敛;工作区不干净时相关缺陷项自动降级为 H 并说明 |
| 8 聚合 | (原无,分片引入方差)P0/P1 vs P2 边界分类用 3 个独立子代理多数投票 |

保留未动:五维度分类学、变异探针与取证日志、契约证据顺序(schema→`--help`→E2E)、反锚定(先读生产代码)、Pre-Screen Funnel、大白话写作风格约束。

`architecture-fit-review` 对照:范围有界(git diff 有限区间)天然抑制开放性,但同样缺 1/2/4/5——后续改造候选。

## 6. 引用登记(访问级别)

一级(全文读过):Zhu/Hall/May CSUR 1997;Sadowski et al. CACM 2018;Zheng et al. NeurIPS 2023;
Jia & Harman TSE 2011;Petrović & Ivanković 2018;Beller et al. 2021(Facebook);Inozemtseva &
Holmes 2014;Huang et al. ICLR 2024;CRITIC(Gou et al. ICLR 2024);Self-Refine(Madaan et al.
2023);CheckEval;Haldar & Hockenmaier 2025;Google Testing Blog 2020;PIT/Stryker 官方文档;
ISTQB 术语表与 CTFL v4.0 §5.1.3;Porter 系列三篇(1995/1997/1998);Rigby & Bird 2013;
Czerwonka et al. 2015;Bacchelli & Bird 2013;Beller et al. 2014;Kemerer & Paulk 2009;
AutoCommenter/MetaMateCR/RovoDev/RevMate 四篇工业报告;AlAmir 2026;Dalal & Mallows 1988
(经 Dalal–Lyu–Mallows 综述全文)。
二级(摘要级或经开放二手交叉验证):Votta 1993 与 Eick 1992(付费,数字转引自开放的 Porter/Petersson
全文);Jönsson & Svingby 2007(摘要);Guest 2006(经二手);Weyuker 1986(经 CSUR 综述复述);
Mills 1972(经引文);Guerdan 2025(摘要);Prometheus 2(摘要)。
三级(工程博客,非同行评审):Thinking Machines 非确定性博客;Meta Engineering 2025(变异+LLM)。
已在调研中剔除:无法定位的 "Luo et al. ICSE 2025"(多家替代工业研究已覆盖同一论点)。

---
*调研方法:4 个并行研究代理 + 人工核验一手来源(全文/摘要/二手交叉验证分级见上),2026-11-07。*
