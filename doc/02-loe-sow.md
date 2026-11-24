# Northbridge Logistics – IT Service Desk Analytics
## Level of Effort (LOE) and Statement of Work (SOW) – Proof of Concept

> **Simulated engagement for portfolio purposes.** The client, stakeholders, rates and dates are fictional. Hours represent a realistic **client engagement** estimate (including meetings, reviews and testing) and are not the time taken to build the portfolio version.

| Document control | Detail |
|---|---|
| Document ID | NBL-SOW-001 |
| Version / status | v1.1 – **draft, not approved** (aligned to requirements v1.2) |
| Date | November 23, 2026 |
| Baseline | `01-requirements.md` v1.2 (NBL-REQ-001). If this document and the requirements disagree, the requirements govern |
| Client | Northbridge Logistics (fictional) |

---

# Part A – Level of Effort

## A1. Estimate by work package

| # | Work package | Main tasks | Req. ref | Hours |
|---|---|---|---|---:|
| 1 | Project management and client communication | Kickoff, 2 weekly status updates, decision log, final demo | §2 | 6 |
| 2 | Environment setup and access | Workspace and Lakehouse, encrypted Fabric REST connection, ServiceNow API account check, Entra test user, access checklist with client IT | §3, §6, SEC-04 | 4 |
| 3 | API ingestion | Pipeline Copy activity: REST source with basic auth via the Fabric connection, pagination rules (`sysparm_limit`/`sysparm_offset`), JSON landing in `Files/bronze/`; count from Aggregate API; notebook loads `bronze_incident` and checks completeness and duplicates | §6, AC-01, AC-02, AC-08 | 8 |
| 4 | Transformation and data quality | UTC handling, `fact_incident`, `dim_date`, `dim_team` (with "Unassigned"), `dim_priority` with SLA targets, `security_team_map`, data quality counts | §4, §5, §7, AC-03 | 8 |
| 5 | Orchestration | Daily pipeline that runs the notebook; check run history | §10, AC-06 | 2 |
| 6 | Semantic model and DAX | Direct Lake model, relationships, active `opened_at` and inactive `resolved_at` date relationships, 4 measures, snapshot logic for Open Incidents, descriptions | §4, §7 | 6 |
| 7 | Row-level security | "Team Lead" role using `USERPRINCIPALNAME()`, Viewer access for test user, positive and negative tests | §8, AC-05 | 3 |
| 8 | Report | One page: 4 KPI cards, 3 charts, slicers, "data as of" timestamp, PoC label | §9, AC-07 | 4 |
| 9 | Testing and KPI validation | Manual check of up to 10 incidents, acceptance test log | §11, AC-04 | 5 |
| 10 | Fabric assessment | Workspace roles and RLS, pipeline run history, Best Practice Analyzer, AI readiness → 5 findings, 3 quick wins, next steps | §12, AC-09 | 4 |
| 11 | Documentation and handover | README (overview, architecture, how to run, limitations), walkthrough with Priya | §12, AC-09 | 4 |
| | **Subtotal** | | | **54** |
| | Contingency (15%) – PDI data gaps, access delays, rework after review | | | 8 |
| | **Total estimate** | | | **62** |

## A2. Estimated fees

| Item | Value |
|---|---|
| Blended rate (illustrative) | CAD $165 / hour |
| Estimated fees (62 hours) | **CAD $10,230** (excluding taxes) |
| Not included | Fabric capacity, Power BI Pro licenses, ServiceNow licensing |

The PoC uses about a third of the CAD $25,000–$30,000 budget Dana mentioned. The rest stays available for a production phase, which will be estimated separately once the PoC results are reviewed.

## A3. Estimating assumptions

- Client IT provides workspace, connection and test-user access within **2 business days** of kickoff.
- ServiceNow incident volume is small enough for a **full load** on each run.
- KPI rules are taken as defined in requirements v1.2 and do not change during the PoC.
- Client feedback and decisions are returned within **2 business days**.
- One round of review comments on the report and documentation is included.

---

# Part B – Statement of Work

## B1. Purpose

