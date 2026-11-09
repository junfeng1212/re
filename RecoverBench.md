## Reviewer idAb [Weak accept]

**W1:** The evaluation is narrow relative to the deployment claims. The 416 cases derive from only 104 independent tasks, with original trajectories averaging approximately 13.89 steps. The evaluation primarily uses one executor model, injected rollback failures, and a known recovery step.

**A1:** Thanks for your constructive feedback regarding our evaluation scope and experimental design. We hope the following replies could address your concerns:

**RecoverMemBench provides a sufficient testbed while balancing peer benchmark scales and evaluation costs.** Spanning 1,445 execution steps across 10 repositories, our 312 resumption (at 25%, 50%, and 75% steps) and 104 rollback cases sufficiently expose consistent bottlenecks across 11 memory systems, matching the scale of mainstream benchmarks like LoCoMo and LongMemEval. Moreover, with the dataset construction alone exceeding $1,800 USD, scaling it up further would make downstream evaluation prohibitively expensive; our current size strikes the ideal balance for affordable yet rigorous evaluation.

**Our settings ensure a strictly controlled evaluation that isolates memory recovery capabilities from confounding factors.** By building our dataset from tasks solved three consecutive times without interruption and fixing the executor, injected errors, and recovery steps, we prevent external variables (e.g., base model limits, agent framework design, or random error types) from distorting the evaluation of task recovery.

**Beyond the static dataset, a core contribution of our work is a generalizable data construction pipeline.** As the first systematic study of agent task recovery, we provide not only a ready to use benchmark, but also a reproducible pipeline for interruption, error injection, and state restoration that readily extends to other models and failure scenarios.

-------

**W2:** The comparison with version-aware baselines materially qualifies the method advantage.

**A2:** We thank the reviewer for this insightful comment regarding the version aware baselines in Appendix C.1. We clarify how these results support our core contributions and highlight the practical advantages of our method below: 

**RecoverMemBench is the first work to systematically investigate the task recovery challenge and diagnose the fundamental bottlenecks of existing memory systems.** Prior agent memory benchmarks overlook interrupted task recovery, which is a critical capability for real world deployment. Our primary contribution lies in establishing this evaluation paradigm and uncovering two root causes of recovery failure: unreliable semantic retrieval and state mismatch caused by overwrite updates. In fact, we designed the version-aware baseline experiment in Appendix C.1 specifically to validate this diagnostic finding. The performance gains achieved by equipping baselines with stepwise snapshots directly prove our findings that synchronizing memory state with the restored environment is essential for task rollback.

**Version-aware baselines incur prohibitive storage costs that exceed RecoverMem by 19.57 to 307.75 times, rendering them impractical for real-world scenarios.** Since existing memory systems are not natively designed for state rollback, preserving historical states requires duplicating their full memory banks at every step. As shown in Table 4 (summarized below), maintaining recovery effectiveness via version preservation requires between 36.00 MiB and 566.26 MiB of storage. In contrast, RecoverMem achieves the highest recovery rate (43.27%) using only 1.84 MiB by recording task state updates at each step. Furthermore, **naive snapshot preservation fails to resolve the semantic retrieval bottleneck.** By simultaneously overcoming prohibitive storage overheads and semantic retrieval bottlenecks, RecoverMem provides a practical benchmark and design paradigm for next-generation lightweight, recovery-aware agent memory.

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

**Our rollback setting aligns with practical deployment workflows, where restoring the environment inherently specifies the rollback step $k$.** In real-world scenarios, environment rollback is a targeted operation: people inspects the failed trajectory, locates the erroneous step, and reverts the workspace to the last valid state s_k. Because executing this environment restoration already identifies step $k$, RecoverMem directly uses $k$ to retrieve the exact task state $m_k$ without requiring extra assumptions. 


-------

**Q2:** How can the system retain knowledge from a failed suffix without reintroducing invalid progress?

**A4:** Thanks for your question regarding our experience retention mechanism.

