# Northbridge Logistics - IT Service Desk Analytics
## Proof of Concept (PoC) Requirements

> **Simulated engagement for portfolio purposes.** Northbridge Logistics, its stakeholders, and the discovery call are fictional. All data comes from a ServiceNow Personal Developer Instance (PDI) and is demonstration data, not real operational records.

| Document control | Detail |
|---|---|
| Document ID | NBL-REQ-001 |
| Version / status | v1.3 - PoC scope; **draft, not approved** (v1.3: validated pagination method recorded; v1.2: ingestion moved to a Fabric pipeline Copy activity with an encrypted Fabric connection; no Azure subscription required) |
| Date | November 23, 2026 |
| Business lead | Dana Reyes, IT Service Manager (fictional) |
| Sponsor | Mark, Director of IT (fictional) |
| Role of this document | **Single source of truth** for the PoC LOE, SOW and technical build |

---

## 1. Background

Northbridge's IT Service Manager spends 4 to 5 hours every Monday exporting ServiceNow incidents to Excel and building pivot tables. KPI calculations change from week to week, so the Director of IT does not trust the numbers enough to use them in a staffing discussion with the CFO.

Northbridge has Microsoft Fabric. Before committing to a full implementation, the client wants a **proof of concept** showing that ServiceNow data can be ingested automatically into Fabric and turned into consistent, secured service-desk KPIs in Power BI.

## 2. PoC objectives

| ID | Objective |
|---|---|
| OBJ-01 | Show automated ingestion from the ServiceNow REST API into Fabric, with authentication and pagination |
| OBJ-02 | Model the data in a simple star schema in a Fabric Lakehouse |
| OBJ-03 | Calculate four KPIs consistently from written definitions |
| OBJ-04 | Show that team leads can see only their own team's data (RLS) |
| OBJ-05 | Deliver one report page that replaces the core of the Monday Excel report |
| OBJ-06 | Assess the Fabric setup and document findings, quick wins and next steps |

## 3. Users and access (PoC)

| Persona | Access in the PoC | How it is tested |
|---|---|---|
| Dana (IT Service Manager), Mark (Director of IT) | All teams | Workspace owner account |
| Team lead | Own assignment group only | Test user created in Microsoft Entra ID, with **Viewer** role on the workspace |
| Priya (analyst / maintainer) | Audience for the README and handover | Not tested as a separate user |

Licensing note: the PoC runs on a Fabric **trial capacity**. In production on an F8 capacity, report viewers would need Power BI Pro licenses (see Future Enhancements and the LOE).

## 4. KPI definitions

Timestamps are normalized to UTC for duration calculations and shown in Eastern Time for display. ServiceNow incident states used: 1 New, 2 In Progress, 3 On Hold, 6 Resolved, 7 Closed, 8 Canceled.

| KPI | Definition | PoC calculation rule |
|---|---|---|
| **Open Incidents** | Incidents not yet resolved, as of the latest data load | Count of incidents where `state` is New, In Progress or On Hold. On Hold **counts as open** in the PoC |
| **Incidents Created** | Incidents opened in the selected period | Count of incidents by `opened_at` date |
| **MTTR (hours)** | Average time from opening to resolution | Average of (`resolved_at` minus `opened_at`) in calendar hours for incidents currently in Resolved or Closed state with both timestamps present. Uses the current `resolved_at` value from ServiceNow; the PoC does not independently verify earlier or final resolutions for reopened incidents because state-change history is out of scope. Canceled incidents are excluded |
| **SLA Breach %** | Share of resolved incidents that missed their resolution target | For the selected period, divide the number of eligible Resolved or Closed incidents where `resolution_hours > sla_target_hours` by the total number of eligible Resolved or Closed incidents, then multiply by 100. Eligible incidents must have both timestamps and a defined SLA target. Exclude P5, canceled incidents and incomplete timestamps |

**Rules that apply to all KPIs**
- Incidents are attributed to their **current** assignment group. Transfer history is not tracked in the PoC.
- Incidents with no assignment group are shown as **"Unassigned"** rather than dropped.
- Incidents missing `opened_at` are excluded and counted in the data quality check. Incidents without `resolved_at` are not included in MTTR or SLA Breach %.
- **Date filter interpretation:** Incidents Created uses `opened_at`; MTTR and SLA Breach % use `resolved_at` for the selected period. Open Incidents is a **current snapshot** as of the latest load and is not a historical open count; the date slicer should not change this snapshot KPI.

