Thanks for your thorough, meticulous, and impartial advice which considerably eases the enhancement of our research. We meticulously amended the writing inaccuracies and optimized our work.

### Questions
**Q1.3: The examples used in the benchmarks are also small scale: a 2D continuous environment with a few obstacles, suggesting poor scalability. What about a hybrid task where, instead of reaching a goal avoiding some obstacles, the agent would need to look for a key and open a door on the other side of the map. This deserves clarification.**
感谢您有价值的提议！我们使用minigrid构造了您描述的任务。且为了更全面的分析算法有效性，我们构造了三个目标相同，但地图大小、边界范围不同的场景。We use minigrid to construct a set of maze tasks with the same goal: 
Agent must take the key to open the door and go to the goal point. 
Scenario A is a small map (square boundary), scenario B (square boundary) and scenario C (rectangle boundary) are big maps. The observation space of each scenario is the same, but the scene scale, boundary shape, and obstacle placement are different. [See anonymous link for visualization of the tasks](https://anonymous.4open.science/r/ICML_reviewer_3-0622/README.md). 
动作空间：智能体需要选择离散化方向（前后左右）以及对应方向行走的格数（向前走几格（1至4格），连续参数空间是$[0.5,4.5],环境端会将连续动作离散化保证走的是整数单位：0.5-1.5是一格，1.5-2.5是两格，以此类推。）我们按照文中实验设定将所有方法在上述三个场景进行评估，表1结果显示，我们的方法在两个较为复杂的场景（B,A）上超过了所有基线，这进一步证明我们方法的有效性，目前的SOTAHyAR-TD3较差的表现说明其无法学习到任务的逻辑。
在A上，我们的方法优势稍微不明显，这也印证了在简单的场景确实不存在较困难的探索场景，因此其他基线也可以学到不错的策略。
**Q1.1: The first one is that it is unclear how the proposed method helps in the so-called hard exploration tasks.** 

以往的混合动作空间算法大多需要同时对离散动作和连续动作同时选择，由离散和连续动作空间结合而成的复杂决策空间提升了策略的决策难度，即必须同时选对离散动作和最优参数才能得到高分。为了缓解策略在混合动作空间中探索困难，我们借鉴以往单一连续动作空间控制中的离散化思想[]。将每个离散动作对应的参数划分为多个离散化区间.并最终对混合动作空间进行重构，
具体而言，就是将扁平化决策：$\pi(s)\to (a_{discrete},a_{continious}$转化为树状动作空间：$\pi(s)\to a_{discrete},\pi(s,a_{discrete})\to a_{continious_region},,\pi(s,a_{discrete},a_{continious_region})\to a_{continious}$.
这样做有两个好处，
- 在策略具有较强的探索利用能力的前提下，串行选择降低了并行选择的难度，因为在大多数场景离散动作的选择决定性更强，e.g. 
在迷宫中方向的选择比速度的选择更关键，相邻的参数对环境的影响相似因此我们对参数空间进行了切分即同一子区间内的参数语义相似即选对子区间后策略只需要在较小的子区间探索最优参数即使不是最优其对应的奖励也是正向的引导策略偏好对这一子区间进行采样。
此外，三层次串行决策的好处是每一层的决策都较为简单，即先从最重要的离散动作中选择，再从离散的参数子区间进行选择，最后在子区间中国年确定参数。
- 我们的框架相比于以往的并行决策，引入了更多信息辅助策略进行决策，在决策子区间时网络的输入会加入所选离散动作的one-hot编码辅助策略选择参数子区间（每个动作在每个状态可能对应一个可行区间例如在迷宫中向前走这一离散动作往往对应较高的速度），同理策略选择参数时也会在子区间和离散动作的辅助下进行决策。
- 但以上前提都是在策略具有较强的探索利用能力的前提下才能在三层串行决策中有效训练。为此，在efficient-zero[2]中使用MCTS辅助RL算法在非稳定场景中进行决策的启发下，我们雇佣MCTS实现对树状动作空间进行高效探索，并使用Q网络评估的Q值计算每一层节点的价值。从而增强了策略在层次化决策中的能力。

**Q1.2: Algorithm 1 is confusing, requiring traversing all sub-regions first, then doing MCTS and eventually, training the transformer. This suggests that a very good initialization of the value and policy networks is needed, with samples covering most of the interesting parts of the state-action space.** 

- 感谢您指出表述模糊的部分，新版本对这一块进行了优化。并不是一定要全部遍历后才能进行MCTS，我们这样做的原因是在实验过程中发现先对节点进行遍历可以提升策略的学习速度。但其实不遍历最终的策略表现水平也可以达到相似的水平（超越其他baselines），在chase上略差。我们在新版本附录中公开了在三个场景上对比遍历和不遍历的方法的测试分数。表4的结果展示，二者最终收敛到同一水平，但遍历可以实现更快的收敛。表5记录了两种方法的收敛速度。
- 感谢您富有见地的提问！我们在训练我们的算法时是随机初始化的。但您提供了一个很好的思路，我们也好奇初始化参数会会对方法产生什么影响，以及会带来多大的提升。我们在三个新建立的场景上与我们的方法进行了对比并在三个场景上进行了验证。表6中的结果显示，随机初始化参数在经过多轮训练后可以达到和初始化参数相同的效果。这也侧面证明我们的MCTS+RL的结合是有效的。

进一步，我们分析了两种策略学习的速度。表5显示，较好的初始化确实会有效提升策略的学习效率。上述实验被放到了新版本中。

**Q2: The proposed methodology appears too incremental. The authors propose a strategy to encode/represent continuous parameterized actions which can be efficient for MCTS but, unfortunately, there is no formal analysis besides the empirical evaluation. 
For example, why a homogeneous partitioning? It is unclear how this strategy would extend to other hybrid settings beyond the ones used in the experiments.**

感谢您指出表述模糊的地方，我们在新版本中对齐次划分子区间做了进一步解释：
- 第一个原因是我们希望使用简单的技术做出一个任务无关的子区间划分方法。那么顺序（齐次）切分是最直接的方法，我们只需要设定区间数目。
- 另一个使用齐次分割的原因是，对于大多数场景，参数往往代表离散动作的程度（速度等），因此相近的参数往往对环境的影响相似，例如在迷宫中智能体在同一方向上选择走1格和走2格效果是相似的。
- 这一方式可以直接扩展到参数维度是多维的场景，例如两维的参数（比如坐标），我们的齐次切分依旧可以使语义相似的参数在一个区间。可视化请看这个匿名链接（）
- 此外，我们进一步验证，对于特定的任务(场景B)，表7显示人为构造的参数区间与齐次切分的有效性没有较大差异。不过收敛速度确实会更快。

**Q3: The paper claims to address several challenges: hybrid action spaces, hard (sparse) exploration problems, and eventually mult-agent systems. It is confusing where is the main contribution of the work.**

我们为粗糙的表达给你带来的困扰道歉，在新版本中我们进一步突出了我们的动机。我们的核心是观测到混合动作空间中复杂的离散-连续关系会造成的探索空间爆炸进而导致的探索困难，并提出重构动作空间并借助transfomer-based MCTS辅助强化学习来缓解这一问题。在这一主要贡献基础上，我们在附录中（以防读者混淆）展示了我们的方法拓展到多智能体场景的潜力，并将我们的方法和CTDE框架结合的效果举例。


**Q4: Finally, there is a vast literature combining discrete (for example combined task and motion planning) that is not mentioned at all in this work.**

抱歉我们以往更多关注了混合动作空间控制这一领域的相关工作。我们对离散化这一领域进行了调研，并将这一部分放到了背景部分。