**RecoverMem prevent valid progress from failed explorations by anchoring the task state strictly to $m_k$ while selectively distilling the failed suffix $\{m_{k+1}, \dots, m_T\}$ into a failure experience module.** Given the rollback step $k$, RecoverMem retrieves the exact snapshot $m_k$ so that the established progress and remaining subgoals strictly reflect the restored environment $s_k$, completely discarding any invalid state updates recorded after step $k$. Meanwhile, our Reflection mechanism summarizes the flawed trajectory from $m_{k+1}$ to $m_T$ and appends only the failure lessons into the dedicated failed field of $m_k$ to get $\tilde{m}_k$. This way allows the agent to retain knowledge of prior exploration faults without mistaking undone actions for completed work.


We hope our responses have addressed your concerns. If you find them helpful, we would appreciate your consideration of increasing your score.

-------
-------


## Reviewer KFmi [Clear Rejection]

**W1:** The problem setting is not well motivated: Practical systems persist session logs and checkpoint the environment and memory together; providing the full previous trajectory, which fits in context here, is not compared.

**A1:** Thank you for the suggestion. We evaluated full history directly, where the fresh agent receives the raw actions and observations up to the recovery step k in its initial context instead of retrieved memory. Table R1 reports recovery performance and Table R2 reports history lengths.

**Table R1.** Full history versus RecoverMem.

| Setting | Resumption@25% RR | Resumption@25% AS | Resumption@50% RR | Resumption@50% AS | Resumption@75% RR | Resumption@75% AS | Rollback RR | Rollback AS |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Full history | 35.57 | 102.13 | 50.96 | 182.94 | 43.29 | 153.57 | 32.69 | 191.02 |
| RecoverMem (Ours) | **49.03** | **57.55** | **70.20** | **83.88** | **87.50** | **26.83** | **43.27** | **89.35** |

**Full history improves over w/o Memory but remains far below RecoverMem.** Rollback RR rises from 25.96% to 32.69%, yet averaged over the three resumption settings, full history reaches 43.27% RR and 146.21% AS, compared with 68.90% and 56.09% for RecoverMem. Under rollback, RecoverMem improves RR by 10.58 points and lowers AS by over 100 points.

**Full history fits our setting poorly, first because of its length and noise.** As the history grows, raw observations obscure critical context, and agents struggle to reliably extract information situated in the middle of long sequences (Liu et al., 2024). Meanwhile, full history often incur the overflow of context window. Consequently, the RR drops from 50.96% at Resumption@50% to 43.29% at Resumption@75%.

**Under rollback, a full history fails to indicate which past explorations were erroneous.** The agent can read the entire history before step k, but the raw record does not mark which edits were erroneous, so the agent may re-enter the same failed path. RecoverMem instead records completed progress, remaining subgoals, and failed attempts with their observed outcomes, and its Reflection module turns the failed suffix into explicit lessons (removing it lowers RR from 43.27% to 38.46%, Table 2).

**It's costable for checkpointing memory system at every step.** Because existing systems are not designed for checkpointing, this duplicates the full memory bank at every step and costs 36.00 to 566.26 MiB, requiring 19.51X - 307.75X storage costs compared with RecoverMem. 

| Method | RR (%) | AS (%) | Storage (MiB) | Storage Overhead vs. Ours |
| :--- | :---: | :---: | :---: | :---: |
| A-MEM (Stepwise) | 43.27 | 70.56 | 36.00 | 19.57X |
| Mem0 (Stepwise) | 42.31 | 62.16 | 203.66 | 110.68X |
| MemGPT (Stepwise) | 41.34 | 66.33 | 70.53 | 38.33X |
| MemoryOS (Stepwise) | 39.42 | 112.78 | 566.26 | 307.75X |
| **RecoverMem (Ours)** | **43.27** | **89.35** | **1.84** | **1.00X** |

Overall, our work firstly proposes a benchmark to investigate task recovery abilities of current memory systems, which is often ignored. And our analysis shows that these methods fail to handle task recovery setting due to their issues in both memory update and memory retrieval. We believe our work can provide new insight for future memory system research.

----

**W2:** The rollback mismatch exists only because the environment is rolled back but the memory is not; RecoverMem's step-indexed retrieval then amounts to a checkpoint.

**A2:**  Thanks for your questions about our proposed method. We hope the following replies can address your concerns:

