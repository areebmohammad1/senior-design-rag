# EpistemicRAG — Final Presentation Run Sheet

## 1. Presentation Overview

- **Project:** EpistemicRAG — Local Context-Aware Knowledge Copilot with Epistemic Tracing
- **Presenter:** Mohammad Areeb
- **Presentation date:** November 25, 2026
- **Presentation format:** Recorded slide presentation with voice-over
- **Recorded duration:** 6 minutes 37 seconds
- **Video hosting:** UC OneDrive
- **Presentation files:** `EpistemicRAG_Presentation_Draft.pptx` and `EpistemicRAG_Presentation_Draft.pdf`

## 2. Speaker Order and Handoffs

Mohammad Areeb is the sole presenter for all ten main slides. There are no speaker handoffs.

| Slide | Topic | Speaker |
|---|---|---|
| 1 | Title and purpose | Mohammad Areeb |
| 2 | Team and faculty advisor | Mohammad Areeb |
| 3 | Project abstract | Mohammad Areeb |
| 4 | User stories | Mohammad Areeb |
| 5 | Simplified D0 architecture | Mohammad Areeb |
| 6 | Major project constraints | Mohammad Areeb |
| 7 | Progress review | Mohammad Areeb |
| 8 | Expected end-of-term outcomes | Mohammad Areeb |
| 9 | Division of work | Mohammad Areeb |
| 10 | Expo demonstration | Mohammad Areeb |

## 3. Timekeeper and Rehearsal

- **Rehearsal date:** November 18, 2026
- **Rehearsal duration:** Approximately 7 minutes
- **Presenter and timekeeper:** Mohammad Areeb
- **Target duration:** 5–10 minutes
- **Final recording duration:** 6 minutes 37 seconds
- **Full-group rehearsal:** Mohammad Areeb is the sole presenter under the approved presentation arrangement. The presentation was rehearsed individually, with no speaker handoffs required.

## 4. Questions and Answers

**Q1: How will the system prevent unsupported answers?**

**Owner: Mohammad Areeb**

The retrieval pipeline will evaluate available evidence against a similarity threshold. When sufficient evidence is unavailable, the system is designed to refuse unsupported claims.

**Q2: How will citations be verified?**

**Owner: Mohammad Areeb**

Generated citations will reference indexed document chunks. Evaluation will check whether cited chunks exist and support the corresponding claims.

**Q3: Can the system run without internet access?**

**Owner: Mohammad Areeb**

The architecture uses local document storage, embeddings, ChromaDB, and Ollama. Air-gapped operation remains a design requirement to be validated during implementation.

**Q4: What are the largest remaining risks?**

**Owner: Mohammad Areeb**

CPU inference latency, memory consumption, retrieval quality, and citation accuracy. These require implementation and benchmarking.

## 5. Offline Backup Verification

The following backup materials and playback checks were verified:

- **Local PowerPoint presentation:** Verified
- **Local PDF presentation:** Verified
- **Local MP4 recording:** Verified
- **Playback on a second computer:** Verified

Offline copies of the presentation and recording are available as backups if the primary presentation method encounters technical difficulties.
