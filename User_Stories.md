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
* **Primary:** Academic Researcher (queries private literature corpus and inspects citation provenance).
* **Primary / Operational:** Grant & Program Manager (synthesizes multi-project progress and generates compliance reports from past awards and expenditure records).
* **Hidden / Governance:** Institutional Compliance Officer (demands zero outbound network transmission of proprietary manuscripts or unannounced grant proposals).
* **Hidden / Downstream:** Resource-Constrained Workstation User (requires local execution within 16 GB system RAM on CPU hardware).

### User Stories
* **US-01 (Primary - Academic Researcher):** As an **academic researcher**, I want **sentence-level attribution links back to exact source text chunks for every generated claim**, so that **I can independently verify claims and eliminate literature synthesis hallucinations**.
* **US-02 (Hidden / Compliance):** As an **institutional compliance officer**, I want **document parsing, vector indexing, and model inference to run strictly on localhost without outbound network calls**, so that **confidential research manuscripts and grant proposals comply with institutional data governance policies**.
* **US-03 (Primary - Program Manager):** As a **grant program manager**, I want **the copilot to synthesize milestones and reportable outcomes across multi-year award documentation with strict fallback refusals when records lack relevant data**, so that **I can compile audited sponsor progress reports without risking fabricated accomplishments**.
* **US-04 (Hidden / Resource):** As a **low-resource workstation user**, I want **the application to cap resident memory consumption under 12 GB RAM on CPU-only hardware**, so that **the system runs concurrently with desktop productivity software without OS thrashing**.

> **INVEST Self-Check:** All stories are independent, negotiable, valuable to distinct academic and administrative roles, estimable within the two-semester lifecycle, scoped small to functional boundaries, and testable via quantifiable retrieval, socket, and memory metrics without prescribing UI widgets.

---

## 2. Use Case

### UC-01: Query Corpus with Epistemic Tracing and Calibrated Fallback (Expands US-01 & US-03)
* **Primary Actor:** Academic Researcher / Grant Program Manager
* **Secondary Actors:** ChromaDB (Vector Store), Ollama (Local LLM Daemon)
* **Preconditions:**
  1. Corpus documents (.pdf, .txt, .md) are indexed in local ChromaDB.
  2. The local Ollama service is active on `localhost:11434`.
  3. The epistemic confidence threshold $\tau$ is initialized from an empirical sweep range ($0.40 \le \tau \le 0.85$, default baseline set to $\tau = 0.65$).

#### Main Success Flow
1. **Actor:** Submits a natural-language query targeting the local corpus.
2. **System:** Embeds the query and retrieves top-$k$ candidate chunks ($k=4$) from ChromaDB using cosine similarity.
3. **System:** Evaluates the highest candidate similarity score against the calibrated threshold ($\text{score} \ge \tau$).
4. **System:** Injects the valid chunks into a grounded prompt and streams an answer where each assertive sentence includes an inline anchor (e.g., `[Ref: Chunk_ID]`).
5. **Actor:** Clicks an inline anchor to verify source attribution.
6. **System:** Displays the verbatim text snippet, file name, page/section reference, and similarity score.

#### Alternate Flow: Calibrated Low-Confidence Refusal (US-03)
* **3a.** If the maximum candidate chunk similarity score falls below the calibrated threshold ($\text{score} < \tau$):
  * **3a.1. System:** Suppresses generative context injection to prevent ungrounded extrapolation.
  * **3a.2. System:** Emits an explicit epistemic fallback: *"The indexed corpus does not contain sufficient evidence to answer this inquiry with verifiable certainty (Score: [score] < Threshold: [\tau])."*
  * **3a.3. System:** Displays the closest retrieved passage with a "Low Confidence" flag and halts generation.

#### Exception Flow: Local LLM Engine Timeout
* **4a.** If Ollama fails to respond within 30 seconds:
  * **4a.1. System:** Catches connection timeout and logs the failure to `logs/copilot_error.log`.
  * **4a.2. System:** Displays an offline notification prompting the user to check local daemon status.
  * **4a.3. System:** Retains the input query in the client buffer.

#### Postconditions:
* The user receives a verifiable cited response or a calibrated fallback refusal.
* Zero queries or document contents leave `127.0.0.1`.

---

## 3. Given / When / Then Acceptance Criteria

### Criteria for UC-01 (Query & Epistemic Tracing)

* **AC-01.1 (Main Flow - Provenance Attribution):**  
  **Given** an indexed corpus in ChromaDB and a query whose top chunk similarity meets or exceeds the empirically calibrated threshold $\tau \in [0.40, 0.85]$ (baseline $\tau = 0.65$),  
  **When** the user submits the query and generation finishes,  
  **Then** 100% of factual assertions contain a valid citation anchor referencing an indexed `Chunk_ID`, and selecting the anchor reveals the source excerpt with metadata in $\le 500\text{ ms}$.  
  *(Metric Grounding: 500 ms aligns with standard interactive UI perception thresholds for indexed local database lookups).*

* **AC-01.2 (Alternate Flow - Empirical Fallback Refusal):**  
  **Given** a query where all retrieved chunks score below the operational threshold $\tau$,  
  **When** the retrieval evaluation pipeline scores the candidate chunks,  
  **Then** the system returns an explicit refusal message in $\le 2.0\text{ seconds}$, generates exactly 0 ungrounded sentences, and reports the top candidate's similarity score alongside $\tau$.  
  *(Metric Grounding: Retrieval scoring is purely vector math on local embeddings and requires no generative LLM compute; $\tau$ is empirically tuned via a 40-query test set to maximize precision against out-of-domain distractors).*

* **AC-01.3 (Exception Flow - Local Daemon Failure):**  
  **Given** the local Ollama process is terminated, unresponsive, or suspended by the OS,  
  **When** an inference request is dispatched to `http://localhost:11434`,  
  **Then** the application triggers an execution timeout after $30.0\text{ seconds}$, displays error status `ERR_LOCAL_ENGINE_UNAVAILABLE`, retains the unsent query in the text input buffer, and records the failure in `logs/copilot_error.log`.  
  *(Metric Grounding: On standard 16 GB CPU hardware, time-to-first-token for quantized 3B models takes 3–8 seconds; any silence beyond 30 seconds indicates thread starvation, process termination, or deadlock).*

---

### Empirical Calibration & Baseline Protocol
To eliminate arbitrary metric selection, system parameters will be empirically calibrated during milestone evaluation:
1. **Confidence Threshold ($\tau$ Sweep):** We evaluate cosine similarity thresholds across the range $[0.40, 0.85]$ in increments of $0.05$ against a validation set of 20 grounded research questions and 20 out-of-domain distractor questions, selecting the $\tau$ that maximizes classification F1-score (balancing hallucination prevention with recall).
2. **Resource & Latency Bounds:** Timeout and memory caps (12 GB RAM) are validated against CPU profiling logs to ensure headless operation without triggering operating system swap thrashing.
