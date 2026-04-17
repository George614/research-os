# Agent Quiz

## Question 1
In the context of the shift from LLM-as-a-Judge to Agent-as-a-Judge, which characteristic defines the 'Self-Evolving' stage of development?

- [x] The ability to synthesize and refine internal evaluation rubrics on-the-fly during operation.
- [ ] The use of predefined agentic workflows to decouple monolithic inference into structured steps.
- [ ] The implementation of adaptive routing within a fixed decision space based on intermediate feedback.
- [ ] The employment of multi-agent debate protocols to mitigate the inherent parametric biases of a single model.

**Hint:** Focus on the ability of the judge to modify its own internal judgment standards dynamically.

## Question 2
The Mind2Web 2 benchmark introduces a tree-structured rubric design. What is the primary function of 'critical nodes' within this aggregation logic?

- [x] They act as gating conditions that determine if the scores of non-critical child nodes should be meaningfully aggregated.
- [ ] They calculate the weighted average of all leaf nodes to provide a fine-grained continuous score.
- [ ] They enable parallelized retrieval strategies to locate information across the vast online landscape.
- [ ] They serve as memory markers to track historical reasoning states across multi-step search horizons.

**Hint:** Consider how a failure in a fundamental requirement affects the validity of subsequent, more granular assessments.

## Question 3
According to research on non-verifiable LLM post-training, what was the observed effect of using 'non-reasoning' judges compared to 'reasoning' judges?

- [x] Policies trained with non-reasoning judges exhibited severe reward hacking, achieving high rewards despite declining gold-standard scores.
- [ ] Non-reasoning judges demonstrated higher inter-annotator agreement with human consensus than reasoning judges on static benchmarks.
- [ ] Non-reasoning judges were more effective at identifying 'situational preferences' in complex, multi-turn dialogues.
- [ ] Policies trained with non-reasoning judges required less computational overhead while achieving comparable generalization.

**Hint:** Think about the consequences of evaluating final outputs without validating the intermediate steps of the model's logic.

## Question 4
How does the DevAI benchmark address the limitations of existing code generation benchmarks like SWE-bench?

- [x] By providing hierarchical solution requirements that allow agentic judges to provide intermediate feedback rather than just outcome-based scores.
- [ ] By focusing exclusively on algorithmic efficiency and functional correctness via deterministic unit tests.
- [ ] By utilizing a fixed sandbox environment that prevents agents from interacting with external APIs or real-time web data.
- [ ] By measuring the 'pass@k' metric across thousands of identical prompts to ensure statistical significance.

**Hint:** Consider the difference between looking at a final 'resolve rate' versus evaluating the steps taken to build a complete system.

## Question 5
The concept of 'Generation-Verification Asymmetry' is central to the Mind2Web 2 framework. Which statement best explains this principle?

- [x] While generated answers can vary substantially in structure and time-varying content, the underlying task requirements remain easy for an agent to verify.
- [ ] Verification requires significantly more computational power than generation, necessitating the use of Small Language Models (SLMs) as judges.
- [ ] The agent can generate multiple solutions but is inherently unable to verify the logical consistency of its own reasoning path.
- [ ] High-quality human verification tokens are required to train a judge, whereas generation tokens can be synthetically produced.

**Hint:** Think about the relationship between the diversity of possible correct answers and the fixed nature of the task's criteria.

## Question 6
What is the primary role of the 'Memory' module in Agent-as-a-Judge frameworks as described in Section 3.4 of the Survey?

- [x] To retain intermediate states and personalized context that support multi-step reasoning and consistent judgment across interactions.
- [ ] To store the model's parametric weights to prevent catastrophic forgetting during supervised fine-tuning.
- [ ] To cache external tool responses to reduce the latency associated with API calls during real-time verification.
- [ ] To allow the judge to simulate human-like cognitive fatigue to better align with the success rates observed in human benchmarks.

**Hint:** Consider how a judge handles a task that takes dozens of steps over a long duration.

## Question 7
In the 'CourtEval' or 'ChatEval' methodology, how is 'collectively robustness' achieved?

- [x] Through decentralized deliberation where distinct roles, such as prosecutors and defense attorneys, expose conﬂicting arguments.
- [ ] By aggregating scalar scores from thousands of independent Small Language Models (SLMs) using a simple majority vote.
- [ ] By ensuring the judge model has no access to its own previous output patterns to prevent self-favoritism bias.
- [ ] Through the injection of random noise into the input prompt to test the stability of the model's judgment across different perturbations.

**Hint:** Think about legal proceedings and how different perspectives contribute to a final decision.

## Question 8
Which metric is used to measure the deviation of an AI judge's results from the consensus reached by human expert evaluators?

- [x] Judge Shift
- [ ] Alignment Rate
- [ ] Krippendorff's Alpha
- [ ] Constraint Success Rate (CSR)

**Hint:** This term implies a movement or divergence from a baseline standard.

## Question 9
According to the Mind2Web 2 study, how did agentic search systems perform relative to human participants on 'tedious' tasks requiring high attention to detail?

- [x] They sometimes outperformed humans, who were prone to oversight and carelessness due to cognitive fatigue in long-horizon tasks.
- [ ] They failed significantly because humans can visit up to 375 webpages in an hour, a speed agents currently cannot match.
- [ ] They showed a systematic bias toward time-invariant tasks, whereas humans were unable to handle real-time, time-varying information.
- [ ] They were found to be 97% less expensive, but consistently 20% less reliable than an individual human judge.

**Hint:** Consider the limitations of human working memory and the repetitive nature of web search.

## Question 10
In the Agent-as-a-Judge framework, what is the critical difference between 'evidence collection' and 'correctness verification' tools?

- [x] Evidence collection gathers observable artifacts like execution results, while correctness verification checks logical or factual consistency.
- [ ] Evidence collection is used for training-time optimization, whereas correctness verification is exclusively for inference-time scaling.
- [ ] Correctness verification requires a 'Gold-Standard' oracle, whereas evidence collection relies on multi-agent debate.
- [ ] Evidence collection is restricted to text-based environments, whereas correctness verification uses multimodal signals.

**Hint:** Distinguish between the act of finding information and the act of validating its accuracy.

## Question 11
The SPELL and SPICE frameworks utilize 'Self-Play' for alignment. Which component in the SPELL architecture is responsible for reducing variance in the reward signal?

- [x] An ensemble-based verifier that aggregates multiple independent binary judgments through majority voting.
- [ ] A 'Challenger' model that generates tasks specifically designed to maximize the probability of solver failure.
- [ ] A history memory $H$ that stores only the most recently solved question-answer pairs for curriculum learning.
- [ ] The information asymmetry between the 'Questioner' and 'Responder' roles during document grounding.

**Hint:** Think about how multiple evaluations of the same answer could lead to a more stable final score.

## Question 12
Which of the following is a key reason why LLM-as-a-Judge is considered inadequate for evaluating modern agentic search systems?

- [x] The long search horizons and complex citation-backed answers outpace the cognitive window of single-pass, static LLM evaluation.
- [ ] Traditional LLM judges are unable to generate scores on a continuous scale, resulting in only binary success/failure outputs.
- [ ] LLM-as-a-Judge requires access to the internal weights of the model being evaluated, which is often commercially restricted.
- [ ] Static benchmarks have already been fully memorized by most frontier models, rendering single-pass scores meaningless.

**Hint:** Think about the depth and duration of the task compared to the nature of a single prompt-response interaction.
