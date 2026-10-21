# RAGRight

**RAG Architecture Advisor for Product Managers**

Most PMs building RAG products make architecture decisions based on what their engineer recommends, what they read in a blog post, or what the LLM framework defaults to. RAGRight challenges that. It asks you 7 questions about your specific product and returns a justified architecture recommendation across 6 layers, with a plain-English reason for every choice.

The output is not a generic "use hybrid retrieval" answer. It is a specific recommendation shaped by your answers: whether your content is a live data feed or a static policy document changes the chunking strategy. Whether your users ask ambiguous multi-intent questions or exact SKU lookups changes the retrieval type and reranker. Whether a wrong answer causes a bad experience or a legal liability changes the prompt requirements. RAGRight makes those distinctions explicit.

---

## Two Modes

**Run my own interview**
Start the 7-question interview and type your own answers. The deterministic decision engine processes your inputs and returns a recommendation specific to your product. No API key needed.

**Watch demo**
No input needed. A pre-built live e-commerce catalogue scenario auto-plays through all 7 questions. All answers and the final recommendation are deterministic.

---

## The 7 Questions

| # | Question | What it decides |
|---|----------|----------------|
| Q1 | What kind of documents will you index? | Chunking strategy |
| Q2 | What types of questions will users ask? | Retrieval type |
| Q2a | (branch) If exact query goes wrong, is that high or low risk? | Precision bar |
| Q3 | How often does your source content change? | Freshness policy |
| Q3a | (branch) What triggers the change - human editor or automated feed? | Pipeline vs event-driven re-indexing |
| Q4 | Do different users need access to different documents? | Access control layer |
| Q5 | How sensitive is a wrong answer? | Prompt requirements and reranking |
| Q6 | What is your latency budget per query? | Reranking approach |
| Q7 | Will users ask precise or ambiguous questions? | Query transformation |
| Q7a | (branch) Give me one real example query | Vocabulary vs intent mismatch |

Branch follow-ups fire automatically when answers trigger them.

---

## The 6 Architecture Layers

1. **Chunking Strategy** - How source content is split before indexing. This determines the unit of retrieval - a single database record, a sentence-level fact, a semantic paragraph, or a parent section with child passages. The wrong chunking strategy is the most common reason RAG systems return contextually correct but factually wrong answers.

2. **Retrieval Type** - How the system finds relevant chunks at query time. Keyword search (BM25) finds exact matches. Dense embedding search finds semantic paraphrases. Hybrid runs both. Query transformation rewrites or expands ambiguous queries before retrieval. The choice is driven by how users actually phrase their questions.

3. **Reranking** - A second pass that re-orders retrieved chunks before they reach the LLM. The first retrieval pass optimises for recall (find anything possibly relevant). Reranking optimises for precision (surface the one chunk that actually answers this specific query). For access-controlled systems, pre-retrieval metadata filtering replaces reranking - the filter runs before the vector search, not after.

4. **Freshness Policy** - How often the index is rebuilt and how failures are caught. An index that is one week stale in a daily-updated product catalogue will serve wrong prices and stock levels with full confidence. The policy matches re-indexing frequency to how fast the source content changes, and adds alerting so silent failures are caught.

5. **Prompt Requirements** - The instructions given to the LLM at generation time. RAGRight outputs three minimum rules for every use case: ground the answer in retrieved chunks only, cite the source for every claim, and refuse or ask when evidence is missing rather than guessing. High-stakes and access-controlled use cases get additional rules.

6. **Top 3 Failure Modes** - The three failure patterns most likely given the specific combination of chunking, retrieval, reranking, freshness, and access control choices made. Each failure mode includes a concrete action to prevent it. Generic failure modes (like "the model hallucinates") are excluded - only failures specific to the architecture are surfaced.

---

## Chunking Strategies Covered

The solution checks answers in this order and picks the first strategy that matches:

