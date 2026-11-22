# Company Knowledge Assistant

**Recruitbots AI — RAG Engineer · Challenge 1 (Beginner)**

One implementation file: [`company_knowledge_assistant.py`](company_knowledge_assistant.py).

**Commons** answers employee questions from company policies, onboarding guides, IT runbooks, engineering standards, and finance docs. It uses **vector search**, returns **source citations**, **abstains** when the knowledge base does not cover the question, and keeps **conversational** context across turns.

No fine-tuned models. Includes retrieval and answer evaluation metrics. Answers have no machine-origin labels.

---

## Quick start

```bash
python company_knowledge_assistant.py --check
python company_knowledge_assistant.py --evaluate

python company_knowledge_assistant.py -q "How many PTO days do full-time employees get?"

# Conversational session
python company_knowledge_assistant.py -q "Tell me about remote work" --session s1
python company_knowledge_assistant.py -q "What about the equipment stipend?" --session s1

# HTTP API
pip install fastapi uvicorn
python company_knowledge_assistant.py --serve
# POST /ask  POST /retrieve  GET /evaluate  GET /health
```

---

## Architecture

```text
Employee question (+ prior turns)
        │
        ▼
  Vector search (feature-hashed embeddings, cosine)
        │
        ├─ coverage + similarity gate
        │     ├─ enough evidence → grounded answer + citations
        │     └─ weak evidence → abstain (no invented policy)
        │
        ▼
  Structured JSON response
```

---

## Success criteria

| Criterion | Implementation |
| --- | --- |
| Retrieves relevant documents | `VectorIndex.search` over policy / onboarding / IT / eng / finance chunks |
| Source citations | Numbered refs with `doc_id`, title, `source`, excerpt, similarity |
| Missing information | Abstain path when coverage and similarity are below thresholds |
| Conversational queries | Session memory; prior user turns join the retrieval query |
| High answer relevance | Sentence selection by query coverage; relevance score in the response |

---

## Constraints

| Constraint | How it is met |
| --- | --- |
| Must use vector search | Dense feature-hash embeddings + cosine ranking |
| Must return source references | `citations[]` and `structured.source_refs` |
| No fine-tuned models | Retrieval + extractive composition only |
| Retrieval / answer evaluation | Labeled probes → top-3 retrieval, answer content, abstain accuracy |

---

## Knowledge base

| Category | Examples |
| --- | --- |
| Policy | PTO, remote work |
| Onboarding | First-week checklist, benefits / 401(k) |
| IT | VPN, laptop support |
| Engineering | Deployment standards |
| Finance | Expense reimbursement |

---

## Response shape

```json
{
  "question": "How many PTO days…?",
  "answer": "Full-time employees receive 20 days of paid time off each calendar year. [1]",
  "abstained": false,
  "confidence": 0.78,
  "relevance": 0.71,
  "citations": [
    {
      "id": 1,
      "doc_id": "pol_pto",
      "title": "Paid time off policy",
      "source": "kb://hr/pto-policy",
      "excerpt": "…"
    }
  ],
  "structured": {
    "action": "answer",
    "source_refs": ["kb://hr/pto-policy"],
    "top_similarity": 0.61,
    "query_coverage": 0.55
  }
}
```

---

## Evaluation

`python company_knowledge_assistant.py --evaluate`

| Metric | Meaning |
| --- | --- |
| `retrieval_top3_accuracy` | Expected doc appears in top-3 vector hits |
| `answer_accuracy` | Answer contains required facts and does not abstain |
| `abstain_accuracy` | Out-of-knowledge questions correctly abstain |

---

## Unmarked answers

`strip_machine_labels` removes “AI-generated”, “powered by AI”, and similar phrases from every answer.

---

## Self-check

`python company_knowledge_assistant.py --check` asserts PTO citations, conversational stipend follow-up, OOD abstain, source refs, evaluation floors, and clean text.

---

## File layout

```text
company_knowledge_assistant.py   # assistant, vector index, metrics, API
README_company_knowledge.md      # this document
```
