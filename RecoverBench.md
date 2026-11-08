## Reviewer idAb [Weak accept]

Thanks for your thoughtful comments on our work. We hope the following replies could address your concerns.

**W1:** The evaluation is narrow relative to the deployment claims. The 416 cases derive from only 104 independent tasks, with original trajectories averaging approximately 13.89 steps. The evaluation primarily uses one executor model, injected rollback failures, and a known recovery step.

**A1:** Thanks for your constructive feedback regarding our evaluation scope and experimental design. We hope the following replies could address your concerns:

**RecoverMemBench provides a sufficient testbed while balancing peer benchmark scales and evaluation costs.** Spanning 1,445 execution steps across 10 repositories, our 312 resumption (at 25%, 50%, and 75% steps) and 104 rollback cases sufficiently expose consistent bottlenecks across 11 memory systems, matching the scale of mainstream benchmarks like LoCoMo and LongMemEval. Moreover, constructing these dataset alone cost over $1,800 USD, making this scale essential for affordable evaluation. 

**Our settings ensure a strictly controlled evaluation that isolates memory recovery capabilities from confounding factors.** By building our dataset from tasks solved three consecutive times without interruption and fixing the executor, injected errors, and recovery steps, we prevent external variables (e.g., base model limits, agent framework design, or random error types) from distorting the evaluation of task recovery.

**Beyond the static dataset, a core contribution of our work is a generalizable data construction pipeline.** As the first systematic study of agent task recovery, we provide not only a ready to use benchmark, but also a reproducible pipeline for interruption, error injection, and state restoration that readily extends to other models and failure scenarios.


2. 强调我们的这个样本量已经足够，其他的方法也是这个数量。 此外，评估成本是很大的， 构建这个数据集我们就花了1800usd，所以评估起来的成本也很大。
3.   使用这些setting是为了提供一个严谨controllable的评测 -》隔离base model的能力，failures噪声对评测结果的影响，让我们的分析更reliable -》以GPT 4o所有成功样本的base数据为例，使用成功样本作为base数据，能够隔离base模型的强弱对最终评测的影响。
4.  **核心不仅仅在于数据，同时也有构造数据的pipeline。** 强调我们的工作不仅仅提供了一个可以直接评估的benchmark，也提出了一个严谨的数据构造pipeline以评估Agent Memory在task recovery ability上面能力。
5.   强调贡献： 我们提出一个benchmark来评测了一个过去bench都忽略的一个问题： task recovery abilities

-------

**W2:** The comparison with version-aware baselines materially qualifies the method advantage.

**A2:** We thank the reviewer for this insightful comment regarding the version aware baselines in Appendix C.1. We clarify how these results support our core contributions and highlight the practical advantages of our method below: 

**RecoverMemBench is the first work to systematically investigate the task recovery challenge and diagnose the fundamental bottlenecks of existing memory systems.** Prior agent memory benchmarks overlook interrupted task recovery, which is a critical capability for real world deployment. Our primary contribution lies in establishing this evaluation paradigm and uncovering two root causes of recovery failure: unreliable semantic retrieval and state mismatch caused by overwrite updates. In fact, we designed the version-aware baseline experiment in Appendix C.1 specifically to validate this diagnostic finding. The performance gains achieved by equipping baselines with stepwise snapshots directly prove our findings that synchronizing memory state with the restored environment is essential for task rollback.

**Version-aware baselines incur prohibitive storage costs that exceed RecoverMem by 19.57 to 307.75 times, rendering them impractical for real-world scenarios.** Since existing memory systems are not natively designed for state rollback, preserving historical states requires duplicating their full memory banks at every step. As shown in Table 4 (summarized below), maintaining recovery effectiveness via version preservation requires between 36.00 MiB and 566.26 MiB of storage. In contrast, RecoverMem achieves the highest recovery rate (43.27%) using only 1.84 MiB by recording compact structured state updates at each step. Furthermore, naive snapshot preservation fails to resolve the semantic retrieval bottleneck. We believe that exposing this steep trade-off between state consistency and storage efficiency will motivate future research on lightweight, recovery aware agent memory.


| Method | RR (%) | AS (%) | Storage (MiB) | Storage Overhead vs. Ours |
| :--- | :---: | :---: | :---: | :---: |
| A-MEM (Version-aware) | 43.27 | 70.56 | 36.00 | 19.57X |
| Mem0 (Version-aware) | 42.31 | 62.16 | 203.66 | 110.68X |
| MemGPT (Version-aware) | 41.34 | 66.33 | 70.53 | 38.33X |
| MemoryOS (Version-aware) | 39.42 | 112.78 | 566.26 | 307.75X |
| **RecoverMem (Ours)** | **43.27** | **89.35** | **1.84** | **1.00X** |

-------

**Q1:** How would RecoverMem identify valid state when there is no reliable global rollback step?

