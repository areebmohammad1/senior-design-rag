# Peer Design Review Notes — EpistemicRAG

**Team:** Mohammad Areeb; Patrick Caudill  
**Project:** Local Context-Aware Knowledge Copilot with Epistemic Tracing

## Feedback Received

**Reviewing team:** PolyPredict  
**Original comment (verbatim):**

> I think the team did a good job explaining their project and how they plan to keep it secure and affordable. I liked their focus on privacy and reducing AI hallucinations. One thing they could improve is explaining how they will test the accuracy and overall performance of their system. They could also provide more details about how they plan to evaluate their results and determine if the system is performing well enough. Overall, the project has a good idea, but adding more information about testing and performance would make it even better.

## Action Items

| ID | Criterion | Issue / reviewing team | Proposed action | Owner | Priority | Due date |
|---|---|---|---|---|---|---|
| PR-01 | Clarity | Testing methods unclear; **PolyPredict** | Specify evaluation metrics for retrieval relevance, citation correctness, and unsupported-claim frequency | Mohammad Areeb | Proposed must-fix | December 4, 2026 |
| PR-02 | Feasibility | Performance acceptance criteria unclear; **PolyPredict** | Specify measurement procedure and target thresholds for latency, peak RAM, and response quality | Patrick Caudill | Proposed must-fix | December 4, 2026 |

## Review Summary

**Most feedback:** The received feedback focused on **clarity** of the evaluation plan and **feasibility** of measurable performance criteria.

**Top two issues raised while reviewing other teams:** (1) Missing edge-case and error-recovery strategies, including timeouts, invalid inputs, and network disruption; (2) insufficient measurable performance targets and concrete API contracts.

**Reviews our team provided:**

- **Team PolyPredict:** Praised clear goals, high-level architecture, user workflows, and component separation. Recommended explicit timeout/invalid-input handling, quantitative evaluation metrics, and concrete API schemas.
- **Team Pulse:** Praised repository organization, project vision, and core modules. Recommended clearer hardware/resource and heavy-load latency budgets, fallback strategies for network/service disruption, and module API contracts.
