We are immensely grateful to you for recognizing the importance and novelty of our approach. Your valuable suggestions inspire us to improve our work further.

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.

### Weakness
**W1: The major weakness is perhaps the tasks used to evaluate the effectiveness of the proposed methods are not challenging. Checking how the proposed method behave on a more complicated environment would make the proposal much more convincing.**

感谢您有价值的提！我们使用minigrid构造了三个更难的场景，这三个场景共享相同的任务逻辑：经典的拿钥匙开门迷宫任务，Agent must take the key to open the door and go to the goal point，之所以说这一任务较困难是因为即使在单一离散空间的设定下主流强化学习算法TD3,PPO等都无法完全solve这一任务。这三个场景三个目标相同，但地图大小、边界范围不同的场景。We use minigrid to construct a set of maze tasks with the same goal: 

场景介绍：Scenario A is a small map (square boundary), scenario B (square boundary) and scenario C (rectangle boundary) are big maps. The observation space of each scenario is the same, but the scene scale, boundary shape, and obstacle placement are different. [See anonymous link for visualization of the tasks](https://anonymous.4open.science/r/ICML_reviewer_3-0622/README.md). 
动作空间：智能体需要选择离散化方向（前后左右）以及对应方向行走的格数（向前走几格（1至4格），连续参数空间是$[0.5,4.5],环境端会将连续动作离散化保证走的是整数单位：0.5-1.5是一格，1.5-2.5是两格，以此类推。）我们按照文中实验设定将所有方法在上述三个场景进行评估，表1结果显示，我们的方法在两个较为复杂的场景（B,A）上超过了所有基线，这进一步证明我们方法的有效性，目前的SOTAHyAR-TD3较差的表现说明其无法学习到任务的逻辑。
在A上，我们的方法优势稍微不明显，这也印证了在简单的场景确实不存在较困难的探索场景，因此其他基线也可以学到不错的策略。我们在新版本中增加了这组实验来进一步证明我们方法的有效性。
### Detailed Comments
**C1: There is an ablation experiment on the effect of segmenting subregions of continuous actions. However, it does not tell how the hierarchy of discrete-subregion-continuous is determined. I think choosing the hierarchy does matter for the performance of the method. Do you have to manually figure out the hierarchy case by case, or how do you determine the hierarchy if it is not naturally given in the problem?**

感谢您有深度的提问，我们在新版本中对这部分的介绍进一步优化。这种层次化动作空间是任务无关的，即对于任何场景都是 所有离散动作k-连续动作子空间c-确定连续动作。为了实现任务无关化，我们使用最直接的顺序切分对参数空间进行划分即用户只需要设置期望的子空间数，我们的方法会对原始隐空间进行均匀切分。
为了进一步印证我们这种任务无关的层次划分方式是有效的。我们在两个不同的场景与人为设计的层次架构进行对比，针对不同离散动作，我们将其对应的常规有效区间设置的比其他区间小20%，方便策略选择到最优连续参数。我们的方法子区间均为4，表2显示人为构造的参数区间与齐次切分的有效性没有较大差异。不过收敛速度确实会更快。
