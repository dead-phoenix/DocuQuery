# DocuQuery

A semantic search chatbot for navigating large document repositories using natural language. Built for MANE's Legal and IP department as a student project at École Centrale Méditerranée, it lets employees ask plain-language questions and get accurate answers drawn from a corpus of internal legal documents — no search terms, no manual browsing.

---

## Overview

Large organizations accumulate legal documents across scattered systems — contracts, policies, IP guidelines, compliance frameworks. Finding the right answer in that corpus is slow and error-prone. DocuQuery wraps a fine-tuned sentence embedding model and a Streamlit interface around that corpus, turning it into a conversational Q&A system.

The core approach is semantic similarity: user questions are encoded into vectors and matched against a pre-indexed FAQ database derived from the document corpus. When no close match exists, a local LLM (Vicuna) generates a response from scratch.

---

## How It Works

```
User question
     │
     ▼
Encode with SentenceTransformer (multi-qa-mpnet-base-dot-v1)
     │
     ▼
Cosine similarity against pre-encoded FAQ database
     │
     ├── High similarity  → Return matched answer, ask for satisfaction
     │
     ├── Medium similarity → Show answer + suggest similar questions
     │                       User can confirm or select an alternative
     │
     └── Low similarity   → Fall back to Vicuna LLM for open-ended answer
```

User satisfaction is tracked after each response via Yes/No buttons. If unsatisfied, the bot prompts for clarification or escalates to Vicuna.

---

## Models

| Component | Model | Purpose |
|---|---|---|
| Sentence embedding | `multi-qa-mpnet-base-dot-v1` (SBERT) | Encode questions and FAQ entries into semantic vectors |
| Similarity | Cosine similarity | Match user query to closest FAQ entry |
| Fallback LLM | Vicuna | Generate open-ended answers for low-similarity queries |

### Why `multi-qa-mpnet-base-dot-v1`?

After comparing several BERT and SBERT variants (`bert-base-uncased`, `all-distilroberta-v1`, `all-mpnet-base-v2`, `all-MiniLM` variants), this model consistently produced the highest cosine similarity scores on legal Q&A pairs. It was trained specifically on question-answer pairs, making it well-suited for retrieval tasks. It also handles long-range dependencies better than smaller models, important for legal text where context spans multiple sentences.

---

## Data

The FAQ database was built from MANE's internal legal documents (`.docx`, `.doc`, `.pdf`), converted to plain text, cleaned, and restructured into question-answer pairs. Due to a signed NDA, the data files are not included in this repository.

The original dataset contained 122 FAQ pairs across 68 documents. This was augmented to ~1,650 pairs using paraphrasing, synonym replacement, and sentence shuffling — then used to fine-tune the SBERT model for the domain.

To run the project, you need to supply your own `faq_data.json`. See the format below.

### `faq_data.json` format

```json
[
  {
    "filename": "document_name.pdf",
    "context": "Relevant excerpt from the document...",
    "question": "What is a CDA?",
    "answer": "A CDA (Confidential Disclosure Agreement) is signed when..."
  }
]
```

---

## Tech Stack

- Python
- [Sentence-Transformers](https://www.sbert.net/) — SBERT embeddings
- [Streamlit](https://streamlit.io/) — web interface
- [Vicuna](https://github.com/lm-sys/FastChat) — local LLM fallback
- MongoDB (optional) — for structured storage of augmented Q&A data

---

## Project Structure

```
DocuQuery/
├── app.py                  # Streamlit web application
├── chatbot_in_terminal.py  # Terminal version of the chatbot
├── faq_data.json           # ⚠️ NOT INCLUDED — proprietary data (see above)
├── requirements.txt
└── README.md
```

---

## Installation

```bash
git clone https://github.com/yourusername/DocuQuery.git
cd DocuQuery
pip install -r requirements.txt
```

You will also need to install and run [Vicuna locally](https://github.com/lm-sys/FastChat) for the LLM fallback to work.

Then add your own `faq_data.json` following the format above.

```bash
streamlit run app.py
```

---

## Development Timeline

The project ran over 7 weeks (October–December 2023):

| Phase | Duration |
|---|---|
| Data access and document review | 18 days |
| Data preparation and format conversion | 4 days |
| LLM (Alpaca-LoRA / LLaMA) exploration | 8 days |
| Transition to BERT, restructuring data | 9 days |
| BERT vs SBERT comparison, Streamlit frontend | 7 days |
| Data augmentation (~1,650 pairs generated) | part of week 6 |
| SBERT fine-tuning and full integration | 9 days |

The project started with LLMs (Alpaca-LoRA, DialoGPT) but pivoted to SBERT after the dataset size (68 files, 122 FAQs) proved too small for effective LLM fine-tuning. BERT's efficiency with limited data and its task-specific fine-tuning capability made it the right fit.

---

## Limitations

- **Closed knowledge base**: The bot can only answer questions covered by the FAQ database. Novel queries outside the corpus rely entirely on Vicuna, which has no access to the documents.
- **Static index**: The FAQ database must be manually updated when documents change. There is no live document ingestion pipeline.
- **Vicuna latency**: The LLM fallback is significantly slower than the similarity-based path.
- **Language**: The system was primarily tested on English and French legal text. Performance on other languages is untested.

---

## Areas for Future Work

- Replace the static FAQ database with a live vector store (e.g., FAISS, Chroma) for real-time document ingestion
- Add file retrieval — return the source document alongside the answer
- Implement continuous learning from user satisfaction feedback
- Expand to HR, compliance, and operational policy documents
- Add role-based personalization for different employee functions

---

## Project Context

- **Client**: [MANE](https://www.mane.com/) — global fragrance and flavor company, Legal & IP department
- **Institution**: École Centrale Méditerranée
- **Supervisor**: Anne-Laure Mealier
- **Year**: 2023–2024
