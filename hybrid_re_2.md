We are deeply grateful for your recognition of our paper's motivation, performance, and potential academic impact. Your positive feedback is highly encouraging. Such feedback motivates us to further our research in this promising field. Thank you for your thorough and constructive review.
### Weakness
**W1: It would be nice if the authors can spend more space explaining how the Q values are propagated in Section 3.1 (4). How is $V_{new}$ computed? Are Q values also propagated to the previous time step like other MCTS approaches? How are $`\mathcal{X}_{kw}`$ and $`\mathcal{Q}_{kw}`$ used (are they paired or just two sets)? How does the algorithm handle stochastic dynamics?**

Thanks for your suggestion, we will cover this part in more detail in the new version.

- How is $V_{new}$ computed?: Following the Q-value calculation process in the mainstream DRL algorithm [2], we take the current state, the selected discrete action, and the selected continuous subregion as the input of the Q-value network and output the state-action Q-value as $V_{new}$. The reason why we choose Q value as $V_{new}$ is that the numerical value of Q value directly represents the reward expectation of the selected hybrid action at the current state, which can be used as a measure of the nodes in MCTS. With the increased training steps, the Q network will be continuously optimized to achieve a stable state-action evaluation, which helps MCTS eliminate the suboptimal policy caused by the inaccurate evaluation of node value in stochastic dynamic scenarios.
- Are Q values also propagated to the previous time step like other MCTS approaches?: Different from other works that directly use MCTS for sequential path planning, we only use MCTS to make decisions in the hierarchical action space according to the current state and do not need to consider the historical state information, so we do not need to propagate to the previous time step.
- How are $`\mathbb{X}_{kw}`$ and $`\mathbb{Q}_{kw}`$ used: These two sets are paired, and each subregion maintains two such sets. After a complete sampling, the selected continuous action will be added to the set of candidate actions $`\mathbb{X}_{kw}`$, at the same time, the Q value corresponding to the sampled action is added to $`\mathcal{Q}_{kw}`$. Finally, we select the best parameter action previously sampled in subregion $\hat{w}$ under discrete action $\hat{k}$ as the final parameter action, i.e., $`\hat{x}_{\hat{k},\hat{w}}=\arg\max_{x\in\mathbb{X}_{\hat{k},\hat{w}}}{\mathbb{Q}_{\hat{k},\hat{w}}}(x)`$
- How does the algorithm handle stochastic dynamics?: Your question is very in-depth. (1)We use the Q-value as the node value in MCTS. Compared with other traditional node value optimization methods, using Q values has an advantage: Q values can be continuously optimized. As the number of training steps and sampling increases, the critic is optimized by TD-learning according to mainstream DRL theory. This ensures that the nodes of MCTS are still effectively evaluated in stochastic dynamic scenarios. (2) Change the exploration mode of MCTS. We used the traditional discounted UCB in the main experiment. When faced with random dynamic scenarios, we can replace traditional UCB with UCB-V [2], which is more effective in stochastic dynamics scenarios. The main idea of UCB-V is to add variability estimation term based on UCB to improve the modeling ability of uncertain environments.

**W2: It seems like the discussions in the paper are mostly focused on having only one discrete component and one continuous component in the action space. It would be nice if the authors can discuss how the algorithms can be adapted to handle scenarios with multiple discrete components and multiple continuous components.**

Thanks for your suggestion, this section has been supplemented in the new version. The previous version mainly discussed the method for the mainstream hybrid action space setting,i.e. "one discrete component and one continuous component", but our method can also be extended to more complex setting:
- Our approach can be directly extended to the case where each discrete component have multiple continuous components, that is, perform uniform splitting for each continuous component and maintain an independent MCTS policy introduced in our paper, each policy selects a parameter within the respective continuous action space based on the current state.
- When the continuous components corresponding to different discrete actions are different. Our method first sets the output dimension according to the maximum number of continuous components to realize the unified decision of all discrete actions and the corresponding continuous components. That is, when the maximum number of continuous parameters is 4, we set the output dimension of the continuous action policy to be 4. When the selected discrete action corresponds to two continuous actions, we default to only use the first two output dimensions to interact with the environment.  In addition, we truncate the gradient of the output excess dimension of the continuous action to prevent the noise gradient from affecting the policy training.