## 5. SLA rule (simplified 24/7)

All priorities use a **24/7 calendar-hour clock** in the PoC.

| Priority | Policy target | PoC target (calendar hours) |
|---|---|---|
| 1 (Critical) | 4 hours | 4 |
| 2 (High) | 8 hours | 8 |
| 3 (Moderate) | 3 business days | 72 |
| 4 (Low) | 5 business days | 120 |
| 5 (Planning) | No target | Excluded from SLA Breach % |

**Known simplification and limitation:** Northbridge's policy measures P3 and P4 in **business days**, while the PoC substitutes **72 and 120 elapsed calendar hours**. These are not equivalent: depending on when a ticket opens, the PoC deadline may be **earlier or later** than the policy deadline, so the PoC may **overstate or understate SLA breaches**. The PoC result is a technical demonstration, **not a valid measure of production SLA compliance**. Business-hours and holiday-aware SLA calculation is listed under Future Enhancements.

The ServiceNow SLA timer table (`task_sla`) is **not** used.

## 6. Data source

| Item | Detail |
|---|---|
| Source | ServiceNow PDI, Table API: `GET /api/now/table/incident` |
| Authentication | Basic authentication with a dedicated read-only integration account (`svc_fabric_api`, roles `itil`, `snc_basic_auth_api_access`, `snc_read_only`). Credentials are stored **only in an encrypted Fabric REST connection** used by the pipeline, which supports scheduled runs; do not store plaintext secrets in Git, notebook code, logs, screenshots, or Lakehouse `Files`. For production, evaluate OAuth and Azure Key Vault |
| Ingestion method | Fabric Data Factory pipeline **Copy activity** (REST source → Lakehouse `Files/bronze/` as JSON). A second Copy activity saves the record count from the Aggregate API (`/api/now/stats/incident?sysparm_count=true`) for the completeness check |
| Pagination | Copy activity **pagination rules** with an offset placeholder in the relative URL: `...&sysparm_limit=10&sysparm_offset={offset}`; rule QueryParameters `{offset}` = Range start 0, step 10; EndCondition `$.result` = Empty; MaxRequestNumber 50 as a safety limit. A stable sort (`ORDERBYsys_id`) prevents page drift. Validated: 8 requests (7 data pages + 1 empty page), 67 records |
| Fields | `sys_id`, `number`, `opened_at`, `resolved_at`, `closed_at`, `state`, `priority`, `category`, `assignment_group` |
| Load type | Full load on each run (the PDI volume is small: 67 incidents at last check) |
| History | Whatever demo data exists in the PDI. No 24-month history |

## 7. Data model (star schema)

| Table | Grain / purpose |
|---|---|
| `bronze_incident` | Raw API records as received (loaded by the notebook from the JSON files the Copy activity lands in `Files/bronze/`) |
| `fact_incident` | One row per incident: keys, timestamps, state, `resolution_hours`, `sla_target_hours`, `is_sla_breached` |
| `dim_date` | One row per calendar date |
| `dim_team` | One row per assignment group, including "Unassigned" |
| `dim_priority` | One row per priority, with its SLA target hours |
| `security_team_map` | User email mapped to team, used by the RLS role |

Relationships are one-to-many and single-direction, from the dimensions to the fact table. The semantic model uses **Direct Lake** on the Lakehouse tables. Date-based measures must use the appropriate event date (`opened_at` for Created; `resolved_at` for MTTR and SLA Breach %) rather than assuming one date relationship serves every KPI.

## 8. Security (RLS)

- **SEC-01:** a "Team Lead" RLS role filters `dim_team` to the teams mapped to the signed-in user (`USERPRINCIPALNAME()` matched against `security_team_map`).
- **SEC-02:** the workspace owner (acting as Dana or Mark) sees all teams.
- **SEC-03:** team leads get the workspace **Viewer** role only. RLS does not apply to Admin, Member or Contributor roles.
- **SEC-04:** no credentials appear in notebooks, pipelines or the Git repository.

## 9. Report (one page)

- KPI cards: Open Incidents, Incidents Created, MTTR (hours), SLA Breach %
- Incidents created by week (line chart)
- SLA Breach % by team (bar chart)
- Open incidents by priority (bar chart)
- Slicers: date range, team, priority. The date range applies to period-based measures and charts; **Open Incidents remains the latest-load snapshot** (team and priority filters still apply)
- A "data as of" timestamp showing the last successful load

