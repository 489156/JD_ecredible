# Care-Bridge: AI-Powered Dual-Safety and Administrative Automation Platform

## 1. Goals and Success Metrics
- **Documentation efficiency:** 50% reduction in time spent on documentation within 3 months by shifting to voice-led capture, auto-filling, and proactive nudges.
- **Regulatory risk mitigation:** 0% documentation-related penalty exposure for the next LTC evaluation via standardized narratives, rule-based compliance checks, and immutable audit trails.
- **Caregiver well-being:** 25% reduction in reported burnout within 6 months through cognitive load reduction, stress detection, and supportive interventions.
- **Ethical safety:** 90% accuracy in high-risk acoustic event detection (falls, screams, abuse) with <2s alert latency; objective evidence retained via an immutable ledger.

## 2. Product Modules

### Module A: AI Administrative Copilot (LLM-Centric)
- **Voice-to-Regulation (V2R):** Mobile/desktop mic capture ➜ ASR ➜ colloquial-to-regulatory normalization via a fine-tuned small LLM (sLLM) trained on paired colloquial/regulatory text. Output structured notes mapped to LTC evaluation schemas.
- **Cognitive Load Optimization:** Zero/one-click task flows with icon-based quick actions for common items (meals, mobility, mood). Contextual defaults from schedule and care plans auto-fill forms to minimize typing.
- **Real-time Risk Flagging:** Rules + LLM moderation to detect punitive or biased language; suggests neutral alternatives and blocks submission until corrected.
- **Data Ingestion Pipeline:** Secure OCR/ICR for scanned/handwritten backfiles (3–5 years). Human-in-the-loop labeling to achieve ≥95% handwriting accuracy, plus PII masking prior to model use.

### Module B: Dual-Safety Guardian (Acoustic Intelligence)
- **Abnormal Acoustic Detection (AAD):** On-device/edge SED (falls, screams, glass breaking, aggression). Latency <2s; offline-first buffering.
- **Mutual Respect Monitoring:** Korean-language tone/sentiment models to distinguish abusive language (elder→caregiver) and abusive tone (caregiver→elder) without full speech reconstruction.
- **Automated Incident Ledger:** 30s pre/post audio window stored as encrypted chunks (unique keys per incident) in tamper-proof storage with hash-chained metadata and role-restricted access.
- **Caregiver Distress Alert:** Stress inference during V2R usage (prosody, tempo, pauses). Private “take a break” nudges and anonymized roll-ups for managers.

### Module C: Family & Government Connect
- **Trust Transparency Portal:** Sanitized daily summaries for guardians (positive/neutral only) with consent-aware access; HIPAA/PIPA-compliant data minimization.
- **Financial Discrepancy System (FDS):** Cross-check scheduled care time, logged tasks, and acoustic presence signals to surface fraud/neglect anomalies; produces auditor-ready reports.
- **AI Safety Certification Badge:** Real-time badge API tied to system uptime and safety coverage; revokes if uptime <99.99% or guardianship controls fail.

## 3. Architecture Overview
- **Client Apps:** Flutter/React Native with native modules for low-latency audio capture and on-device SED. Accessibility-first UI (large touch targets, high-contrast Deep Teal primary, Warm Yellow/Coral accent for alerts).
- **APIs:** FastAPI (Python 3.11+) on K8s. Public-facing API gateway with WAF and OAuth2/OIDC (Keycloak/Okta). gRPC between services for low-latency calls; REST for external portals.
- **Services:**
  - `admin-copilot-service` (LLM orchestration, risk flagging, templating)
  - `speech-ingestion-service` (ASR interface, diarization, streaming)
  - `acoustic-guardian-service` (event aggregation from edge devices, incident ledger)
  - `family-portal-service` (summary generation, role/consent enforcement)
  - `analytics-fds-service` (anomaly detection, reports)
- **Data Stores:**
  - PostgreSQL for regulatory notes, tasks, schedules, audit metadata.
  - Secure NoSQL (e.g., DynamoDB/Cosmos) for encrypted audio segments and incident payloads.
  - Object storage (S3-compatible) for artifacts with per-object KMS keys; hash-chained manifests for tamper evidence.
  - Data Lake (parquet in object storage) for anonymized text/audio features; governed via Lake Formation/Glue catalog.
- **AI/ML Stack:**
  - Custom sLLM hosted privately (vLLM/FastAPI inference) with retrieval augmentation from compliance snippets.
  - ASR via on-prem models (Whisper finetune) with diarization; no external calls for PHI.
  - Edge SED (TFLite/PyTorch Mobile) auto-updated via signed model bundles.
