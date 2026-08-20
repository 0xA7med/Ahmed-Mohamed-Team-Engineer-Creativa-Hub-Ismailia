# Esnad (إسناد) — Clinical RAG for the WHO Hypertension Guideline

&gt; **Ask your documents — get cited answers. Zero hallucination.**

Esnad is a clinical question-answering assistant built with Retrieval-Augmented Generation (RAG) over the **WHO Guideline for the Pharmacological Treatment of Hypertension in Adults (2021, 61 pages)**. Ask in plain language — every answer is cited to the exact section and page, strictly from the guideline. Out-of-scope or patient-specific dosing questions are **safely refused, never invented**.

The name carries both meanings of the Arabic *isnād*: the chain of documented attribution — and a companion you lean on.

Built for the **AI Clinical Decision Support Lite Hackathon — Creativa Hub Ismailia**
**Team Engineer** · Ahmed Mohamed · 2026

---

## 🎯 Final measured results

| Area | Metric | Score |
|---|---|---|
| Retrieval | Precision@3 / Precision@5 | 0.667 / 0.471 |
| Retrieval | Top-1 Accuracy | 0.643 |
| Retrieval | Hit Rate@5 | **1.000** |
| Generation | Faithfulness (LLM-as-judge) | 0.981 (53/54 claims) |
| Generation | Citation Integrity / Coverage | 1.0 / 1.0 |
| Safety | Refusal Accuracy | **1.000 (6/6 behaviors)** |

Every number is reproducible from the evaluation outputs in this repository — nothing is hand-typed.

---

## 🏗️ Architecture

    WHO Guideline PDF
          │
          ▼
    ① Parsing — Docling (layout-aware; dual export: provenance-rich JSON + readable Markdown)
          │
          ▼
    ② Chunking — HybridChunker (section-aware; 150 chunks, ≤512 tokens, 100% metadata)
          │
          ▼
    ③ Embeddings — BGE-M3 (1024-dim semantic vectors)
          │
          ▼
    ④ Retrieval — FAISS dense top-k
          │
          ▼
    ⑤ Generation — Qwen2.5-7B-Instruct (4-bit), grounded with [Source N] citations
          │
          ▼
    ⑥ Safety — 3 layers: risk classifier → grounding rules → confidence guard

No monolithic frameworks — every stage is wired by hand so each one stays measurable, replaceable, and explainable.

---

## 🧪 Key engineering decisions — all backed by measurement

1. **Evaluation before building.** A golden set of 15 clinical questions (written from the guideline, with known answers) was used to compare **5 retrieval variants** on identical questions, chunks, and index. **Dense-only retrieval won** (P@5 0.471, Top-1 0.643); hybrid BM25+RRF and cross-encoder reranking added complexity with no gain.

2. **The reranker paradox.** The re-ranker *failed* at ranking (Top-1 dropped 0.64 → 0.57) — but its raw scores cleanly separated in-scope (min 0.479) from out-of-scope (max 0.129) questions. It was **repurposed as a confidence guard** with data-derived thresholds: refuse below **0.304**, high confidence above **0.987**.

3. **Twin chunks.** The guideline's executive summary repeats body-text recommendations almost verbatim, flooding top results with near-duplicates. Measured insight: deduplication *during ranking* hurt Precision@K (0.47 → 0.36), so dedup (Jaccard ≥ 0.7) was **moved to context assembly** — the model sees 5 unique sources instead of repeated ones.

---

## 🛡️ Safety by design

- **Risk classifier** — emergencies are redirected to care; patient-specific cases get a caution label
- **6-rule grounding prompt** — answer only from sources, cite every claim, say when it's not there, no off-document advice, no patient-specific dosing, always add a medical disclaimer
- **Confidence guard** — scores below 0.304 trigger a safe refusal with a clear reason

Tested on 6 risky scenarios: **6/6 correct behaviors**.

---

## 📱 Live demo

The notebook ships with an **Esnad-branded Gradio web UI** (last cell): the guideline is loaded, analyzed, and ready for questions — with colored status badges (answered / general guidance / safely refused / redirected) and evidence cards showing section + page for every answer.

---

## 🧰 Tech stack — 100% open source

| Tool | Role | Why |
|---|---|---|
| Docling | PDF parsing | Layout-aware; preserves real page numbers — the basis of page-level citations |
| HybridChunker | Chunking | Section-aware splitting; 150 chunks ≤ 512 tokens |
| BGE-M3 | Embeddings | Strong multilingual retrieval; 8192-token context |
| FAISS | Vector index | Exact similarity search in milliseconds, fully local |
| rank_bm25 | Keyword baseline | Classic BM25 for the hybrid variant |
| bge-reranker-v2-m3 | Cross-encoder | Failed at ranking → repurposed as the confidence guard |
| Qwen2.5-7B-Instruct (4-bit) | Generation | Open-source, follows strict rules, runs on a free Kaggle GPU |
| Gradio | Demo UI | Esnad phone-style interface |

Runs entirely on free Kaggle GPUs. No API keys. No per-query costs.

---

## 🚀 How to run

1. Open the notebook in **Kaggle**
2. Settings → **Internet: On**, **Accelerator: GPU T4** (or P100)
3. Attach the WHO Hypertension Guideline PDF as input data
4. **Run → Restart & Run All**
5. The last cell launches the Esnad demo UI (public link via `share=True`, valid 72h)

---

## 📁 Repository structure

    ├── Ahmed-Mohamed-Team-Engineer-Creativa-Hub-Ismailia.ipynb   # main notebook (all stages)
    ├── README.md
    ├── data/
    │   └── WHO Hypertension Guideline.pdf                       # source document
    ├── outputs/
    │   ├── chunks.json                                          # 150 chunks with full metadata
    │   ├── eval_baseline_dense.json                             # golden-set retrieval scores
    │   ├── eval_variant_comparison.json                         # all 5 retrieval variants
    │   ├── safety_battery_results.json                          # 6/6 safety scenarios
    │   └── eval_report_card.json                                # final metrics
    └── presentation/
        └── pitch_deck.pptx                                      # Esnad pitch deck

---

## 💼 Business model

B2B subscription per facility (hospitals, clinic networks, telemedicine platforms, medical education centers), plus an on-premise licence for institutions requiring full data control. Fully open-source stack → no per-query API costs, no patient data leaves the facility. The same verified engine serves any document-heavy domain — legal, education, engineering — by swapping the input documents, with zero code changes.

---

## 👤 Author

**Ahmed Mohamed** — Team Engineer · Creativa Hub Ismailia
AI Clinical Decision Support Lite Hackathon · 2026