| Strategy | Triggered when |
|----------|---------------|
| **Record-level indexing** | Content is a live structured data feed - inventory, pricing, catalogues, databases. No chunking; each record is one retrieval unit with metadata fields. |
| **Hybrid: Fact-Proposition + Semantic** | Medical or clinical content combined with precise exact-match queries. Fact-Proposition chunks isolate single sentences (lab ranges, dosage limits); Semantic chunks cover surrounding explanation. |
| **Semantic** | A single structured table or framework that is not long, legal, or medical. Groups content by meaning rather than character count. |
| **Parent-Child (Hierarchical)** | Long documents or legal content where retrieved passages only make sense with surrounding section context. |
| **Recursive** | Policy docs, FAQs, or content with clear headings and consistent structure. Respects document boundaries rather than cutting mid-section. |
| **Fact-Proposition** | Precise exact-match queries (codes, thresholds, eligibility rules) with no other condition above met. Each claim becomes its own retrievable unit. |
| **Semantic (default)** | All other cases - varied or loosely structured text where meaning clusters naturally. |

---

## Demo Scenario: Live E-commerce Catalogue

The built-in demo runs a product catalogue use case with these characteristics:

- Structured data (SKUs, pricing, stock levels, regional availability)
- Mixed query types (exact SKU lookup + natural language search)
- Daily automated updates via ERP feed - no human in the loop
- Region and subscription-tier-based access control
- Under 500ms latency requirement
- Ambiguous multi-intent queries ("show me something good for hiking")

**Resulting recommendation:** Record-level indexing, Hybrid + Query Transformation retrieval, Metadata reranking (latency-constrained), Automated re-indexing pipeline with change-detection alerts.

---

## Stack

- Single-file HTML artifact - no backend, no build step, no framework dependencies
- Deterministic JS decision engine - zero API calls, runs entirely in the browser
- No data stored or transmitted

---

## Project Structure

```
ragright/
  index.html        # The app - landing screen, chat UI, decision engine, rec card renderer
  designpage.html   # Info page explaining the architecture
  system_prompt.py  # RAGRight system prompt (reference)
  requirements.txt  # No Python dependencies needed - local serve options only
  .env.example      # API key template for local dev reference
  README.md
```

---

## Example Case Runs

These are real test runs through the RAGRight interview. Each shows the 7 questions, the answers given, and the architecture recommendation produced.

---

### Test 1 - ProbeLens (PM Framework Tool)

ProbeLens is a PM problem decomposition tool backed by a single framework document. Users ask how to apply an 8-row structured table to specific product problems.

| Question | Answer |
|----------|--------|
| Q1: Document type | A single framework document - an 8-row table covering WHY, WHAT, WHO, WHERE, WHEN, HOW, SO WHAT and HOW WILL WE KNOW. Each row has a question, deeper questions, a knife to use, an example and a walk-away insight. |
| Q2: Query types | Pure natural language only - users ask how to apply a framework question to their problem. No codes or IDs. |
| Q3: Content change frequency | Rarely - the framework is fixed and changes only if the PM manually updates it. |
| Q4: Access control | No - all users see the same single framework document. |
| Q5: Wrong answer sensitivity | Medium - a wrong answer sends a PM down the wrong framework path and wastes workshop time, but no legal or safety consequences. |
| Q6: Latency budget | Flexible (under 2 seconds) - users are in a thinking session, not a real-time product page. |
| Q7: Query intent | Mix - precise ("what does the WHO knife mean") and ambiguous ("my metric dropped 20%, what should I do"). Branch: intent mismatch is the bigger problem - the user does not know which framework row to start with. |

**RAGRight recommendation:**

- **Chunking:** Semantic - the document is a single structured table where meaning clusters by row. Semantic chunking keeps each framework dimension together rather than cutting mid-row.
- **Retrieval:** Dense + Query Transformation - pure natural language queries need dense embedding search, and ambiguous intent queries need transformation to identify which framework dimension applies before retrieval.
- **Reranking:** LLM reranker - intent-level ambiguity (user asks about a metric drop but does not know if they need WHO, WHAT, or WHY) requires LLM-level understanding to surface the right framework row.
- **Freshness:** Re-index on document events only - the framework is static and only needs re-indexing when the PM publishes a new version.
- **Prompt requirements:** Ground answers in retrieved framework rows only. Cite the specific row and column (e.g. "WHO row, Knife column"). When a query maps to multiple rows, ask the user which dimension to start with rather than guessing.

