# IITB Insti-Assist — A RAG-Powered AI Assistant for IIT Bombay

**Scope:** Academic Assistant (course registration, grading policy, academic calendar, exam rules)

it is a Retrieval-Augmented Generation (RAG) system, that answers questions about IIT Bombay's academic rules, grading (SPI/CPI), registration categories, academic calendar dates, exam malpractice penalties, and medals/prizes and is grounded in real IIT Bombay Academic Office documents. The assistant refuses to answer ("I don't know...") whenever a question falls outside what its retrieved context supports.

## Setup

```bash
python -m venv venv
source venv/bin/activate        # on Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Run

```bash
jupyter notebook IITB_Insti_Assist.ipynb
```

Run all cells top to bottom (**Kernel → Restart & Run All**). The last code cell launches an
inline Gradio chat interface right inside the notebook — ask it things like:

- "What grade do I get if I'm caught copying in an exam?"
- "How is CPI calculated?"
- "When is the last date to drop a course in Spring 2026?"
- "What is Category III academic standing?"
- "What's the hostel mess refund policy?"

## How it works (pipeline)

```
5 real IITB Academic Office documents

        │ paragraph + sentence-based chunking (with 1-sentence overlap on splits)

   ~30-40 text chunks, each tagged with doc title + source URL

        │ sentence-transformers/all-MiniLM-L6-v2

   embedding vectors

        │ FAISS IndexFlatIP  (cosine similarity via normalized inner product)

   top-k retrieval per query, gated by a MIN_SIMILARITY threshold

        │ Gemini API (Gemini-2.5-flash), system prompt forces "context-only" answering

   grounded answer + list of source chunks used

        │ Gradio ChatInterface (rendered inline in the notebook)

   chat UI with multi-turn memory + "Sources used" footer on every answer
```

## Notes

- The 5 source documents are embedded as plain-text Python strings directly in the notebook (see the corpus cell) so the retrieval pipeline runs fully offline so only the final LLM call needs network access.
