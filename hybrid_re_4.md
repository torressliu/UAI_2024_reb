We are immensely grateful to you for recognizing the importance and novelty of our approach. Your valuable suggestions inspire us to improve our work further. Based on your feedback, we analyzed our work and added extended experiments to the new version. Besides, the writing is optimized in the new version.
### Detailed Comments 
**C1: The necessity and the reason why using transformer? Of course, transformer is always a fashion choice in DL, but I will still wonder if there is a better choice.**

Please allow me to explain why transformer is more efficient in our framework. Since there naturally exists hierarchical dependencies during the action selection process, i.e., state$\to$discrete action→discrete region$\to$continuous action, we use a Transformer-style sequence model to model such dependencies and generate their Q-values (as Transformer is suitable for modeling sequence dependence). We regard the embedded states, actions, regions, etc, as input tokens (like words in NLP). Besides, Qi ,et al.\footnote{Han, Qi, et al.``On the connection between local attention and dynamic depth-wise convolution." ICLR (2021).} demonstrate that transformer has a better ability to capture relevance.

**C2: For you to split the control process into 3 levels, especially the sub-area choice level makes me confused even though you have proved it through ablation experiments. I would wonder about the insight behind this. Or could this problem be solved by other optimization methods like bandit in continuous space?**

Your idea is valuable for our research.
Since previous work has shown that effective policies for hybrid action control need to learn dependencies between discrete action and continuous action [2], we design the hierarchical action structure to explicitly model the correlations between discrete actions and continuous actions. Such a hierarchical decision is a flow from coarse-grained (discrete actions) to fine-grained (continuous actions). Compared with the bandit method (i.e. flattening the hybrid action space without model the dependence between discrete actions and continuous action, partitioning the sub-interval and then selecting the action), our method can simplify the exploration space by using the dependency between discrete actions and continuous actions (that is, directly abandoning the exploration of the continuous action space corresponding to irrelevant discrete actions). 

We compare our method and the discounted UCB bandit method in three scenarios. Table 1 shows that the exploration effect of our method is better, indicating that our hierarchical decision can better explore the optimal strategy.
Table 1. Win rate (Average of 10 runs):
| Method      | Chase | Move| Hard Goal|
| :-----------: | :-----------: | :------------: | :-----------: |
| Ours|$0.81\pm 0.08$|$0.97\pm 0.04$|$o.58\pm 0.13$|
| bandit for continous space |$0.73\pm 0.11$|$0.86\pm 0.07$|$0.27\pm 0.13$|


因为在以往的工作中证明了离散动作和对应的连续参数的有效策略需要学习到两者间的依赖关系的[2]，我们设计成层次决策是为了（1）显性建模离散动作和连续动作的相关性，（2）这样的层次决策是由粗粒度（离散动作）到细粒度（连续动作）的流程，相比于将混合动作空间扁平化后切分子区间再选择混合动作减少大量次优选择区间（剪枝，直接排除包含次优离散动作的决策空间以及次优连续参数子区间的决策空间），且所选离散动作的one-hot向量作为连续子区间决策的输入可以进一步辅助决策。为了验证我们方法的优越性，我们在三个场景上比较了我们的方法以及将动作空间扁平化切分后使用折扣UCB方法进行决策。表1显示我们的方法探索效果更好，说明我们的层次化决策可以更好的探索到更优策略。