---

---

### Test 2 - GlobalBenefits (HR Policy Knowledge Base)

An internal HR tool for employees to look up benefits, compliance rules and policy entitlements. Documents vary by country, employment type and policy version, and are updated quarterly.

| Question | Answer |
|----------|--------|
| Q1: Document type | Mix of HR policy documents, employee benefits guides, compliance checklists and internal FAQs. Documents vary by country, employment type and policy version. New versions published quarterly; old ones withdrawn. |
| Q2: Query types | Mix - natural language ("can contractors add parents to health insurance") and exact references ("what does clause 4.2 of the India benefits policy say"). Branch: high risk - a wrong answer on a policy clause or plan code can cause an employee to make a benefits decision based on incorrect information. |
| Q3: Content change frequency | Quarterly - new policy versions every quarter, old ones withdrawn, with possible mid-quarter updates when regulations change in a specific country. |
| Q4: Access control | Yes - controlled by country, employment type and role. A contractor in India sees only contractor policies for India. A full-time employee in the US cannot see executive compensation documents. |
| Q5: Wrong answer sensitivity | High stakes - wrong answers on benefits eligibility, reimbursement limits or compliance rules have direct financial and legal consequences for employees and the company. |
| Q6: Latency budget | Flexible (under 2 seconds) - employees can wait a moment for an accurate answer. Accuracy matters more than speed. |
| Q7: Query intent | Mix - precise ("what is the reimbursement limit for internet bills for full-time employees in India under the April 2026 policy") and ambiguous ("what benefits do I get as a new joiner"). Branch: intent mismatch - "what do I get as a new joiner" could mean health insurance, joining bonus, laptop policy, leave entitlement or probation benefits. |

**RAGRight recommendation:**

- **Chunking:** Parent-Child (Hierarchical) - long policy documents with legal language where a retrieved clause only makes sense with its surrounding section context. Child chunks are retrieved for precision; parent sections are sent to the LLM for context.
- **Retrieval:** Hybrid + Query Transformation - exact clause references need keyword search (BM25) while natural language eligibility questions need dense search. Ambiguous new-joiner queries need transformation to identify which benefit category to retrieve before running the search.
- **Reranking:** Metadata reranking - the most common failure is returning the correct policy topic from the wrong country, employment type or policy version. Metadata reranking filters retrieved chunks by the user's country and employment type before they reach the LLM.
- **Freshness:** Scheduled batch re-indexing on quarterly publish cycle, plus event-driven re-indexing when a policy version is withdrawn mid-quarter. Alerting required - a stale index serving a withdrawn policy is a compliance risk.
- **Prompt requirements:** Ground answers in retrieved policy sections only. Cite the policy name, country, version date and clause number for every claim. If the retrieved evidence is from a different country or employment type than the user's, say so explicitly and do not answer. When a new-joiner query spans multiple benefit categories, list each category separately with its source rather than combining them.

---

---

### Test 3 - DosageRef (Clinical Drug Reference)

A clinical drug reference tool for healthcare providers. Users look up drug dosages, contraindications, and interaction warnings. This test was run against the v1 logic to evaluate reasoning drift vs the v2 deterministic engine.

| Question | Answer |
|----------|--------|
| Q1: Document type | Clinical drug monographs - structured reference documents with fixed sections: Indications, Dosage, Contraindications, Interactions, Adverse Effects, Pharmacokinetics. One monograph per drug. About 2,400 documents. Each monograph is 4-8 pages with consistent heading structure across all documents. |
| Q2: Query types | Mostly exact lookups - "what is the adult oral dose of metformin for type 2 diabetes", "is amoxicillin contraindicated in penicillin allergy". Some natural language around interactions ("can I give this with warfarin"). Branch: high risk - a wrong dosage or missed contraindication is a patient safety event. |
| Q3: Content change frequency | Low but critical. Monographs update when the FDA issues label changes or new safety alerts. Roughly 50-100 updates per year across 2,400 documents. Updates are pushed by a third-party drug database vendor as XML files. No human editor in the loop - fully automated. |
| Q4: Access control | No access control. All clinical users (doctors, nurses, pharmacists) see all monographs. |
| Q5: Wrong answer sensitivity | High stakes. A wrong dose, missed contraindication, or outdated interaction warning is a direct patient safety event with liability consequences. |
| Q6: Latency budget | Under 1 second. Clinicians are mid-workflow - at a bedside or in a dispensing queue. |
| Q7: Query intent | Mostly precise. Clinicians ask specific questions with clear intent. Some variation: "what is the renal dose adjustment for vancomycin" vs "how do I dose vancomycin in kidney disease" - same intent, different vocabulary. Vocabulary mismatch (clinical term vs lay term) is the bigger problem. |