**That's a scenario that prior memory benchmarks do not evaluate and that RecoverMemBench evaluates systematically.** We agree that the mismatch arises because the environment is rolled back while memory is not, and we adopt this setting by design: memory persists outside the agent process, and the failed suffix contains lessons that prevent repeated exploration, so rolling memory back together with the environment would discard them. The challenge is therefore to retain the full memory while keeping it consistent with the restored environment, a requirement that prior memory benchmarks do not evaluate and that RecoverMemBench examines systematically.

**RecoverMem is not a plain checkpoint**, for three reasons: 
i) Each entry is a structured task state record (completed progress, remaining subgoals, failed attempts) rather than a copy of the memory bank. 
ii) After rollback, it reflects on entries m_{k+1} to m_T to update the experience module, which a checkpoint would discard; removing this Reflection lowers RR from 43.27% to 38.46% and raises AS from 89.35% to 117.62% (Table 2). 
ii) Snapshots alone do not resolve the retrieval bottleneck, since entries from different steps of the same task remain semantically similar (Section 3.2.1).

**We also test the checkpoint view directly by equipping four baselines with stepwise snapshots (Appendix C.1, Table 4).** Because existing systems are not designed for rollback, this duplicates the full memory bank at every step and costs 36.00 to 566.26 MiB, whereas RecoverMem matches the best stepwise baseline (A-MEM, 43.27% RR) with only 1.84 MiB.
Such memory efficiency is critical in real-world scenarios, directly affecting operational speed.

| Method | RR (%) | AS (%) | Storage (MiB) | Storage Overhead vs. Ours |
| :--- | :---: | :---: | :---: | :---: |
| A-MEM (Stepwise) | 43.27 | 70.56 | 36.00 | 19.57X |
| Mem0 (Stepwise) | 42.31 | 62.16 | 203.66 | 110.68X |
| MemGPT (Stepwise) | 41.34 | 66.33 | 70.53 | 38.33X |
| MemoryOS (Stepwise) | 39.42 | 112.78 | 566.26 | 307.75X |
| **RecoverMem (Ours)** | **43.27** | **89.35** | **1.84** | **1.00X** |

----

**W3:** The benchmark does not support the claims: 104 tasks that GPT-4o solves reliably, with a median of 11 steps, one agent, one backbone, and one run per setting, far from the hundreds of actions in the motivation. Interruption points are set from the known length of successful runs, and rollback errors are injected by an LLM rather than made by the agent.

**A3:**
Thanks for your constructive feedback regarding our evaluation scope and experimental design. We hope the following replies could address your concerns:

**RecoverMemBench provides a sufficient testbed while balancing peer benchmark scales and evaluation costs.** Spanning 1,445 execution steps across 10 repositories, our 312 resumption (at 25%, 50%, and 75% steps) and 104 rollback cases sufficiently expose consistent bottlenecks across 11 memory systems, matching the scale of mainstream benchmarks like LongMemEval and GAIA. Moreover, with the dataset construction alone exceeding $1,800 USD, scaling it up further would make downstream evaluation prohibitively expensive; our current size strikes the ideal balance for affordable yet rigorous evaluation.

**Our settings ensure a strictly controlled evaluation that isolates memory recovery capabilities from confounding factors.** By building our dataset from tasks solved three consecutive times without interruption and fixing the executor, injected errors, and recovery steps, we prevent external variables (e.g., base model limits, agent framework design, or random error types) from distorting the evaluation of task recovery.

**Beyond the static dataset, a core contribution of our work is a generalizable data construction pipeline.** As the first systematic study of agent task recovery, we provide not only a ready to use benchmark, but also a reproducible pipeline for interruption, error injection, and state restoration that readily extends to other models and failure scenarios.


------

[‼️]**W4:** The evaluation of RecoverMem is not convincing: It receives the git diff (Figure 7) and the rollback step index, which the baselines do not; with step-wise snapshots, baselines reach similar recovery rates (Appendix C.1).

**A4:** Thanks for your thoughtful views about our evaluation pipeline. We hope the following replies can make our evaluation more clear:

**git diff cannot be accessed for all methods.** In Figure 7, that git diff is not the "git.diff" gile instead of the description in "" 

**Accessing rollback step is the default setting of task rollback.** Our rollback setting aligns with practical deployment workflows, where restoring the environment inherently specifies the rollback step $k$. Because, in real-world scenarios, environment rollback is a targeted operation: people inspects the failed trajectory, locates the erroneous step, and reverts the workspace to the last valid state s_k. However, in the default setting of current memory methods, the retrieving is based on semantic query. So, they are unable to utilize that rollback step index.

