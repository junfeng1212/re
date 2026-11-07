## Reviewer Wh7d [2: Weak rejection]
-----
**W1:** Storing multiple conclusions with confidence scores can improve performance. However, BeliefMem relies on uncalibrated LLM confidence scores and heuristic Noisy-OR updates. It is unclear whether these updates reliably represent uncertainty, particularly when observations are correlated or contradictory.

**A1:** 这个首先使用 Fig3回复一下，我们向memory里面插入了完全相反但会被召回的记忆，但是在几个step的refine之后，我们发现BeliefMem完全可以修正这个错误的记忆，具体地，在召回阶段BeliefMem的召回记忆中 正确的记忆的rank完全超过了 之前人工插入的记忆。
第二这个校准的问题，展示一下最佳的概率的Conclusion是否很真实的一致

此外，还得说，我们做了移除probability的实验，打乱或者只给多种的结论反而会降低beliefmem的表现

-----

**W2:** BeliefMem is motivated by interactive agent settings, where memory influences future actions and observations. However, the evaluation primarily focuses on conversational QA, with ALFWorld as the only interactive agent benchmark. This provides limited evidence of its effectiveness in broader agentic settings.

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

**A2:** 用appendix里面打乱prob的实验回答这个问题，顺便做一个不同数量的conflicting candidates的影响的实验

------

**W3:** Incomplete discussion of related work. The paper would benefit from a more comprehensive comparison with recent studies on persistent memory corruption, poisoned experience retrieval, and error propagation in self-evolving agents. The distinction between BeliefMem and these approaches should be clarified.

**A3:** related work. 要和self evolving agent讨论

-----

**W4:** Unclear recovery mechanism under incorrect high-confidence beliefs. When an initially incorrect belief already dominates an agent's actions, the agent may avoid behaviors that could generate contradictory evidence. BeliefMem primarily updates confidence after recseiving new observation, but does not explain how corrective evidence can be actively obtained in such situations.

**A4:** Fig 4的实验回复这个问题

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


**W2:** The ALFWorld success rates contain values that cannot be obtained from the stated number of episodes. SR is the number of solved episodes divided by the total number and times 100. Appendix C.1 states that the Seen set has 140 episodes and the Unseen set has 134. So the number of solved episodes equals 1.4×SR on Seen and 1.34×SR on Unseen, and this must be an integer. In Table 5, w/o Add has a Seen SR of 19.87. It gives 19.87×1.4=27.82 solved episodes. However, 27 solved episodes give 19.29% and 28 give 20.00%, so 19.87% is not possible.The same problem appears in 25.29, 42.32, 27.65 and 29.76 of Table 5, and in the 53.75 for Table 2. Moreover, B.3 reports only one seed 42, and the paper does not mention averaging over multiple runs. These values therefore cannot come from the reported setting. Furthermore, in Table 8, w/o decay and λ = 1.0 should be identical according to Eq. 9. However, the authors report different results.

**A2:** in Table, the 53.75 for Table 2.


**Q1:** How were the success rates in Table 2 and Table 5 computed? Why do most values correspond to an integer number of solved episodes while some do not?

**Q2:** In Table 8, how does w/o decay differ from λ = 1.0 in the implementation? According to Eq. 9, the two settings should be identical.

**Q3:** 3.Which reset value is used for a competing candidate in Merge? Section 3.3 states 1/|H_sub(c)|, while Appendix B.2 states 0.25.

**Q4:** Since the retrieval score in Eq. 9 does not use the candidate scores p, does the “w/o belief-aware retrieval” ablation only remove the scores from the retrieved text given to the agent?