**RAGRight recommendation (v1 logic - reasoning drift noted):**

Note: This test was run against the v1 logic before the deterministic engine was built. The v1 system correctly identified Fact-Proposition chunking as the right choice but drifted in the retrieval and reranking reasoning - it suggested semantic search as the primary retrieval method rather than keyword-first hybrid, underweighting the exact-match nature of clinical dose lookups. The v2 deterministic engine corrects this: for precise exact-match queries with clinical/medical content, it selects Hybrid (BM25-first) retrieval with a cross-encoder reranker, not dense-first.

- **Chunking:** Fact-Proposition - each drug monograph is broken into atomic clinical facts: one chunk per dosage instruction, one per contraindication, one per interaction warning. This prevents a dense context window from burying the exact dose figure the clinician needs behind surrounding explanation text.
- **Retrieval (v1 output, noted as drift):** Dense + keyword boost - v1 weighted semantic search too heavily given the precise nature of the queries. Correct answer per v2 engine: Hybrid (BM25-first) with dense as secondary, since clinicians use exact clinical terminology that BM25 catches reliably.
- **Reranking:** Cross-encoder reranker with confidence threshold - re-ranks retrieved chunks by match to the specific query (drug name + indication + parameter). If confidence is below threshold for exact dose or contraindication queries, refuse rather than guess.
- **Freshness:** Event-driven re-indexing on vendor XML push - fully automated, no human trigger needed. Each update tagged with FDA label date. Chunks from superseded label versions marked deprecated immediately on new version arrival.
- **Prompt requirements:** Answer only from retrieved monograph chunks. Cite drug name, section heading, and FDA label date for every claim. For dose queries: return the exact figure only, not a paraphrase. If the retrieved chunk is from a label older than 90 days, append "verify against current label." Refuse if no chunk scores above threshold rather than synthesising from context.

---

---

---

### Test 4 - Multi-tenant SaaS (Contract and Policy Knowledge Base)

A SaaS platform where each customer uploads their own contracts, internal policies, and knowledge base articles. Strict data isolation: no document from one tenant should ever be visible to another tenant.

| Question | Answer |
|----------|--------|
| Q1: Document type | Each customer uploads their own documents - contracts, internal policies, and knowledge base articles. Strict multi-tenant SaaS. No document from one tenant should ever be visible to another. |
| Q2: Query types | Natural language questions. Users ask things like "what does my contract say about termination" or "what is our refund policy". |
| Q3: Content change frequency | Monthly. Customers upload new documents occasionally, no automated feed. |
| Q4: Access control | Yes, strict tenant isolation. Each customer can only see their own documents. No user should ever see another tenant's data, even if they ask for it directly. |
| Q5: Wrong answer sensitivity | High stakes. These are legal contracts and compliance documents. A wrong answer could expose the customer to legal or financial risk. |
| Q6: Latency budget | Under 1 second. Accuracy matters more than speed. |
| Q7: Query intent | Mostly precise. "What does clause 7 say about liability" or "what is our notice period for termination" - single clear intent. |

**RAGRight recommendation:**

