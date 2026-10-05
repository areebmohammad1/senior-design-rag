# CS 5001: Senior Design Deliverable 4
## User Stories, Use Case, and Acceptance Criteria

**Project Title:** Local Context-Aware Knowledge Copilot with Epistemic Tracing  
**Team Identifier:** EpistemicRAG  
**Repository:** https://github.com/areebmohammad1/senior-design-rag  
**Team Members:** Mohammad Areeb (Computer Science), Patrick Caudill (Geosciences)  
**Faculty Advisor:** Dr. Andre Curtis-Trudel (Department of Philosophy / Center for Humanities and Technology)  

---

## 1. Stakeholder Map & User Stories

### Stakeholder Map
* **Primary:** Academic Researcher / Student (queries local corpus and verifies cited answers).
* **Secondary:** Faculty Advisor (evaluates synthesized literature and checks attribution integrity).
* **Hidden:** Institutional Compliance Officer (demands zero data exfiltration across network boundaries).
* **Hidden:** Low-Resource Workstation User (requires local execution within 16 GB system RAM).

### User Stories
* **US-01 (Primary):** As an **academic researcher**, I want **sentence-level attribution links back to exact source text chunks for every generated claim**, so that **I can independently verify evidence and eliminate literature synthesis hallucinations**.
* **US-02 (Hidden / Compliance):** As an **institutional compliance officer**, I want **document parsing, vector indexing, and model inference to run strictly on localhost without outbound network calls**, so that **confidential research manuscripts comply with institutional data governance policies**.
* **US-03 (Primary):** As an **academic researcher**, I want **an explicit refusal of extrapolation when corpus retrieval similarity falls below a defined certainty threshold**, so that **I am not misled by plausible-sounding speculative generation**.
* **US-04 (Hidden / Resource):** As a **low-resource workstation user**, I want **the application to cap resident memory consumption under 12 GB RAM on CPU-only hardware**, so that **the system runs concurrently with desktop tools without operating system thrashing**.

> **INVEST Self-Check:** All stories are independent, negotiable, valuable to distinct stakeholders, estimable within our semester timeline, sized small to single functional boundaries, and testable via quantifiable retrieval, network socket, and memory metrics without prescribing UI widgets.

---

## 2. Use Case

### UC-01: Query Corpus with Epistemic Tracing (Expands US-01 & US-03)
* **Primary Actor:** Academic Researcher
* **Secondary Actors:** ChromaDB (Vector Store), Ollama (Local LLM Daemon)
* **Preconditions:**
  1. Corpus documents (.pdf, .txt, .md) are indexed in local ChromaDB.
  2. The local Ollama service is active on `localhost:11434`.

#### Main Success Flow
1. **Actor:** Submits a natural-language query targeting the local corpus.
2. **System:** Embeds the query and retrieves top-$k$ candidate chunks ($k=4$) from ChromaDB.
3. **System:** Confirms the top candidate chunk cosine similarity score is $\ge 0.65$.
4. **System:** Injects chunks into a grounded prompt and streams a response where each assertive sentence includes an inline anchor (e.g., `[Ref: Chunk_ID]`).
5. **Actor:** Clicks an inline anchor to verify source attribution.
6. **System:** Displays the verbatim text snippet, file name, page number, and similarity score.

#### Alternate Flow: Low-Confidence Refusal (US-03)
* **3a.** If top candidate chunk cosine similarity is $< 0.65$:
  * **3a.1. System:** Suppresses generative context injection.
  * **3a.2. System:** Emits: *"The indexed corpus does not contain sufficient evidence to answer this query with verifiable certainty."*
  * **3a.3. System:** Displays the closest candidate chunk's score with a low-confidence tag and halts.

#### Exception Flow: Local LLM Engine Timeout
* **4a.** If Ollama fails to respond within 30 seconds:
  * **4a.1. System:** Catches connection timeout and logs the failure to `logs/copilot_error.log`.
  * **4a.2. System:** Displays an offline notification prompting the user to check local daemon status.
  * **4a.3. System:** Retains the input query in the input buffer.

#### Postconditions:
* The user receives a verifiable cited response or a deterministic refusal.
* Zero queries or document contents leave `127.0.0.1`.

---

## 3. Acceptance Criteria

* **AC-01.1 (Main Flow - Provenance Attribution):**  
  **Given** an indexed document corpus in ChromaDB and a query with top chunk cosine similarity $\ge 0.65$,  
  **When** the user submits the query and generation finishes,  
  **Then** 100% of factual sentences contain a valid citation anchor referencing an indexed `Chunk_ID`, and clicking the anchor reveals the source excerpt with metadata in $\le 500\text{ ms}$.

* **AC-01.2 (Alternate Flow - Epistemic Fallback):**  
  **Given** a loaded corpus where all retrieved chunks score $< 0.65$ cosine similarity relative to the query,  
  **When** the retrieval evaluation pipeline scores the candidate chunks,  
  **Then** the system returns an explicit refusal message within $2.0\text{ seconds}$, generates exactly $0$ ungrounded sentences, and reports the top candidate's similarity score.

* **AC-01.3 (Exception Flow - Local Daemon Failure):**  
  **Given** the local Ollama process is terminated or non-responsive,  
  **When** an inference request is dispatched to `http://localhost:11434`,  
  **Then** the application aborts after $30.0\text{ seconds}$, displays error status `ERR_LOCAL_ENGINE_UNAVAILABLE`, retains the unsent query in the text input buffer, and records the failure in `logs/copilot_error.log`.
