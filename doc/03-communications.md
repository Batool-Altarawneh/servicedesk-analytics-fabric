# Northbridge Logistics – IT Service Desk Analytics
## Client Communications

> **Simulated engagement for portfolio purposes.** The client, stakeholders and emails are fictional.

---

## 1. Discovery recap email

**To:** Dana Reyes (IT Service Manager)
**Cc:** Mark (Director of IT); Kevin Osei (Systems Administrator)
**Date:** November 24, 2026
**Subject:** Service Desk Analytics – discovery recap, proposed PoC and next steps

Hi Dana,

Thank you again for walking me through your current reporting process. Below is a summary of what we discussed and how I suggest we move forward.

**What we heard**
- The Monday Excel report takes 4–5 hours a week, and KPIs are calculated differently from week to week.
- Mark needs numbers he can trust for the staffing discussion with the CFO.
- You and Mark need to see all four teams; each team lead should see only their own team.
- Data should be refreshed daily and ready before the 8:30 a.m. stand-ups.

**Proposed approach: start with a proof of concept**
Before committing to a full implementation, I recommend a short, two-week proof of concept (PoC). It will show that ServiceNow incident data can be loaded automatically into Fabric and turned into four consistent KPIs (Open Incidents, Incidents Created, MTTR and SLA Breach %), secured by team, on one Power BI report page.

To keep the PoC small, it uses a simplified SLA clock (24/7 for all priorities). Business-hours SLAs, the holiday calendar, reopen tracking, 24-month history and the company-average comparison are listed as future enhancements. PoC SLA results are a technical demonstration and should not be used as a measure of actual SLA compliance.

**Attached for review**
1. `01-requirements.md` – PoC requirements, including the KPI definitions and acceptance criteria
2. `02-loe-sow.md` – Level of effort (62 hours, about CAD $10,230 before taxes) and statement of work

**What I need from you**

| # | Request | Owner | Needed by |
|---|---|---|---|
| 1 | Review the KPI definitions and confirm they match how you want the numbers calculated | Dana | Nov 26 |
| 2 | Confirm who approves the SOW and signs off acceptance (we assumed Mark) | Dana / Mark | Nov 26 |
| 3 | Approve the SOW | Mark | Nov 27 |
| 4 | Provide a Fabric workspace, a test user for the RLS check, and a read-only ServiceNow API account (credentials will be held in an encrypted Fabric connection) | Kevin | Nov 30 |
| 5 | Let me know roughly how many incidents are created per month | Dana | Nov 27 |

**Next steps**
- Once the SOW is approved, we start the PoC on **November 30**, with the demo planned for the week of **December 7**.
- I'll send a short status update every **Friday**, covering progress, decisions needed and risks. Please let me know if another format works better for you and Mark.

Kevin, I'd appreciate 20 minutes this week to go through the access list. Could you suggest a time?

Thanks,
[Your name]
BI / Fabric Consultant
