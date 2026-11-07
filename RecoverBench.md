## Reviewer idAb [Weak accept]

**W1:** The evaluation is narrow relative to the deployment claims. The 416 cases derive from only 104 independent tasks, with original trajectories averaging approximately 13.89 steps. The evaluation primarily uses one executor model, injected rollback failures, and a known recovery step.

**A1:** We appreciate your thoughtful comments on our work. We hope the following responses address your concerns.

1. 强调贡献： 我们提出一个benchmark来评测了一个过去bench都忽略的一个问题： task recovery abilities
2.  **416 cases derive from only 104 independent tasks & averaging approximately 13.89 steps.** ：太大的样本量会极具增加Agent的评测成本，因为每个样本agent都需要多次LLM的推理 -》 选取这个数字是出于评估成本的考虑
3.  **uses one executor model, injected rollback failures, and a known recovery step.**： 使用这些setting是为了提供一个严谨controllable的评测 -》隔离base model的能力，failures噪声对评测结果的影响，让我们的分析更reliable -》以GPT 4o所有成功样本的base数据为例，使用成功样本作为base数据，能够隔离base模型的强弱对最终评测的影响。
4.  **核心不仅仅在于数据，同时也有构造数据的pipeline。** 强调我们的工作不仅仅提供了一个可以直接评估的benchmark，也提出了一个严谨的数据构造pipeline以评估Agent Memory在task recovery ability上面能力。

**W2:** The comparison with version-aware baselines materially qualifies the method advantage.

**A2:** Thanks for your insightful views for our work. 

**RecoverMemBench uncovers the piror work's drawbacks on task recovery ability.** 
强调benchmark是核心贡献 -》 基于benchmark分析出过去方法在task recovery上的不足-》基于分析我们提出一个直接的方法，RecoverMem

**Although version-aware baselines are capable of RecoverMem, they incur substantial storage requirements.** 尽管version-aware baselines和我们的方法能力相当，但是他们需要几十倍额外的存储空间【Appendix Table 4】。这使得他们的方法unpractial。


**Q1:** How would RecoverMem identify valid state when there is no reliable global rollback step?

**A3:** 强调我们的bench里面 task rollback setting就是需要先知道global rollback step； 这个也符合现在practical的场景：human先分析为什么agent失败 -》 找到原因及问题step -》 直接rollback 【主要强调setting符合practial场景】


**Q2:** How can the system retain knowledge from a failed suffix without reintroducing invalid progress?
解释方法的机制

**A4:** 当需要rollback的时候， 比如rollback到 k step， RecoverMem会reflect k+1 到 final step T的记忆来总结出错误的exploration，从而避免reintroducing invalid progress。


