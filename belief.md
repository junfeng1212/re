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

