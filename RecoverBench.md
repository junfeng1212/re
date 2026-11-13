## Reviewer idAb [Weak accept]

**[W1:]** The evaluation is narrow relative to the deployment claims. The 416 cases derive from only 104 independent tasks, with original trajectories averaging approximately 13.89 steps. The evaluation primarily uses one executor model, injected rollback failures, and a known recovery step.

**[A1:]** Thanks for your constructive feedback regarding our evaluation scope and experimental design. We hope the following replies could address your concerns:

**RecoverMemBench provides a sufficient testbed while balancing peer benchmark scales and evaluation costs.** Spanning 1,445 execution steps across 10 repositories, our 312 resumption (at 25%, 50%, and 75% steps) and 104 rollback cases sufficiently expose consistent bottlenecks across 11 memory systems, matching the scale of mainstream benchmarks like ALFWorld [1] and LongMemEval [2]. Moreover, with the dataset construction alone exceeding $1,800 USD, scaling it up further would make downstream evaluation prohibitively expensive; our current size strikes the ideal balance for affordable yet rigorous evaluation.

**Our settings ensure a strictly controlled evaluation that isolates memory recovery capabilities from confounding factors.** By building our dataset from tasks solved three consecutive times without interruption and fixing the executor, injected errors, and recovery steps, we prevent external variables (e.g., base model limits, agent framework design, or random error types) from distorting the evaluation of task recovery.

**Beyond the static dataset, a core contribution of our work is a generalizable data construction pipeline.** As the first systematic study of agent task recovery, we provide not only a ready to use benchmark, but also a reproducible pipeline for interruption, error injection, and state restoration that readily extends to other models and failure scenarios.

[1] ALFWorld: Aligning Text and Embodied Environments for Interactive Learning. ICLR2021.
[2] LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory. ICLR2025.

-------

**[W2:]** The comparison with version-aware baselines materially qualifies the method advantage.

**[A2:]** We thank the reviewer for this insightful comment regarding the version aware baselines in Appendix C.1. We clarify how these results support our core contributions and highlight the practical advantages of our method below: 

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

**[Q1:]** How would RecoverMem identify valid state when there is no reliable global rollback step?

**[A3:]** Thanks for your question regarding our method design.

**Our rollback setting aligns with practical deployment workflows, where restoring the environment inherently specifies the rollback step $k$.** In real-world scenarios, environment rollback is a targeted operation: people inspects the failed trajectory, locates the erroneous step, and reverts the workspace to the last valid state s_k. Because executing this environment restoration already identifies step $k$, RecoverMem directly uses $k$ to retrieve the exact task state $m_k$ without requiring extra assumptions. 


-------

**[Q2:]** How can the system retain knowledge from a failed suffix without reintroducing invalid progress?

**[A4:]** Thanks for your question regarding our experience retention mechanism.

**RecoverMem prevent valid progress from failed explorations by anchoring the task state strictly to $m_k$ while selectively distilling the failed suffix $\{m_{k+1}, \dots, m_T\}$ into a failure experience module.** Given the rollback step $k$, RecoverMem retrieves the exact snapshot $m_k$ so that the established progress and remaining subgoals strictly reflect the restored environment $s_k$, completely discarding any invalid state updates recorded after step $k$. Meanwhile, our Reflection mechanism summarizes the flawed trajectory from $m_{k+1}$ to $m_T$ and appends only the failure lessons into the dedicated failed field of $m_k$ to get $\tilde{m}_k$. This way allows the agent to retain knowledge of prior exploration faults without mistaking undone actions for completed work.

Overall, our work firstly investigates the practical task recovery setting in agent memory, and provides a foundation for evaluating
and improving agent memory for reliable and efficient task recovery.

We hope our responses have addressed your concerns. If you find them helpful, we would appreciate your consideration of increasing your score.

-------
-------


## Reviewer KFmi [Clear Rejection]

**[W1:]** The problem setting is not well motivated: Practical systems persist session logs and checkpoint the environment and memory together; providing the full previous trajectory, which fits in context here, is not compared.

**[A1:]** Thanks for your thoughtful views on our work. To address your concerns, we evaluated full history directly, where the fresh agent receives the raw actions and observations up to the recovery step k in its initial context instead of retrieved memory. Our experiments show that appending full history into context fits our settings poorly, since observations during interactions are often ambiguous and implicit,

**Table R1.** Full history versus RecoverMem.