**A3:** Thanks for your question regarding our method design.

Our rollback setting aligns with practical deployment workflows, where restoring the environment inherently specifies the rollback step $k$. In real-world scenarios, environment rollback is a targeted operation: people inspects the failed trajectory, locates the erroneous step, and reverts the workspace to the last valid state s_k. Because executing this environment restoration already identifies step $k$, RecoverMem directly uses $k$ to retrieve the exact task state $m_k$ without requiring extra assumptions. 

强调我们的bench里面 task rollback setting就是需要先知道global rollback step； 这个也符合现在practical的场景：human先分析为什么agent失败 -》 找到原因及问题step -》 直接rollback 【主要强调setting符合practial场景】

-------

**Q2:** How can the system retain knowledge from a failed suffix without reintroducing invalid progress?


**A4:** Thanks for your question regarding our experience retention mechanism.

RecoverMem prevent valid progress from failed explorations by anchoring the task state strictly to $m_k$ while selectively distilling the failed suffix $\{m_{k+1}, \dots, m_T\}$ into a failure experience module. Given the rollback step $k$, RecoverMem retrieves the exact snapshot $m_k$ so that the established progress and remaining subgoals strictly reflect the restored environment $s_k$, completely discarding any invalid state updates recorded after step $k$. Meanwhile, our Reflection mechanism summarizes the flawed trajectory from $m_{k+1}$ to $m_T$ and appends only the failure lessons into the dedicated failed field of $m_k$ to get $\tilde{m}_k$. This way allows the agent to retain knowledge of prior exploration faults without mistaking undone actions for completed work.

-------

## Reviewer KFmi [Clear Rejection]

**W1:** The problem setting is not well motivated: Practical systems persist session logs and checkpoint the environment and memory together; providing the full previous trajectory, which fits in context here, is not compared.

**A1:** 
1. 给出full history时候的表现 -》 分析不好的原因 【full history的noisy 太大，并且上下文窗口有限制】 -》 强调一直保留full context和 checkpoint of environment and memory 会到来巨大的额外存储成本，使得不practical 【给出Appendix Table4】

2. baseline里面有方法是直接把 原始context保存下来的 【MemGPT】，如果直接存储checkpoint的话，会带来70x的存储成本

3. 强调核心贡献 

----

**W2:** The rollback mismatch exists only because the environment is rolled back but the memory is not; RecoverMem's step-indexed retrieval then amounts to a checkpoint.

**A2:**  这个说法正是我们的工作highlight的地方【贡献】-》但是过去的方法都忽略了这个问题 -》此外， 虽然能直接存储checkpoint但是额外的成本很难控制 导致方法并不practical 给出Appendix Table4】

----

**W3:** The benchmark does not support the claims: 104 tasks that GPT-4o solves reliably, with a median of 11 steps, one agent, one backbone, and one run per setting, far from the hundreds of actions in the motivation. Interruption points are set from the known length of successful runs, and rollback errors are injected by an LLM rather than made by the agent.

**A3:**
1. 强调贡献： 我们提出一个benchmark来评测了一个过去bench都忽略的一个问题： task recovery abilities
2. 强调我们的这个样本量已经足够，其他的方法也是这个数量。 此外，评估成本是很大的， 构建这个数据集我们就花了1800usd，所以评估起来的成本也很大。
3.   使用这些setting是为了提供一个严谨controllable的评测 -》隔离base model的能力，failures噪声对评测结果的影响，让我们的分析更reliable -》以GPT 4o所有成功样本的base数据为例，使用成功样本作为base数据，能够隔离base模型的强弱对最终评测的影响。
4.  **核心不仅仅在于数据，同时也有构造数据的pipeline。** 强调我们的工作不仅仅提供了一个可以直接评估的benchmark，也提出了一个严谨的数据构造pipeline以评估Agent Memory在task recovery ability上面能力。

------

**W4:** The evaluation of RecoverMem is not convincing: It receives the git diff (Figure 7) and the rollback step index, which the baselines do not; with step-wise snapshots, baselines reach similar recovery rates (Appendix C.1).

**A4:**
1. 强调bench是核心贡献 -》 分析出过去方法在memory存储和召回上的不足，所以找到了方法
2. 强调比较是公平的， 我们完全follow他们原始的论文里面的setting。并且在Appendix里面Table 4也展示了，即使他们能用stepwise snapshot之后，他们引来的额外的成本是巨大的。

----

**W5:** It is tailored to SWE tasks and evaluated only on the authors' own benchmark, not on existing memory benchmarks.

**A5:**  强调bench是核心贡献 -> 现有的memory bench把这个问题给忽略， 我们是第一个系统性地研究这个问题 -》 其他的benchmark无法研究这个问题

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
2. snapshot-adapted它引来的额外的成本是巨大的。这使得这些方法每一步都存储起来， 就变得unpractical

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