## 10. Refresh

A Fabric Data Factory pipeline runs the ingestion and transformation notebook **once per day**. Run success or failure is visible in the pipeline's run history.

## 11. Acceptance criteria

| ID | Criterion | How it is verified |
|---|---|---|
| AC-01 | All incidents are ingested | Row count in `bronze_incident` equals the count returned by the ServiceNow Aggregate API in the same run |
| AC-02 | Pagination works | Use a page size smaller than the incident total so multiple API pages are retrieved, with no duplicate `sys_id` values. If the PDI contains too few records to demonstrate multiple pages, document this test as not yet demonstrated rather than claiming it passed |
| AC-03 | The star schema is valid | All relationships are one-to-many with no orphaned keys in `fact_incident` |
| AC-04 | KPIs are correct | The four KPIs match manual checks on up to 10 eligible demo incidents, including the documented date and exclusion rules. The SLA check validates the **PoC's simplified** targets, not Northbridge's production policy |
| AC-05 | RLS works | The test team-lead user sees only their own team; the owner account sees all teams |
| AC-06 | Refresh is automated | The pipeline completes successfully on its schedule at least once |
| AC-07 | The report is complete | The page contains every element listed in section 9 |
| AC-08 | Credentials are protected | No passwords or tokens appear in Git, notebooks, Lakehouse `Files`, logs or screenshots; credentials exist only in the encrypted Fabric connection used by the scheduled pipeline |
| AC-09 | Assessment and documentation are done | The Fabric assessment and README are in the repository |

## 12. Deliverables

| Deliverable | Location |
|---|---|
| Requirements (this document) | `docs/01-requirements.md` |
| LOE and SOW | `docs/02-loe-sow.md` |
| Client recap email | `docs/03-communications.md` |
| Ingestion and transformation notebook | `fabric/` (exported) |
| Pipeline, semantic model and report | Fabric workspace + screenshots in `images/` |
| Fabric assessment (5 findings, 3 quick wins, next steps) | `docs/04-fabric-assessment.md` |
| README (overview, architecture, how to run, PoC limitations) | `README.md` |

**Fabric assessment scope:** workspace roles and RLS setup, pipeline run history, a Best Practice Analyzer scan of the semantic model, and AI readiness (table, column and measure descriptions).

## 13. Assumptions and risks

| ID | Item | Mitigation |
|---|---|---|
| R-01 | The PDI has few demo incidents, and some are missing an assignment group or resolution date | Show missing groups as "Unassigned" and report missing dates. To demonstrate pagination, use a page size smaller than the number of records; if fewer than two records are available, record the limitation |
| R-02 | The PDI hibernates when idle and can be reclaimed | Wake it before each session; keep all code in GitHub |
| R-03 | The Fabric trial ends after 60 days | Take screenshots and export items as they are built |
| R-04 | Timestamps are returned in UTC | Convert to Eastern Time for display only; calculate durations in UTC |
| R-06 | Copy activity pagination rules can be fiddly to configure | **Occurred and resolved.** A separate offset query parameter was not applied (20 identical pages, stopped only by MaxRequestNumber); RFC5988 Link-header paging fetched only 1 page. Fixed with an `{offset}` placeholder in the relative URL. Lesson: always validate paging against a known total and check for duplicate keys |
| R-05 | Converting business-day targets to 72/120 calendar hours can **overstate or understate** P3/P4 SLA breaches | Label results as PoC-only in the report and README; implement policy-accurate business-hours/holiday rules in a future phase |
| A-01 | PoC results show feasibility; they are not production acceptance | The full implementation needs a separate SOW |

## 14. Future enhancements (out of PoC scope)

- Business-hours SLA clock for P3/P4 (Mon-Fri, 8 a.m.-6 p.m. ET) with an Ontario holiday calendar
- Reopen tracking and Reopen Rate KPI (needs state-change history)
- Team attribution based on assignment history
- 24-month history, year-over-year trends, and mapping for the 2024 reorganization
- Company-average comparison for team leads, using a separate summary table outside RLS
- Incremental loading, alerts on refresh failure, Azure Key Vault for secrets
- Hourly refresh
- Fabric Git integration and deployment pipelines (Development, Test, and Production)
- Production licensing (Power BI Pro for 5 viewers on F8), the P1 15-minute response SLA, and training for Priya
