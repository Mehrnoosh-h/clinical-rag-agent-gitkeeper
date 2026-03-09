# 🏥 Clinical RAG Agent

An end-to-end **Retrieval-Augmented Generation (RAG)** pipeline that makes synthetic EHR clinical notes fully queryable in plain English — from raw PDF to an agentic chat interface.

> ⚠️ All patient data used in this project is entirely **synthetic and fictional**, generated for NLP/RAG testing purposes only. Do not use for clinical decision-making.

---

## 📌 Demo

Ask the assistant questions like:

| Question | Answer |
|----------|--------|
| *What was the patient's heart rate in the ER?* | 112 bpm |
| *What was the discharge diagnosis?* | Community-acquired pneumonia (right lower lobe) |
| *What antibiotics were given?* | Ceftriaxone 1g IV q24h + Azithromycin 500mg IV daily |
| *What are Avery's active medications?* | Metformin, Lisinopril, Atorvastatin, Sertraline, Fluticasone/salmeterol, Albuterol |

---

## 🏗️ Pipeline Overview

```
PDF → Clean → Chunk → Embed → FAISS Index → Retrieve → GPT-4o → Answer
                                                ↑
                                          Agent (tool-calling)
```

| Step | Description |
|------|-------------|
| 1. Data Loading | Extract text from a multi-section clinical PDF using PyPDF |
| 2. Text Cleaning | Strip repeated page headers and normalise whitespace |
| 3. Text Chunking | 3-pass hierarchical chunking (section → sub-section → merge) |
| 4. Embeddings & Index | Encode chunks with `MiniLM-L6-v2` and store in FAISS |
| 5. Retrieval | Cosine-similarity search with parent-context promotion |
| 6. Generation | Grounded Q&A with GPT-4o (`temperature=0`) |
| 7. Agentic RAG | Tool-calling agent (gpt-4o-mini) that decides what to search |
| 8. Gradio UI | Chat interface powered by the agent |

---

## 🔍 Chunking Strategy

One of the core design decisions in this project is the **3-pass chunking** approach, chosen specifically because clinical documents are dense and hierarchically structured:

1. **Section-level split** — divide on major document sections (Patient Snapshot, ED Note, Discharge Summary, etc.) to preserve clinical context boundaries.
2. **Sub-section split** — detect internal note headers (`HISTORY OF PRESENT ILLNESS`, `PHYSICAL EXAM`, `LABS`, etc.) and split further, improving retrieval precision.
3. **Merge pass** — adjacent sub-chunks within the same section are merged up to `MAX_CHARS=1200`, preventing overly small fragments that hurt retrieval quality.

This approach outperforms naive token-count chunking on clinical text where context boundaries carry meaning.

---

## 🤖 Agentic vs. Baseline RAG

The notebook implements **both** approaches so you can compare them directly:

| | Baseline RAG | Agentic RAG |
|---|---|---|
| Retrieval | Always triggered, fixed query | Model decides what to search |
| Tool use | None | `search_notes` tool via OpenAI function calling |
| Best for | Simple, direct questions | Ambiguous or multi-part questions |
| Model | GPT-4o | GPT-4o-mini |

---

## 🛠️ Tech Stack

- **PDF parsing** — `pypdf`
- **Embeddings** — `sentence-transformers` (`all-MiniLM-L6-v2`, 384-dim)
- **Vector search** — `faiss-cpu` (IndexFlatIP / cosine similarity)
- **LLM** — OpenAI `gpt-4o` (generation) + `gpt-4o-mini` (agent)
- **UI** — `gradio`
- **No LangChain** — pipeline is built from scratch for full transparency

---

## 📁 Repository Structure

```
clinical-rag-agent/
│
├── notebook/
│   └── clinical_notes_rag.ipynb     # Main notebook (full pipeline)
├── data/
│   └── synthetic_clinical_notes_for_rag.pdf
├── LICENSE
└── README.md
```

---

## 🚀 Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/Mehrnoosh-h/clinical-rag-agent.git
cd clinical-rag-agent
```

### 2. Install dependencies

```bash
pip install pypdf pymupdf sentence-transformers faiss-cpu openai gradio python-dotenv
```

### 3. Set your OpenAI API key

Create a `.env` file in the project root:

```
OPENAI_API_KEY=sk-...
```

### 4. Run the notebook

```bash
jupyter notebook notebook/clinical_notes_rag.ipynb
```

Run all cells top to bottom. The Gradio chat UI will launch automatically in the final cell.

---

## ⚙️ Configuration

Key constants in the notebook that you can tune:

| Constant | Default | Description |
|----------|---------|-------------|
| `EMBED_MODEL_NAME` | `all-MiniLM-L6-v2` | Sentence embedding model |
| `MAX_CHARS` | `1200` | Maximum characters per chunk |
| `MIN_CHARS` | `300` | Minimum before merging into previous chunk |
| `k` | `3` | Number of top chunks retrieved per query |

---

## 📄 License

This project is released under the MIT License. The clinical data used is entirely synthetic and carries no real patient information.