| Setting | Resumption@25% RR | Resumption@25% AS | Resumption@50% RR | Resumption@50% AS | Resumption@75% RR | Resumption@75% AS | Rollback RR | Rollback AS |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| w/o Memory | 22.11 | 182.16 | 38.47 | 151.59 | 44.23 | 141.37 | 25.96 | 263.53 |
| Full history | 35.57 | 102.13 | 50.96 | 182.94 | 43.29 | 153.57 | 32.69 | 191.02 |
| RecoverMem (Ours) | **49.03** | **57.55** | **70.20** | **83.88** | **87.50** | **26.83** | **43.27** | **89.35** |

**Full history improves over w/o Memory but remains far below RecoverMem.** Rollback RR rises from 25.96% to 32.69%, yet averaged over the three resumption settings, full history reaches 43.27% RR and 146.21% AS, compared with 68.90% and 56.09% for RecoverMem. Under rollback, RecoverMem improves RR by 10.58 points and lowers AS by over 100 pp.

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

**[W2:]** The rollback mismatch exists only because the environment is rolled back but the memory is not; RecoverMem's step-indexed retrieval then amounts to a checkpoint.

**[A2:]**  Thanks for your questions about our proposed method. We hope the following replies can address your concerns:

**That's a scenario that prior memory benchmarks do not evaluate and that RecoverMemBench evaluates systematically.** We agree that this mismatch arises because the environment is rolled back while the external memory persists, and we adopt this setting by design. Specifically, rolling back both the environment and the memory system simultaneously would erase lessons learned from past failures, causing agents to repeat prior errors. Conversely, rolling back only the environment preserves those failure experiences, yet it introduces the risk of stale memory states misleading the agent. Consequently, the core challenge is to retain lessons from prior failing attempts while keeping memory consistent with the restored environment.

**RecoverMem is not a plain checkpoint**, for three reasons: 
i) Each entry is a structured task state record (completed progress, remaining subgoals, failed attempts) rather than a copy of the memory bank. 
ii) After rollback, it reflects on entries $m_{k+1}$ to $m_T$ to update the experience module, which a checkpoint would discard; removing this Reflection lowers RR from 43.27% to 38.46% and raises AS from 89.35% to 117.62% (Table 2). 
ii) Snapshots alone do not resolve the retrieval bottleneck, since entries from different steps of the same task remain semantically similar (Section 3.2.1).

**We also test the checkpoint view directly by equipping four baselines with stepwise snapshots (Appendix C.1, Table 4, provided below).** Because existing systems are not designed for rollback, this duplicates the full memory bank at every step and costs 36.00 to 566.26 MiB, whereas RecoverMem matches the best stepwise baseline (A-MEM, 43.27% RR) with only 1.84 MiB.
Such memory efficiency is critical in real-world scenarios, directly affecting operational speed.

| Method | RR (%) | AS (%) | Storage (MiB) | Storage Overhead vs. Ours |
| :--- | :---: | :---: | :---: | :---: |
| A-MEM (Stepwise) | 43.27 | 70.56 | 36.00 | 19.57X |
| Mem0 (Stepwise) | 42.31 | 62.16 | 203.66 | 110.68X |
| MemGPT (Stepwise) | 41.34 | 66.33 | 70.53 | 38.33X |
| MemoryOS (Stepwise) | 39.42 | 112.78 | 566.26 | 307.75X |
| **RecoverMem (Ours)** | **43.27** | **89.35** | **1.84** | **1.00X** |

----

**[W3:]** The benchmark does not support the claims: 104 tasks that GPT-4o solves reliably, with a median of 11 steps, one agent, one backbone, and one run per setting, far from the hundreds of actions in the motivation. Interruption points are set from the known length of successful runs, and rollback errors are injected by an LLM rather than made by the agent.

**[A3:]** Thanks for your constructive feedback regarding our evaluation scope and experimental design. We hope the following replies could address your concerns:

**RecoverMemBench provides a sufficient testbed while balancing peer benchmark scales and evaluation costs.** Spanning 1,445 execution steps across 10 repositories, our 312 resumption (at 25%, 50%, and 75% steps) and 104 rollback cases sufficiently expose consistent bottlenecks across 11 memory systems, matching the scale of mainstream benchmarks like ALFWorld [1] and LongMemEval [2]. Moreover, with the dataset construction alone exceeding $1,800 USD, scaling it up further would make downstream evaluation prohibitively expensive; our current size strikes the ideal balance for affordable yet rigorous evaluation.