- **Chunking:** Parent-Child - legal contracts and policy documents contain answers that only make sense with surrounding clause context. Child chunks are retrieved for precision; parent sections are passed to the LLM for full context.
- **Retrieval:** Dense (Embedding) - natural language questions about contracts and policies match well against dense vector embeddings, which capture semantic meaning even when the user's phrasing differs from the document's wording.
- **Reranking:** Metadata Pre-filtering (mandatory) - access control must be enforced at the index query level, not at reranking. A tenant_id filter must be applied before any vector search runs. Post-retrieval filtering is the classic bypass failure: the restricted chunk enters the context window before it gets removed.
- **Freshness:** Scheduled batch re-indexing (monthly) + event triggers for urgent uploads. Monthly customer uploads allow scheduled batches, but a newly uploaded contract must be queryable immediately - event-driven re-indexing on upload is required.
- **Prompt requirements:** Ground answers in retrieved chunks only. Name the document title and section for every claim. Never paraphrase a legal or policy clause without quoting the exact passage. If retrieved chunks contradict each other, surface both and flag the conflict explicitly. Never acknowledge the existence of documents belonging to any other tenant - respond "I have no information on that" rather than "I cannot share that", to avoid confirming other tenants' data exists in the system.
- **Top failure modes:** Access control bypass (metadata filter applied post-retrieval lets a restricted chunk enter the context window); confident wrong answer on missing evidence (LLM synthesises from partial context when no good chunk exists); embedding model drift (old and new vectors become incompatible after a model upgrade without full corpus re-embedding).

---

---

### Test 5 - Hospital Network (Patient Support Knowledge Base)

A patient-facing support tool for a hospital network. Patients ask post-surgery and medication questions directly, with no staff review before answers are served. Wrong answers have direct clinical consequences.

| Question | Answer |
|----------|--------|
| Q1: Document type | Discharge instructions, medication FAQs, appointment prep guides, and post-surgery care sheets. Mix of PDFs and plain text written by different clinical teams. About 800 documents total. Highly variable structure — discharge instructions follow a loose template but clinical teams customise heavily. FAQs are 1-2 pages, care sheets 3-10 pages, prep guides vary widely. |
| Q2: Query types | Mostly natural language from patients ("is it normal to feel dizzy after my medication", "what foods should I avoid after my procedure"). Some staff exact lookups ("show me the prep guide for colonoscopy"). Branch: high risk — patients access answers directly with no staff review. A patient acting on wrong post-op instructions could end up back in the ER. |
| Q3: Content change frequency | Mixed cadences. Medication FAQs update monthly when formulary changes. Care sheets and prep guides update quarterly when clinical protocols are reviewed. Discharge instructions are ad hoc — a doctor may revise one at any time. Doctors email updated PDFs to a shared inbox with no central team processing them consistently. Multiple versions coexist — 3 versions of the same knee replacement care sheet found in the system. |
| Q4: Access control | Role-based and site-based. Patients see only documents relevant to their care type (cardiac patient does not see orthopedic instructions). Staff see everything. 3 hospital sites with protocol differences — a patient at Site B must not receive Site A's instructions. |
| Q5: Wrong answer sensitivity | High stakes. Wrong site protocol means wrong wound care, wrong medication timing, wrong follow-up schedule. Refusal preferred over guessing — "I found conflicting information, please call your care team" is better than a confident wrong answer. |
| Q6: Latency budget | Flexible for patients (2-3 seconds acceptable if accuracy improves). Under 1 second for staff who are mid-workflow. |
| Q7: Query intent | Both precise and vague. Scared patients ask things like "something feels wrong after my surgery." Vocabulary mismatch exists (patient says "leaking", docs say "exudate") but intent mismatch is harder — "my leg is swollen" could be normal post-op swelling, a DVT warning sign, a medication reaction, or a wound complication. Four different answers from four different documents. |

**RAGRight recommendation:**

