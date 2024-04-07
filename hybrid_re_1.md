We are immensely grateful to you for recognizing the importance and novelty of our approach. Your valuable suggestions inspire us to improve our work further.

If you think the following response addresses your concerns, we would appreciate it if you could kindly consider raising the score.

### Weakness
**W1: The major weakness is perhaps the tasks used to evaluate the effectiveness of the proposed methods are not challenging. Checking how the proposed method behave on a more complicated environment would make the proposal much more convincing.**

Thanks for your valuable questions! We will add validation on complex scenarios in the new version. We use minigrid to construct three more difficult scenarios that share the same task logic: The agent must take the key to open the door and go to the goal point. The reason why these task is difficult is that even in the setting of a single discrete space, the mainstream DRL algorithms such as TD3 and PPO. cannot completely solve these tasks due to the difficulty of exploration. These three scenes have the same targets, but different map sizes and boundary ranges.

**Environment Introduction:** Scenario A is a small map (square boundary), scenario B (square boundary) and scenario C (rectangle boundary) are big maps. The observation space of each scenario is the same, but the scene scale, boundary shape, and obstacle placement are different. [See anonymous link for visualization of the tasks](https://anonymous.4open.science/r/ICML_reviewer_3-0622/README.md). 

**Action space:** Discretization direction: left, right, forward, back; Continuous action: Speed, the number of squares to walk in the corresponding direction (1 to 4). The continuous action space is [0.5,4.5], and the environment will discretize the continuous actions to ensure that they are in integer units: [0.5,1.5) is one step, [1.5,2.5) is two, and so on.

We evaluate all methods in the above three scenarios according to the experimental settings in this paper. The results in Table 1 show that our method outperforms all baselines in three complex scenarios, which further proves the effectiveness of our method. The poor performance of the current SOTA method, i.e. HyAR-TD3, indicates the inefficiency of its exploration ability.

Table 1. Win rate (Average of 10 runs):
| Method      | Scenario A (%)| Scenario B (%)| Scenario C (%)|
| :-----------: | :-----------: | :------------: | :-----------: |
| HyAR-TD3|$40$|$20$|$30$|
| HPPO |$30$|$0$|$30$|
| PDQN |$10$ |$0$|$10$|
| HHQN |$30$ |$10$|$20$|
| **Ours** |$60$ |$40$|$50$|

### Detailed Comments
**C1: There is an ablation experiment on the effect of segmenting subregions of continuous actions. However, it does not tell how the hierarchy of discrete-subregion-continuous is determined. I think choosing the hierarchy does matter for the performance of the method. Do you have to manually figure out the hierarchy case by case, or how do you determine the hierarchy if it is not naturally given in the problem?**

Thanks for your insightful questions, we will improve the presentation of this part in the new version. our hierarchical structure is task-agnostic. We only need to specify the number c of subregions in the continuous action space. For any task, the hierarchical action architecture can be designed as follow: the first level is all discrete actions, the second level is continuous action subregions corresponding to each discrete action and the third level is continuous action in subregion. Notably, to achieve task-agnostic, we use the most direct sequential splitting to partition the continuous action space, that is, the user only needs to set the desired number of subspaces, and each continuous action space is uniformly split into c portions.