**Our settings ensure a strictly controlled evaluation that isolates memory recovery capabilities from confounding factors.** By building our dataset from tasks solved three consecutive times without interruption and fixing the executor, injected errors, and recovery steps, we prevent external variables (e.g., base model limits, agent framework design, or random error types) from distorting the evaluation of task recovery.

**Beyond the static dataset, a core contribution of our work is a generalizable data construction pipeline.** As the first systematic study of agent task recovery, we provide not only a ready to use benchmark, but also a reproducible pipeline for interruption, error injection, and state restoration that readily extends to other models and failure scenarios.

[1] ALFWorld: Aligning Text and Embodied Environments for Interactive Learning. ICLR2021.
[2] LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory. ICLR2025.

------

[‼️]**[W4:]** The evaluation of RecoverMem is not convincing: It receives the git diff (Figure 7) and the rollback step index, which the baselines do not; with step-wise snapshots, baselines reach similar recovery rates (Appendix C.1).

**[A4:]** Thanks for your thoughtful views about our evaluation pipeline. We hope the following replies can make our evaluation more clear:

**Git diff cannot be accessed for all methods.** In Figure 7, that git diff is not the "git.diff" file instead of the description in "" 

**Accessing rollback step is the default setting of task rollback.** Our rollback setting aligns with practical deployment workflows, where restoring the environment inherently specifies the rollback step $k$. Because, in real-world scenarios, environment rollback is a targeted operation: people inspects the failed trajectory, locates the erroneous step, and reverts the workspace to the last valid state s_k. However, in the default setting of current memory methods, the retrieving is based on semantic query. So, they are unable to utilize that rollback step index.

**Targeted experiments in Appendix C.1 further substantiate this finding.** Evaluations on our benchmark demonstrate that baselines' overwrite updates frequently trigger state inconsistencies between memory and the environment. While stepwise snapshotting addresses this issue, it fails to address retrieval bottleneck: because all recorded memories are closely related to the task query, semantic differentiation blurs, preventing the agent from retrieving the exact state required for recovery. Furthermore, stepwise baselines need to duplicate their full memory banks at every step,  requiring 19.51X - 307.75X storage costs compared with RecoverMem. That undermines their applicabilities in real-world scenarios as efficiency is one of most important parts of memory system. 


----

**[W5:]** It is tailored to SWE tasks and evaluated only on the authors' own benchmark, not on existing memory benchmarks.

**[A5:]**  Thank you for the thoughtful comment. We hope the following replies can address your concern.

**Existing memory benchmarks evaluate how retained knowledge benefits later tasks, and they define neither an interruption nor an environment restore step, so they cannot measure task recovery, which is why we build RecoverMemBench instead of reusing them.** We choose SWE-bench because Git lets us restore the environment to any earlier step, which task rollback requires, whereas in domains with irreversible or costly actions such a controlled rollback is hard to construct. We discuss that in our Limitation section (Appendix D), while the two failure modes we identify (semantic retrieval among similar entries and overwrite updates) stem from memory design rather than from coding. Extending RecoverMemBench to other domains with restorable environments is the main direction of our future work.

---

**[W6:]** The failure analysis is weakly supported: the "full memory bank" test uses the evaluation question as the retrieval query, and the questions match RecoverMem's record fields. The link between answer correctness and recovery success is not shown.

**[A6:]** Thanks for your comments about our analysis. We hope the responses below will resolve your concerns.

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

**[Q1:]** Without memory, resuming at 25% is close to restarting from scratch, yet the recovery rate is only about 22%, although the agent solves all these tasks in three runs. What explains this, and what prompt does the recovered agent receive?

**[A7:]** Thank you for the valuable feedback! We address your concerns below. Resumption@25% is not equivalent to restarting from scratch, because recovery starts from the restored environment s_k rather than s_0, and **the first k steps may already have modified the repository or added files such as reproduction scripts (82.69% samples have modified the files in repo when resumpation@25%)**. Without memory, the fresh agent receives only the task description, so it cannot tell which changes are already present or which subtasks are complete, and it may repeat finished work (e.g., re-editing code that is already modified) or build on a wrongly assumed state. Such behavior makes the run unreliable and adds steps, which is reflected in the AS of 182.16% in Table 1. The 22.11% RR therefore does not contradict the three uninterrupted successes: those runs show that the agent can solve each task from a clean state, whereas RecoverMemBench measures whether it can recover the state of a partially modified environment, which base capability alone does not provide.

