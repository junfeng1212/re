## Reviewer Wh7d [2: Weak rejection]
-----
**W1&Q1:** Storing multiple conclusions with confidence scores can improve performance. However, BeliefMem relies on uncalibrated LLM confidence scores and heuristic Noisy-OR updates. It is unclear whether these updates reliably represent uncertainty, particularly when observations are correlated or contradictory. & Could the authors evaluate the calibration and robustness of belief scores, particularly under correlated or contradictory observations?

**A1:**  We thank the reviewer for this thoughtful comment. 

**BeliefMem assigns the highest belief score to true conclusion, showing the reliability of belief score.** In experiments of Figure 4(b), we track the the Top-1 rate, defined as the fraction of attributes whose true conclusion receives the highest score. As shown in Figure 4(b) (also provided below), with gradual updates, our method BeliefMem rises the Top-1 rate from 7.69% to 84.62%, which means that 84.62% of true conclusions receive the highest belief score among all candidates. In contrast, Mem0 and A-MEM only improve the Top-1 rate to 46.15% and 42.31%, respectively. These results show that BeliefMem can calibrate the belief scores reliably using Noisy-OR update over steps.


| Step | BeliefMem | Mem0 | A-MEM |
| --- | --- | --- | --- |
| 1 | 7.69% | 7.69% | 7.69% |
| 50 | 15.38% | 38.46% | 38.46% |
| 100 | 23.08% | 42.31% | 38.46% |
| 200 | 46.15% | 42.31% | 38.46% |
| 250 | 46.15% | 57.69% | 53.85% |
| 300 | 61.54% | 57.69% | 53.85% |
| 400 | 65.38% | 50.00% | 46.15% |
| 500 | 84.62% | 46.15% | 42.31% |

Additionally, in Table 7 (provided below), Shuffling the belief scores of all candidates of each attribute reduces F1 from 40.51 to 29.72 on multi-hop questions and from 51.88 to 31.22 on temporal questions in LoCoMo benchmark. This experiments further show the effectiveness of belief scores in our work, which can reliably provide the uncertainty in external environement.

| Method | Multi-hop | Temporal | Open-domain | Single-hop |
| :--- | :---: | :---: | :---: | :---: |
| Shuffling Probabilities | 29.72 / 24.10 | 31.22 / 27.61 | 17.93 / 14.81 | 32.17 / 25.12 |
| **BeliefMem (ours)** | **40.51 / 32.24** | **51.88 / 45.78** | **28.73 / 25.12** | **48.41 / 42.07** |

**BeliefMem corrects the contradictory memory over observations.** In experiments of Figure 3, we select several attributes of which contradictory memory receive the highest belief score. Over 10 steps (5 valid observations and 5 correlated observations), BeliefMem can refine 60.80% of contradictory memory into the correct memory (the correct memory outranks the contradictory) with only average 4.75 steps. In contrast, prior deterministic memory method only refine 33.30% of contradictory memory into right memory while using averge 9.45 steps.  These results show the robustness of belief score in BeliefMem due to its memory update over noisy observations.

| Method | Correction Rate$up$ | Correction Steps$down$ |
| --- | --- | --- |
| BeliefMem | 60.80% | 4.75 |
| Deterministic memory system | 33.30% | 9.45 |

Overall, our work firstly proposes the probabilistic paradigm of memory system, enabling the memory system to preserve the uncertainty in POMDP, to address the self-reinforcing error. We believe BeliefMem can provide insights for future agent memory research.



-----

**W2&Q2:** BeliefMem is motivated by interactive agent settings, where memory influences future actions and observations. However, the evaluation primarily focuses on conversational QA, with ALFWorld as the only interactive agent benchmark. This provides limited evidence of its effectiveness in broader agentic settings. & Could the authors evaluate BeliefMem in more complex online agent settings where memory influences the agent's actions and subsequent observations?

**A2:** ‼️[跑一个额外的benchmark实验]

----