**Targeted experiments in Appendix C.1 further substantiate this finding.** Evaluations on our benchmark demonstrate that baselines' overwrite updates frequently trigger state inconsistencies between memory and the environment. While stepwise snapshotting addresses this issue, it fails to address retrieval bottleneck: because all recorded memories are closely related to the task query, semantic differentiation blurs, preventing the agent from retrieving the exact state required for recovery. Furthermore, stepwise baselines need to duplicate their full memory banks at every step,  requiring 19.51X - 307.75X storage costs compared with RecoverMem. That undermines their applicabilities in real-world scenarios as efficiency is one of most important parts of memory system. 

Overall, our work firstly systematically investigate a crucial but ignored capabililty of agent memory, task recovery ability, and provide a solid foundation for future research.


----

**W5:** It is tailored to SWE tasks and evaluated only on the authors' own benchmark, not on existing memory benchmarks.

**A5:**  Thank you for the comment. **Existing memory benchmarks evaluate how retained knowledge benefits later tasks, and they define neither an interruption nor an environment restore step, so they cannot measure task recovery, which is why we build RecoverMemBench instead of reusing them.** We choose SWE-bench because Git lets us restore the environment to any earlier step, which task rollback requires, whereas in domains with irreversible or costly actions such a controlled rollback is hard to construct. We discuss that in our Limitation section (Appendix D), while the two failure modes we identify (semantic retrieval among similar entries and overwrite updates) stem from memory design rather than from coding. Extending RecoverMemBench to other domains with restorable environments is the main direction of our future work.

---

**W6:** The failure analysis is weakly supported: the "full memory bank" test uses the evaluation question as the retrieval query, and the questions match RecoverMem's record fields. The link between answer correctness and recovery success is not shown.

**A6:** Thanks for your comments about our analysis. We hope the following replies can make our work more clear:

**The full memory bank test is a diagnostic applied only to the four baselines (Figure 4), so it cannot favor RecoverMem.** It asks whether the needed state is stored at all, and a state-specific query is the appropriate probe because the task query q is identical at every step and cannot separate entries from different steps (Section 3.2.1). Correct rates rise from 34.50% to 60.40% for A-Mem and from 36.40% to 64.75% for MemoryOS, which shows that the information is stored but not retrieved by q.

**The questions are generated from environment evidence (tool calls, Git snapshots, diffs) by Deepseek that sees no method’s memory, and they are graded by code against the environment (Appendix B.2), so they are fixed independently of any method’s record schema.** They ask about modified files, localization findings, and verification results, which any recovery must know, so the overlap with RecoverMem’s fields reflects that both target task state. All methods receive the same questions and the same answering model.

**Link to recovery success.** Every question is generated from the environment state at the interruption step, namely the modified files, the localization findings, and the verification results (Appendix B.2 and B.3), which is exactly the task state a recovered agent needs to avoid repeating completed work or skipping remaining work. A memory that retrieves this information and answers correctly therefore supplies the state required for recovery, whereas a memory that returns Missing or Wrong answers leaves the agent to rediscover or misjudge that state, consistent with the extra steps and failures in Table 1. Several examples (Appendix B.2) are provided as follows:

> Task State QA Example
> 
> 
> **Example 1**
> * **Question:** Immediately before original step 7, answer about that original pre-injection history and workspace: Which repository-relative source file was explicitly located for Table in the recorded investigation?
> * **Answer:**
> ```json
> {"path": "astropy/io/votable/tree.py"}
> 
> ```
> 
> 
> 
> 
> ---
> 
> 
> **Example 2**
> * **Question:** Immediately before original step 9, answer about that original pre-injection history and workspace: Relative to the initial workspace, which repository-relative files are added, modified, or deleted in the checkpoint workspace? Report net state, not whether a file was ever edited.
> * **Answer:**
> ```json
> {
>   "added": ["test_version_check.py"],
>   "deleted": [],
>   "modified": ["sphinx/extension.py"]
> }
> 
> ```
>

----

