We are deeply grateful for your recognition of our paper's motivation, performance, and potential academic impact. Your positive feedback is highly encouraging. Such feedback motivates us to further our research in this promising field. Thank you for your thorough and constructive review.
### Weakness
**W1: It would be nice if the authors can spend more space explaining how the Q values are propagated in Section 3.1 (4). How is computed? Are Q values also propagated to the previous time step like other MCTS approaches? How are and used (are they paired or just two sets)? How does the algorithm handle stochastic dynamics?**

**W2: It seems like the discussions in the paper are mostly focused on having only one discrete component and one continuous component in the action space. It would be nice if the authors can discuss how the algorithms can be adapted to handle scenarios with multiple discrete components and multiple continuous components.**

### Detailed Comments
**C1: What would happen if a subregion contains mostly bad actions but also some good actions (for example, in Figure 1C, the average Q for the third subregion is actually the lowest). Would MCTS consider this subregion as bad subregion (with low 
$Q_r$)?**

您的问题非常有深度，我们会在新版本强调这一部分。是的，顺序切分确实会存在上述潜在风险。我们为了缓解这一问题带来的负面形象采取了两个技术：
- 一个是借助MCTS强大的探索利用能力来摆脱次优子区间，即策略会有较大几率选择访问次数少的子区间来提升探索到最优区间的可能性。
- 第二个是最主要的方法，即如图2右所示，我们使用$max(Q_{ki})$对子区间进行赋值，即每个区间的最优动作代表该区间的好坏，这样可以引导策略选择最优参数所在的区间而不被平均Q值所影响。

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

感谢您有价值的建议，我们在新版本中使用语言+示意图的形式对MLP架构进行详细介绍。即
