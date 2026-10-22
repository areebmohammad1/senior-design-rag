# CS 5001: Senior Design Deliverable (Design D0)
## High-Level Architecture & System Design
**Project Title:** Local Context-Aware Knowledge Copilot with Epistemic Tracing (EpistemicRAG)  
**Team Identifier:** EpistemicRAG  
**Repository:** https://github.com/areebmohammad1/senior-design-rag  
**Team Members:** Mohammad Areeb (Computer Science), Patrick Caudill (Geosciences)  
**Faculty Advisor:** Dr. Andre Curtis-Trudel (Department of Philosophy / Center for Humanities and Technology)

---

## 1. Title, Goal Statement, Conventions, & System Inputs/Outputs

### System Goal Statement
An air-gapped, zero-cost knowledge synthesis copilot running locally within 16 GB system RAM on standard CPU hardware that deterministically attributes generated claims to source document chunks, eliminating hallucinations with zero network egress.

### Basic System Inputs and Outputs
* **Basic Input:** Local text/document files (`.pdf`, `.txt`, `.md`) for corpus ingestion, and natural language query strings submitted by the user.
* **Basic Output:** Streamed natural-language synthesis text enriched with inline citation anchors (e.g., `[Ref: Chunk_ID]`) linking directly to source text snippets and metadata, or an explicit calibrated fallback refusal.

### Diagram Conventions & Legend
* **Solid Rectangles [C1 – C5]:** Core internal software components built and maintained by the team.
* **Double / Shaded Borders [[C6]]:** External local runtime daemon (Ollama over `localhost:11434` / 127.0.0.1).
* **Solid Directed Arrows:** Synchronous data exchange and defined interface boundaries (I1 – I6).

---

## 2. Block Diagram (D0 High-Level View)

![Block Diagram D0](block_diagram_d0.png)

---

## 3. Component Responsibility Table

| Component ID & Name | Single Responsibility Statement | Interfaces In / Out | Primary Owner |
| :--- | :--- | :--- | :--- |
| **C1: User Interface (UI)** | Renders the web/CLI interface for document selection, query submission, citation anchor inspection, and response rendering. | **In:** User Input<br>**Out:** I1 | Mohammad Areeb |
| **C2: Orchestration & Session Manager** | Coordinates execution flow between UI, ingestion, vector store, and provenance coordinator while maintaining session state. | **In:** I1<br>**Out:** I2, I5 | Mohammad Areeb |
| **C3: Document Ingestion Engine** | Parses raw document files (`.pdf`, `.md`, `.txt`), chunks text, extracts metadata, and generates embeddings for vector indexing. | **In:** I2<br>**Out:** I3 | Patrick Caudill |
| **C4: Epistemic Provenance Coordinator** | Evaluates similarity score confidence thresholds against calibrated limits ($\tau$), constructs grounded prompts, and attaches citation metadata. | **In:** I5, I4<br>**Out:** I4, I6 | Patrick Caudill |
| **C5: Vector Storage Engine** | Manages local persistence, indexing, and nearest-neighbor cosine similarity queries over document embeddings using ChromaDB. | **In:** I3, I4<br>**Out:** I4 | Patrick Caudill |
| **C6: Local Inference Daemon** | Executes quantized local LLM inference (Llama 3.2 3B) via Ollama over `127.0.0.1` without outbound network access. | **In:** I6<br>**Out:** I6 Stream | Mohammad Areeb |

---

## 4. Interface Specification Table

| Interface ID | Connected Components | Input Data & Types | Output Data & Types | Protocol / Format | Failure / Error Handling |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **I1** | C1 $\leftrightarrow$ C2 | `query` (string), `session_id` (string) | `response_stream` (text/event-stream), `citations` (JSON list) | HTTP / REST & JSON over localhost | Catches validation/network errors; returns HTTP 400/500 JSON payload to C1. |
| **I2** | C2 $\rightarrow$ C3 | `file_path` (string), `chunk_size` (int) | `ingest_status` (bool), `chunk_count` (int) | In-Process Python Function Call | Logs parsing failures to `logs/copilot_error.log`; raises `IngestionError`. |
| **I3** | C3 $\rightarrow$ C5 | `chunks` (list[dict]), `embeddings` (list[float]) | `upsert_count` (int), `status` (string) | DB API Client (ChromaDB Native Python SDK) | Handles disk full/write lock errors; retries write or raises `StorageError`. |
| **I4** | C4 $\leftrightarrow$ C5 | `query_vector` (list[float]), `top_k` (int) | `matched_chunks` (list[dict]), `scores` (list[float]) | In-Process Python API / ChromaDB Query | Handles empty vector store; returns empty result list to C4 to trigger fallback. |
| **I5** | C2 $\rightarrow$ C4 | `user_query` (string), `confidence_threshold` (float) | `grounded_response` (string), `provenance_map` (dict) | In-Process Python Function Call | Catches sub-threshold scores; generates calibrated refusal string. |
| **I6** | C4 $\rightarrow$ C6 | `formatted_prompt` (string), `temperature` (float) | `token_stream` (text/event-stream) | HTTP REST Stream (`http://localhost:11434/api/generate`) | Catches socket timeout (>30.0s); emits `ERR_LOCAL_ENGINE_UNAVAILABLE` to C2/C1. |

### Example Interface Payload (Interface I1 Request/Response)
```json
// I1 Request Payload (C1 -> C2)
{
  "session_id": "sess-98234-a",
  "query": "What are the core reportable outcomes for award HRD-2026?",
  "top_k": 4
}

// I1 Response Payload (C2 -> C1)
{
  "status": "success",
  "response_text": "The project achieved milestone M2 under award HRD-2026 [Ref: Chunk_882].",
  "citations": [
    {
      "chunk_id": "Chunk_882",
      "file_name": "Grant_Report_2026.pdf",
      "page_number": 14,
      "similarity_score": 0.78,
      "verbatim_snippet": "Milestone M2 was successfully completed on schedule with zero cost overruns."
    }
  ]
}