Regarding the prompt, the w/o Memory agent receives only the task description q, using the default mini-SWE-agent[1] prompt for SWE-bench, with no memory content and no summary of earlier steps. For the other methods, the prompt is identical and the retrieved memory m = Read(M, q) is placed in the initial context (Section 2.1), so memory is the only variable across methods. ‼️We will add the full prompt to the appendix.

**[Q2:]** Additional steps include failed runs and exclude the cost of memory construction and reflection. The "over 35%" gain in the abstract is relative to the average baseline, not the strongest one.

**[A8:]** Thanks for the insightful review. Our responses follow.

**Scope of Additional Steps (AS)**. AS measures only the agent’s execution steps for completing the task, relative to the uninterrupted run (Eq. 6), and it is not a measure of total system cost. It counts the steps of failed recovery runs, which is why Section 2.4 requires reading AS together with RR, and it excludes memory construction, reflection, and latency, as stated in Appendix C.2. We will therefore write “additional steps” instead of “additional cost” in the abstract and throughout the paper. To cover the excluded part, Table 5 reports the token and storage footprint of each memory system: RecoverMem uses 1,206.27 tokens and 1.84 MiB per sample, lower than A-Mem, MemGPT, MemoryOS, and MemoryBank.

And we will refine the wording in the abstract to clarify that this value represents the average baseline.

We hope our responses have addressed your concerns. If you find them helpful, we would appreciate your consideration of increasing your score.

-----

## Reviewer yXDZ (Weak Rejection)

**[W1&Q1&Q3:]** 1. Added value over straightforward recovery approaches is not sufficiently established without evaluating full interaction history. The snapshot comparison also requires further analysis. &&  Additional evidence on full-history recovery. && Clarification of the rollback advantage.

**[A1:]** We thank the reviewer for these constructive questions. our replies in brief: (1) restoring the complete interaction history yields lower RR and higher AS than RecoverMem in every setting, even though the history fits in the agent’s context window; (2) under rollback, RecoverMem’s advantage over the snapshot-adapted baselines lies primarily in storage, and these baselines in fact adopt the mechanism that our analysis identifies; (3) we will revise our cost claims to refer to execution steps and storage.

### Recovery with the full interaction history

**Setup.** A fresh mini-SWE-agent receives the raw trajectory in its initial context. For task resumption, the history covers steps $1$ to $k$, which constitutes the complete history at interruption. For task rollback, we evaluate both variants as you suggested. Full history ($\le k$) keeps steps $1$ to $k$ and discards the failed suffix. Full history (all) keeps steps $1$ to $T$ and explicitly marks steps $k+1$ to $T$ as undone by rollback. We do not truncate the history to the 4,096-token budget imposed on all memory systems (Appendix B.1), so this comparison favors the full-history baseline.

**Table R1.** RR (%, ↑) / AS (%, ↓).

| Method | Resump.@25% | Resump.@50% | Resump.@75% | Resump. Avg. | Rollback |
| :--- | :---: | :---: | :---: | :---: | :---: |
| w/o Memory | 22.11 / 182.16 | 38.47 / 151.59 | 44.23 / 141.37 | 34.94 / 158.37 | 25.96 / 263.53 |
| Full history ($\le k$) | 35.57 / 102.13 | 50.96 / 182.94 | 43.29 / 153.57 | 43.27 / 146.21 | 32.69 / 191.02 |
| Full history (all, marked) | - | - | - | - | 19.23 / 268.74 |
| RecoverMem (Ours) | 49.03 / 57.55 | 70.20 / 83.88 | 87.50 / 26.83 | 68.90 / 56.09 | 43.27 / 89.35 |

**Table R2.** Average history length in tokens of each task.

| Setting | Resumption@25% | Resumption@50% | Resumption@75% | Rollback@k | Rollback@all |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Tokens | 2,643 | 4,625 | 6,177 | 4,251 | 8,348|

**RecoverMem outperforms full history in every setting, including when the history fits.** All histories fit within the context window of GPT-4o (Table R2), so Table R1 directly shows the case in which the agent has access to the complete history. Even under this condition, full history reaches an average resumption RR of 43.27%. This is below 9 of the 11 memory systems in Table 1 and 25.63 pp below RecoverMem, and it comes with 146.21% AS compared with 56.09%. RecoverMem achieves both higher RR and lower AS in every setting. 

