Thanks for your thorough, meticulous, and impartial advice which considerably eases the enhancement of our research. We meticulously amended the writing inaccuracies and optimized our work.

### Questions
**Q1.1: The examples used in the benchmarks are also small scale: a 2D continuous environment with a few obstacles, suggesting poor scalability. What about a hybrid task where, instead of reaching a goal avoiding some obstacles, the agent would need to look for a key and open a door on the other side of the map. This deserves clarification.**

Thanks for your valuable advice! We will add validation on complex scenarios in the new version. We use minigrid to construct a set of maze tasks with the same goal: Agent must take the key to open the door and go to the goal point. These three scenes have the same three targets, but different map sizes and boundary ranges.

**Environment Introduction:** Scenario A is a small map (square boundary), scenario B (square boundary) and scenario C (rectangle boundary) are big maps. The observation space of each scenario is the same, but the scene scale, boundary shape, and obstacle placement are different. [See anonymous link for visualization of the tasks](https://anonymous.4open.science/r/ICML_reviewer_3-0622/README.md). 

**Action space:** Discretization direction: left, right, forward, back; Continuous action: Speed, the number of squares to walk in the corresponding direction (1 to 4). The continuous action space is [0.5,4.5], and the environment will discretize the continuous actions to ensure that they are in integer units: [0.5,1.5) is one step, [1.5,2.5) is two, and so on.

We evaluate all methods in the above three scenarios according to the experimental settings in this paper. The results in Table 1 show that our method outperforms all baselines in three complex scenarios, which further proves the effectiveness of our method. The poor performance of the current SOTA method, i.e. HyAR-TD3, indicates the inefficiency of its exploration ability.

Table 1. Win rate (Average of 10 runs):
| Method      | Scenario A (%)| Scenario B (%)| Scenario C (%)|
| :-: | :-: | :-: | :-: |
| HyAR-TD3|$40$|$20$|$30$|
| HPPO |$30$|$0$|$30$|
| PDQN |$10$ |$0$|$10$|
| HHQN |$30$ |$10$|$20$|
| **Ours** |$60$ |$40$|$50$|

The results in Table 1 show that our method outperforms all baselines in three complex scenarios, which further proves the effectiveness of our method. The poor performance of the current SOTA method, i.e. HyAR-TD3, indicates the inefficiency of its exploration ability.

**Q1.2: It is unclear how the proposed method helps in the so-called hard exploration tasks.** 

Most of the previous hybrid action space algorithms need to select discrete actions and continuous actions at the same time. The complex decision space composed of discrete and continuous action spaces increases the difficulty of policy decisions, that is, the correct discrete actions and efficient continuous parameters must be selected at the same time. To alleviate the difficulty of policy exploration in the hybrid action space, we divide the parameters corresponding to each discrete action into multiple discretization regions. Finally, the flat hybrid action space is reconstructed into a hierarchical structure. Specifically, we transform the flat decision: $' \pi(s)\to (a_{discrete},a_{continious} '$into a decision tree-like form: $`\pi(s)\to a_{discrete},\pi(s,a_{discrete})\to a_{continious_region},,\pi(s,a_{discrete},a_{continious_region})\to  a_{continious}`$.

Hierarchical decision making has two benefits：
- Under the premise that the policy has strong exploration and exploitation ability, hierarchical decision can reduce the difficulty of parallel selection. Because, such a hierarchical decision is a flow from coarse-grained (discrete actions) to fine-grained (continuous action sub-area). For example, in a maze, the choice of direction is more important than the choice of speed. Hierarchical decision making can first find the optimal direction by a small amount of exploration to ensure that the policy obtains positive feedback, and then find good continuous action subintervals from the optimal direction by further sampling, and finally quickly explore the optimal parameters in a small subregion.
- All the above prerequisites can be effectively trained in hierarchical decision-making only if the policy has strong exploration and exploitation ability. To this end, inspired by the use of MCTS to aid RL algorithms in decision making in stochastic scenarios[1], we employ the MCTS to efficiently explore the tree-shaped action space and compute the value of nodes at each layer using Q-values evaluated by Q-networks. By doing this, the ability of the policy in hierarchical decision making is enhanced. 
- Our framework has the potential to be improved, as we currently use discounted UCB[1]. We have recently tried a variety of more advanced UCB calculation methods, UCB-V, to verify in multiple scenarios, and found that the effect can be further improved. This further embodies that the combination of our constructed hierarchical decision framework and MCTS is effective.

**Q1.3: Algorithm 1 is confusing, requiring traversing all sub-regions first, then doing MCTS and eventually, training the transformer. This suggests that a very good initialization of the value and policy networks is needed, with samples covering most of the interesting parts of the state-action space.** 

Thanks for pointing out the vague part, we will explain it in detail and add additional experiments in the new version. It is not necessary to traverse all sub-regions before performing MCTS, the reason we do this is that we found in the experiment process that traversing the sub-regions first can improve the learning speed of the policy. The policy without traversing all sub-regions can also achieve a similar level of performance (outperforming the other baselines) with the help of MCTS's powerful ability to balance exploration and exploitation. 

We expose the performance of the methods. Method 1: traversing all sub-regions first. Method 2:  without traversing all sub-regions first and directly doing MCTS. The results in Table 2 show that both methods perform at similar levels, with method 1 performing slightly better on Chase. Table 3 shows the convergence speeds of the two methods, "traversing all sub-regions first" indeed helpful to improve the exploration efficiency of the policy and improve the learning speed.

Table 2. Performance (Average of 5 runs):
| Method      | Chase | Move| Hard Goal|
| :-: | :-: | :-: | :-: |
| Method 1|$0.81\pm 0.08$|$0.97\pm 0.04$|$0.58\pm 0.13$|
| Method 2|$0.72\pm 0.05$|$0.95\pm 0.08$|$0.56\pm 0.17$|

Table 3. Learning speed (Average of 5 runs):
| Method      | Chase | Move| Hard Goal|
| :-: | :-: | :-: | :-: |
| Method 1|$2h12min$|$1h26min$|$8h41min$|
| Method 2|$2h48min$|$2h05min$|$10h13min$|

**Q2: The proposed methodology appears too incremental. The authors propose a strategy to encode/represent continuous parameterized actions which can be efficient for MCTS but, unfortunately, there is no formal analysis besides the empirical evaluation. For example, why a homogeneous partitioning? It is unclear how this strategy would extend to other hybrid settings beyond the ones used in the experiments.**

Thanks for pointing out the ambiguity, and we've highlighted the reason for using homogeneous partitioning in the new version.
感谢您指出表述有歧义的地方，我们在新版本中对齐次划分子区间做了进一步解释：
- The first reason is that we want to design a task-agnostic subregion partition method. And sequential (homogeneous) segmentation is the most direct method, we only need to specify the number of regions and do not need more prior knowledge to design the action structure for each task.
- Another reason to use homogeneous segmentation is that, for most scenarios, the parameters tend to represent the degree (speed, etc.) of discrete actions, so close parameters tend to have similar effects on the environment. For example, in a maze, if the agent chooses to go 1 grid and go 2 grids in the same direction, the effect will be similar.
- This approach can be directly extended to the case where the parameter dimension is multi-dimensional, such as two-dimensional parameters (e.g., coordinates), and our homogeneous partition can still keep semantically similar parameters in the same interval. Visualize this anonymous link ()
- When the continuous parameters corresponding to different discrete actions are different. Our method first sets the output dimension according to the maximum number of continuous actions to realize the unified decision of all discrete actions and the corresponding continuous actions. That is, when the maximum number of continuous parameters is 4, we set the output dimension of the continuous action policy to be 4. When the selected discrete action corresponds to two continuous actions, we default to only use the first two output dimensions to interact with the environment.  In addition, we truncate the gradient of the output excess dimension of the continuous action to prevent the noise gradient from affecting the policy training.

We construct a complex mixed action space task using the Goal scenario to verify the effectiveness of our approach for diverse hybrid action settings. The agent needs to get past the opposing defender and kick the ball into the goal. Discrete actions correspond to a diverse number of continuous actions.

*Action space (discrete action-continuous action) :* Shot-force (p), move -coordinate (x,y), dribble -no continuous component. 

Table 4 shows that our method outperforms other methods in this task. It indicates that our above techniques can indeed enable the extension of our approach to more complex hybrid action space control.

Table 4. Performance (Average of 5 runs):
| Method| Goal with complex action space|
| :-: | :-: |
| **Ours**|$0.51\pm 0.14$|
| HyAR-TD3|$0.36\pm 0.08$|
| HPPO|$0.06\pm 0.11$|
| PDQN|$0.17\pm 0.09$|
| HHQN|$0.14\pm 0.18$|

**Q3: The paper claims to address several challenges: hybrid action spaces, hard (sparse) exploration problems, and eventually mult-agent systems. It is confusing where is the main contribution of the work.**

We further highlight our contribution in the new version. Our core motivation, which does not involve multi-agents, is to address exploration difficulties due to complex discrete-continuous relationships in hybrid action Spaces. We only use multi-agent scenarios to verify that our method can perform well compared to other methods in more complex scenarios, such as multi-agent scenarios.

**Q4: Finally, there is a vast literature combining discrete (for example combined task and motion planning) that is not mentioned at all in this work.**

Thank you for your advice. We'll update this section to the new version of the background section.

抱歉我们以往更多关注了混合动作空间控制这一领域的相关工作。我们对离散化这一领域进行了调研，并将这一部分放到了背景部分。
and neighboring parameters in the same speed space have similar effects on the environment so we split the parameter space meaning that the parameters in the same subinterval are semantically similar meaning that after choosing the right subinterval, the policy only needs to explore the optimal parameter in the smaller subinterval and even if it is not optimal, the corresponding reward is positive guidance policy preference sampling this subinterval.

In addition, the benefit of three-level serial decision making is that the decision of each level is relatively simple, that is, policy first selects from the most important discrete actions, then selects from the discrete parameter subregions, and finally determines the parameters in the subinterval Chinese years.
- 我们的框架相比于以往的并行决策，引入了更多信息辅助策略进行决策，在决策子区间时网络的输入会加入所选离散动作的one-hot编码辅助策略选择参数子区间（每个动作在每个状态可能对应一个可行区间例如在迷宫中向前走这一离散动作往往对应较高的速度），同理策略选择参数时也会在子区间和离散动作的辅助下进行决策。