We construct a complex mixed action space task using the Goal scenario to verify the effectiveness of our approach for diverse hybrid action settings. The agent needs to get past the opposing defender and kick the ball into the goal. Discrete actions correspond to a diverse number of continuous actions.

*Action space (discrete action-continuous action) :* Shot-force (p), move -coordinate (x,y), dribble -no continuous component. 

Table 1 shows that our method outperforms other methods in this task. It indicates that our above techniques can indeed enable the extension of our approach to more complex hybrid action space control.

Table 1. Performance (Average of 5 runs):
| Method| Goal with complex action space|
| :-: | :-: |
| **Ours**|$0.51\pm 0.14$|
| HyAR-TD3|$0.36\pm 0.08$|
| HPPO|$0.06\pm 0.11$|
| PDQN|$0.17\pm 0.09$|
| HHQN|$0.14\pm 0.18$|

### Detailed Comments
**C1: What would happen if a subregion contains mostly bad actions but also some good actions (for example, in Figure 1C, the average Q for the third subregion is actually the lowest). Would MCTS consider this subregion as bad subregion (with low $Q_r$)?**

Your question is insightful, and this is a problem we are trying to solve. We use a simple technique to mitigate this issue and will emphasize this part in the new version. As shown in the right part of Figure 2 in the submission, we use $max(Q_{ki})$ to assign values to subregions, that is, the highest Q value of each region represents the good or bad of its corresponding region, which can guide the policy to choose the region where the optimal continuous parameter lies without being affected by the average Q value.

We compare "using maximum Q-value" (ours) with "using average Q-value" (basline )in three tasks. The results in Table 2 show that "using maximum Q-value" is significantly better, which proves that our method can promote the exploration of policies and find the optimal continuous parameters.

Table 2. Performance (Average of 5 runs):
| Method      | Chase | Move| Hard Goal|
| :-: | :-: | :-: | :-: |
| Method 1|$0.81\pm 0.08$|$0.97\pm 0.04$|$0.58\pm 0.13$|
| Method 2|$0.72\pm 0.05$|$0.95\pm 0.08$|$0.56\pm 0.17$|

**C2: In Section 3.1, "continuous parameter space $X_k$ marked with rounded rectangles in light origin" -> "continuous parameter space $X_k$ marked with rounded rectangles in light orange"?**

we correct this in the new version.

**C3: In Equation (6), is $\mathcal{Q_{\hat{k},\hat{w}}}$ same as $\mathcal{Q_{\hat{k}\hat{w}}}$?**

Thanks for your careful review, these two are the same. We will correct this in the new version.

**C4: Why are the results of other algorithms not presented for Chase and Hard Goal in Figure 5?**

There are two reasons for this：
- To highlight the obvious advantages of our method compared with the current hybrid action space SOTA algorithm HyAR-TD3[1] in complex tasks.
- Refer to our Table 1 which already contains the scores of all methods in all scenarios. The results in the last two rows show that the other baselines basically fail to learn an effective policy on these two complex scenarios (fluctuating around the 0 score), so we do not show the curves for the other baselines on the right of Figure 7.

**C5: Why are the curves noisier (with more spiky intervals) in the ablation studies (Figure 7) compared to Figure 5? What is the environment for Figure 7?**

Thanks for pointing out our vague statement. We will correct it in the new version. We use Hard Move introduced in Appendix B. The curve in Figure 7 is jittered because it is the mean-variance curve of five random seeds. In the main experiment, ten random seeds were used.
**C6: It may be good to describe the MLP architecture in more detail in Section 5.2 or in the appendices.**

Thanks for your valuable suggestions, we will introduce the MLP architecture in detail in the form of language + figure in the new version of the appendix. The detailed architecture of MLP is (1) We first provide a fully connected network [input_dim,64] for each input (state, discrete action, subinterval) as the feature extraction layer. Then the three vectors are concatenated by the last dimension and fed into 6 fully connected layers (with relu activation function) instead of Transformers of equal depth (corresponding to the double-layer transformer we used in the paper).