Deliver a proof of concept showing that Northbridge's ServiceNow incident data can be ingested automatically into Microsoft Fabric and turned into consistent, secured service-desk KPIs in Power BI, as defined in requirements v1.2.

## B2. In scope

1. ServiceNow Table API ingestion with authentication and pagination
2. Fabric Lakehouse with a simple star schema
3. Four KPIs: Open Incidents, Incidents Created, MTTR (hours), SLA Breach %
4. Simplified 24/7 SLA calculation (requirements §5)
5. Basic row-level security by team
6. One Power BI report page
7. A short Fabric assessment and README documentation

## B3. Out of scope

Everything listed in requirements §14, including:
- Business-hours SLA calculation and holiday calendar
- Reopen tracking and history-based team attribution
- 24-month history, year-over-year trends and reorganization mapping
- Company-average comparison for team leads
- Incremental loading, failure alerts and hourly refresh
- Git integration and deployment pipelines
- Production deployment, licensing procurement and formal end-user training

Anything not listed in B2 is out of scope unless added through change control (B8).

## B4. Deliverables

| # | Deliverable | Acceptance reference |
|---|---|---|
| D1 | Ingestion pipeline (Copy activities) and transformation notebook | AC-01, AC-02, AC-03, AC-08 |
| D2 | Daily schedule on the pipeline | AC-06 |
| D3 | Direct Lake semantic model with 4 measures and RLS | AC-04, AC-05 |
| D4 | One-page Power BI report | AC-07 |
| D5 | Fabric assessment (`docs/04-fabric-assessment.md`) | AC-09 |
| D6 | README and handover walkthrough | AC-09 |
| D7 | Acceptance test log | AC-01 to AC-09 |

## B5. Timeline (proposed)

| Week | Dates | Activities | Milestone |
|---|---|---|---|
| 0 | Nov 24–27, 2026 | Requirements and SOW approval, access requests | SOW signed |
| 1 | Nov 30 – Dec 4, 2026 | Setup, ingestion, transformation, pipeline | Data in Lakehouse, refreshed daily |
| 2 | Dec 7–11, 2026 | Semantic model, RLS, report, testing, assessment, documentation | PoC demo and handover walkthrough |
| – | By Dec 16, 2026 | Client acceptance review | PoC accepted |

The PoC finishes before the holiday period, which leaves January for a decision on the production phase.

## B6. Roles and responsibilities

| Role | Name | Responsibilities |
|---|---|---|
| Sponsor and approver | Mark, Director of IT | Approves this SOW and signs off acceptance |
| Business lead | Dana Reyes, IT Service Manager | Answers KPI questions, reviews the report, makes day-to-day decisions |
| Client IT | Kevin Osei, Systems Administrator | Fabric workspace, Entra test user, ServiceNow API account |
| Maintainer | Priya, Analyst | Attends the handover walkthrough |
| Consultant | BI / Fabric consultant | Builds, tests and documents the deliverables; status reporting |

## B7. Client responsibilities and assumptions

- Provide the access listed in A3 within 2 business days of kickoff.
- Enter the ServiceNow integration credentials into the Fabric connection (or allow the consultant to do so under client control).
- Name one person to make decisions and approve acceptance.
- Acknowledge that PoC SLA results use simplified targets and are **not** a measure of production SLA compliance (requirements §5, R-05).
- Acknowledge that the PoC uses demonstration data, not production records.

## B8. Change control

A change request is raised for any addition or change to B2. Each request states the change, its reason, the hours and fee impact, and the timeline impact. No change-request work starts until the sponsor approves it in writing.

## B9. Acceptance

The consultant demonstrates the deliverables against acceptance criteria AC-01 to AC-09 and provides the test log. The client has **3 business days** to accept or list specific defects against those criteria. Defects are fixed and re-tested once within the estimate. If a criterion cannot be shown because of a data limitation (for example AC-02 with too few PDI records), the limitation is recorded rather than marked as passed.

## B10. Risks

As listed in requirements §13. The main schedule risk is access provisioning (A3); the main data risk is a small or incomplete PDI dataset (R-01).

## B11. Approval

| | Name | Signature | Date |
|---|---|---|---|
| Client sponsor | Mark, Director of IT | | |
| Consultant | | | |
