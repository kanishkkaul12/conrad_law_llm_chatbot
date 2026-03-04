# Conrad – AI Contract Review & Analysis Assistant

Conrad is an AI-first legal assistant designed to help review, search, and analyse long-form contracts.
It uses Retrieval-Augmented Generation (RAG) built on top of LangChain, FAISS vector search, and a Chainlit UI to let users ask natural language questions about contracts and receive grounded, clause-level answers.

> **Note:** This was a 3-person team project. See [My Contributions](#my-contributions--kanishk-kaul) section for a breakdown of what I specifically built.

---

## 1. Project Goals

Conrad is built to:

- Reduce time spent manually reading long contracts
- Surface relevant clauses and obligations given a free-form query
- Provide concise, plain-language summaries of complex legal language
- Enable experimentation with modern RAG patterns on real-world contract data (CUAD, ContractNLI)

This project is a proof-of-concept for AI-assisted contract analysis, not a production legal tool.

---

## 2. Key Features

- **Retrieval-Augmented Generation (RAG)**
  Uses FAISS as a vector store to retrieve the most relevant contract chunks before answering.

- **Semantic Clause Search**
  Ask questions like *"What are the termination rights?"* or *"Who bears data protection liability?"* and Conrad retrieves and quotes the most relevant clauses.

- **Summarisation and Explanation**
  Produces short, interpretable summaries of dense clauses to make contracts easier to understand for non-lawyers.

- **Multi-Document Support**
  Index and query multiple contracts or templates in one vector store.

- **Interactive Chat UI (Chainlit)**
  Chat-style interface to interact with the assistant, test prompts, and refine questions in real time.

- **Optional Observability via LangSmith**
  If configured, traces each run for debugging, prompt iteration, and evaluation.

---

## 3. Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.10+ |
| RAG Framework | LangChain |
| Vector Store | FAISS (local, disk-persisted) |
| LLM / Embeddings | OpenAI (configurable via `.env`) |
| Chat UI | Chainlit |
| Observability | LangSmith (optional) |
| Data | CUAD, ContractNLI |

---

## 4. Repository Structure

```
conrad_law_llm_chatbot/
├── cuad_faiss_index/        # Persisted FAISS index built on CUAD dataset
├── data/                    # Raw and processed contract datasets
├── notebook/                # Experimentation and analysis notebooks
├── src/                     # Core pipeline code
├── cuad_analysis.ipynb      # CUAD dataset exploration and indexing
├── test.py                  # Evaluation and prompt testing scripts
├── requirements.txt         # Python dependencies
└── README.md
```

---

## 5. Getting Started

```bash
# 1. Clone the repo
git clone https://github.com/kanishkkaul12/conrad_law_llm_chatbot.git
cd conrad_law_llm_chatbot

# 2. Install dependencies
pip install -r requirements.txt

# 3. Set your environment variables
cp .env.example .env
# Add your OpenAI API key to .env

# 4. Run the app
chainlit run app.py
```

---

## 6. My Contributions — Kanishk Kaul

This was a collaborative 3-person project. My specific contributions were:

### System Architecture
Designed the end-to-end RAG pipeline — including document chunking strategy, embedding model selection, retrieval flow, and how context is injected into the LLM prompt. The goal was to ensure retrieved chunks were semantically relevant and small enough to avoid context window issues while retaining full clause meaning.

### Vector Database Construction
Built and optimized the FAISS index on the CUAD dataset (`cuad_faiss_index/`). This involved:
- Preprocessing and chunking 500+ contracts from the CUAD dataset
- Generating embeddings using OpenAI's embedding models
- Persisting the FAISS index to disk for fast reuse across sessions
- Experimenting with chunk size and overlap to balance retrieval precision and recall

### Prompt Engineering
Designed, tested, and iterated on prompt strategies to improve answer grounding and reduce hallucinations. Key experiments included:
- Varying system prompt instructions (strict grounding vs. interpretive summarization)
- Testing few-shot examples for legal clause explanation tasks
- Adjusting how retrieved context is formatted and ordered in the prompt
- Evaluating outputs qualitatively across different query types (factual, interpretive, enforcement scenarios)

---

## 7. Product Decisions & Learnings

### Why RAG over fine-tuning?
Fine-tuning requires large labeled datasets and retraining whenever the contract domain changes. RAG lets the system generalize to unseen contracts at query time without retraining — a far better fit for a product serving diverse legal documents across industries. It also makes the system's answers auditable since they're grounded in retrieved source text.

### Why FAISS?
For a local proof-of-concept, FAISS offered fast approximate nearest-neighbor search without requiring a hosted vector database service. In a production system, this would move to Pinecone or Weaviate for scalability, multi-tenancy, and metadata filtering.

### Key Failure Modes Identified
Through structured prompt testing, we identified three main failure categories:

| Failure Mode | Description | Mitigation Applied |
|---|---|---|
| Retrieval mismatch | Vague queries without legal context produced off-target chunk retrievals | Added query reformulation step in prompt |
| Chunking boundary issues | Long clauses split across chunks degraded answer coherence | Tuned chunk size and overlap parameters |
| Hallucination on gaps | Model filled in missing information when retrieved context was insufficient | Added explicit grounding instruction: *"Only answer from the provided context"* |

### What I'd Build Next
- **Confidence scoring** — a layer that flags low-certainty answers so users know when to seek human review
- **Structured evaluation harness** — using RAGAS or a custom eval suite to measure retrieval precision, answer faithfulness, and context relevance at scale
- **Clause comparison** — allow users to diff two contract versions and highlight material changes
- **Risk summarisation** — auto-generate a risk summary flagging high-risk clauses (indemnity, IP ownership, termination, liability caps) at the top of each session

---

## 8. Dataset References

- **CUAD** (Contract Understanding Atticus Dataset) — 510 contracts with 13,000+ expert-labeled clause annotations across 41 clause types. [Paper](https://arxiv.org/abs/2103.06268)
- **ContractNLI** — Natural language inference dataset for contracts, teaching models logical entailment relationships between clauses. [Paper](https://arxiv.org/abs/2110.01799)

---

## 9. Disclaimer

Conrad is a research and learning project. It is not a substitute for professional legal advice. Do not rely on its outputs for real legal decisions.