**Q1:** Could the authors evaluate the calibration and robustness of belief scores, particularly under correlated or contradictory observations?
同A1

---

**Q2:** Could the authors evaluate BeliefMem in more complex online agent settings where memory influences the agent's actions and subsequent observations?

同A2

------

## Reviewer tp79 [2: Weak rejection]
----
**W1&Q1:** Misalignment between evaluation and core motivation. LoCoMo primarily evaluates long-term conversational memory and question answering, rather than agent self-evolution or self-reinforcing errors. It lacks the closed-loop interaction required to investigate how incorrect memories influence subsequent actions and observations. Although ALFWorld provides an interactive setting, the experiments do not sufficiently establish the claimed self-reinforcing error mechanism./ Could the authors provide a closed-loop evaluation demonstrating how incorrect memories influence future actions, reinforce themselves, and are eventually corrected by BeliefMem?

**A1:** We thank the reviewer for the suggestion and provide the requested closed-loop evaluation below. The experiment proceeds in two stages. First, misleading observations create an incorrect memory. Second, the agent completes a sequence of tasks and writes each trajectory back to memory, so actions in earlier tasks determine the memory available to later tasks.

**Setting.** The experiment uses all 46 Cool & Place tasks of ALFWorld. Each task requires the agent to cool an object with fridge 1 and then place the cooled object at a target receptacle, so success requires using the fridge. BeliefMem is compared with Mem0 and A-MEM. The agent solves the 46 tasks one by one. And the order of each task is random. After each task, the trajectory is written into memory through the native write operation of each method, and the next task retrieves from the updated memory.

**Constructing the incorrect memory.** All methods start from an empty memory and receive the same observations: the agent tries to open fridge 1, and the environment returns “Nothing happens.” Each method writes the observations through the native write operation. After writing, Mem0 and A-MEM store “Fridge 1 cannot be opened.” BeliefMem stores two competing conclusions about the fridge: “Fridge 1 cannot be opened” with probability 0.92 and “Fridge 1 can be opened” with probability 0.25. Fridge 1 is then restored, and the evaluation on the 46 tasks begins. After the restoration, the environment never returns a fridge failure, so the stored conclusion “Fridge 1 cannot be opened” is incorrect in every task.

**Metrics.** The evaluation reports the success rate (SR) and the *fridge rate*, defined as the fraction of tasks with at least one use of fridge 1. As a reference, each method also runs on the same tasks in the same order without the incorrect memory.

| Method | SR without incorrect memory (%) | SR with incorrect memory (%) | Fridge rate, tasks 1 to 20 (%) | Fridge rate, tasks 21 to 46 (%) |
| --- | --- | --- | --- | --- |
| Mem0 | 32.61 | 23.91 | 15.00 | 34.62 |
| A-MEM | 28.26 | 19.57 | 10.00 | 19.23 |
| BeliefMem | 71.74 | 67.39 | 100.00 | 100.00 |

**Incorrect memory changes future actions.** Without the incorrect memory, all three methods use fridge 1 in all 46 tasks. With the incorrect memory, Mem0 and A-MEM use fridge 1 in only 15% and 10% of the first 20 tasks. Because every task requires fridge 1, the reduced fridge use directly lowers the SR. The SR of Mem0 drops from 32.61% to 23.91%, and the SR of A-MEM drops from 28.26% to 19.57%.

**Incorrect memory reinforces itself.** After fridge 1 is restored, the environment never returns a fridge failure. However, the number of memory entries supporting “Fridge 1 cannot be opened” grows from 3 after the first task to 11 after the last task in Mem0, and from 2 to 18 in A-MEM. These new entries come from failed tasks where the agent avoided fridge 1 and attributed the failure to the fridge. Avoiding the fridge therefore generates new evidence for the incorrect conclusion, even though the environment provides none.

