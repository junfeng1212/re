## Reviewer idAb [Weak accept]

**W1:** The evaluation is narrow relative to the deployment claims. The 416 cases derive from only 104 independent tasks, with original trajectories averaging approximately 13.89 steps. The evaluation primarily uses one executor model, injected rollback failures, and a known recovery step.

**A1:** We appreciate your thoughtful comments on our work. We hope the following responses address your concerns.

1. 强调贡献： 我们提出一个benchmark来评测了一个过去bench都忽略的一个问题： task recovery abilities
2.  **416 cases derive from only 104 independent tasks & averaging approximately 13.89 steps.** ：太大的样本量会极具增加Agent的评测成本，因为每个样本agent都需要多次LLM的推理 -》 选取这个数字是出于评估成本的考虑
3.  **uses one executor model, injected rollback failures, and a known recovery step.**： 使用这些setting是为了提供一个严谨controllable的评测 -》隔离base model的能力，failures噪声对评测结果的影响，让我们的分析更reliable -》以GPT 4o所有成功样本的base数据为例，使用成功样本作为base数据，能够隔离base模型的强弱对最终评测的影响。
4.  **核心不仅仅在于数据，同时也有构造数据的pipeline。** 强调我们的工作不仅仅提供了一个可以直接评估的benchmark，也提出了一个严谨的数据构造pipeline以评估Agent Memory在task recovery ability上面能力。

-------

**W2:** The comparison with version-aware baselines materially qualifies the method advantage.

**A2:** Thanks for your insightful views for our work. 

**RecoverMemBench uncovers the piror work's drawbacks on task recovery ability.** 
强调benchmark是核心贡献 -》 基于benchmark分析出过去方法在task recovery上的不足-》基于分析我们提出一个直接的方法，RecoverMem

**Although version-aware baselines are capable of RecoverMem, they incur substantial storage requirements.** 尽管version-aware baselines和我们的方法能力相当，但是他们需要几十倍额外的存储空间【Appendix Table 4】。这使得他们的方法unpractial。

-------

**Q1:** How would RecoverMem identify valid state when there is no reliable global rollback step?

**A3:** 强调我们的bench里面 task rollback setting就是需要先知道global rollback step； 这个也符合现在practical的场景：human先分析为什么agent失败 -》 找到原因及问题step -》 直接rollback 【主要强调setting符合practial场景】

-------

**Q2:** How can the system retain knowledge from a failed suffix without reintroducing invalid progress?
解释方法的机制

**A4:** 当需要rollback的时候， 比如rollback到 k step， RecoverMem会reflect k+1 到 final step T的记忆来总结出错误的exploration，从而避免reintroducing invalid progress。

-------

## Reviewer KFmi [Clear Rejection]

**W1:** The problem setting is not well motivated: Practical systems persist session logs and checkpoint the environment and memory together; providing the full previous trajectory, which fits in context here, is not compared.

**A1:** 
1. 给出full history时候的表现 -》 分析不好的原因 【full history的noisy 太大，并且上下文窗口有限制】 -》 强调一直保留full context和 checkpoint of environment and memory 会到来巨大的额外存储成本，使得不practical 【给出Appendix Table4】

2. baseline里面有方法是直接把 原始context保存下来的 【MemGPT】，如果直接存储checkpoint的话，会带来70x的存储成本

3. 强调核心贡献 

----

**W2:** The rollback mismatch exists only because the environment is rolled back but the memory is not; RecoverMem's step-indexed retrieval then amounts to a checkpoint.

**A2:** 认可这个说法 -》 但是过去的方法都忽略了这个问题 -》 虽然能直接存储checkpoint但是额外的成本很难控制 导致方法并不practical 给出Appendix Table4】

强调 benchmark是核心 -》 过去的方法忽视了这个问题 -》 benchmark通过设计practical的setting找出问题所在

----

**W3:** The benchmark does not support the claims: 104 tasks that GPT-4o solves reliably, with a median of 11 steps, one agent, one backbone, and one run per setting, far from the hundreds of actions in the motivation. Interruption points are set from the known length of successful runs, and rollback errors are injected by an LLM rather than made by the agent.

**A3:**
1. 强调贡献： 我们提出一个benchmark来评测了一个过去bench都忽略的一个问题： task recovery abilities
2.  **416 cases derive from only 104 independent tasks & averaging approximately 13.89 steps.** ：太大的样本量会极具增加Agent的评测成本，因为每个样本agent都需要多次LLM的推理 -》 选取这个数字是出于评估成本的考虑
3.  **uses one executor model, injected rollback failures, and a known recovery step.**： 使用这些setting是为了提供一个严谨controllable的评测 -》隔离base model的能力，failures噪声对评测结果的影响，让我们的分析更reliable -》以GPT 4o所有成功样本的base数据为例，使用成功样本作为base数据，能够隔离base模型的强弱对最终评测的影响。
4.  **核心不仅仅在于数据，同时也有构造数据的pipeline。** 强调我们的工作不仅仅提供了一个可以直接评估的benchmark，也提出了一个严谨的数据构造pipeline以评估Agent Memory在task recovery ability上面能力。
5.  我么会在limitation里面讨论这一点


------

**W4:** The evaluation of RecoverMem is not convincing: It receives the git diff (Figure 7) and the rollback step index, which the baselines do not; with step-wise snapshots, baselines reach similar recovery rates (Appendix C.1).

**A4:**
1. 强调bench是核心贡献 -》 分析出过去方法在memory存储和召回上的不足，所以找到了方法
2. 强调比较是公平的， 我们完全follow他们原始的论文里面的setting。并且在Appendix里面Table 4也展示了，即使他们能用stepwise snapshot之后，他们引来的额外的成本是巨大的。