**Q1:** Without memory, resuming at 25% is close to restarting from scratch, yet the recovery rate is only about 22%, although the agent solves all these tasks in three runs. What explains this, and what prompt does the recovered agent receive?

**A7:** Thank you for the question. Resumption@25% is not equivalent to restarting from scratch, because recovery starts from the restored environment s_k rather than s_0, and **the first k steps may already have modified the repository or added files such as reproduction scripts (82.69% samples have modified the files in repo when resumpation@25%)**. Without memory, the fresh agent receives only the task description, so it cannot tell which changes are already present or which subtasks are complete, and it may repeat finished work (e.g., re-editing code that is already modified) or build on a wrongly assumed state. Such behavior makes the run unreliable and adds steps, which is reflected in the AS of 182.16% in Table 1. The 22.11% RR therefore does not contradict the three uninterrupted successes: those runs show that the agent can solve each task from a clean state, whereas RecoverMemBench measures whether it can recover the state of a partially modified environment, which base capability alone does not provide.

Regarding the prompt, the w/o Memory agent receives only the task description q, using the default mini-SWE-agent[1] prompt for SWE-bench, with no memory content and no summary of earlier steps. For the other methods, the prompt is identical and the retrieved memory m = Read(M, q) is placed in the initial context (Section 2.1), so memory is the only variable across methods. We will add the full prompt to the appendix.

**Q2:** Additional steps include failed runs and exclude the cost of memory construction and reflection. The "over 35%" gain in the abstract is relative to the average baseline, not the strongest one.

**A8:** 

**Scope of Additional Steps (AS)**. AS measures only the agent’s execution steps for completing the task, relative to the uninterrupted run (Eq. 6), and it is not a measure of total system cost. It counts the steps of failed recovery runs, which is why Section 2.4 requires reading AS together with RR, and it excludes memory construction, reflection, and latency, as stated in Appendix C.2. We will therefore write “additional steps” instead of “additional cost” in the abstract and throughout the paper. To cover the excluded part, Table 5 reports the token and storage footprint of each memory system: RecoverMem uses 1,206.27 tokens and 1.84 MiB per sample, lower than A-Mem, MemGPT, MemoryOS, and MemoryBank.

And we will refine the wording in the abstract to clarify that this value represents the average baseline.

We hope our responses have addressed your concerns. If you find them helpful, we would appreciate your consideration of increasing your score.

-----

## Reviewer yXDZ (Weak Rejection)

**W1:** The added value over straightforward recovery approaches is not sufficiently established. The introduction argues that full interaction history is limited by context length and noise, but the paper does not directly evaluate this alternative or report the original history lengths in tokens. Since the recovery point (k) is known, rollback could use the history up to (k), or the full history with subsequent actions explicitly marked as undone. These comparisons are important for determining whether the proposed memory mechanism provides a substantive benefit.

**A1:** Thank you for the suggestion. We evaluated full history directly, where the fresh agent receives the raw actions and observations up to the recovery step k in its initial context instead of retrieved memory. Table R1 reports recovery performance and Table R2 reports history lengths.

**Table R1.** Full history versus RecoverMem.

| Setting | Resumption@25% RR | Resumption@25% AS | Resumption@50% RR | Resumption@50% AS | Resumption@75% RR | Resumption@75% AS | Rollback RR | Rollback AS |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Full history | 35.57 | 102.13 | 50.96 | 182.94 | 43.29 | 153.57 | 32.69 | 191.02 |
| RecoverMem (Ours) | **49.03** | **57.55** | **70.20** | **83.88** | **87.50** | **26.83** | **43.27** | **89.35** |

**Table R2.** Average history length in tokens.

| Setting | Resumption@25% | Resumption@50% | Resumption@75% | Rollback |
| :--- | :---: | :---: | :---: | :---: |
| Tokens | 2,643 | 4,625 | 6,177 | 4,251 |

**Full history improves over w/o Memory but remains far below RecoverMem.** Rollback RR rises from 25.96% to 32.69%, yet averaged over the three resumption settings, full history reaches 43.27% RR and 146.21% AS, compared with 68.90% and 56.09% for RecoverMem. Under rollback, RecoverMem improves RR by 10.58 points and lowers AS by over 100 points.