**Deterministic memory does not resolve the error.** When the agent occasionally interacts with Fridge 1 and succeeds in opening it, Mem0 and A-MEM add a new entry like "Fridge 1 can be opened" in memory bank, while retaining the contradictory "Fridge 1 cannot be opened" along with its associated entries. These contradictory entries then coexist in the retrieved context, and the behavior of the agent remains inconsistent. In tasks 21 to 46, the fridge rate of Mem0 and A-MEM reaches only 34.62% and 19.23%.

**BeliefMem corrects the incorrect memory.** BeliefMem retrieves both conclusions together with the corresponding probabilities, so the agent sees that fridge 1 may still work. In the first task, the agent initially searches for other ways to cool the object. After 36 steps, the agent tries fridge 1 because “Fridge 1 can be opened” still has probability 0.25, and the fridge opens. Merge then incorporates the new observation in two ways (Section 3.3): the Noisy-OR update raises “Fridge 1 can be opened” from 0.25 to 0.83, and the competing conclusion “Fridge 1 cannot be opened” is reset from 0.92 to 0.25.

From the second task on, the agent uses fridge 1 in every task. BeliefMem reaches a fridge rate of 100% and an SR of 67.39%, close to the SR of 71.74% without the incorrect memory.

**Summary.** The experiment covers the closed-loop evaluation. In deterministic memory, the incorrect conclusion suppresses the only action able to refute the conclusion. The resulting failures add entries that support the incorrect conclusion, and the incorrect conclusion persists after contradicting evidence appears. BeliefMem keeps the alternative conclusion visible. The visible alternative leads the agent to test fridge 1 within the first task, and the resulting observation corrects the memory. We will add the experiment to the revision.

----

**W2:** Insufficient analysis of noise from competing hypotheses. Preserving multiple candidate conclusions may introduce conflicting or misleading information during retrieval, potentially degrading downstream decisions. While the paper examines retrieval size, it does not systematically investigate how the number of conflicting candidates or their confidence distributions affect task performance. More targeted ablations are necessary.

**A2:** Thanks for your insightful comments on our work. We hope the following replies could address your concerns.

**Probabilistic memory addresses conflicting or misleading information preserved in multiple candidate conclusions.** In ablation studies, we remove the all probabilities of all candidates of each attribute $c$ (w/o belief-aware retrieval). As shown below, removing these probabilities only achieve the average performance of 28.50 / 19.02 on LoCoMo benchmark. However, when BeliefMem with these probabilities, it improves the average performance from 28.50 / 19.02 to 42.38 / 36.30. These results show that preserving multiple candidate conclusions can impact the decisions of agents due to conflicting information over there. Our BeliefMem can handle this challenge by ranking these conflicting information using probabilities, enabling agents to know which candidate conclusion is most reliable.

**Probability distribution provided in BeliefMem is effective and reliable.** In Table 9, we conduct experiments where we shuffle the probability of each candidate given an attribute $c$. As shown in Table 9 (also provided below), shuffling probabilities results in notable performance degradation on the LoCoMo benchmark, degrading performance by an average of 14.62% / 13.39%.

| Method | Multi-hop | Temporal | Open-domain | Single-hop | AVG |
| :--- | :---: | :---: | :---: | :---: | :---: |
| w/o belief-aware retrieval | 27.12 / 18.98 | 25.67 / 15.43 | 15.89 / 10.12 | 45.31 / 31.55 | 28.50 / 19.02 |
| Shuffling Probabilities | 29.72 / 24.10 | 31.22 / 27.61 | 17.93 / 14.81 | 32.17 / 25.12 | 27.76 / 22.91 |
| **BeliefMem (ours)** | **40.51 / 32.24** | **51.88 / 45.78** | **28.73 / 25.12** | **48.41 / 42.07** | **42.38 / 36.30** |

