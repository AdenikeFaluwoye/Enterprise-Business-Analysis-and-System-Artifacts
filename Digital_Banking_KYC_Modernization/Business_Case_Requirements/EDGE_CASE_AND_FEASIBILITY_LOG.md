# Edge Case & Technical Feasibility Log
**Project:** Digital Banking KYC & Identity Verification Modernization  
**Author:** Adenike Faluwoye, Senior Business Systems Analyst  
**Key Contributors:** QA Lead, Tech/Dev Lead, Compliance Lead  

---

## 1. Overview
During initial requirements elicitation workshops, edge cases (system failure paths, irregular user inputs) and technical constraints are identified to ensure system resilience and prevent rework during development.

---

## 2. Edge Case & Exception Log

| ID | Trigger / Scenario | Source Role | Impacted Component | Business & System Handling Rule | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **EC-01** | Government ID upload image is blurry or glare obscures text. | QA Lead | Mobile Web Front-End / OCR Engine | Front-end client runs client-side image quality check. If rejected, prompt user real-time: *"Image unclear, re-take photo."* Max 3 attempts before routing to manual queue. | Approved |
| **EC-02** | Third-party ID Verification API (Trulioo) times out (> 10s). | Tech Lead | Integration Middleware | System logs API Gateway 504 timeout, sets application status to `PENDING_MANUAL_REVIEW`, and triggers asynchronous retry worker twice before alerting Operations. | Approved |
| **EC-03** | User's legal name on ID differs from credit bureau file due to marriage/hyphenation. | Compliance | Compliance Engine / Fuzzy Logic | System invokes fuzzy matching logic (threshold = 85% match score). If score < 85%, route to Compliance Queue with highlighted mismatch fields. | Approved |

---

## 3. Technical Feasibility & Architectural Constraints

| Constraint ID | Architecture / Legacy Constraint | Implication on Requirement | Agreed Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **TC-01** | Core Banking System only supports overnight batch processing for account creation. | Real-time instant account activation cannot write directly to the core DB synchronously. | Implement an asynchronous queue (AWS SQS/Kafka) to buffer approved onboarding payloads, allowing UI instant approval response while core updates complete within 15 minutes. |
| **TC-02** | Sanction Screening Engine database updates daily at 02:00 AM UTC. | Potential 24-hour gap for newly issued FINTRAC red-flag alerts. | Enable real-time webhook listener for emergency sanction broadcast updates from Compliance vendor. |