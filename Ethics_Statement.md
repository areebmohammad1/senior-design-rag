# EpistemicRAG — Ethics Statement

**Project:** Local Context-Aware Knowledge Copilot with Epistemic Tracing  
**Team:** Mohammad Areeb (Computer Science); Patrick Caudill (Geosciences)  
**Advisor:** Dr. Andre Curtis-Trudel  
**Project phase:** Design/documentation; implementation has not begun. Unless stated otherwise, safeguards below are *planned*, not implemented or tested.

## 1. Privacy Summary

EpistemicRAG is designed for **one user per local installation**, under that person's operating-system account. The user explicitly selects files to ingest and receives a notice that their text and derived embeddings will be indexed locally. Normal operation will not upload documents or queries to cloud services. The proposed data inventory follows the D0/D1 document–chunk–embedding–query–citation model.

| Data item | Source | Purpose | Consent | Storage | Access | Retention |
|---|---|---|---|---|---|---|
| Original PDF/TXT/Markdown file | User-selected local file | Extract source text | File selection plus ingestion notice/confirmation | Remains at original path; no permanent app copy planned | OS user and authorized local process | User controls original; deletion from index does not erase source |
| Document ID, filename/type, hash | Ingested file | Identify and trace sources | Same ingestion confirmation | Local metadata/index | OS user and local app | Until indexed document is deleted |
| Chunk ID, text, index, optional page/section | Extracted file text | Retrieval and page/section attribution | Same ingestion confirmation | Local ChromaDB | OS user and local app | Until indexed document is deleted |
| Embedding vector (384 dimensions), model ID | Local embedding model | Similarity search | Same ingestion confirmation | Local ChromaDB | OS user and local app | Until indexed document is deleted |
| Query ID, query text, similarity threshold | User query | Retrieve evidence and produce response | User submits query | Transient process memory | Local app | Request only; no query history |
| Citation ID, chunk ID, sentence index, similarity score | Local retrieval/tracing | Link response claims to evidence | User submits query | Transient process memory | Local app | Session only |
| Generated response | Local Ollama model | Display answer or evidence-based refusal | User submits query | Transient process memory/UI | OS user | Session only; no saved answer history |
| Operational errors (without content) | Application runtime | Diagnose failures | Disclosed in local notice | Local log, if enabled | OS user | Short configurable period; exact duration TBD |
| Temporary session token | App startup | Authenticate local API requests | App launch | Process memory | Local browser session and backend | Expires on application shutdown |

A planned **Delete Indexed Document** action removes its metadata, chunks, and embeddings. It does not delete the original file or external backups. The application will avoid recording document contents, query text, generated responses, or session tokens in logs. Users will be informed that deletion from an index is not a guarantee of secure erasure from disk snapshots or backups. The team will define the log-retention period before implementation.

## 2. Security Summary

The proposed interface is a lightweight local browser UI backed by Python and Ollama. The application and Ollama endpoints will bind to `127.0.0.1`, rather than publicly accessible network interfaces. Loopback access alone does **not** prevent requests from other local processes or malicious websites, so protected operations will require a randomly generated per-launch session token, securely passed to the intended browser session. The token will not be made available from an unauthenticated token-issuing endpoint.

| Concern | Planned control or limitation |
|---|---|
| Default credentials | No preset application usernames/passwords |
| Authentication and token lifetime | Unpredictable token generated at launch, checked on protected requests, expired at shutdown |
| Unauthenticated endpoints | No unprotected ingestion, retrieval, deletion, or generation endpoints; restrict origins and cross-site requests |
| Secrets in source control | No cloud API keys required in normal operation; do not commit tokens or credentials; use ignored local configuration if ever necessary |
| Local data access | Rely on OS account permissions and limit file access to intended paths |
| HTTPS | Loopback-only HTTP is proposed for local deployment; any future remote access would require a separate TLS/authentication design |
| Rate limiting/resource safety | Limit concurrent ingestion and generation jobs to protect CPU and 16 GB RAM budget |
| Evidence integrity | Display source-linked citations and refuse unsupported claims; citations do not by themselves guarantee factual correctness |

**Git-history audit (performed during assignment preparation).** The team cloned the repository with commit history and ran two read-only `git log --all -p` pattern searches. The broad keyword search produced benign documentation references (including `token_stream` and `first-token`). A second search for recognizable OpenAI/GitHub/AWS credential formats, private-key headers, and password assignments returned no matches. **No exposed credentials were identified by these checks.** They were limited pattern-based searches, not a comprehensive secret scanner or a guarantee that no secrets exist. The searches excluded PNG files.

| Security self-audit question | Answer |
|---|---|
| Are default credentials used? | None planned |
| Are application API requests authenticated? | Planned session-token check; not implemented |
| Is the system exposed to remote clients? | No remote access planned; localhost-only binding |
| Are secrets hard-coded or committed? | None identified in the limited history checks; future prevention required |
| Have the controls been penetration-tested? | No; implementation has not begun |

