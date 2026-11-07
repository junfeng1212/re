## Reviewer idAb

Thanks for your comments on our work, which is curcial to make our work more clear and solid.

**W1:** The evaluation is narrow relative to the deployment claims. The 416 cases derive from only 104 independent tasks, with original trajectories averaging approximately 13.89 steps. The evaluation primarily uses one executor model, injected rollback failures, and a known recovery step.

**A1:** Appreciate your thoughtful comments on our work. We hope the following replies could address your concerns.

**416 cases derive from only 104 independent tasks & averaging approximately 13.89 steps.** 这个地方主要说明现在Agent的评估需要很大的成本，这个成本无论是从运行的时间，还有运行花费的金钱上面来考虑，都是巨大的。所以呢在这里我们一共制造出来416个样本，这416个样本其实对于一份Agent for task来说，凭空的来说，它已经算比较大了。同时我们相对normal水平的stop数，它也是它也是一个对于评估成本控制的一个考量。如果在这种相对简单，stop数相对少的任务上都没有办法表现很好的话，那么在更长任务上的难度只会更大。

**uses one executor model, injected rollback failures, and a known recovery step.** 这里我们其实主要是为了考虑，我们得用这个benchmark得提供一个可控的环境，这个环境它能够独立的评测Memory System在Task Recovery Ability上面的能力，而不会受到其他的因素，比如Base Model的能力或者Agent的设计的影响，这样有助于我们分析。例如我们这里用的是GPT 4O，收集到它所有可以解决的数据。然后我们通过人主动的截断任务和插入errors来构造出task resumption和task rollback的场景，这样能够分析出Memory system在这个setting下对于模型Agent的表现的影响，而不会因为模型的强弱影响到最终的结果。

我们的benchmark它其实主要就是提出了一个两个setting，它能够评测被过去benchmark忽略的一个memory system的能力，也就是task recovery ability。同时我们，我觉得我们认为我们的贡献不仅仅在于这一个数据集，而是在于构建评测这个问题的pipeline，即如何构造一个可控的环境和数据。


**W1:** The evaluation is narrow relative to the deployment claims. The 416 cases derive from only 104 independent tasks, with original trajectories averaging approximately 13.89 steps. The evaluation primarily uses one executor model, injected rollback failures, and a known recovery step.

**A1:** We appreciate your thoughtful comments on our work. We hope the following responses address your concerns.

**416 cases derive from only 104 independent tasks & averaging approximately 13.89 steps.** The scale of RecoverMemBench reflects the high cost of evaluating LLM agents. Each recovery case requires building the memory from the recorded trajectory and then running a complete agent execution in an executable environment. Both the running time and the monetary cost therefore grow with the number of cases, the number of execution steps per case, and the number of evaluated memory systems. For example, evaluating the 13 configurations in Table 1 on the 416 cases requires more than 5,000 recovery runs. Given this cost, 416 cases constitute a relatively large evaluation set for agent tasks. For the same reason, we build the benchmark from tasks with a moderate number of steps, which keeps the overall evaluation cost manageable. Moreover, because of this moderate length, our results are likely to underestimate the difficulty of task recovery. Existing memory systems already fail to recover reliably on tasks of about 14 steps, so recovery on longer tasks is expected to be even harder. Longer trajectories produce more records that describe the same task objects at different stages of execution, and this is precisely the condition under which semantic retrieval fails in our analysis (Section 3.2).

**uses one executor model, injected rollback failures, and a known recovery step.** These design choices are deliberate. RecoverMemBench aims to provide a controlled environment that evaluates the task recovery ability of memory systems independently of other factors, such as the capability of the base model and the design of the agent. To this end, we fix the executor to GPT-4o and retain only the tasks that it resolves in all three uninterrupted runs. We then construct the two settings from these successful trajectories. For task resumption, we interrupt each trajectory at predetermined steps. For task rollback, we inject an error after a sampled step and, once the run fails, restore the environment to that step. Because the executor solves every retained task without interruption, a failure after recovery can be primarily attributed to the memory system rather than to the limited capability of the executor. In addition, fixing the recovery step ensures that all memory systems are evaluated from identical recovery points. The differences in recovery rate and additional steps therefore reflect differences between the memory systems themselves. This controlled design also enables the analysis in Section 3.2, which traces recovery failures to unreliable semantic retrieval and to inconsistencies between memory and the environment.

More broadly, the main contribution of RecoverMemBench is its two settings, task resumption and task rollback. Together, they evaluate task recovery, a capability of memory systems that prior benchmarks overlook. Beyond the dataset itself, we believe that the pipeline for constructing this evaluation is also a key contribution. The pipeline specifies how to build a controlled environment and data for task recovery: selecting tasks that the executor solves reliably, interrupting their trajectories at controlled steps, and injecting errors followed by environment rollback. None of these steps is tied to a particular executor model. The same pipeline can therefore be applied to other executors and to other domains in which environment states can be restored.

**W2:** The comparison with version-aware baselines materially qualifies the method advantage.

**A2:** Thanks for your insightful views for our work. 

**RecoverMemBench uncovers the piror work's drawbacks on task recovery ability.** 这一段主要是强调我们的核心贡献，我们提出了这个Benchmark，然后通过我们的Benchmark分析现在方法的不足。 基于这些不足，我们提出了一个直接的方法来解决这个问题。[ 这里的思路主要是为了强调我们核心的contribution是这个Benchmark，而不是这个方法]

**Although version-aware baselines are capable of RecoverMem, they incur substantial storage requirements.** 过去的方法，他们使用这个Version-Aware的版本之后，他们虽然能够和我们的方法有差不多表现，但是他们会带来上千倍、几十倍的额外的存储成本。Agent memory是一个相对practical的方法，所以这是一个无法被忽略的问题。我们也觉得这也是未来我们需要研究的一个问题。[这里的思路主要是强调，虽然这些被hack和我们方法的表现差不多，但是他们需要额外上，几十倍甚至上百倍的成本。]



Q1: How would RecoverMem identify valid state when there is no reliable global rollback step?



说明在我们的setting， 这个rollback的能力是支持哪些“在Agent在任务上失败之后，我们分析Agent的 Trajectory能够拿到在哪一个step的表现不佳，从而rollback的前提是 我们得知道哪一步出现了问题” 【强调practical的场景】

Q2: How can the system retain knowledge from a failed suffix without reintroducing invalid progress?
解释方法的机制