Additionally, **BeliefMem assigns the highest belief score to true conclusion.** In experiments of Figure 4(b), we track the the Top-1 rate, defined as the fraction of attributes whose true conclusion receives the highest score. As shown in Figure 4(b) (also provided below), with gradual updates, our method BeliefMem rises the Top-1 rate from 7.69% to 84.62%, which means that 84.62% of true conclusions receive the highest belief score among all candidates. In contrast, Mem0 and A-MEM only improve the Top-1 rate to 46.15% and 42.31%, respectively. These results comprehensively show BeliefMem's reliability in preserving the uncertainty and providing it for agent's actions.

| Step | BeliefMem | Mem0 | A-MEM |
| --- | --- | --- | --- |
| 1 | 7.69% | 7.69% | 7.69% |
| 50 | 15.38% | 38.46% | 38.46% |
| 100 | 23.08% | 42.31% | 38.46% |
| 200 | 46.15% | 42.31% | 38.46% |
| 250 | 46.15% | 57.69% | 53.85% |
| 300 | 61.54% | 57.69% | 53.85% |
| 400 | 65.38% | 50.00% | 46.15% |
| 500 | 84.62% | 46.15% | 42.31% |

**Increasing the number of conflicting candidates improve the performance of BeliefMem.** We further conduct the experiments with different number of conflicting candidates |H_sub(c)|

