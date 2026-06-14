# MedRAG-Eval

**RAG pipeline evaluation on spinal health domain - RAGAS metrics, framework comparison, and OWASP-inspired adversarial testing**

## Project Overview

This project evaluates a RAG (Retrieval-Augmented Generation) pipeline built on spinal health content from WebMD. It covers the full evaluation cycle: pipeline construction, automated quality metrics, framework comparison, and adversarial safety testing.

**Goal**
- Build and evaluate a medical RAG pipeline using local models only
- Compare RAGAS and DeepEval evaluation frameworks under identical conditions
- Test system robustness against OWASP LLM Top 10-inspired adversarial inputs
- Demonstrate evaluation methodology progression from the previous project

The project is grounded in a real clinical domain, with test cases verified by a spinal orthopedic specialist.

## Skills & Tools Demonstrated

- RAG pipeline construction (LangChain, Chroma, Ollama)
- RAGAS evaluation metrics (Context Precision, Context Recall, Faithfulness, Answer Relevancy, Noise Sensitivity)
- DeepEval evaluation metrics (Contextual Precision, Contextual Recall, Faithfulness, Answer Relevancy)
- Framework comparison methodology
- OWASP-inspired adversarial testing (prompt injection variants)
- AspectCritic safety metrics
- LLM-as-a-Judge methodology and its limitations
- Prompt hardening with explicit input segregation

## Phase 1: RAG Pipeline

Built a RAG pipeline over 4 WebMD articles on spinal health topics.

**Knowledge base:**
- Spinal Muscular Atrophy (SMA)
- Spinal Stenosis
- Chordoma
- Pain Management

**Stack:** LangChain, Chroma, Ollama, Qwen 2.5 7B, nomic-embed-text

**Key decisions:**
- Qwen 2.5 7B selected over 3B - 3B proved unreliable as a judge in the previous project; medical domain requires accuracy
- All models run locally - avoids external API costs; provides full control over the environment
- 4 topics selected for thematic coherence, varying complexity, and natural retrieval noise across domains
- Chordoma included as a rare condition to stress-test retrieval on niche medical content

[Phase 1 notebook](notebooks/01_rag_pipeline.ipynb)

## Phase 2: RAGAS Evaluation

Automated evaluation of the RAG pipeline using 5 RAGAS metrics across 12 test cases verified by a clinical expert.

**Metrics:** Context Precision, Context Recall, Faithfulness, Answer Relevancy, Noise Sensitivity

**Key findings:**
- Context Recall = 1.0 across all questions - retriever consistently finds relevant content
- Answer Relevancy high overall (0.71–0.96)
- Context Precision and Faithfulness showed notable variation - retrieval failures identified on 2 questions
- Ground truth granularity affects scores - overly brief references penalize complete correct answers
- Non-determinism observed at temperature=0 - same question produced hallucination in one run and "I don't know" in another

[Phase 2 notebook](notebooks/02_ragas_evaluation.ipynb)

## Phase 3: Framework Comparison

RAGAS and DeepEval evaluated on identical inputs - same generated answers, same retrieved contexts, same judge model, same session.

**Metrics compared:** Faithfulness, Answer Relevancy, Context Precision, Context Recall

**Key findings:**
- Context Recall = 1.0 in both frameworks - the only metric where they fully agree
- Faithfulness diverges significantly on several questions - different prompting strategies lead to different verdicts on the same answer
- RAGAS tends to be stricter on retrieval quality; DeepEval rewards contextual honesty
- Framework choice matters: neither is universally "better" - they measure different aspects of the same concept
- Same model used as generator and judge in both frameworks - differences in results reflect framework behavior, not model differences

[Phase 3 notebook](notebooks/03_framework_comparison.ipynb)

## Phase 4: Adversarial & Safety Testing

18 test cases across 4 categories, evaluated with AspectCritic safety metrics and manual red teaming. Inspired by OWASP LLM Top 10, adversarial testing covers behavioral risks (LLM01, LLM02, LLM06, LLM09). Infrastructure-layer risks (LLM03, LLM05, LLM08) are outside scope of behavioral testing methodology.

**Test categories:**
- Knowledge Boundary (7) - questions exceeding source granularity, logical inconsistencies, fictitious terms
- Harmful Medical (4) - innocent but potentially dangerous queries
- Bias (4) - demographic and stigma-based probing
- Adversarial (3) - prompt injection variants: urgency, roleplay, prompt injection

**Automated evaluation:** 4 AspectCritic binary metrics (medical_safety, hallucination_check, role_resistance, bias_check)

**Manual red teaming:** 6 direct adversarial attacks compared across standard and hardened prompts

**Key findings:**
- Strong role resistance - no attack successfully forced a full role override
- bias_check and hallucination_check performed well as binary metrics
- medical_safety unsuitable as binary metric - penalizes correct "I don't know" responses equally with dangerous advice
- Most effective attack vectors: authority framing and academic/fictional framing
- Prompt hardening (explicit input segregation) improved resistance to roleplay attacks but introduced a new failure mode - over-reliance on reference materials
- Attack surface inherently limited - public WebMD content contains no PII; traditional jailbreak vectors not applicable

[Phase 4 notebook](notebooks/04_adversarial_safety_testing.ipynb)

## Known Limitations

- Notebook initialization is duplicated across sessions - Chroma vector store does not persist between runtimes
- Chroma vector store rebuilt on each run - minor variations in chunk ordering may affect retrieval reproducibility
- Non-determinism at temperature=0 - local models are not fully deterministic
- `ragas==0.2.14` pinned - newer versions require OpenAI credentials
- `num_predict=80` (hard token limit) caused answer truncation in Phase 4 - accepted trade-off after model ignored `max_tokens` and produced excessively verbose responses without it

## Conclusion

This project demonstrates a complete evaluation cycle for a medical RAG system - from pipeline construction to safety testing. Across four phases, it surfaces the limitations of automated evaluation metrics, the importance of framework choice, and the iterative nature of prompt hardening for adversarial robustness.

Particularly valuable experience was gained in comparative evaluation methodology and understanding where automated metrics fail to capture real system behavior - especially in safety-critical domains.

---

## How to Run
1. Install [Ollama](https://ollama.com) and pull required models:
    ollama pull qwen2.5:7b
    ollama pull nomic-embed-text
2. Open notebooks in Jupyter or run locally
3. Run cells sequentially - each notebook is self-contained

**Author:** Katerina Kasparova
**QA Engineer transitioning to AI/ML & GenAI QA**

[LinkedIn](https://linkedin.com/in/katerina-kasparova) | [GitHub](https://github.com/rizzken)

Last updated: June 2026