**Raw history degrades as trajectories lengthen.** Relative to w/o Memory, full history boosts RR at @25% (+13.46 pp) and @50% (+12.49 pp), but underperforms at @75% (-0.94 pp) and inflates AS from @50% onward. Worse, its RR declines between @50% and @75% (50.96% $\rightarrow$ 43.29%) despite more completed steps, whereas RecoverMem steadily scales (70.20% $\rightarrow$ 87.50%). Consistent with the lost-in-the-middle phenomenon (Liu et al., 2024), lengthy raw logs mix outdated details with actual progress, forcing the agent to figure out from scratch what is done and what remains. RecoverMem prevents this via a bounded state tracking completed progress, remaining subgoals, and failed attempts.

**Under rollback, keeping undone actions actively misleads the agent.** While steps $1$ to $k$ raise RR over w/o Memory (25.96% $\rightarrow$ 32.69%), appending the marked suffix drops RR to 19.23% and inflates AS to 268.74%. Since we only mark that steps were undone rather than why they failed, the agent treats the reverted path as viable and repeats it. RecoverMem instead restores the state-aligned memory $m_k$ (matching $s_k$) and uses Reflection to record the suffix as a failed attempt to avoid. Under the same information scope, RecoverMem beats Full history ($\le k$) even without Reflection (38.46% vs. 32.69% RR; 117.62% vs. 191.02% AS), while Reflection adds +4.81 pp in RR.

These results show that the challenge exposed by RecoverMemBench does not stem from a lack of access to past interactions. The complete history is available, yet it does not support reliable recovery. Recovery requires a task state that is consistent with the environment from which execution resumes, which is the central finding of our analysis and the design principle of RecoverMem. 

We added Tables R1 to R3 and this discussion and Appendix C.3 (highlighted in red).

### Advantage over the snapshot-adapted baselines and cost claims

**Snapshot-adapted baselines incur prohibitive storage costs that exceed RecoverMem by 19.57 to 307.75 times, rendering them impractical for real-world scenarios.** Since existing memory systems are not natively designed for state rollback, preserving historical states requires duplicating their full memory banks at every step. As shown in Table 4 (summarized below), maintaining recovery effectiveness via version preservation requires between 36.00 MiB and 566.26 MiB of storage. In contrast, RecoverMem achieves the highest recovery rate (43.27%) using only 1.84 MiB by recording compact structured state updates at each step. Furthermore, naive snapshot preservation fails to resolve the semantic retrieval bottleneck. 

**The snapshot-adapted baselines adopt the mechanism our analysis identifies.** These variants are not existing memory systems as released. We equip them with per-step snapshots retrieved by step index, which is the solution implied by our diagnosis of memory-environment mismatch (Section 3.2.2). Averaged over the four baselines, this adaptation raises RR from 35.58% to 41.59% and lowers AS from 147.84% to 77.96%, which validates the diagnosis as a central finding of RecoverMemBench. In this setting, RecoverMem contributes a practical realization of the same mechanism.

**Under resumption, the advantage does not come from storage.** Memory-environment mismatch arises only after rollback. Under resumption, $\mathcal{M}_k$ is built from steps $1$ to $k$ and is consistent with $s_k$ by construction (Eq. (3)). The snapshot at step $k$ therefore coincides with the original memory and leaves the retrieval failure of Section 3.2.1 unaddressed. RecoverMem’s resumption gains (68.90% RR versus 50.91% for MemGPT, and 56.09% AS versus 104.90% for A-MEM) stem from its structured task-state record, which answers 86.3% of task-state questions correctly, compared with at most 39.3% for the four evaluated baselines (Figure 3(a)). The same property explains why full history falls short.

**Cost claims.** We report three cost components separately: execution steps (AS, Table 1), memory construction tokens (Table 5), and storage (Tables 4 and 5). Since these do not establish total cost (Appendix C.2), we will replace “additional cost” with “additional execution steps” in the abstract, Section 1, and Section 4.2, and scope each claim to the component it measures. Relative to the original memory systems (Table 1), RecoverMem requires the fewest execution steps in every setting. Relative to the snapshot-adapted baselines (Table 4), its advantage lies in storage alone.


Thank you for your thoughtful comments and valuable suggestions. We respond to each point in detail below.

------