- **Chunking:** Hierarchical + metadata-rich at two levels. Level 1 (semantic): break documents by clinical section (Symptoms, Medications, When to Call Doctor) rather than fixed token counts. For a post-op care sheet, chunk by symptom cluster, not page breaks. Level 2 (safety): add mandatory metadata tags per chunk — document_id, version_date, site_location, clinical_context (e.g. "cardiac_post_op", "wound_care"), risk_level ("normal", "call_doctor", "go_to_ER"). This enables the system to disambiguate normal swelling from DVT and enforce site-based access at retrieval time.
- **Retrieval:** Hybrid (semantic + keyword boosting for clinical safety terms) with mandatory metadata filtering. Semantic search bridges vocabulary mismatch ("oozing" maps to exudate chunks). Keyword boost ensures safety terms ("fever", "bleeding", "call doctor") always surface even on vague queries. Metadata filter by patient site, care type, and document version before ranking.
- **Reranking:** Two-stage with conflict detection. Stage 1: rerank by semantic relevance plus clinical safety signals (chunks with "call doctor", "ER", "urgent" rank higher when query contains symptom language). Stage 2: if reranked results contain contradictory advice or multiple versions of the same instruction, do not synthesise — flag as conflicting and trigger refusal response. This catches the 3-version problem and routes patients to call their care team rather than guessing.
- **Freshness:** Versioned documents with quarantine gate and weekly audit. New uploads get a version_date and upload_timestamp. Newly uploaded docs sit in an unverified index for 24-48 hours; staff must confirm the canonical version before it is queryable by patients. Automated weekly check for duplicate document IDs with different version_dates alerts the clinical operations team. Retain last 2 versions per document for audit purposes.
- **Prompt requirements:** Answer only from retrieved chunks with no inference beyond what the documents state. Cite source document, clinical section, and version date for every claim. If chunks conflict: "Your care instructions have conflicting information. Please call your care team immediately." If query is vague or ambiguous: ask a clarifying question ("Did this happen immediately after surgery or days later?") before retrieving. If no relevant chunk found: "I don't have information about that. Please contact your care team."
- **Top failure modes:** (1) Confidently serving outdated discharge instructions — prevented by quarantine gate, manual approval before patient-facing indexing, and weekly version audit tied to clinical sign-off. (2) Ambiguous query retrieves wrong clinical context (normal swelling docs instead of DVT warning) — prevented by clarifying follow-up questions in the prompt when query contains vague symptom language, plus risk-level reranking that always surfaces "when to go to ER" chunks on symptom queries. (3) Role-based access fails silently, patient at Site B gets Site A protocol — prevented by mandatory metadata filtering at retrieval (not post-retrieval), query logging with filter verification, and monthly audit sampling 100 patient queries to confirm correct site and care type metadata was enforced.

---

---

---

### Test 6 - FinServ Trading Desk (Financial Services Knowledge Base)

An internal knowledge base for a financial services firm covering regulatory compliance docs, trading desk SOPs, risk policy manuals, and real-time market data summaries. Three distinct user types with different access rights and latency requirements in the same system. Note: this test run was completed before the rec card rendering fix - the raw output had formatting artifacts (asterisk prefixes, duplicated closing section). The architecture content is clean and logged here correctly.

| Question | Answer |
|----------|--------|
| Q1: Document type | Regulatory compliance docs, trading desk SOPs, risk policy manuals, and real-time market data summaries - all four in the same system. Compliance docs and risk manuals are unstructured PDFs. SOPs are Word docs with numbered sections. Market data summaries are short auto-generated text blurbs from a data feed. About 2,000 documents. Most docs are 10-50 pages; market summaries are 1-2 paragraphs each. |
| Q2: Query types | Mix. Traders do exact lookups (current margin requirements for a specific instrument). Compliance officers ask natural language questions about regulatory obligations. Risk managers search by SOP title or process name. All three user types in the same system. Branch: high risk for wrong exact lookup - a trader acting on a wrong margin requirement could breach position limits, trigger a margin call, or create a regulatory violation. |
| Q3: Content change frequency | Tiered. Compliance docs update monthly when regulators publish new rules. Risk manuals update weekly. SOPs update ad hoc, once or twice a month. Market data summaries update every 15 minutes via automated feed. Branch: ownership undefined - no versioning process exists. When someone uploads a new SOP, the old one stays in the index alongside it. Users sometimes get answers from outdated versions without knowing it. |
| Q4: Access control | Role-based. Traders see market summaries and trading SOPs only. Compliance officers see everything. Risk managers see risk manuals, SOPs, and compliance docs but not raw market data summaries. No cross-role access by default. |
| Q5: Wrong answer sensitivity | Compliance: high stakes - wrong answer could mean an illegal trade, regulatory fine, or audit failure. SOP: medium - wrong escalation path causes operational delay and rework but is recoverable. Market data: medium to high depending on timing - a stale rate during a volatile session could mean a bad trade execution. |
| Q6: Latency budget | Traders need under 500ms (live markets). Compliance officers fine with under 2 seconds (doing research). Risk managers flexible up to 5 seconds. |
| Q7: Query intent | Mix, but multi-intent is common especially from compliance officers. Real example: "What do I need to check before approving a large derivatives trade?" - could pull from risk limits, compliance rules, SOP steps, and counterparty docs all at once. Intent mismatch is the bigger problem - the user does not know which policy document owns the answer. |