## 3. Licensing Summary

The team **provisionally** selected `AGPL-3.0-or-later` for its own source code and prefers PyMuPDF for PDF parsing. This is not yet a final compatibility determination. Versions will be pinned and license texts checked before installation/distribution. Open-source application code and separately licensed model weights must be treated distinctly.

| Dependency / asset | Version | License (SPDX where available) | Type / obligations and decision |
|---|---|---|---|
| EpistemicRAG application code | Initial version TBD | `AGPL-3.0-or-later` (provisional) | Strong copyleft; provide corresponding source and required notices when applicable |
| Python runtime | TBD | `Python-2.0` (verify version's license) | Permissive; preserve applicable notices |
| Ollama software | TBD | `MIT` (verify selected release) | Permissive; preserve license/copyright notices |
| ChromaDB | TBD | `Apache-2.0` (verify selected release) | Permissive; preserve license/NOTICE as applicable |
| sentence-transformers | TBD | `Apache-2.0` (verify selected release) | Permissive; preserve license/NOTICE as applicable |
| PyMuPDF | TBD | AGPLv3 or commercial terms; verify exact SPDX expression for selected release | Strong copyleft; confirm distribution/source obligations or obtain alternative license |
| Llama 3.2 3B model | Model build TBD | Meta Llama 3.2 Community License (custom, not SPDX) | Separate model license and acceptable-use terms; **weights will not be bundled** |
| Project datasets / non-code assets | None selected for distribution | N/A | User-provided documents are not project-distributed datasets; any future assets require individual review |

**Compatibility and release decision.** The team will verify actual dependency versions, transitive licenses, the selected PyMuPDF terms, and Meta's model-use conditions before publishing a release. The AGPL selection does not relicense Llama weights. Users will be instructed to obtain Ollama and the model separately. Merely making a GitHub repository public does not itself grant an open-source license; a license file must be added when the team finalizes this choice. PyMuPDF may be replaced by a permissively licensed parser if compliance or compatibility cannot be established.

## 4. Attribution List

The following are **planned technologies**, not claims that packages, models, or third-party assets have already been incorporated. The team will retain applicable copyright/license notices in `README.md`, `NOTICE`, or equivalent release documentation as required.

| Resource / author | Official source URL | License / attribution note |
|---|---|---|
| Python Software Foundation — Python | https://www.python.org/ | Python license; verify installed version |
| Ollama contributors — Ollama | https://github.com/ollama/ollama | MIT; preserve license and copyright notices |
| Chroma contributors — ChromaDB | https://github.com/chroma-core/chroma | Apache-2.0; verify release notices |
| Sentence Transformers contributors — sentence-transformers | https://github.com/UKPLab/sentence-transformers | Apache-2.0; verify release notices |
| Artifex / PyMuPDF contributors — PyMuPDF | https://pymupdf.readthedocs.io/ | AGPL/commercial; review https://pymupdf.readthedocs.io/en/latest/about.html |
| Meta — Llama 3.2 | https://www.llama.com/llama3_2/ | Meta Llama 3.2 Community License; model obtained separately |

No adapted third-party code, bundled external datasets, or non-code media assets have been identified in the documentation-stage project. If any are introduced, their authors, source URLs, licenses, and required attribution will be recorded before distribution. User-imported research papers remain the user's own materials, not redistributed project assets.

## 5. AI-Use Disclosure

> “Generative AI tools may be used for drafting and code assistance but not for the design decisions or written analysis that are assessed in this course.”

Both team members report using Gemini and ChatGPT to assist with drafts and formatting of EpistemicRAG documentation. No AI-assisted EpistemicRAG implementation code has been written yet. The table is grouped under the course's **Generally OK** category; AI use is not a substitute for independently assessed design decisions or written analysis.

| Tool | Used by | Task | Column | How verified |
|---|---|---|---|---|
| Google Gemini | Mohammad Areeb and Patrick Caudill | Drafting and organizing previous senior-design documentation | Generally OK | Both members reviewed drafts against assignment instructions and agreed project requirements and design artifacts |
| ChatGPT | Mohammad Areeb and Patrick Caudill | Drafting and formatting project documentation, including this ethics statement | Generally OK | Both members checked the final content against repository documentation, team decisions, and assignment requirements |

**Not a substitute for:** Requirements, stakeholder priorities, architecture, data-model decisions, ethical trade-offs, and assessed written analysis remain the responsibility of the team. Each member must inspect and revise AI-assisted drafts so the final submission reflects the team's own conclusions. AI outputs are not evidence of implementation or testing.

**Individual acknowledgments**

Mohammad Areeb — Signature: Mohammad Areeb                   Date: 11/11/2026  
Patrick Caudill — Signature: Patrick Caudill                 Date: 11/11/2026

By signing, each member confirms that the disclosure accurately describes their AI use and that the submitted architecture, data model, priorities, and design rationale reflect the team's own work.
