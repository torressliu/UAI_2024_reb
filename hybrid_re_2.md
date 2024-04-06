We are deeply grateful for your recognition of our paper's motivation, performance, and potential academic impact. Your positive feedback is highly encouraging. Such feedback motivates us to further our research in this promising field. Thank you for your thorough and constructive review.
### Weakness
**W1: It would be nice if the authors can spend more space explaining how the Q values are propagated in Section 3.1 (4). How is $V_{new}$ computed? Are Q values also propagated to the previous time step like other MCTS approaches? How are $`\mathcal{X}_{kw}`$ and $`\mathcal{Q}_{kw}`$ used (are they paired or just two sets)? How does the algorithm handle stochastic dynamics?**

感谢您的提议，在新版本中我们会对这一部分进一步详细介绍。
- 我们遵循efficent zero[1] 的思想，按照传统强化学习算法中的Q值计算方式[2]，将当前状态s,所选离散动作k,所选连续子区间c作为Q值网络的输入并输出状态-动作Q值作为$V_{new}$。我们选用Q值作为$V_{new}$的原因是Q值的数值大小直接表示当前状态下所选动作的奖励期望，可以作为MCTS中表示节点优劣的度量值。
- 和其他工作中直接使用美MCTS作为全局路径规划算法不同，我们的方法仅借鉴了MCTS的思想来辅助强化学习算法在固定三层层次化动作空间中串行选择动作，并不需要考虑历史状态信息，因此不用回传到以往时间步。
- $`\mathcal{X}_{kw}`$ and $`\mathcal{Q}_{kw}`$是两个集合：当策略每选择一次动作，会将混合动作放入 动作集$`\mathcal{X}_{kw}`$并将该状态-动作对的{V_new}放入$`\mathcal{Q}_{kw}`$中，该集合中的值用于MCTS中的反向传播。
- 您的提问非常有深度。这也是我们考虑使用Q值作为MCTS中节点值的主要原因 the Q value evaluated by the critic is used as the node's value, and as the training progresses, critic is optimized (section 3.1 explains how the values of the nodes are assigned). This makes UCB effective in unstable scenarios. 随着训练的进行，策略通过大量的探索来优化Q网络，使Q值的评估逐渐稳定和准确，进而稳定的Q值回你

**W2: It seems like the discussions in the paper are mostly focused on having only one discrete component and one continuous component in the action space. It would be nice if the authors can discuss how the algorithms can be adapted to handle scenarios with multiple discrete components and multiple continuous components.**

感谢您的建议，新版本对这一部分进行了补充。确实文章主要讨论现实场景中主流的“one discrete component and one continuous component”模式，但我们的方法也可以扩展到其他不常见的动作空间：
- 每个离散动作对应n个连续组件。我们会对连续参数组成的动作空间进行均匀划分为c个子区间，当选择到相应的子区间时，策略网络会输出一个n维的向量代表选择的连续参数。[为方便您的理解，请点击这个匿名链接来看可视化的示例]()。
- 不同离散动作对应的连续组件个数不同。针对这种设定，我们的方法首先按照最大连续组建个数进行输出维数设定从而实现对所有离散动作以及对应连续组件进行统一决策，即当最大连续组件为4时，我们设定连续参数策略输出维度为4，当所选离散动作对应2个连续组件时我们默认只用输出的前两个维度输入环境。此外，为了防止训练梯度回传的混乱，在训练期间我们会将连续组件策略的输出多余维度的梯度阶段，防止噪声梯度影响策略训练。
我们将Goal场景改成了离散组件对应多样化数目的连续组件.动作空间 (离散动作-连续组件):射门-力度(p), 移动-坐标（x,y）,盘带-无连续组件。智能体需要过掉对方后卫并将球踢入球门。表1显示我们的方法在这一场景比其他方法表现更优秀。说明我们上述技术确实可以使我们的方法扩展到更复杂的混合动作空间控制中。
### Detailed Comments
**C1: What would happen if a subregion contains mostly bad actions but also some good actions (for example, in Figure 1C, the average Q for the third subregion is actually the lowest). Would MCTS consider this subregion as bad subregion (with low 
$Q_r$)?**

您的问题非常有深度，我们会在新版本强调这一部分。是的，顺序切分确实会存在上述潜在风险。我们为了缓解这一问题带来的负面形象采取了两个技术：
- 一个是借助MCTS强大的探索利用能力来摆脱次优子区间，即策略会有较大几率选择访问次数少的子区间来提升探索到最优区间的可能性。
- 第二个是最主要的方法，即如图2右所示，我们使用$max(Q_{ki})$对子区间进行赋值，即每个区间的最高Q值代表该区间的好坏，这样可以引导策略选择最优参数所在的区间而不被平均Q值所影响。
为此在三个场景对使用最大Q值作为区间价值和使用平均Q值作为区间价值这两种方法进行了对比。表2的结果显示，使用最大Q值效果要明显更好一些，证明这一方法可以促进策略的探索，找到最优连续参数。

**C2: In Section 3.1, "continuous parameter space $X_k$ marked with rounded rectangles in light origin" -> "continuous parameter space $X_k$ marked with rounded rectangles in light orange"?**

感谢您仔细的阅读，我们在新版本进行修改

**C3: In Equation (6), is $\mathcal{Q_{\hat{k},\hat{w}}}$ same as $\mathcal{Q_{\hat{k}\hat{w}}}$?**

感谢您细致的审稿，这两个是一样的。是我们的写作错误，在新版本中进行了改正

**C4: Why are the results of other algorithms not presented for Chase and Hard Goal in Figure 5?**

我们这么做的原因有二
- 为了突出我们的方法在复杂场景与目前的混合动作空间的SOTA算法HyAR-TD3[]相比，具有较为明显的优势。
- 第二个原因是参考我们的表1已经包含了所有方法在所有场景的分数。最后两行的结果显示其他基线在这两个复杂场景上基本无法学到有效策略（在0分上下波动），因此我们没有在图7右侧展示其他基线的曲线。

**C5: Why are the curves noisier (with more spiky intervals) in the ablation studies (Figure 7) compared to Figure 5? What is the environment for Figure 7?**

感谢您指出我们表述模糊的地方，在新版本我们会改正。我们在图七使用的是附录B中介绍的Hard Move,图七中的曲线较为抖动是因为这是五个种子的均值方差曲线。而我们的主实验使用的是十个种子。
**C6: It may be good to describe the MLP architecture in more detail in Section 5.2 or in the appendices.**

感谢您有价值的建议，我们在新版本中使用语言+示意图的形式对MLP架构进行详细介绍。即我们为每个输入(状态，离散动作，子区间)提供一层全联接网络[dim,64]作为特征提取层，并将三个向量按最后一个维度拼接后输入到5层全联接层代替同等深度的transformer. 