**RAGRight recommendation:**

- **Chunking:** Hierarchical + semantic boundaries - document type-aware chunking. Extract SOP sections by heading/number (preserve numbered structure). Chunk PDFs by topic and semantic coherence (300-500 tokens). Keep market data summaries as atomic units - do not split 1-2 paragraph blurbs. This prevents exact lookups from drowning in context and allows multi-intent queries to pull coherent sub-documents (risk limits, compliance rules, and SOP steps as separate retrieved chunks).
- **Retrieval:** Hybrid (dense + sparse) with role-based filtering and query expansion - exact lookups (trader asking "EUR/USD margin") require BM25 to catch precise terms. Ambiguous multi-intent queries (compliance officer) need dense retrieval to find semantically related chunks across documents. Intent mismatch means the user does not know which doc owns the answer, so query expansion rewrites "approve large derivatives trade" into sub-queries ("What are the risk limits? What compliance checks? What is the escalation SOP?"). Role-based filtering applied at retrieval time before ranking.
- **Reranking:** Cross-encoder reranker with confidence scoring - re-ranks BM25 and dense results by semantic relevance. For exact lookups (margin requirements), if no chunk scores above 0.80 on exact term match and semantic fit, refuse to answer and escalate to a human rather than guess.
- **Freshness:** Tiered invalidation with version tracking. Market data: re-index every 15 minutes, auto-replace old summaries. Risk manuals: automatic re-index weekly on fixed schedule, old chunks tagged with expiration timestamp. Compliance docs and SOPs: human-triggered re-index with version control - when a new SOP is uploaded, mark the old one deprecated (archive for audit, not delete) and re-index. Use document version ID in every chunk's metadata. At query time, retriever filters out deprecated versions unless explicitly requested.
- **Prompt requirements:** Answer only from retrieved chunks. If the question requires information not in the retrieved set, say "I don't have that information in the current policy docs - escalate to compliance/risk/trading operations." Cite exact document name, section number (if SOP), and update date for every claim. If confidence score is below threshold (0.75 for high-stakes, 0.65 for medium), or if retrieved chunks conflict, output: "I found two conflicting answers in the policy docs - please confirm with compliance/risk owner before proceeding."
- **Top failure modes:** (1) Stale exact lookups (margin requirements, position limits) - trader gets yesterday's requirement from a non-invalidated chunk. Version-tag every market data and risk manual chunk with timestamp; filter out chunks older than the SLA (15 min for market data, 7 days for risk manuals); automated alerts if a query returns a chunk older than SLA. (2) Intent mismatch returns incomplete answers - compliance officer asks "approve large derivatives trade" and gets only risk limits, not compliance rules or SOP steps. Pre-process ambiguous multi-intent queries with an LLM intent classifier that rewrites them into sub-queries; retrieve top-k for each, then merge ranked results. (3) Outdated SOP coexists with new one, user gets wrong instructions - two versions of the trade reconciliation SOP both rank high. Implement strict versioning: when a new SOP is uploaded, the old one is immediately tagged deprecated=true and moved to an archive index; retriever filters out deprecated docs by default; include version and date in every chunk and in the prompt output.

---

---

## What RAGRight Is Not

It is not a RAG implementation. It does not build your index, write your retrieval code, or connect to your data source. It is a decision tool that helps you choose the right architecture before you build, so you are not undoing the wrong choices six months later.
