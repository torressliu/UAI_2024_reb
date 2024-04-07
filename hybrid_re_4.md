We are immensely grateful to you for recognizing the importance and novelty of our approach. Your valuable suggestions inspire us to improve our work further. Based on your feedback, we analyzed our work and added extended experiments to the new version. Besides, the writing is optimized in the new version.
### Detailed Comments 
**C1: The necessity and the reason why using transformer? Of course, transformer is always a fashion choice in DL, but I will still wonder if there is a better choice.**

We will highlight this part in the new version. Since there naturally exists hierarchical dependencies during the action selection process, i.e., state$\to$discrete action$\to$discrete region$\to$continuous action, we use a Transformer-style sequence model to model such dependencies and generate their Q-values (as the Transformer is suitable for modeling sequence dependence). We regard the embedded states, actions, regions, etc, as input tokens (like words in NLP). Besides, Qi,et al.[1] demonstrate that transformer has a better ability to capture relevance than MLP.

**C2: For you to split the control process into 3 levels, especially the sub-area choice level makes me confused even though you have proved it through ablation experiments. I would wonder about the insight behind this. Or could this problem be solved by other optimization methods like bandit in continuous space?**

Your advice is valuable for our research. We will improve this part and add additional experiments in the new version.
- Since previous work has shown that effective policies for hybrid action control need to learn dependencies between discrete action and continuous action [2], we design the hierarchical action structure to explicitly model the correlations between discrete actions and continuous actions.
- Such a hierarchical decision is a flow from coarse-grained (discrete actions) to fine-grained (continuous action sub-area). That is, in this hierarchical architecture, when the algorithm finds the optimal subinterval, it only needs to repeat sampling in this interval without going back to the upper level to explore other subintervals, ultimately improving the exploration efficiency.
-  Compared with the straightforward bandit method (i.e. flattening the hybrid action space, partitioning the sub-interval and then selecting the action), our method can simplify the exploration space by using the dependency between discrete actions and continuous actions (that is, directly abandoning the exploration of the continuous action space corresponding to irrelevant discrete actions). 

We compare our method and the Discounted UCB bandit method in three scenarios. We split the same subregions, i.e. 8, for each continuous action space for both methods to ensure a fair comparison. Table 1 shows that the exploration effect of our method is better, indicating that our hierarchical decision can better explore the optimal policy.

Table 1. Win rate (Average of 10 runs):
| Method      | Chase | Move| Hard Goal|
| :-----------: | :-----------: | :------------: | :-----------: |
| Ours|$0.81\pm 0.08$|$0.97\pm 0.04$|$0.58\pm 0.13$|
| Bandit for continous space |$0.73\pm 0.11$|$0.86\pm 0.07$|$0.27\pm 0.13$|


[1]: Han, Qi, et al.``On the connection between local attention and dynamic depth-wise convolution." ICLR (2021)

[2]: Li, Boyan, et al. "Hyar: Addressing discrete-continuous action reinforcement learning via hybrid action representation." ICLR (2022)
