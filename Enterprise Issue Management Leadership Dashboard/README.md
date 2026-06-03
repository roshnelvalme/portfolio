# Enterprise Issue Management Leadership Dashboard

<a href = "../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/BriefingPortfolio.png">
<img alt="Briefing Portfolio Page of # Enterprise Issue Management Leadership Dashboard" width="400px" src="../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/BriefingPortfolio.png">
</a>

[Link to Hosted Report](https://app.powerbi.com/view?r=eyJrIjoiODE1MzdiMTEtODhiMi00NTdiLTg2OTItMTRmNDBkNjE5NGVmIiwidCI6IjlhYzIwMTdiLTBiNjAtNDgzZC1iYjkzLWVjOWU3YTU1MDE3MCJ9)

## Project Description

This project is a Power BI leadership dashboard built to support recurring review of enterprise issue management activity, covering intake, ownership, status progression, aging, and resolution across a large issue portfolio.

This portfolio version uses sample data.  Company-specific terminology, internal references, and sensitive details were removed or generalized to protect confidentiality while preserving the structure and analytic intent of the original solution.

## The Problem

The team managing enterprise issues was working from a SharePoint list with no structured reporting layer.  Leaders could see individual records, but had no way to quickly assess the open workload, identify aging or high-risk items, track recent activity, or understand where issues were coming from and who owned them.  Every leadership review required someone to manually filter and summarize a list, a process that was time-consuming and inconsistent.

The goal was to replace that with a structured Power BI report that could support recurring leadership review cycles, surface the right operational questions at a glance, and allow drill-through to individual issue detail when something needed close attention.

## How the Report Solved the Problem

The core problem was one of visibility: leaders had access to a list but not to answers.  Knowing that an issue existed was different from knowing whether it was aging, who owned it, what had changed recently, or whether it was close to resolution.  The report addressed each of those gaps through a layered page structure: a portfolio summary for breadth, activity-focused views for recency, and two drill-through detail pages for depth.

### Portfolio Summary: From List to Leadership View

The Briefing Portfolio page replaced the manual summarization step entirely.  Rather than asking someone to filter a SharePoint list before a meeting, leaders could open a single page and immediately see the full picture: total issues tracked, a count of currently open items, median and maximum age among open issues, and total issues closed.  Those KPIs answered the first leadership question of what the state of the portfolio was right now, without any interaction required.

The date range slicers for Received Date and Last Updated added a scoping capability that the list view never had.  Leaders could narrow the visible issues to a specific intake window or activity period, changing the table below without affecting the KPI cards, which remained anchored to the full dataset through portfolio-level measures that ignore slicer context.  This distinction meant the filtering to see a specific time period would not accidentally make the headline numbers appear smaller than they actually were.

The status distribution chart and reporting organization breakdown gave leaders an at-a-glance view of where issues were concentrated, making it easier to identify whether a particular region or status category warranted closer attention before the detailed table was reviewed.

### Activity Views: Surfacing What Changed

Two table views addressed the recency problem, specifically the question of what had moved since the last review.  The "Enterprise Issues Updated in Designated Time Frame" view reflected the full date-range slicer selection, letting leaders focus the issue list on a specific window of activity.  The "Enterprise Issues Updated Last 7 Days" view used a fixed rolling window anchored to the refresh date, surfacing issues that had seen recent movement regardless of their received date, making it useful for quickly identifying what needed attention before the next review cycle.

Both views included an Executive Summary column that let leaders read a plain-language summary of each issue without navigating away from the page, reducing the number of clicks needed to assess whether a given item warranted closer review.

### Drill-Through Detail Pages: Two Depths of Context

For issues that did warrant closer attention, the report offered two drill-through pages designed for different audiences and purposes.

The **Executive Details** page was oriented toward leadership review.  It surfaced the problem statement, executive summary, the specific ask, and desired outcome in a structured layout, providing the framing a leader would need to understand the issue and engage in a meaningful conversation about it.  A Status History table below showed every status the issue had passed through, with the exact date and time it entered and exited each state, making the full lifecycle of an issue verifiable at a glance.  Ticket numbers and a timestamped notes log provided additional operational context without requiring anyone to navigate back to the source system. The header displayed the current status and how many days the issue had been in that state, drawn directly from the status history measure.

The **Issue PM** page was structured for ownership and accountability review.  In addition to the executive summary, it displayed a Background section explaining the full context of the issue, a Recommendations section documenting the suggested path forward, and a separate Impact and Risk section covering scope of impact and disposition.  The POC panel surfaced the submitter, region, facility, primary points of contact, Team Management POCs, and resolution group, making ownership unambiguous for any selected issue.

### The Complete Picture

Together, the pages formed a reporting experience that matched the way leadership actually reviewed issues: start with the portfolio to assess overall health, use the activity views to identify what had changed since last time, and drill into specific items to understand context, ownership, and history before a decision or escalation.  That workflow, which previously required someone to manually build and narrate a filtered list, was embedded directly into the report structure.

## Data Model

The model is organized into a primary issues table, a status history table derived from SharePoint version history, two child lookup tables for multi-value fields, a generated date table, and a disconnected sort table for visual ordering.  All DAX measures are isolated in a dedicated measures table.

### Primary Issues Table

The main fact table holds one row per enterprise issue and captures full lifecycle context: title, problem statement, executive summary, current status, process step, resolution group assignments, primary and secondary council points of contact, reporting organization, facility, submitter, ticket numbers, notes and updates, source channel, scope of impact, intake POC, desired outcome, the specific ask, risk score, subject matter expert contacts, team management POC, team management received date, and last modified date.

Two columns were grouped to normalize raw source values into reporting-friendly categories:

- **Status (groups)** maps the full set of status values into three buckets: `Open`, `Closed`, `Exclude from Reporting` so that visuals and measures can filter by business-meaningful state without being tightly coupled to the specific terminology used in the source list.
- **Reporting Organization (groups)** normalizes inconsistent organization values into a consistent set for display and filtering, including cases where the same organization was recorded under different label formats. 

The issues table also includes an ID Plus Title column sorted by the numeric ID, giving visuals and drill-through pages a readable issue label that sorts correctly without requiring a separate sort column.

Issue Age is calculated in Power Query at load time using `Duration.Days()` against a fixed reference date (July 31, 2024), keeping the measure layer clean and ensuring consistent aging logic across all visuals.  The original report used the refresh date so leaders had an up-to-date experience.

The status history table is the most technically involved part of the model.  Rather than relying solely on the current state of the SharePoint list, the original implementation used a combination of SharePoint's Version History API, Power Automate, and JSON to capture every recorded change to key fields over the life of each issue.  That version history was stored in a JSON array format in the SharePoint list and parsed in Power Query into a flat, relational structure.

The resulting table holds one row per status period per issue, with the following columns:

|Column |Description |
|:---|:---|
|`Id`| Links each history row back to the parent issue|
| `Status` | The status value held during this period |
| `Date In` | When the issue entered this status |
| `Date Out` | When the issue left this status; blank indicates the current status |
| `Days in Step` &nbsp;&nbsp;&nbsp;| Whole days spent in this status period, calculated in Power Query |

The `Date Out = BLANK()` convention is deliberate.  It identifies the current open-ended status row cleanly, without needing a separate flag or a `MAX()` calculation to find the latest record.  This is what allows the Status History JSON - Days in Current Status measure to simply use a `LOOKUPVALUE()` rather than a more complex aggregation pattern.

Intermediate columns from the JSON parsing process such as `Array ID`, `Version Label` are removed in Power Query before the table reaches the model, keeping the schema clean while preserving the parsing logic for maintainability.

### Multi-Value Field Tables

Two SharePoint fields support multiple values per issue: `Issue Source` and `Team Mgmt POC`.  Rather than handling these as delimited strings in the main table, each is expanded in Power Query into a separate child table with one row per value per issue.

The **Source** child table holds the issue Id and the source channel value, supporting accurate count and filter behavior in source distribution visuals.

The **Team Management POCs** child table holds the Issue ID, a numeric lookup value, the contact name, and the associated email address.  Because a single issue can have more than one POC, the `Team Mgmt POCs` measure uses `CONCATENATEX` with an `ALL()` + ID filter pattern to reassemble the names into a readable string for display in detail cards, while preserving the relational structure for filtering.

### Date Table

The date table is generated by a reusable M function rather than a static table, covering the full range of issue activity in the dataset.  Only the `Date` column is retained after generation, keeping the table lightweight.  The table is marked as a Date Table in the model.

### Refresh Date Table

The model includes a dedicated Last Refresh Date table used to anchor all time-relative measures. In the original production version, this table used a dynamic calculation that converted UTC time to Eastern Time with full daylight saving time awareness. It used Power Query to determine DST boundaries programmatically by finding the second Sunday in March and the first Sunday in November for the current year, then applying the appropriate UTC offset. This ensured that any refresh date visual showed the date and time in Eastern Time regardless of daylight saving time. For this portfolio version, that logic was replaced with a fixed value to ensure that measures referring to a rolling window (such as issues opened, modified, or closed in the last seven days) produce consistent results against the static sample dataset.

### Status Sort Table

A disconnected lookup table controls the display order of status values in the status history visual.  It is hardcoded as an embedded M table and carries no relationships to the rest of the model, keeping sort order logic separated from data model structure and filter behavior.

## DAX Measures

All measures are housed in a dedicated @Measures table to centralize the measures in one location.

### Portfolio-Level vs Contextual Measures

A deliberate design pattern runs through the measure layer: most key metrics exist in two versions.

Portfolio-level measures use `ALL()` to override the active filter context so KPI cards on summary pages always reflect the full dataset regardless of slicer selections.  This prevents a common reporting issue where a headline number changes unexpectedly when a user clicks a filter elsewhere on the page.

Contextual measures respect whatever filters are active, making them appropriate for visuals designed for interactive exploration such as filtering by team, status, organization, or time window.  These are suffixed with Contextual to make placement intent clear.

### Measure Summary

| Measure | Pattern | Purpose |
|:--------------------------|:-------------|:--------------------------------------------------------|
| `EIL # of Issues` | Basic | Total row count of the issues table |
| `Count EIL Excluding Duplicate and Deleted` &nbsp;&nbsp;&nbsp;&nbsp;| Portfolio | Full issue count excluding data quality exclusions |
| `Count EIL Excluding Duplicate`
`and Deleted (Contextual)` | Contextual &nbsp;&nbsp;&nbsp;| Same exclusion logic, filter-context aware |
| `EIL Median Age of Open Issues` | Portfolio | Median days open, full dataset |
| `EIL Median Age of`
`Open Issues (Contextual)` | Contextual | Median days open, respects filters |
| `EIL Max Age of Open Issues` | Portfolio | Maximum days open, full dataset |
| `EIL Max Age of`
`Open Issues (Contextual)` | Contextual | Maximum days open, respects filters |
| `EIL # of Open Issues (Contextual)` | Contextual | Count of issues in open status groups |
| `EIL # of Issues Closed (Contextual)` | Contextual | Count of issues with a closed status history row |
| `EIL # of Issues Opened Last 7 Days` | Portfolio | Issues received within 7 days of refresh date |
| `EIL # of Issues Modified Last 7 Days` | Portfolio | Issues modified within 7 days of refresh date |
| `EIL # of Issues Closed Last 7 Days`| Portfolio | Issues that entered closed status within 7 days of refresh date |
| `Status History JSON -`
`Days in Current Status` | Detail | Days in current status for a single selected issue, via LOOKUPVALUE on blank Date Out |
| `Team Mgmt POCs` | Detail | Concatenated POC names for a single selected issue |
| `Last Refresh Date` | Utility | MAX of the refresh date column for display |

The three "last 7 days" measures reference the Last Refresh Date table rather than TODAY(), which ensures they work correctly against a static dataset and makes the date anchor explicit and auditable.

<a href = "../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/ExecutiveDetails.png">
<img align="left" alt="Executive Details Page of Enterprise Issue Management Leadership Dashboard" width="250px" src="../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/ExecutiveDetails.png">
</a>
<a href = "../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/IssuesUpdatedLast7Days.png">
<img align="left" alt="Issues Updated Last 7 Days Page of Enterprise Issue Management Leadership Dashboard" width="250px" src="../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/IssuesUpdatedLast7Days.png">
</a>
<a href = "../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/IssuesUpdatedinDesignatedTimeFrame.png">
<img align="left" alt="Issues Updated in Designated Time Frame Page of Enterprise Issue Management Leadership Dashboard" width="250px" src="../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/IssuesUpdatedinDesignatedTimeFrame.png">
</a>
<a href = "../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/IssuePM.png">
<img align="left" alt="Issues PM Page of Enterprise Issue Management Leadership Dashboard" width="250px" src="../Enterprise%20Issue Management%20Leadership%20Dashboard/resources/IssuePM.png">
</a>