**Full history fits our setting poorly, first because of its length and noise.** The average histories at Resumption@50%, Resumption@75%, and rollback (4,625, 6,177, and 4,251 tokens) exceed the 4,096 token memory budget shared by all methods (Appendix B.1), and individual samples are longer than these averages. Even when the history fits, many raw observations bury the information the agent needs, and agents use information in long contexts unreliably, especially when it lies in the middle (Liu et al., 2024). This is consistent with RR falling from 50.96% at 50% to 43.29% at 75%, although more of the task has been completed. 

**Under rollback a full history does not tell the agent which past explorations were wrong.** The agent can read the entire history before step k, but the raw record does not mark which edits were erroneous, so the agent may re-enter the same failed path. RecoverMem instead records completed progress, remaining subgoals, and failed attempts with their observed outcomes, and its Reflection module turns the failed suffix into explicit lessons (removing it lowers RR from 43.27% to 38.46%, Table 2).

**Checkpointing existing memory systems at every step is not a practical alternative either, because of storage.** Table 4 shows that this costs 36.00 to 566.26 MiB, compared with 1.84 MiB for RecoverMem. This includes MemGPT, which retains the original history: its version-aware variant requires 70.53 MiB, which is 38.33 times the storage of RecoverMem, yet reaches only 41.34% RR under rollback.

Overall, our work propose RecoverMemBench, which firstly investigate the important but underexplored ability of agent memory in task recovery setting. Our analysis demonstrates that current memory system fails to handle task resumption and task rollback because of their drawbacks in both memory update and retrieval, providing solid foundation for future agent memory research.


------

**W2:** The snapshot comparison also requires further analysis. In Appendix C.1, Table 4, snapshot-adapted A-MEM matches RecoverMem’s rollback recovery rate of 43.27% and incurs fewer additional steps: 70.56% versus 89.35%. RecoverMem does use less storage, but an advantage in rollback success or execution overhead over snapshot-adapted A-MEM is not evident.

**A2:** We thank the reviewer for this insightful comment regarding the snapshot-adapted baselines in Appendix C.1. We clarify how these results support our core contributions and highlight the practical advantages of our method below: 

**The snapshot experiment in Appendix C.1 is a controlled test of a diagnosis produced by RecoverMemBench, not a competing baseline.** RecoverMemBench shows that existing systems fail under rollback because overwrite updates keep the records from steps k+1 to T accessible after the environment is restored to s_k (Section 3.2.2). If this diagnosis is correct, equipping the same systems with stepwise snapshots, so that the memory M_k matches s_k, should improve recovery. This is what we observe: rollback RR improves for three of the four baselines (e.g., Mem0 from 26.92% to 42.31%) and AS decreases for all four (e.g., Mem0 from 179.42% to 62.16%), which supports our finding that keeping memory consistent with the restored environment is essential for rollback. The experiment therefore validates the conclusion drawn from our benchmark, and the remaining question is whether this consistency can be obtained at a practical cost, which we address next.

**Snapshot-adapted baselines incur prohibitive storage costs that exceed RecoverMem by 19.57 to 307.75 times, rendering them impractical for real-world scenarios.** Since existing memory systems are not natively designed for state rollback, preserving historical states requires duplicating their full memory banks at every step. As shown in Table 4 (summarized below), maintaining recovery effectiveness via version preservation requires between 36.00 MiB and 566.26 MiB of storage. In contrast, RecoverMem achieves the highest recovery rate (43.27%) using only 1.84 MiB by recording compact structured state updates at each step. Furthermore, naive snapshot preservation fails to resolve the semantic retrieval bottleneck. 

We believe that exposing these analysis in our work will motivate future research on lightweight, recovery-aware agent memory.


| Method | RR (%) | AS (%) | Storage (MiB) | Storage Overhead vs. Ours |
| :--- | :---: | :---: | :---: | :---: |
| A-MEM (Version-aware) | 43.27 | 70.56 | 36.00 | 19.57X |
| Mem0 (Version-aware) | 42.31 | 62.16 | 203.66 | 110.68X |
| MemGPT (Version-aware) | 41.34 | 66.33 | 70.53 | 38.33X |
| MemoryOS (Version-aware) | 39.42 | 112.78 | 566.26 | 307.75X |
| **RecoverMem (Ours)** | **43.27** | **89.35** | **1.84** | **1.00X** |

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
