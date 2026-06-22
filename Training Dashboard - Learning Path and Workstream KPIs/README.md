# Training Dashboard: Power BI Automation for LMS Course, Learning Path, and Workstream KPIs

<a href="../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/LearningPath-Reset.png">
<img alt="Learning Path Overview Page of # Training Dashboard: Power BI Automation for LMS Course, Learning Path, and Workstream KPIs" width="400px" src="../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/LearningPath-Reset.png">
</a>

[Link to Hosted Report](https://app.powerbi.com/view?r=eyJrIjoiOTY1ZWQ5MGItODU1Zi00ZjlmLWFmYjUtYTBiMjg5NWVlZDBmIiwidCI6IjlhYzIwMTdiLTBiNjAtNDgzZC1iYjkzLWVjOWU3YTU1MDE3MCJ9)

## Executive Summary

Replaced manual Excel-based training tracking with an automated Power BI dashboard that combines LMS exports, course mappings, learning path data, and workstream assignments into repeatable completion KPIs. The report helps leaders monitor learning path progress, workstream readiness, course completion, source-file freshness, and learner-level follow-up needs.

## Project Description

This project transformed a manual training KPI process into a repeatable Power BI reporting solution.  The business previously monitored training progress in Excel by exporting LMS reports, manually entering values into spreadsheets, maintaining separate workstream mappings, and relying on formulas to calculate completion metrics.  The LMS could produce course and learning path exports, but had no way to connect training activity to the higher-level workstreams the business used to manage and evaluate readiness.  That gap made reporting time-consuming, error-prone, and difficult to audit.  The goal was to eliminate the manual cycle by accepting five structured CSV inputs (course results, learning path results, a workstream-to-course mapping, a workstream-to-user mapping, and relevant ad hoc training) and having Power BI handle all shaping, relationship resolution, calculation, and visualization automatically.

This portfolio version uses sample data.  Theming, company-specific terminology, internal references, and sensitive details were removed or generalized to protect confidentiality while preserving the structure and analytic intent of the original solution.

## The Problem

The business needed a single, trustworthy view of training progress that its existing tools could not provide.  Several challenges drove the project:

- **No workstream layer in the LMS.**  The LMS organized training into courses and learning paths, but the business evaluated readiness at a higher level, by workstream.  Connecting those two views required manual effort every reporting cycle.
- **Manual and error-prone calculations**.  KPIs were recalculated by hand in Excel each period.  Any change to a course mapping, a learning path, or a user assignment required manual updates across multiple spreadsheets.
- **No distinction between required and optional training**. Completion rates needed to reflect required training logic, while inactive and discontinued courses needed to be removed during data preparation so they did not distort completion KPIs.
- **No audit trail.**  The business could not easily verify whether workstream course mappings were complete, whether expected courses appeared in the LMS exports, or whether the report was using the most current source files.
- **Scattered learning-level visibility.**  Identifying which specific users had incomplete training required manually scanning rows across multiple LMS exports rather than viewing a consolidated summary.

## How the Report Solved the Problem

### Sources

The Sources page addressed a trust issue that existed in the previous Excel process: users needed to confirm that the report was reading the correct input files before relying on any of the KPIs.  The page displays a simple table listing the source files in the configured folder, including the course results, learning path results, workstream map, workstream training assignments, and relevant ad hoc training, along with each file's last modified timestamp.  This gave users an immediate, at-a-glance check that the data had been refreshed with the latest LMS exports before drawing conclusions from the dashboard.

### Learning Path Overview

The Learning Path Overview page reports training progress at the learning path level.  A KPI card in the upper right displays the overall learning path completion percentage for the current filter context, and a matrix table lists each learning path alongside the number of users assigned, the number who completed all required courses, and the resulting completion rate.  Slicers for learner name and learning path title allow users to narrow the view to specific populations or programs, with companion text boxes that display the active selections for clarity.

### Workstream Overview

The Workstream Overview page translates course and learning path activity into the higher-level workstream categories the business uses to evaluate training readiness.  Each row in the matrix represents a workstream and shows how many users are assigned to it, how many have completed all required workstream training, and the resulting completion percentage.  A KPI card in the upper right reflects the blended completion rate across all workstreams in the current filter context.  Slicers for learner name and workstream allow leadership to isolate specific business areas or user groups, and companion text boxes confirm which selections are active.

### Workstream Audit

The Workstream Audit page provides a data-quality and reconciliation view designed to surface gaps in the workstream course mappings.  The matrix displays each learner alongside two counts: the total number of courses mapped to their assigned workstreams in the workstream-course mapping file, and the number of those mapped courses that actually appear in the combined course results.  Differences between the two columns indicate that courses defined in the mapping may be missing from the LMS export, pointing to potential data-quality issues or gaps in the source files that need resolution before completion KPIs can be considered fully reliable.

### Course Overview

The Course Overview page provides operational visibility into training progress at the individual course level.  A KPI card in the upper right shows the blended course completion percentage for all courses in the current filter context, and the matrix table lists each course title alongside the number of users assigned, the number who have completed it, and the resulting completion rate.  The page includes slicers for learner name, course type, course title, optional status, and whether the course has been completed, giving users granular control over what the table reflects.

### People Summary

The People Summary page consolidates training status at the individual learner level, turning the report into an action list for follow-up rather than a data export to manually scan.  Each row represents a learner and shows the total number of courses assigned to them, the number completed, the overall course completion percentage, and three duration columns (total assigned training time, time spent on completed courses, and time remaining) all formatted in hours and minutes.  An Optional slicer at the top left allows users to toggle between all courses and required courses only, adjusting every metric in the table accordingly.  The page makes it easy to identify who is furthest behind, how much training time remains for each person, and where follow-up should be prioritized.

## Data Model

The semantic model follows a star schema pattern with `CombinedCourseResults` as the central fact table, supported by dimension and bridge tables that cover courses, learning paths, people, and workstreams.  Two utility tables support data-load monitoring and refresh timestamping.

### Tables

**CombinedCourseResults** is the central fact table.  It is built in Power Query by appending the two LMS reports (learning path results report and the course results report), then applying a series of transformations: defaulting null learning path IDs and titles for courses that fall outside a learning path, removing rows where the course title contains the words "inactive" or "discontinued," deriving a course type classification (Online, Video, Instructor-Led, Self-Study, or N/A) from keywords in the course title, anti-joining against a separate exclusion list to remove courses that should be excluded from the main learning path and workstream completion model, and reducing duplicate learner-course rows when a course appears in both the learning path export and the course results export.  The resulting table retains the learner-course and learning path grain needed for reporting and includes the learner's name fields, the associated learning path when applicable, the course title and type, the optional flag, the completion date, assignment names, and the foreign keys used for relationships.

**Courses** is a dimension table that holds one row per unique active course.  It is derived from the same source reports as the fact table, deduplicated by Course ID, anti-joined against the exclusion list, and filtered to remove inactive and discontinued titles.  Course duration is parsed from a text representation and converted to total minutes for use in duration calculations.

**Learning Path** is a dimension table containing one row per learning path, built by selecting and deduplicating learning path ID and title from the learning path results report.  A synthetic "Not in Learning Path" row with a reserved ID is appended to provide a consistent label for course results that fall outside any learning path.

**LearningPathCourses** maps each learning path to its component courses, including the optional flag for each course.  It is used to support audit and reference queries rather than driving the primary completion measures.

**People** is a dimension table with one row per unique learner, derived by grouping the combined source data on People ID and extracting the first and last name, then constructing a full display name.

**Workstreams** is a dimension table listing each unique workstream with its ID and title, derived from the workstream map input file.

**WorkstreamMap** is a bridge table that resolves the many-to-many relationship between learners and workstreams.  Each row associates a People ID with a Workstream ID, enabling workstream-level user assignment counts and completion logic.

**WorkstreamCourse**s is a bridge table that maps courses to workstreams.  Each row associates a Course ID with a Workstream ID, enabling workstream-level user assignment counts and completion logic.

**Source** is a utility table that reads the configured source folder and returns each file's name and last modified timestamp, converted to Eastern Time.  It powers the Sources page.

**LastRefreshDate** is a utility table that captures the current UTC timestamp at model refresh and converts it to Eastern Time for display in the report footer.

## DAX Measures

All measures are stored in a `@Measures` table and organized into display folders by functional area.

### Base Course Completion Measures

`Course Users Assigned`  counts the distinct number of learners appearing in `CombinedCourseResults` for the current filter context.  The measure uses `ISINSCOPE` to detect whether the visual is operating at the course level; when it is, it evaluates a direct `DISTINCTCOUNT` on `People ID`, and when it is not, it iterates over the course IDs using `SUMX` to ensure correct aggregation behavior across higher-level visuals.

`Course Users Completed`  filters `Course Users Assigned` to rows where the course completion date is not blank, producing the count of learners who have a recorded completion for each course in context.

`Course Completion %` divides `Course Users Completed` by `Course Users Assigned` using `DIVIDE` to handle zero-denominator cases safely.

### Learning Path Completion Measures

`LP Users Assigned` counts distinct learners per learning path by iterating path titles with `SUMX` and calculating a distinct people count within each, ensuring that users assigned to multiple learning paths are counted separately for each one.

`LP Users Completed` uses a more complex pattern to ensure that completion is evaluated correctly against only required courses.  It builds a summary table using `SUMMARIZE` filtered to non-optional courses, calculates both a required course count and a completed course count per person per learning path and then counts the rows where the two counts are equal.  This approach avoids overcounting completions when a learner has completed some but not all required courses in a learning path.

`LP Completion %` divides `LP Users Completed` by `LP Users Assigned` using `DIVIDE` to handle zero-denominator cases safely.

### Workstream Completion Measures

`WS Users Assigned` iterates over each distinct Workstream ID in the current context using `SUMX` over `WorkstreamMap`, counting the distinct people assigned to each workstream and summing the results.

`WS Users Completed` is the most complex measure in the model.  For each workstream in context, it resolves the set of assigned people using `CALCULATETABLE` on `WorkstreamMap` and the set of required workstream courses using `CALCULATETABLE` on `WorkstreamCourses`.  It then adds columns to a filtered version of `WorkstreamMap` representing the number of required workstream courses assigned to each person and the number they have completed.  A final `COUNTROWS` on a filtered version of that result counts only the rows where the completion count equals the assignment count, meaning the user has finished every required workstream course.

`WS Completion %` divides `WS User Completed` by `WS Users Assigned` using `DIVIDE` to handle zero-denominator cases safely.

### Workstream Audit Measures

`WS # of Courses from WSCourses` counts the number of courses mapped to each workstream in the `WorkstreamCourses` table, iterating over workstream IDs from `WorkstreamMap` to respect the current filter context.

`WS # of Courses from CCR` counts how many of those same mapped courses actually appear in `CombinedCourseResults` for the current filter context.  Comparing this measure against `WS # of Courses from WSCourses` on the Workstream Audit page reveals any discrepancies between the defined workstream mapping and what is present in the source data.

### Duration Measures

Three base duration measures sum estimated course duration in minutes from the `Courses` dimension via `RELATED`.  `Duration of Courses Assigned` sums the duration for all assigned rows in context.  `Duration of Courses Completed` applies an additional filter for non-blank completion dates.   `Duration of Courses to be Completed` subtracts the completed duration from the assigned duration.

Three companion formatting measures convert each of those minute totals into an hours-and-minutes string using `TRUNC`,  `MOD`, and `FORMAT`, producing the `hh:mm` display shown on the People Summary page.

### Filter Display Measures

`Selected People View`,  `Selected LP View`, and `Selected WS View` are display measures that use `ISFILTERED`,  `FILTERS`,  `TOPN`, and `CONCATENATEX` to build a readable list of the currently active slicer selections for their respective dimension.  Each measure defaults to "All" when no filter is applied and appends a count indicator when the number of selected values exceeds the display limit.  The measures power the companion text boxes on the Learning Path Overview, Workstream Overview, and Workstream Audit pages that confirm which filters are active.

### Refresh Measure

`Last Refresh Date` returns the maximum value from `LastRefreshDate[Last Refresh Date (ET)]`, making the model refresh timestamp available for display in the report footer across all pages.

<a href = "../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/Sources.png">
<img align="left" alt="Sources Page of Training Dashboard: Power BI Automation for LMS Course, Learning Path, and Workstream KPIs" width="250px" src="../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/Sources.png">
</a>
<a href = "../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/WorkStreamOverview-Reset.png">
<img align="left" alt="Workstream Overview Page of Training Dashboard: Power BI Automation for LMS Course, Learning Path, and Workstream KPIs" width="250px" src="../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/WorkStreamOverview-Reset.png">
</a>
<a href = "../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/WorkstreamAudit-Reset.png">
<img align="left" alt="Workstream Audit Page of Training Dashboard: Power BI Automation for LMS Course, Learning Path, and Workstream KPIs" width="250px" src="../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/WorkstreamAudit-Reset.png">
</a>
<a href = "../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/CourseOverview-Reset.png">
<img align="left" alt="Course Overview Page of Training Dashboard: Power BI Automation for LMS Course, Learning Path, and Workstream KPIs" width="250px" src="../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/CourseOverview-Reset.png">
</a>
<a href = "../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/PeopleSummary.png">
<img align="left" alt="People Summary Page of Training Dashboard: Power BI Automation for LMS Course, Learning Path, and Workstream KPIs" width="250px" src="../Training%20Dashboard%20-%20Learning%20Path%20and%20Workstream%20KPIs/resources/PeopleSummary.png">
</a>