**[W2&Q2:]** It is unclear how the gains depend on the underlying model. Additional evidence on model dependence. &&
Could you evaluate a small set of executors with different capabilities while keeping the memory construction model fixed—for example, different sizes within one Qwen family, together with a stronger Qwen, DeepSeek, or other model? The key comparison is RecoverMem’s gain over full-history and snapshot baselines under each executor, with that model’s uninterrupted task performance also reported.

**[A2:]**

**Benchmark construction controls for task solvability, not for the executor.** For each executor, RecoverMemBench includes only tasks that it solves in all three uninterrupted runs, so a recovery failure cannot be attributed to a task beyond the executor's capability. The executor's ability to use memory, however, may still affect recovery, which the following experiment examines.

**Setup.** We add two executors of different sizes from the Qwen-family, Qwen3.8-27B and Qwen3.8-Flash (125B), alongside GPT-4o from the main experiments, and keep GPT-4o-mini as the memory construction model. For each executor, we follow Section 2.3 to construct 104 Resumption@50% samples and 104 rollback samples from the tasks it solves in all three uninterrupted runs. Since the sample sets differ across executors, we compare the margins of RecoverMem within each executor rather than absolute RR across executors. As in A1, we compare against full history and the snapshot-adapted baselines.

**Table R5.** Resumption@50%, RR (%, ↑) / AS (%, ↓).

| Method | GPT-4o | Qwen3.8-27B | Qwen3.8-Flash (125B) |
| :--- | :---: | :---: | :---: |
| Mem0 | 50.97 / 156.78 | 51.92 / 127.88 | 58.65 / 110.81 |
| A-MEM | 65.38 / 110.43 | 55.76 / 104.23 | 61.54 / 103.59 |
| Full history | 50.96 / 182.94 | 50.00 / 139.42 | 56.73 / 118.92 |
| **RecoverMem (Ours)** | **70.20 / 83.88** | **62.50 / 91.75** | **68.27 / 70.18** |

**Table R6.** Task rollback, RR (%, ↑) / AS (%, ↓).

| Method | GPT-4o | Qwen3.8-27B | Qwen3.8-Flash (125B) |
| :--- | :---: | :---: | :---: |
| Mem0 | 26.92 / 179.42 | 19.23 / 151.20 | 22.12 / 140.63 |
| A-MEM | 38.46 / 126.12 | 26.92 / 137.64 | 29.81 / 129.74 |
| Full history ($\le k$) | 32.69 / 191.02 | 25.00 / 163.47 | 28.85 / 148.26 |
| Mem0 (snapshot) | 42.31 / 62.16 | 36.54 / 86.54 | 41.35 / 74.83 |
| A-MEM (snapshot) | 43.27 / 70.56 | 38.46 / 84.21 | 44.23 / 71.96 |
| RecoverMem (Ours) | 43.27 / 89.35 | 40.38 / 93.97 | 46.15 / 82.65 |

**RecoverMem outperforms the original systems and full history under every executor.** Relative to A-MEM, the stronger original baseline, RecoverMem improves RR by +4.82%, +6.74%, and +6.73% under resumption and by +4.81%, +13.46%, and +16.34% under rollback for GPT-4o, Qwen3.8-27B, and Qwen3.8-Flash, respectively, with lower AS in all six cases. Relative to full history, it leads in RR by at least +11.54% under resumption and +10.58% under rollback, again with lower AS in every case.

**The benefit of state-consistent memory grows with executor capability.** With stronger executors (Table R4), RecoverMem's rollback margin widens from +4.81% to +13.46% and +16.34% over A-MEM, and from +10.58% to +15.38% and +17.30% over full history. Snapshot adaptation mirrors this trend, raising rollback RR by +4.81%, +11.54%, and +14.42% for A-MEM, and by +15.39%, +17.31%, and +19.23% for Mem0. While stronger executors read raw history better, they still cannot reliably identify undone progress. Resolving this requires memory consistent with $s_k$, the central diagnosis of RecoverMemBench.

**Relative to the snapshot-adapted baselines, the pattern in A1 holds under every executor.** RecoverMem matches or exceeds their RR by +0.00% to +4.80%, whereas those baselines require fewer additional steps. Its advantage over snapshot adaptation therefore lies in storage, as discussed in A1.

We will add Tables R5 and R6 to Appendix C. We hope our responses have addressed your concerns, and we would be grateful if you would consider raising your score.

-----
