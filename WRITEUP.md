# IITB Insti-Assist

## 1. Chosen scope and why

I chose the **Academic Assistant** scope which answers questions from course registration, grading policy (SPI/CPI), the
academic calendar, and exam/malpractice rules are:

1. **the rules and registration process is precisely documented.** Academic rules (grade definitions, credit formulas, registration categories, calendar dates) are numeric and unambiguous in the source PDFs, which makes it easy to evaluate whether the RAG system is actually grounded (a wrong CPI formula or a wrong exam date is an obvious, checkable failure).
2. **It is really helpful as information is scattered across various documents.** Questions like "what happens if I miss an exam", "how many credits can I register for", or "when does the semester end" come up every semester for every student, and the authoritative answer is buried in long, dense PDFs that most students never fully read.

## 2. Data sources used

Five real, current IIT Bombay Academic Office documents (publicly hosted PDFs, retrieved via web
search and transcribed into the notebook):

1. **UG Rules & Regulations** — the master rulebook: registration categories,
   credit structure, grading system, SPI/CPI formulas, ARP, branch change, minors/honours.
   `acad.iitb.ac.in/files/UG_RULE_BOOK.pdf`
2. **Academic Calendar 2025–2026** — semester start/end dates, registration windows, exam dates,
   grade-submission deadlines. `acad.iitb.ac.in/.../Academic Calendar 2025-26_FINAL.pdf`
3. **Academic Calendar 2026–2027** — same, for the following academic year (is used to test whether the
   assistant can distinguish between two similar-but-different-year documents).
   `acad.iitb.ac.in/.../Academic_Calendar_2026-27_FINAL.pdf`
4. **Disciplinary Actions for Academic Malpractice** — the specific penalty (grade + suspension)
   for each type of exam/assignment malpractice. `iitb.ac.in/newacadhome/punishments201521July.pdf`
5. **Rules for Award of Medals & Academic Prizes (UG & PG)** — CPI thresholds and eligibility
   rules for the President of India Medal, Institute Gold/Silver Medals, and academic prizes.
   `iitb.ac.in/newacadhome/RulesforAwardofMedalsandAcademicprizesforUGandPG.pdf`

## 3. Chunking strategy and why

**Approach:** paragraph-based chunking, with long paragraphs further split at sentence boundaries
and a 1-sentence overlap carried across split points.

**Why i not used fixed-size character chunking?** These documents are rule books snd each paragraph typically states one rule and its qualifying conditions/exceptions together. A fixed character-count splitter would risk cutting a rule away from its threshold number or its exception clause, which is exactly the kind of failure that causes a RAG system to hallucinate or give a technically-true-but-incomplete answer. Respecting paragraph boundaries keeps each rule's logic intact within a chunk.

**Why cap at ~160 words and merge small paragraphs?** Some paragraphs (e.g. a single calendar date line) are very short and merging consecutive short paragraphs up to the cap keeps chunk count manageable and gives the embedding model enough context per chunk to place it correctly in vector space. Conversely, the UG Rulebook has a few very long paragraphs (e.g. the full grading table section); splitting these at sentence boundaries (with 1-sentence overlap) avoids chunks that are too large to embed precisely, while the overlap prevents the "grade" from being separated from the sentence that first names it.

## 4. Known limitations / what I'd improve with more time

- **Data size:** only 5 documents / ~200 chunks. A production assistant would ingest the full UG/PG rule books, department-specific curricula, hostel manuals, and FAQ pages (likely 50+ documents), and re-crawl periodically since the Academic Calendar and Senate-approved rules change most semesters.
- **Chunking:** paragraph+sentence chunking is simple and interpretable but doesn't understand table structure as the Academic Calendar's tabular date ranges lose some of their tabular alignment once flattened to prose. A table-aware parser would improve precision on calendar queries.
- **Embedding model:** all-MiniLM-L6-v2 is fast but general-purpose; a domain-tuned or larger embedding model would likely improve retrieval recall on paraphrased student queries.
- **No re-ranking step:** top-k is taken directly from FAISS cosine similarity. A cross-encoder re-ranker over the top ~20 candidates would help when multiple chunks are topically similar but only one actually answers the question.
- **No automated eval harness:** the "Try it" cell shows a handful of manually chosen examples. With more time I'd build a small labelled test set (in-scope Q/A pairs + deliberately out-of-scope questions) and track precision/recall on the refusal decision, plus answer faithfulness.

## Bonus implemented

**Multi-turn conversational memory:** the Gradio chat interface folds the previous user turn into
the retrieval query, so a follow-up like "what about the second one?" after asking about grading
categories can still retrieve relevant context, without requiring the person to repeat the full
question each time.
