{
  "title": "Agentic Flashcards",
  "cards": [
    {
      "front": "What is the primary limitation of the 'LLM-as-a-Judge' paradigm when evaluating complex, multi-step tasks?",
      "back": "It is constrained by shallow single-pass reasoning and an inability to verify assessments against real-world observations."
    },
    {
      "front": "Which paradigm transition addresses LLM biases by using agents equipped with planning, tool use, and memory?",
      "back": "Agent-as-a-Judge"
    },
    {
      "front": "How does the 'Agent-as-a-Judge' framework evolve robustness compared to monolithic LLM judges?",
      "back": "It shifts from monolithic inference to decentralized deliberation through autonomous multi-agent collaboration."
    },
    {
      "front": "In the Agent-as-a-Judge paradigm, what does the transition 'From Intuition to Execution' entail?",
      "back": "Replacing internal linguistic plausibility checks with objective verification via interaction with external environments."
    },
    {
      "front": "How does the Agent-as-a-Judge paradigm improve evaluation granularity?",
      "back": "By transforming single-pass inference into autonomous, hierarchical reasoning that provides fine-grained feedback on specific components."
    },
    {
      "front": "What characterizes a 'Procedural' Agent-as-a-Judge?",
      "back": "It operates through predefined agentic workflows or fixed protocols that cannot adapt to novel evaluation scenarios."
    },
    {
      "front": "Which development stage of Agent-as-a-Judge is defined by adaptive decision-making through routing and external tool invocation?",
      "back": "Reactive Agent-as-a-Judge"
    },
    {
      "front": "What is the hallmark of a 'Self-Evolving' Agent-as-a-Judge?",
      "back": "The high autonomy to refine internal components, such as synthesizing evaluation rubrics and updating memory on-the-fly."
    },
    {
      "front": "In multi-agent collaboration, what is the purpose of 'Collective Consensus' topologies like horizontal debate?",
      "back": "To leverage agents representing diverse perspectives to counteract the inherent parametric biases of single models."
    },
    {
      "front": "Which multi-agent topology involves organizing agents into structures to address varying error granularities?",
      "back": "Task Decomposition"
    },
    {
      "front": "What are the two primary perspectives for analyzing planning capabilities in agentic judges?",
      "back": "Workflow Orchestration and Rubric Discovery."
    },
    {
      "front": "In Agent-as-a-Judge methodologies, what does 'Rubric Discovery' involve?",
      "back": "The autonomous formulation and refinement of assessment criteria based on discovered context or web searches."
    },
    {
      "front": "What is the difference between 'Evidence Collection' and 'Correctness Verification' in tool integration?",
      "back": "Evidence collection gathers task-relevant artifacts, while verification checks if outputs satisfy explicit logical or factual constraints."
    },
    {
      "front": "How is 'Memory' utilized for intermediate state tracking in agentic evaluation?",
      "back": "It retains evaluation states across reasoning chains to support conditional routing and step-aware assessment."
    },
    {
      "front": "What is the role of 'Personalized Context' in Agent-as-a-Judge memory?",
      "back": "To preserve user-specific preferences and personas to ensure consistency across multiple interactions."
    },
    {
      "front": "How does 'Inference-Time Optimization' adapt judge behavior without updating model parameters?",
      "back": "By controlling judgments through prompts, structured workflows, or multi-agent interactions."
    },
    {
      "front": "What is the 'DevAI' benchmark specifically designed to evaluate?",
      "back": "The code-generation ability of agentic systems on 55 realistic, complete AI development tasks."
    },
    {
      "front": "In the DevAI benchmark, how much did Agent-as-a-Judge reduce evaluation costs compared to humans?",
      "back": "By approximately $97.64\\%$."
    },
    {
      "front": "What is the 'Alignment Rate' metric in AI judging?",
      "back": "The percentage of requirement evaluations that match the human-as-a-judge consensus."
    },
    {
      "front": "Define 'Judge Shift' in the context of automated evaluation.",
      "back": "The measurement of deviation from the human consensus results, where lower values indicate closer alignment."
    },
    {
      "front": "What is 'Mind2Web 2' designed to benchmark?",
      "back": "Agentic search systems on 130 long-horizon, time-varying web browsing and information synthesis tasks."
    },
    {
      "front": "In the Mind2Web 2 'Agent-as-a-Judge' framework, what are the two main aspects evaluated by a rubric?",
      "back": "Correctness and Source Attribution."
    },
    {
      "front": "How does the tree-structured rubric in Mind2Web 2 aggregate scores?",
      "back": "Leaf nodes perform binary judgments, which propagate upward to the root node following specific aggregation logic."
    },
    {
      "front": "What is the difference between 'Critical' and 'Non-critical' nodes in a tree-structured rubric?",
      "back": "Failure of a critical node immediately fails the parent, while non-critical nodes allow for partial scoring."
    },
    {
      "front": "What is 'Partial Completion' as defined in the Mind2Web 2 benchmark?",
      "back": "The average root node score across all tasks, reflecting fine-grained satisfaction of requirements."
    },
    {
      "front": "In the Mind2Web 2 results, which system type outperformed search-augmented LLMs and simple web agents?",
      "back": "Deep Research systems (e.g., OpenAI Deep Research)."
    },
    {
      "front": "What is a 'Process Reward Model' (PRM)?",
      "back": "A model that evaluates the correctness of individual intermediate reasoning steps rather than just the final outcome."
    },
    {
      "front": "Which benchmark tests long-horizon decision-making for web agents using forty thousand step-level preference pairs?",
      "back": "WebRewardBench"
    },
    {
      "front": "What is 'MAJ' (Multi-Agent Judging)?",
      "back": "An evaluation framework that uses multiple interacting small language models (SLMs) to approximate LLM-level judgment accuracy."
    },
    {
      "front": "In the 'JudgeBoard' study, what was the observed effect of MAJ on SLMs?",
      "back": "It substantially improved the reliability and consistency of smaller models, sometimes exceeding larger counterparts."
    },
    {
      "front": "What is 'Simple Test-Time Scaling' (STTS) in the context of LLM-as-a-Judge?",
      "back": "Allocating more computation at inference time to generate deeper reasoning traces and boost evaluation performance."
    },
    {
      "front": "According to the J1-7B study, at what stage is the STTS capability primarily acquired?",
      "back": "During the Reinforcement Learning (RL) phase."
    },
    {
      "front": "What is 'Reward Hacking' in the context of training with non-reasoning judges?",
      "back": "When a policy achieves high rewards from a training judge while performing poorly on a gold-standard evaluator."
    },
    {
      "front": "Why is 'Process-Level Supervision' critical for developing robust judges in non-verifiable domains?",
      "back": "It prevents reward hacking by exposing the judge to the gold-standard's internal reasoning process, not just final labels."
    },
    {
      "front": "What is the 'WebArbiter' framework?",
      "back": "A reasoning-first WebPRM that produces structured justifications and identifies actions most conducive to task completion."
    },
    {
      "front": "How does 'WebArbiter' address the limitations of scalar PRMs?",
      "back": "By providing interpretability through text generation and principles-guided reasoning instead of coarse scalar signals."
    },
    {
      "front": "What are the three roles in the 'SPELL' framework for long-context QA?",
      "back": "Questioner, Responder, and Verifier."
    },
    {
      "front": "How does the 'SPICE' framework improve reasoning during self-play?",
      "back": "By creating information asymmetry where the 'Challenger' has document access and the 'Reasoner' must solve tasks without it."
    },
    {
      "front": "What is the primary challenge related to 'Latency' in Agent-as-a-Judge systems?",
      "back": "Sequential reasoning, tool calls, and multi-agent communication introduce delays problematic for real-time applications."
    },
    {
      "front": "What safety risk is unique to 'Tool-augmented' judges?",
      "back": "Expanding the attack surface for prompt injection, tool misuse, or unintended side effects in external systems."
    },
    {
      "front": "In Agent-as-a-Judge, what is the 'Grounding Filter' used for in the SPELL framework?",
      "back": "To discard questions that can be answered using parametric memory without needing external documents."
    },
    {
      "front": "What does 'Krippendorff's Alpha' measure in AI evaluation studies?",
      "back": "The inter-annotator agreement between different judges (e.g., automated judges vs. a gold-standard)."
    },
    {
      "front": "In the Mind2Web 2 error analysis, what is 'Invalid Attribution'?",
      "back": "When an agent provides expired, fabricated, or incorrect URLs that do not support the claim."
    },
    {
      "front": "What is 'Synthesis Error' in agentic search evaluation?",
      "back": "When an agent correctly retrieves a webpage but distorts or incorrectly extracts the information from it."
    },
    {
      "front": "How does the Agent-as-a-Judge framework apply to the Legal domain in systems like 'AgentsCourt'?",
      "back": "By simulating adversarial debate between agents playing roles such as prosecutor, defense attorney, and judge."
    },
    {
      "front": "In Finance applications, what does 'SAEA' use agent trajectories for?",
      "back": "To audit deployment risks like hallucinations and temporal misalignment (staleness)."
    },
    {
      "front": "What is the 'Gate-then-average' strategy in Mind2Web 2 score aggregation?",
      "back": "Critical nodes act as binary gates; if they fail, the parent fails regardless of the average of non-critical nodes."
    },
    {
      "front": "How is 'Pass@k' calculated in agentic benchmarks?",
      "back": "As the probability that at least one of $k$ generated trajectories successfully completes the task."
    },
    {
      "front": "What is 'Self-Questioning' as a reinforcement learning baseline?",
      "back": "A method where ego agents generate counterfactual critiques of their own reasoning to guide policy fine-tuning."
    },
    {
      "front": "In the Mind2Web 2 study, what factor significantly improved system success in time-varying tasks?",
      "back": "The integration of real-time web browsing/browser interaction capabilities."
    },
    {
      "front": "Which benchmark uses 'Socratic-PRMBench' to analyze reasoning structure errors?",
      "back": "Process Reward Models survey."
    },
    {
      "front": "What distinguishes 'Agentic Benchmarks' from classical ML benchmarks?",
      "back": "Their emphasis on multi-step interaction, environment manipulation, and outcome verification in dynamic environments."
    },
    {
      "front": "What is the purpose of the 'Graph' module in the DevAI Agent-as-a-Judge implementation?",
      "back": "To capture the entire project structure, including files, modules, and dependencies, for deeper code understanding."
    },
    {
      "front": "In the DevAI ablation study, which module provided the most substantial performance boost for the judge?",
      "back": "The 'Locate' module, which targets files relevant to the specific requirements."
    },
    {
      "front": "What is the 'Black-box' setting in the DevAI judging experiment?",
      "back": "A scenario where the judge does not have access to the internal manually collected trajectory data of the evaluand."
    },
    {
      "front": "What is the primary privacy concern for Agent-as-a-Judge systems?",
      "back": "The risk of sensitive data leakage from persistent memory or personalized user interaction histories."
    },
    {
      "front": "What is 'Situational Preference' in LLM judgment?",
      "back": "A phenomenon where explicit rubrics help models maintain consistent preferences across different pairs of answers."
    },
    {
      "front": "In the context of multi-agent judging, what is 'Horizontal Debate'?",
      "back": "A protocol where agents discuss a response as equals to reach a consensus, often inspired by courtroom mechanics."
    },
    {
      "front": "Which benchmark evaluates social interaction strategies between two LLM agents?",
      "back": "SOTOPIA"
    },
    {
      "front": "How does the 'VisualSokoban' benchmark test agents?",
      "back": "It requires spatial reasoning and long-term planning to push boxes to target locations in a puzzle environment."
    },
    {
      "front": "In Agent-as-a-Judge, what is 'Workflow Orchestration'?",
      "back": "The strategic engine that manages how tasks are decomposed into sequences of sub-dimensions or actions."
    },
    {
      "front": "What is the 'Adaptive Router' agent in the AGENT-X framework?",
      "back": "An agent that dynamically selects the most relevant base agents based on intermediate analysis results."
    },
    {
      "front": "What characterizes the 'Relevance' attribute in LLM-as-a-Judge frameworks?",
      "back": "How well a response aligns with the user query, topic, or specific task context."
    },
    {
      "front": "In the 'Meta-Rewarding' framework, how does a model improve its judgment skills?",
      "back": "By using models to critique and refine their own evaluations through meta-judgment loops."
    },
    {
      "front": "What is the function of 'Critical Nodes' in the gating logic of Mind2Web 2?",
      "back": "They serve as gating conditions that must pass (score 1) for the parent node score to be meaningful or perfect."
    },
    {
      "front": "What does 'Cohen\u2019s Kappa' measure when benchmarking AI judges?",
      "back": "The statistical alignment or agreement between an LLM judge's prediction and a manual human judgment."
    },
    {
      "front": "Why is 'Information Asymmetry' used in self-play frameworks like SPICE?",
      "back": "To ensure that the solver model must rely on internalized reasoning rather than simply reading the ground truth from a document."
    },
    {
      "front": "In the Mind2Web 2 benchmark, what is the average number of nodes in a rubric tree?",
      "back": "50 nodes (with a maximum of 603)."
    },
    {
      "front": "What is the 'Success Rate' metric in agentic search?",
      "back": "The percentage of tasks where every criterion in the rubric is fully satisfied (root node score of 1)."
    },
    {
      "front": "In the survey, what is identified as the 'Strategic Engine' of the Agent-as-a-Judge paradigm?",
      "back": "Planning"
    },
    {
      "front": "What is the purpose of 'SFT Distillation' in training reasoning judges?",
      "back": "To teach a smaller model to mimic the chain-of-thought and judging patterns of a more powerful 'gold-standard' judge."
    },
    {
      "front": "In 'WebArbiter', what is the 'Two-stage pipeline' for training?",
      "back": "Reasoning distillation to learn principle-guided logic, followed by reinforcement learning to align verdicts with correctness."
    },
    {
      "front": "What is the 'ABC' in agentic benchmarking?",
      "back": "The Agentic Benchmark Checklist, a set of best practices for building rigorous agent evaluation suites."
    },
    {
      "front": "How does 'LocalSearchBench' evaluate agents?",
      "back": "Through 300 multi-hop QA tasks where agents must use LocalRAG and web-search APIs for life services queries."
    },
    {
      "front": "In the Mind2Web 2 study, what was the performance of OpenAI Deep Research compared to humans?",
      "back": "It achieved $50\\text{--}70\\%$ of human performance while spending half the time."
    },
    {
      "front": "What is the 'Verification of Human Annotation' step in the Mind2Web 2 validation process?",
      "back": "An expert review of discrepancies between human and judge-agent labels to ensure the automated judge is not penalizing valid answers."
    },
    {
      "front": "In the context of 'Meta-Rewarding', what is the 'Self-Taught Evaluator' approach?",
      "back": "Generating synthetic comparisons to train judgment skills without relying on human-labeled data."
    },
    {
      "front": "Which professional domain uses 'Grade-Like-a-Human' for staged evaluation processes?",
      "back": "Education"
    },
    {
      "front": "What is the 'Logic' attribute in LLM-as-a-judge taxonomy?",
      "back": "The internal coherence and correctness of reasoning steps within a response, independent of factual accuracy."
    },
    {
      "front": "In the Mind2Web 2 error analysis, what is 'Partial Missing'?",
      "back": "When an agent provides fewer items or procedural steps than explicitly required by the user prompt."
    }
  ]
}