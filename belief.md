## Reviewer Wh7d [2: Weak rejection]
-----
**W1&Q1:** Storing multiple conclusions with confidence scores can improve performance. However, BeliefMem relies on uncalibrated LLM confidence scores and heuristic Noisy-OR updates. It is unclear whether these updates reliably represent uncertainty, particularly when observations are correlated or contradictory. & Could the authors evaluate the calibration and robustness of belief scores, particularly under correlated or contradictory observations?

**A1:**  We thank the reviewer for this thoughtful comment. 

**BeliefMem assigns the highest belief score to true conclusion.** In experiments of Figure 4(b), we track the the Top-1 rate, defined as the fraction of attributes whose true conclusion receives the highest score. As shown in Figure 4(b) (also provided below), with gradual updates, our method BeliefMem rises the Top-1 rate from 7.69% to 84.62%, which means that 84.62% of true conclusions receive the highest belief score among all candidates. In contrast, Mem0 and A-MEM only improve the Top-1 rate to 46.15% and 42.31%, respectively. These results show that BeliefMem calibrate the belief scores reliably using Noisy-OR update over steps


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

Additionally, in Table 7 (provided below), Shuffling the belief scores of all candidates of each attribute reduces F1 from 40.51 to 29.72 on multi-hop questions and from 51.88 to 31.22 on temporal questions in LoCoMo benchmark. This experiments further show the effectiveness of belief scores in our work.

| Method | Multi-hop | Temporal | Open-domain | Single-hop |
| :--- | :---: | :---: | :---: | :---: |
| Shuffling Probabilities | 29.72 / 24.10 | 31.22 / 27.61 | 17.93 / 14.81 | 32.17 / 25.12 |
| **BeliefMem (ours)** | **40.51 / 32.24** | **51.88 / 45.78** | **28.73 / 25.12** | **48.41 / 42.07** |

**BeliefMem corrects the contradictory memory over noisy observations.** In experiments of Figure 3, we select several attributes of which contradictory memory receive the highest belief score. Over 10 steps (5 valid observations and 5 correlated observations), BeliefMem can refine 60.80% of flawed memory into the correct memory (the correct memory outranks the contradictory) with only average 4.75 steps. In contrast, prior deterministic memory method only refine 33.30% of contradictory memory into right memory while using averge 9.45 steps.  These results show the robustness of belief score in BeliefMem due to its memory update over noisy observations.

| Method | Correction Rate$up$ | Correction Steps$down$ |
| --- | --- | --- |
| BeliefMem | 60.80% | 4.75 |
| Deterministic memory system | 33.30% | 9.45 |

Overall, our work firstly proposes the probabilistic paradigm of memory system, enabling the memory system to preserve the uncertainty in POMDP, to address the self-reinforcing error. We believe BeliefMem can provide insights for future agent memory research.



-----

**W2&Q2:** BeliefMem is motivated by interactive agent settings, where memory influences future actions and observations. However, the evaluation primarily focuses on conversational QA, with ALFWorld as the only interactive agent benchmark. This provides limited evidence of its effectiveness in broader agentic settings. & Could the authors evaluate BeliefMem in more complex online agent settings where memory influences the agent's actions and subsequent observations?

**A2:** 跑一个额外的benchmark

----

**Q1:** Could the authors evaluate the calibration and robustness of belief scores, particularly under correlated or contradictory observations?
同A1

---

**Q2:** Could the authors evaluate BeliefMem in more complex online agent settings where memory influences the agent's actions and subsequent observations?

同A2

------

## Reviewer tp79 [2: Weak rejection]
----
**W1:** Misalignment between evaluation and core motivation. LoCoMo primarily evaluates long-term conversational memory and question answering, rather than agent self-evolution or self-reinforcing errors. It lacks the closed-loop interaction required to investigate how incorrect memories influence subsequent actions and observations. Although ALFWorld provides an interactive setting, the experiments do not sufficiently establish the claimed self-reinforcing error mechanism.

**A1:** 需要讨论一下这一点如何回复

----

**W2:** Insufficient analysis of noise from competing hypotheses. Preserving multiple candidate conclusions may introduce conflicting or misleading information during retrieval, potentially degrading downstream decisions. While the paper examines retrieval size, it does not systematically investigate how the number of conflicting candidates or their confidence distributions affect task performance. More targeted ablations are necessary.

**A2:** Thanks for your insightful comments on our work. We hope the following replies could address your concerns.

