# Clinical RAG for the WHO Hypertension Guideline

**AI Clinical Decision Support Lite Hackathon — Creativa Hub Ismailia**
**Team:** Engineer &nbsp;|&nbsp; **Member:** Ahmed Mohamed

An end-to-end clinical Retrieval-Augmented Generation (RAG) system built over the WHO
"Guideline for the pharmacological treatment of hypertension in adults" (2021).
It answers clinical questions with cited, page-referenced evidence, and refuses
out-of-scope or unsafe requests.

## Architecture
WHO Guideline PDF
│
▼
Docling layout-aware parsing (lossless JSON + provenance)
│
▼
HybridChunker — section-aware chunking, 512 tokens max
150 chunks, 100% metadata coverage (chunk_id, section_title, pages, source)
│
▼
BGE-M3 embeddings (1024-dim, normalized) → FAISS IndexFlatIP
│
▼
Dense retrieval (selected by measurement over 5 variants)
│
▼
Qwen2.5-7B-Instruct (4-bit) — grounded generation, [Source N] citations
│
▼
3-layer safety: risk classification → grounding rules → confidence gating
plain

## Measured Results

| Metric | Score |
|---|---|
| Precision@3 / Precision@5 | 0.667 / 0.471 |
| Top-1 Accuracy | 0.643 |
| Hit Rate@5 | 1.000 |
| Faithfulness (LLM-as-Judge) | 0.981 (53/54 claims) |
| Citation Integrity / Coverage | 1.0 / 1.0 |
| Refusal Accuracy | 1.000 (6/6 behaviors) |

Retrieval method was **not** chosen by assumption: 5 variants (dense, hybrid BM25+RRF,
cross-encoder reranking, dedup) were evaluated on the same 15-question golden set, and
dense-only won. The reranker was repurposed as a **confidence guard** — its scores cleanly
separate in-scope (min 0.479) from out-of-scope (max 0.129) questions, enabling
data-driven refusal thresholds.

## Repository Structure

| Path | Content |
|---|---|
| `notebook/` | Full Kaggle notebook: parsing → chunking → retrieval → evaluation → generation → safety |
| `outputs/chunks.json` | 150 chunks with full metadata |
| `outputs/document_metadata.json` | Document-level metadata (title, publisher, ISBN, licence) |
| `outputs/who_guideline_parsed.md` | Parsed guideline in Markdown |
| `outputs/eval_*.json` | Retrieval evaluation results per variant |
| `outputs/eval_report_card.json` | Final report card — all metrics in one document |

## How to Run

1. Open the notebook in **Kaggle** with GPU (T4) enabled.
2. Add the WHO guideline PDF as a Kaggle dataset input.
3. Run → Restart & Run All. All outputs are written to `/kaggle/working/`.

## Tech Stack

Docling · BGE-M3 · FAISS · rank_bm25 · bge-reranker-v2-m3 · Qwen2.5-7B-Instruct (4-bit NF4) · pandas · matplotlib

All models are open-source (Hugging Face). No closed APIs.

## Source Document

WHO (2021). *Guideline for the pharmacological treatment of hypertension in adults.*
ISBN 978-92-4-003398-6 · Licence: CC BY-NC-SA 3.0 IGO
https://iris.who.int/server/api/core/bitstreams/f062769d-f075-4a00-87af-0a2106e0bd04/content

The PDF itself is not included in this repository (WHO copyright) — use the link above.