- **Eventing & Observability:**
  - Kafka/Redpanda for event bus (alerts, notes, audit events).
  - OpenTelemetry for traces/metrics/logs; SIEM integration.
  - Feature flags (LaunchDarkly/Open-source) for progressive rollout.

## 4. Data Pipelines & Model Lifecycle
- **Backfile Ingestion:**
  1. Secure upload (SFTP/VPN) ➜ checksum verification.
  2. OCR/ICR with layout-aware models; low-confidence extractions routed to labeling UI.
  3. PII masking/anonymization (regex + NER) before landing in the Data Lake.
  4. Versioned labeled datasets stored with data contracts; reproducible ETL via Dagster/Airflow.
- **Training & Validation:**
  - sLLM fine-tuning on colloquial→regulatory pairs; evaluation with domain experts until ≥95% BLEU/accuracy on regulatory phrasing set.
  - SED models validated on Korean LTC soundscape datasets; target >90% F1 and <2s end-to-end alert latency in field tests.
  - Stress/tone models calibrated with ethical review; bias testing across dialects.
- **Deployment & CI/CD:**
  - Terraform IaC for VPC/K8s/KMS/resources; GitOps (ArgoCD) for manifests.
  - Canary releases for edge models with rollback on drift/false positive spikes.
  - Periodic retraining triggered by data quality thresholds and drift monitors.

## 5. Compliance, Security, and Privacy
- **Data minimization:** Edge-first processing; audio not stored unless an incident triggers ledger capture.
- **Encryption:** TLS in transit; per-object KMS keys and envelope encryption at rest. Keys rotated and tenant-scoped.
- **Access Control:** Attribute-based access control (ABAC) with least privilege; manager-only access to raw incidents; guardian portal restricted to sanitized summaries.
- **Auditability:** Immutable incident ledger with hash chaining + write-once storage policy; detailed audit logs for all access.
- **Consent & Policy:** Configurable consent workflows; retention controls; HIPAA/PIPA-compliant logging and breach alerting.

## 6. API Sketches
- `POST /v1/notes/voice` ➜ stream audio, returns structured regulatory note + risk flags.
- `GET /v1/notes/templates` ➜ fetch LTC guideline-aligned templates.
- `POST /v1/acoustic/events` ➜ edge device sends detected event metadata; service responds with alert status and signed upload URL for pre/post audio.
- `GET /v1/incidents/{id}` (manager-only) ➜ retrieve redacted transcript + audit trail.
- `GET /v1/guardian/summary` ➜ sanitized daily summary for authorized guardian.
- `GET /v1/fds/report` ➜ anomaly report cross-linking schedule, tasks, and acoustic presence.

## 7. UX Principles
- **Radical simplicity:** Single-screen flows with icon grids for routine actions; large typography and high-contrast Deep Teal primary with sparing Warm Yellow/Coral accents.
- **Latency targets:** Voice note turnaround <3s; acoustic alert <2s; portal page loads <1s P95.
- **Accessibility:** WCAG 2.1 AA compliance; haptic feedback on mobile; offline capture with sync queues.

## 8. Delivery Plan (4–6 Month MVP)
- **Phase 0 (Week 0–2):** Discovery, data agreements, threat modeling, and design sprints; deliver clickable prototype and data contracts.
- **Phase 1 (Week 2–6):** Build V2R pipeline (ASR + sLLM normalization + risk rules), caregiver app skeleton, and ingestion for backfiles; initial OCR accuracy tuning.
- **Phase 2 (Week 6–10):** Edge SED beta, incident ledger, manager console, and feature flags; begin UAT with 60+ age group (target usability >90/100).
- **Phase 3 (Week 10–14):** Guardian portal (sanitized feed), FDS MVP, compliance hardening, and K8s/Terraform CI/CD.
- **Phase 4 (Week 14–20+):** Reliability scaling (99.99% badge), drift monitoring, red-team audits, and auditor-ready reporting.

## 9. Open Questions for Stakeholders
- Confirm regulatory standard set (national LTC evaluation guideline version) and required languages/dialects for sLLM and ASR.
- Define acceptable false-positive/negative rates for acoustic alerts and distress detection in production versus pilot.
- Clarify consent/opt-out policies for residents and guardians regarding acoustic monitoring and data sharing.
- Provide sample historical records (scans + audio) and scheduling schemas to finalize data contracts and labeling budgets.
- Decide on hosting model (on-prem private cloud vs. dedicated VPC) and data residency requirements.

