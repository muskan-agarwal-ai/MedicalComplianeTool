# Medical Compliance Extraction Platform

> Multi-agent LLM pipeline for pharmaceutical product safety and regulatory compliance extraction. Shipped to production under ISO 27001 and FDA traceability requirements.

---

## The Problem

A pharmaceutical company needed to screen 500+ drug product leaflets across various regulatory compliance categories before market submission. The process was entirely manual, taking days per product, and a single missed extraction could trigger a regulatory failure.

A single-prompt LLM approach was benchmarked first and rejected — it could not meet the traceability requirements and failed on complex multi-section documents. The solution required a more structured, auditable architecture.

---

## Architecture

```
Raw PDF Leaflets
      │
      ▼
┌─────────────────────────┐
│  Document          │
│  Intelligence (OCR)        ← Stage 1: Layout extraction
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│  Vision LLM               ← Stage 2: Section parsing
│               │
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────┐
│  Vector Store           │  ← Stage 3: Semantic chunking
│  + Sentence Transformers     and retrieval
└─────────┬───────────────┘
          │
          ▼
┌─────────────────────────────────────────────┐
│  Multi-Agent Extraction Layer               │
│                                             │
│  Coordinator Agent                          │
│       │                                     │
│       ├── Category Agent 1 (Indications)    │
│       ├── Category Agent 2 (Contraindications)│
│       ├── Category Agent 3 (Dosage)         │
│       ├── Category Agent 4 (Side Effects)   │
│       ├── Category Agent 5 (Interactions)   │
│       ├── Category Agent 6 (Warnings)       │
│       ├── Category Agent 7 (Storage)        │
│       └── Category Agent 8 (Composition)    │
│                                             │
│  Each agent: retrieve → extract → validate  │
└─────────┬───────────────────────────────────┘
          │
          ▼
┌─────────────────────────┐
│  Validator Gates         │  ← Confidence scoring
│  + Schema Enforcement   │      per extraction
└─────────┬───────────────┘
          │
          ├── High confidence → Structured output
          │
          └── Low confidence → Human review queue
                                      │
                                      ▼
                              ┌───────────────┐
                              │  Evaluation   │
                              │  Harness      │
                              └───────────────┘
```

### Why multi-agent over single-prompt

Benchmarked both approaches on 50 real leaflet samples:

| Approach | Accuracy | Traceability | Manual Review Rate |
|---|---|---|---|
| Single-prompt LLM | 71% | None | 40% |
| Multi-agent pipeline | 95% | Full citation | 12% |

The multi-agent design allowed each category to have its own retrieval scope, schema, and confidence threshold. It also made the system auditable: every extracted field traces back to the source chunk in the original document.

---


| Layer | Technology |
|---|---|
| Orchestration | LangChain, custom multi-agent coordinator |
| LLM | Azure OpenAI GPT-4o |
| OCR and layout | Azure Document Intelligence |
| Vector store | FAISS |
| Embeddings | Sentence Transformers |
| API layer | FastAPI |
| Compliance | ISO 27001, FDA traceability |

---

## Evaluation Harness

Before any model change went to production, it was tested against a golden dataset of 500 pharmaceutical products with known correct outputs.

- Hard deployment gate at 95% extraction accuracy
- Caught 3 regressions during development that would have shipped undetected
- Zero hallucination policy: if the model cannot ground a claim to a source chunk, it flags for review rather than generating an answer

Metrics tracked per production run:
- Extraction confidence score per category per document
- Human review rate (rising rate = early signal of reliability problem)
- Schema violation count
- End-to-end latency per document

Evaluation metrics tracked using Ragas:
- Faithfulness: does the extracted answer stay grounded in the retrieved context
- Answer Relevance: is the extraction relevant to the compliance category query
- Context Recall: are the right source chunks being retrieved for each category

Traces logged in LangSmith for latency monitoring and debugging failed extractions.

---

## Results

| Metric | Before | After |
|---|---|---|
| Manual review time | Baseline | 70% reduction |
| Extraction accuracy | ~60% manual | 95% automated |
| Compliance gate pass rate | Manual sign-off | Automated with audit trail |
| Documents processed | Days per product | Minutes per product |

Shipped to production. Used in live UK pharmaceutical regulatory workflows.

---

## Key Design Decisions

**Human-in-the-loop is a feature, not a fallback.** Low-confidence extractions go to a review queue rather than failing silently or hallucinating. This was a requirement from UK regulatory advisors and ended up being what made the system trustworthy in production.

**Citation over generation.** Every extracted field is linked to the source clause in the original document. The model does not summarise or infer — it extracts and cites. This was the only design acceptable under FDA traceability requirements.

**Deduplication at the coordinator level.** When a document section was relevant to multiple compliance categories, each agent retrieved it independently. The coordinator handled conflict resolution before writing final output, preventing duplicate or contradictory extractions.

---

## Compliance and Governance

- ISO 27001 data handling throughout pipeline
- FDA traceability: full audit trail from raw document to structured output
- Human-in-the-loop review queue for low-confidence extractions
- No PII or patient data processed
- All client data handled under NDA

---

