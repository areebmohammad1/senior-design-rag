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
2. **Resource & Latency Bounds:** Timeout and memory caps (12 GB RAM) are validated against CPU profiling logs to ensure headless operation without triggering operating system swap thrashing. text input buffer, and records the failure in `logs/copilot_error.log`.