----

**W5:** It is tailored to SWE tasks and evaluated only on the authors' own benchmark, not on existing memory benchmarks.

**A5:**  强调bench是核心贡献 -> 现有的memory bench把这个问题给忽略了 他们很难评估

---

**W6:** The failure analysis is weakly supported: the "full memory bank" test uses the evaluation question as the retrieval query, and the questions match RecoverMem's record fields. The link between answer correctness and recovery success is not shown.

**A6:**
强调这里面的evaluation question它是关于任务被打断时候environment里面的状态的，而不是整个task query。这些question完全是用我们恢复所需要的信息来生成的。所以它如果方法能够把这些信息给召回的话，它就能够支持task recovery。

我们在Appendices里面提供了几个生成的question的examples，可以看出question完全是task resumption需要的信息

我们这一部分的分析能够揭示现在的方法，它把这些信息存下来了，但是没有办法把它很好地召回，所以在Task resumption里面的表现不佳。

----

**Q1:** Without memory, resuming at 25% is close to restarting from scratch, yet the recovery rate is only about 22%, although the agent solves all these tasks in three runs. What explains this, and what prompt does the recovered agent receive?

**A7:** 这个是因为在25%的Stop的时候，很多队Agent已经对环境做出了改变。

但是我们没有memory的情况下，我们只给了它任务的描述[完全follow swebench的默认设置]，导致它收到任务信息和外部环境是完全不一样的。

从而使它出现了很多Step被用完但没把问题解决的情况，这也是它为什么使用了额外180%的steps；

我们会在论文里面额外增加补充解释，以使我们的文章更加清晰。

**Q2:** Additional steps include failed runs and exclude the cost of memory construction and reflection. The "over 35%" gain in the abstract is relative to the average baseline, not the strongest one.

**A8:** 因为我们这个指标只反映他完成任务带来的额外成本。构建Memory和使用Token的成本，我们是单独汇报；

我们会改进abstract里的描述，使这个更加清晰准确。

-----

## Reviewer yXDZ (Weak Rejection)

**W1:** The added value over straightforward recovery approaches is not sufficiently established. The introduction argues that full interaction history is limited by context length and noise, but the paper does not directly evaluate this alternative or report the original history lengths in tokens. Since the recovery point (k) is known, rollback could use the history up to (k), or the full history with subsequent actions explicitly marked as undone. These comparisons are important for determining whether the proposed memory mechanism provides a substantive benefit.

**A1:**
1. 给出full history时候的表现 -》 分析不好的原因 【full history的noisy 太大，并且上下文窗口有限制】 -》 强调一直保留full context和 checkpoint of environment and memory 会到来巨大的额外存储成本，使得不practical 【给出Appendix Table4】

2. baseline里面有方法是直接把 原始context保存下来的 【MemGPT】，如果直接存储checkpoint的话，会带来70x的存储成本

3. 强调贡献

------

**W2:** The snapshot comparison also requires further analysis. In Appendix C.1, Table 4, snapshot-adapted A-MEM matches RecoverMem’s rollback recovery rate of 43.27% and incurs fewer additional steps: 70.56% versus 89.35%. RecoverMem does use less storage, but an advantage in rollback success or execution overhead over snapshot-adapted A-MEM is not evident.

**A2:** 
1. 强调benchmark是核心的贡献，过去的方法忽视了这个, 基于我们benchmark的分析，我们才知道解决这个问题方案的。
2. 说明虽然过去的方法用了snapshot-adapted, 它会带来相近的结果。但是它引来的额外的成本是巨大的。这使得这些方法每一步都存储起来， 就变得unpractical

**W3:** It is unclear how the gains depend on the underlying model.
Task recovery is evaluated with GPT-4o as the executor and GPT-4o-mini for memory construction (Section 3). It therefore remains unclear how much additional value RecoverMem provides when the executor is stronger or differs in its ability to use long histories. Comparisons across capability levels and within-family model sizes would help establish whether the gains are robust or mainly specific to the current model configuration.

**A3:**
1. 强调为了保证一个严谨的评估。我们固定了base model和保证所有的样本都是Base model在 uninterrupted情况下可以解决的。
2. 这样才能反应，在Task resumption和Task rollback的情况下， Memory system对于的Task recovery capability怎么样？不会受到base model能力的影响

但是为了打消你的顾虑，我们还在用新的base model进行了实验【从sample里面挑一些qwen，不同size的模型，在uninterrupted情况下可以解决的跑】

-------

**Q1:** Additional evidence on full-history recovery.
Could you directly restore the complete interaction history for task resumption, and test both history up to (k) and full history with undone actions explicitly marked for rollback? Please report the history-length distribution and the trade-off between recovery success and cost, including results on cases where the complete history fits within the context window.

**A4:** 同W1

-----

**Q2:** Additional evidence on model dependence.
Could you evaluate a small set of executors with different capabilities while keeping the memory construction model fixed—for example, different sizes within one Qwen family, together with a stronger Qwen, DeepSeek, or other model? The key comparison is RecoverMem’s gain over full-history and snapshot baselines under each executor, with that model’s uninterrupted task performance also reported.

**A5:** 同W3

----

**Q3:** Clarification of the rollback advantage.
Given Table 4, is RecoverMem’s advantage over the stronger snapshot-adapted baselines primarily lower storage consumption? Please clarify the rollback performance and cost claims accordingly, distinguishing storage and execution steps from total cost. Appendix C.2 already notes that the reported measurements do not establish total cost.

**A5:** 同W2