**Shuffling probability distirbution or removing probability notably degrade the performance of BeliefMem.** In Table 9, we conduct experiments where we shuffle the probability of each candidate give an attribute c. As shown in Table 9 (also provided below), shuffling probabilities results in notable performance degradation on LoCoMo benchmark, avergely degraing XXX. That experiments demonstrate that the reliable confidence distributions are the key in BeliefMem since they provide the corrent uncertinty in external environment. 

**Probabilistic memory addresses conflicting or misleading information preserved in multiple candidate conclusions.** In ablation studies, we remove the all probabilities of all candidates of each attribute c (w/o belief-aware retrieval). As shown below, removing these probabilities only achieve the average performance of 28.50 / 19.02 on LoCoMo benchmark. However, when BeliefMem with these probabilities, it improves the average performance from 28.50 / 19.02 to 42.38 / 36.30 (About XX%). These results show that preserving multiple candidate conclusions can impact the decisions of agents due to conflicting information over there. Our BeliefMem can handle this challenge by ranking these conflicting information using probabilities, enabling agents to know which candidate conclusion is most reliable.

| Method | Multi-hop | Temporal | Open-domain | Single-hop | AVG |
| :--- | :---: | :---: | :---: | :---: | :---: |
| w/o belief-aware retrieval | 27.12 / 18.98 | 25.67 / 15.43 | 15.89 / 10.12 | 45.31 / 31.55 | 28.50 / 19.02 |
| Shuffling Probabilities | 29.72 / 24.10 | 31.22 / 27.61 | 17.93 / 14.81 | 32.17 / 25.12 | 27.76 / 22.91 |
| **BeliefMem (ours)** | **40.51 / 32.24** | **51.88 / 45.78** | **28.73 / 25.12** | **48.41 / 42.07** | **42.38 / 36.30** |

Additionally, **BeliefMem assigns the highest belief score to true conclusion.** In experiments of Figure 4(b), we track the the Top-1 rate, defined as the fraction of attributes whose true conclusion receives the highest score. As shown in Figure 4(b) (also provided below), with gradual updates, our method BeliefMem rises the Top-1 rate from 7.69% to 84.62%, which means that 84.62% of true conclusions receive the highest belief score among all candidates. In contrast, Mem0 and A-MEM only improve the Top-1 rate to 46.15% and 42.31%, respectively. These results show that BeliefMem calibrate probabilties reliably using Noisy-OR update over steps.

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

-----

Q2: How does performance change when varying the number of distracting hypotheses and assigning incorrect or closely competing confidence scores?

同W2

------

Q3: How does BeliefMem differ from existing work on persistent memory poisoning and self-reinforcing agent errors?

讨论一下


------

Q4: If an incorrect high-confidence belief prevents the agent from exploring alternatives, what mechanism enables BeliefMem to escape this feedback loop?

同W4

-------

## Reviewer nZa7 [1: clear rejection]


**W1:** The notation is inconsistent in several places. First, the formula above eq. 3 is written as a_t ∼ π(· | o_t, z_t), but z_t is never defined. Second, the letter M is used for two different things. M_t in Eq. 2 denotes the external memory, while M_c in H(c) = {h_1^(c), ..., h_{M_c}^(c)} denotes the number of hypotheses. Third, Eq. 3 updates the memory as Update(M_t, o_t, o_{t+1}) without a_t. However, the paragraph on self-reinforcing error says that the transition (o_t, a_t, o_{t+1}) is written back to memory via Eq. 3. Fourth, in the paragraph on belief update, the authors keep scores unnormalised so that the score of a supported h is not affected by the number of alternative candidates for the same c. However, in Merge, a competing candidate is reset to 1/|H_sub(c)|, which depends directly on the number. Moreover, the retrieval score α_t(c) in Eq. 9 depends only on semantic similarity and time decay but p is not used. So retrieval ranking is not belief-aware.