|H_sub(c)|  ALF-Seen(SR / #Steps↓)  ALF-Unseen(SR / #Steps↓)
2 
4 (default setting)
6


------

**W3:** Incomplete discussion of related work. The paper would benefit from a more comprehensive comparison with recent studies on persistent memory corruption, poisoned experience retrieval, and error propagation in self-evolving agents. The distinction between BeliefMem and these approaches should be clarified.

**A3:** related work. 要和self evolving agent讨论

-----

**W4:** Unclear recovery mechanism under incorrect high-confidence beliefs. When an initially incorrect belief already dominates an agent's actions, the agent may avoid behaviors that could generate contradictory evidence. BeliefMem primarily updates confidence after recseiving new observation, but does not explain how corrective evidence can be actively obtained in such situations.

**A4:** 解释一下 当incorrect high-confidence如何修复，给case study

Fig 4的实验回复这个问题

-----

Q1: Could the authors provide a closed-loop evaluation demonstrating how incorrect memories influence future actions, reinforce themselves, and are eventually corrected by BeliefMem?

A5: We thank the reviewer for the suggestion and provide the requested closed-loop evaluation below. The experiment proceeds in two stages. First, misleading observations create an incorrect memory. Second, the agent completes a sequence of tasks and writes each trajectory back to memory, so actions in earlier tasks determine the memory available to later tasks.

**Setting.** The experiment uses all 46 Cool & Place tasks of ALFWorld. Each task requires the agent to cool an object with fridge 1 and then place the cooled object at a target receptacle, so success requires using the fridge. BeliefMem is compared with Mem0 and A-MEM. The agent solves the 46 tasks one by one. After each task, the trajectory is written into memory through the native write operation of each method, and the next task retrieves from the updated memory.

**Constructing the incorrect memory.** All methods start from an empty memory and receive the same observations: the agent tries to open fridge 1, and the environment returns “Nothing happens.” The observation is ambiguous, because ALFWorld returns the same message for any action without effect. Each method writes the observations through the native write operation. After writing, Mem0 and A-MEM store “Fridge 1 cannot be opened.” BeliefMem stores two competing conclusions about the fridge: “Fridge 1 cannot be opened” with probability 0.92 and “Fridge 1 can be opened” with probability 0.25. Fridge 1 is then restored, and the evaluation on the 46 tasks begins. After the restoration, the environment never returns a fridge failure, so the stored conclusion “Fridge 1 cannot be opened” is incorrect in every task.

**Metrics.** The evaluation reports the success rate (SR) and the fridge rate, defined as the fraction of tasks with at least one use of fridge 1. As a reference, each method also runs on the same tasks in the same order without the incorrect memory.

| Method | SR without incorrect memory (%) | SR with incorrect memory (%) | Fridge rate, tasks 1 to 20 (%) | Fridge rate, tasks 21 to 46 (%) |
| --- | --- | --- | --- | --- |
| Mem0 | 32.61 | 23.91 | 15.00 | 34.62 |
| A-MEM | 28.26 | 19.57 | 10.00 | 19.23 |
| BeliefMem | 71.74 | 67.39 | 100.00 | 100.00 |

**Incorrect memory changes future actions.** Without the incorrect memory, all three methods use fridge 1 in all 46 tasks. With the incorrect memory, Mem0 and A-MEM use fridge 1 in only 15% and 10% of the first 20 tasks. Because every task requires fridge 1, the reduced fridge use directly lowers the SR. The SR of Mem0 drops from 32.61% to 23.91%, and the SR of A-MEM drops from 28.26% to 19.57%.

**Incorrect memory reinforces itself.** After fridge 1 is restored, the environment never returns a fridge failure. Nevertheless, the number of memory entries supporting “Fridge 1 cannot be opened” grows from 3 after the first task to 11 after the last task in Mem0, and from 2 to 18 in A-MEM. These new entries come from failed tasks where the agent avoided fridge 1 and attributed the failure to the fridge. Avoiding the fridge therefore generates new evidence for the incorrect conclusion, even though the environment provides none.

**Deterministic memory does not resolve the error.** When the agent occasionally interacts with Fridge 1 and succeeds in opening it, Mem0 and A-MEM add a new entry like "Fridge 1 can be opened" in isolation, while retaining the contradictory "Fridge 1 cannot be opened" along with its associated entries. These contradictory entries then coexist in the retrieved context, and the behavior of the agent remains inconsistent. In tasks 21 to 46, the fridge rate of Mem0 and A-MEM reaches only 34.62% and 19.23%.

**BeliefMem corrects the incorrect memory.** BeliefMem retrieves both conclusions together with the corresponding probabilities, so the agent sees that fridge 1 may still work. In the first task, the agent initially searches for other ways to cool the object. After 36 steps, the agent tries fridge 1 because “Fridge 1 can be opened” still has probability 0.25, and the fridge opens. Merge then incorporates the new observation in two ways (Section 3.3): the Noisy-OR update raises “Fridge 1 can be opened” from 0.25 to 0.83, and the competing conclusion “Fridge 1 cannot be opened” is reset from 0.92 to 0.25.

From the second task on, the agent uses fridge 1 in every task. BeliefMem reaches a fridge rate of 100% and an SR of 67.39%, close to the SR of 71.74% without the incorrect memory.

**Summary.** The experiment covers the closed-loop evaluation. In deterministic memory, the incorrect conclusion suppresses the only action able to refute the conclusion. The resulting failures add entries that support the incorrect conclusion, and the incorrect conclusion persists after contradicting evidence appears. BeliefMem keeps the alternative conclusion visible. The visible alternative leads the agent to test fridge 1 within the first task, and the resulting observation corrects the memory. We will add the experiment to the revision.


-----

Q2: How does performance change when varying the number of distracting hypotheses and assigning incorrect or closely competing confidence scores?

同W2

------

Q3: How does BeliefMem differ from existing work on persistent memory poisoning and self-reinforcing agent errors?

A3: BeliefMem differs from both lines in the source of the error, the mechanism of persistence, and the remedy. Memory poisoning studies records crafted by an adversary, such as backdoor triggers planted in long-term memory by AgentPoison, malicious records injected purely through queries by MINJA, and payloads from attacker-controlled web content written into memory by Zombie Agents. BeliefMem instead addresses errors without any adversary: the agent infers a plausible but wrong conclusion from partial and ambiguous observations. In prior work on self-reinforcing errors, an incorrect record is reused as a trusted precedent and replicated in later executions. The corresponding remedies decide whether each record is added, kept, or deleted, through quality assessment before addition or consensus validation across related memories. BeliefMem targets a different mechanism: a deterministic conclusion suppresses the action that would test the conclusion, so disconfirming evidence never arrives. BeliefMem therefore changes the representation rather than filtering records, and keeps competing conclusions with confidence scores so that the alternative remains visible and revisable. BeliefMem is not a security defense, since an adaptive attacker could still supply strong false evidence. 

The two directions are complementary, and we will add this discussion to Section 2.


------

Q4: If an incorrect high-confidence belief prevents the agent from exploring alternatives, what mechanism enables BeliefMem to escape this feedback loop?

同W4

-------

## Reviewer nZa7 [1: clear rejection]

**W1:** The notation is inconsistent in several places. First, the formula above eq. 3 is written as a_t ∼ π(· | o_t, z_t), but z_t is never defined. Second, the letter M is used for two different things. M_t in Eq. 2 denotes the external memory, while M_c in H(c) = {h_1^(c), ..., h_{M_c}^(c)} denotes the number of hypotheses. Third, Eq. 3 updates the memory as Update(M_t, o_t, o_{t+1}) without a_t. However, the paragraph on self-reinforcing error says that the transition (o_t, a_t, o_{t+1}) is written back to memory via Eq. 3. Fourth, in the paragraph on belief update, the authors keep scores unnormalised so that the score of a supported h is not affected by the number of alternative candidates for the same c. However, in Merge, a competing candidate is reset to 1/|H_sub(c)|, which depends directly on the number. Moreover, the retrieval score α_t(c) in Eq. 9 depends only on semantic similarity and time decay but p is not used. So retrieval ranking is not belief-aware.

**A1:** 
**Undefined $z_t$ is a typo.** Since we mention that agent should read $o_t$ and $m_t$ to select action $a_t$ in line 199 ("The agent selects $a_t$ conditioned on $o_t$ and $m_t$"), namely $a_t \sim \pi(\cdot \mid o_t, m_t)$, where $m_t = \text{Read}(M_t, o_t)$ is the memory context defined in Eq. 2. We revise this typo in the line 170 of revised version.

**Overloaded $M$.** Using $M_c$ for the number of hypotheses conflicts with the memory $M_t$. We now write $\mathcal{H}(c) = \{h^{(c)}\_1, \dots, h^{(c)}\_{|\mathcal{H}(c)|}\}$ and remove $M_c$, making our writing more clear. We revise it in the line 183 of revised version.

**The update in Eq. 3.** Thanks for your pointing out. We revise Eq. 3 to $M_{t+1} = \text{Update}(M_t, a_t, o_{t+1})$. We also revise the paragraph to state that the action and the resulting observation $(a_t, o_{t+1})$ are written back via Eq. 3 in line 200 of revised version.

**Normalization and the reset to $1/|\mathcal{H}_{\text{sub}}(c)|$.** We thank the reviewer for the careful reading. The phrasing in Section 3.3 conflated two operations with different goals, which we clarify below.

1. Unnormalized probability for supporting evidence. BeliefMem doesn't normalize the probabilities of candidates for the same attribute, so the probabilities need not sum to one. When an observation supports $h$, the Noisy-OR update of $h$ depends only on $p_t^{(c)}(h)$ and $\Delta(o_{t+1}, h)$. The statement in line 266 refers to this property: adding or removing alternative candidates never changes the score that a supported $h$ receives.

2. **Reset for contradicting evidence.** Candidates of the same attribute are mutually exclusive, so evidence for one candidate is evidence against its competitors. Noisy-OR, however, can only increase a score. Without the reset, a competing candidate would keep its accumulated probability and could still outrank the newly supported candidate. The reset lowers each competing candidate to $1/|\mathcal{H}_{\text{sub}}(c)|$, the score under a uniform assignment over $\mathcal{H}_{\text{sub}}(c)$. The competing candidates thus carry no more confidence than an uninformed alternative, yet none is discarded, so later evidence can still restore them. The dependence on $|\mathcal{H}_{\text{sub}}(c)|$ is therefore intended and applies only to contradicted candidates.

We revise Section 3.3 to make them more clear in revised version.

**Belief-aware retrieval.** In BeliefMem, retrieval is belief-aware in what it returns: for each of the Top-K attributes, Eq. 10 returns all candidates together with their probabilities, rather than a single conclusion.

The ablation “w/o belief-aware retrieval” follows this definition. It keeps the ranking unchanged and only removes the probabilities from the returned candidates. This change alone reduces the average F1 on LoCoMo from 42.38 to 28.50.

**We keep $p$ out of the ranking by design.** When we directly use $p$ as a retrieval score to decide which candidates of each attribute $c$, our method will be degraded to deterministic paradigm. Because that means that the candidate with highest probability would be most likely retrieved, and then the agent will obtain the most confident conclusion of each attribute, namely deterministic paradigm.  With our design which provides all candidate with their probabilities, agents can know the uncertainty in external environment. For example, when the conclusion with the highest probability is wrong, agents can know alternative he can visit. Additionally, our experiments shown below, with shuffling the probability distribution of all candidates of each attribute, observe that wrong probability distribution can notably decrease the performance of BeliefMem, from 42.38 / 36.30 to 27.76 / 22.91, validating the effectiveness of our design.

| Method | Multi-hop | Temporal | Open-domain | Single-hop | AVG |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Shuffling Probabilities | 29.72 / 24.10 | 31.22 / 27.61 | 17.93 / 14.81 | 32.17 / 25.12 | 27.76 / 22.91 |
| **BeliefMem (ours)** | **40.51 / 32.24** | **51.88 / 45.78** | **28.73 / 25.12** | **48.41 / 42.07** | **42.38 / 36.30** |

----

**W2&Q1&Q2:** The ALFWorld success rates contain values that cannot be obtained from the stated number of episodes. SR is the number of solved episodes divided by the total number and times 100. Appendix C.1 states that the Seen set has 140 episodes and the Unseen set has 134. So the number of solved episodes equals 1.4×SR on Seen and 1.34×SR on Unseen, and this must be an integer. In Table 5, w/o Add has a Seen SR of 19.87. It gives 19.87×1.4=27.82 solved episodes. However, 27 solved episodes give 19.29% and 28 give 20.00%, so 19.87% is not possible.The same problem appears in 25.29, 42.32, 27.65 and 29.76 of Table 5, and in the 53.75 for Table 2. Moreover, B.3 reports only one seed 42, and the paper does not mention averaging over multiple runs. These values therefore cannot come from the reported setting. Furthermore, in Table 8, w/o decay and λ = 1.0 should be identical according to Eq. 9. However, the authors report different results. / How were the success rates in Table 2 and Table 5 computed? Why do most values correspond to an integer number of solved episodes while some do not? / In Table 8, how does w/o decay differ from λ = 1.0 in the implementation? According to Eq. 9, the two settings should be identical.

**A2:** We thank the reviewer for checking these values so carefully. We check each value back to its evaluation log and found two causes.

**Filling error in Table 2.** The Unseen SR of BeliefMem with the full corpus is 72/134 × 100 = 53.73, not 53.75. That error comes from the manually filling the numbers in latex.

**Incorrect metrics computations caused by API failures.** During the ablation runs of Table 5, some samples are interrupted by Qwen3-Next-80B-A3B-Instruct API failures. In this case, our evaluation would automatically restart each interrupted sample up to 3 times, but the success rate (SR) computation handled these samples incorrectly in two ways.

First, samples that still failed after 3 restarts are judged as invalid evaluation sample since they don't has evaluation results. Therefore, when calculating the SR, they aren't considered. That produces 26/94 × 100 = 27.65 on Seen and 25/84 × 100 = 29.76 on Unseen for w/o belief-based memory, and 22/87 × 100 = 25.29 on Unseen for w/o Add.

Second, for samples that completed after several restarting, every attempt of restarted samples is counted in denominator, so the denominator exceeds the size of the split. This affected 21 Unseen samples of w/o Merge and 9 Seen samples of w/o Add, each restarted 1-3 times. The resulting values were 80/189 × 100 = 42.32 and 31/156 × 100 = 19.87, respectively.

We also check the numbers of all other experiments in our work, and find that they are accurate. So, this metric computation errors only occur when running ablation study experiments due to unstable API rather than intentional things.

**Rerun of the affected settings.** we check the stability of API serive, and re-run the impacted experiments. The updated results are shown below. We will update Table 3, Table 5, and Appendix D.2 accordingly.

Method,Seen SR / #Steps,Unseen SR / #Steps,Avg. SR / #Steps
w/o belief-based memory,30.71% (43/140) / 37.84,24.63% (33/134) / 39.72,27.67% / 38.78
w/o Add,22.14% (31/140) / 39.19,29.10% (39/134) / 41.48,25.62% / 40.34
w/o Merge,39.29% (55/140) / 32.38,53.73% (72/134) / 32.52,46.51% / 32.45
BeliefMem (Ours),58.57% (82/140) / 29.77,61.19% (82/134) / 29.34,59.88% / 29.56

As shown above, after removing belief-based memory, the average SR of BeliefMem decreases from  59.88% to 27.67%, showing the superiority of probabilistic memory paradigm of BeliefMem compared with deterministic paradigm.
Additionally, w/o Add and w/o Merge degrade the performance of our method to 25.62% and 46.51%, respectively, showing consistent trend with analysis in our work. 

**λ = 1.0 versus w/o decay.** These two settings are identical under Eq. 9, since λ^τ = 1 for every τ when λ = 1.0. The two rows in Table 8 come from two separate runs of the same configuration. Their difference ( 2 of 140 Seen samples and 4 of 134 Unseen samples) comes from LLM API inference, which remains even at temperature 0. In the revision, we will merge them into a single row and revise the discussion in Appendix D.4.

Overall, our work firstly propose a probabilisitic memory paradigm in agent memory to address the self-reinforcing error, which can preserve the uncertainty of external environment, making the memory system more reliable and robust.

-----

----

**Q3:** 3.Which reset value is used for a competing candidate in Merge? Section 3.3 states 1/|H_sub(c)|, while Appendix B.2 states 0.25.

**A3:** Thanks for you pointing that. Since our the default setting of |H_sub(c)| is 4 mentioned in Appendix B.3 ("In addition, to reduce cost and latency in BeliefMem, we set the maximum number of candidate conclusions per attribute to 4 during retrieval."). Therefore, the Merge will reduce the competing conclusions' probabilities to 1 / 4 = 0.25.

We will refine the statement of this part to make it more clear in Appendix B.2 and B.3.

------

**Q4:** Since the retrieval score in Eq. 9 does not use the candidate scores p, does the “w/o belief-aware retrieval” ablation only remove the scores from the retrieved text given to the agent?

**A4**: Yes.  "w/o belief-aware retrieval" means that we totally removing the probabilities of all candidates of retrieved attributes. Therefore, under this setting, agents only receive the competing conclusions of retrieved attributes. Since competing conclusions may introduce comflciting information to agents' context, w/o belief-aware retrieval degrades the average performance of BeliefMem from 28.50 / 19.02 to 42.38 / 36.30. These results highlight the effectiveness of the probabilities distribution with candidates.


| Method | Multi-hop | Temporal | Open-domain | Single-hop | AVG |
| :--- | :---: | :---: | :---: | :---: | :---: |
| w/o belief-aware retrieval | 27.12 / 18.98 | 25.67 / 15.43 | 15.89 / 10.12 | 45.31 / 31.55 | 28.50 / 19.02 |

| **BeliefMem (ours)** | **40.51 / 32.24** | **51.88 / 45.78** | **28.73 / 25.12** | **48.41 / 42.07** | **42.38 / 36.30** |


We hope our responses have addressed your concerns. If you find them helpful, we would appreciate your consideration of increasing your score.