**A1:**  1. a_t ∼ π(· | o_t, z_t)中的 z_t是typos 多写了，正确的应该是a_t ∼ π(· | o_t, m_t)
2. H(c) = {h_1^(c), ..., h_{M_c}^(c)} 是不严谨的地方，正确的应该改为 H(c) = {h_1^(c), ..., h_{｜H(c)｜}^(c)}
3. the transition (o_t, a_t, o_{t+1}) 应该使用 (o_t, o_{t+1})
4. Update(M_t, o_t, o_{t+1}) 正确 应该改成 Update(M_t, a_t, o_{t+1})
5.  BeliefMem keeps per-candidate probability {p^(c)_t(h)} instead of a normalized joint posterior over H_sub(c) 意外为 我们维护的belief representation 不是一个normalized的distribution， 即所有的prob的和为1； in Merge， 我们是为了保证已经存在的candidate的概率被更新之后，其他competitive conlusions的概率能够得到限制，从而保证每个candidate的概率不会无限制地被增加
6. 这里我们的方法是在召回完成之后，将各个attribute的所有的conclusions均召回并且伴随着每个的prob，这样子agent才会意识到一些attribute的不确定性；如果直接按照p召回，那么agent接受的召回的memory还是一个确定性的结论。

----

**W2:** The ALFWorld success rates contain values that cannot be obtained from the stated number of episodes. SR is the number of solved episodes divided by the total number and times 100. Appendix C.1 states that the Seen set has 140 episodes and the Unseen set has 134. So the number of solved episodes equals 1.4×SR on Seen and 1.34×SR on Unseen, and this must be an integer. In Table 5, w/o Add has a Seen SR of 19.87. It gives 19.87×1.4=27.82 solved episodes. However, 27 solved episodes give 19.29% and 28 give 20.00%, so 19.87% is not possible.The same problem appears in 25.29, 42.32, 27.65 and 29.76 of Table 5, and in the 53.75 for Table 2. Moreover, B.3 reports only one seed 42, and the paper does not mention averaging over multiple runs. These values therefore cannot come from the reported setting. Furthermore, in Table 8, w/o decay and λ = 1.0 should be identical according to Eq. 9. However, the authors report different results.

**A2:** Table2里面的 53.75属于是表格内容校准错误： 真实的数值是：72/134 * 100 = 53.73 ; 同时，我们也检查了 Table5实验的log，发现因为短暂的API fails 导致在评估时候样本出现失败并且尝试重启3次后也无法成功，导致在计算成功率的时候未纳入总样本量： 25/84 * 100 = 29.76, 22/87* 100 = 25.29； 26/94 * 100 = 27.65；

对于42.32，19.87， 对于部分的样本，重启成功，导致分别有21个样本跑了 1-3次 （w/o Merge on ALFWorld-Unseen） 和9个样本跑了 1-3次 （w/o Add on ALFWorld-Seen）  ，在计算运行总量的时候将这部分重复的run是纳入，从而导致分母超出134: 
80/189 * 100 = 42.32，31/156 * 100 = 0.1987；

感谢你指出这一点。 我们重新在这些setting下完整跑了一些实验

对于w/o decay和 lamada = 1.0 是完全一样的；并且我们可以看到，他们的表现是十分接近的。

-----

**Q1:** How were the success rates in Table 2 and Table 5 computed? Why do most values correspond to an integer number of solved episodes while some do not?

同A2

**Q2:** In Table 8, how does w/o decay differ from λ = 1.0 in the implementation? According to Eq. 9, the two settings should be identical.

同A2

----

**Q3:** 3.Which reset value is used for a competing candidate in Merge? Section 3.3 states 1/|H_sub(c)|, while Appendix B.2 states 0.25.

**A3:** Thanks for you pointing that. Since our the default setting of |H_sub(c)| is 4 mentioned in Appendix B.3 ([In addition, to reduce cost and latency in BeliefMem, we set the maximum number of candidate conclusions per attribute to 4 during retrieval.] ). Therefore, the Merge will reduce the competing conclusions' probabilities to 1 / 4 = 0.25.

We will refine the statement of this part to make it more clear in Appendix B.2 and B.3.

------

**Q4:** Since the retrieval score in Eq. 9 does not use the candidate scores p, does the “w/o belief-aware retrieval” ablation only remove the scores from the retrieved text given to the agent?

**A4**: 是的， 在召回之后，会retrieved memory会给出每一种可能的conclusions，但是不提供prob， 这使得agent无法measure被扣的可能性。 同时我们也做了打乱prob的实验，可以看到agent可以感知到这个一份probability的表现的影响

| Method | Multi-hop | Temporal | Open-domain | Single-hop |
| --- | --- | --- | --- | --- |
| Shuffling Probabilities | 29.72 / 24.10 | 31.22 / 27.61 | 17.93 / 14.81 | 32.17 / 25.12 |
| **BeliefMem (ours)** | **40.51 / 32.24** | **51.88 / 45.78** | **28.73 / 25.12** | **48.41 / 42.07** |

--------


