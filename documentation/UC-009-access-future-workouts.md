# UC-009 — Access Future Workouts

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-009 |
| Version | 0.1 |
| Status | Draft |
| Priority | Medium |
| Primary Persona | User |
| Business Goal | Provide visibility into the planned training load and upcoming schedule |

## Summary
In a "Training" menu accessible from the dashboard, the user can view their future planned training sessions in a calendar view. This allows the user to see what's coming up, understand the planned training load, and prepare for upcoming sessions.

## Actors
- **Primary actor**: User
- **Secondary actors**: None
- **External systems**: PostgreSQL database

## Trigger
The user navigates to the "Training" section from the dashboard (UC-004).

## Preconditions
- [ ] A training plan exists with future sessions (UC-003 completed)
- [ ] The database is accessible

## Expected Outcomes
### On Success
- A calendar view displays all future planned training sessions
- Each session shows the type, planned duration/distance, and brief description
- The user can select a session for detailed view

### On Failure
- If no future sessions exist, a message indicates the plan is complete
- If the database is unavailable, an error message is displayed

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| session_id | UUID | Yes | Existing planned session | 550e8400-... |
| date | Date | Yes | Future date | 2026-06-02 |
| session_type | Enum | Yes | Easy Run / Intervals / Long Run / Recovery / Tempo | Long Run |
| planned_duration | Integer | Yes | In minutes | 75 |
| planned_distance | Float | No | In km | 15.0 |
| description | String | Yes | Session instructions | "Easy pace long run, maintain conversational effort" |
| status | Enum | Yes | Planned / Completed | Planned |

## Happy Path (Step-by-Step)
```
1. USER: Clicks on "Training" in the navigation menu.
2. SYSTEM: Fetches all future planned sessions from the database.
3. SYSTEM: Renders a calendar view with sessions placed on their scheduled dates.
4. USER: Browses the calendar to see upcoming sessions.
5. USER: Clicks on a specific session.
6. SYSTEM: Displays the session detail view with:
   - Session type and date
   - Planned duration and distance
   - Detailed description/instructions
   - Option to export as .FIT (UC-011)
7. USER: Reviews and returns to calendar or dashboard.
```

## Alternative Flows
### A1. No Future Sessions
```
1. SYSTEM detects no remaining planned sessions.
2. SYSTEM displays a message ("All planned sessions are completed. Your AI coach will generate a new plan soon.").
```

### A2. User Views Current Week
```
1. SYSTEM defaults to showing the current week in the calendar.
2. USER can navigate to future weeks using next/previous controls.
```

### A3. User Wants to Export a Session
```
1. USER views a session detail and clicks "Export to .FIT".
2. SYSTEM triggers the export flow (UC-011).
```

## Error Flows
### E1. Database Unavailable
```
1. SYSTEM fails to fetch planned sessions.
2. SYSTEM displays an error message with a retry option.
```

## Business Rules
- **BR1**: Only future (non-completed) sessions are shown in the training calendar.
- **BR2**: The calendar defaults to the current week view.
- **BR3**: Sessions must be displayed on the correct day matching the user's available training schedule (from profile's Current Training section).
- **BR4**: The user cannot manually edit sessions from this view (modifications go through AI chat — UC-007).

## MVP Scope
### Included
- Calendar view of future training sessions
- Session detail view with type, duration, distance, and description
- Navigation between weeks
- Link to export flow (UC-011)

### Excluded (Post-MVP)
- Drag-and-drop session rescheduling
- Manual session creation/editing
- Color-coding by session intensity
- Integration with external calendar apps (Google Calendar, iCal)

### Temporary Assumptions
- Simple week-by-week calendar view
- No month view needed for MVP
- Sessions are read-only in this view

## Non-Functional Requirements
- **Expected usage frequency**: Several times per week
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: Calendar loads < 50 sessions at a time

## Acceptance Criteria
- [ ] GIVEN a user with a training plan WHEN they access Training THEN a calendar view shows future sessions on their scheduled dates
- [ ] GIVEN a session on the calendar WHEN the user clicks it THEN the full session details (type, duration, distance, description) are displayed
- [ ] GIVEN the calendar view WHEN it loads THEN it defaults to the current week
- [ ] GIVEN no future sessions WHEN the user accesses Training THEN an appropriate message is shown
- [ ] GIVEN a session detail view WHEN displayed THEN an "Export to .FIT" option is available (UC-011)

## Success Metrics
- Calendar view loads in under 1 second
- Users check the training calendar at least 2 times per week

## Open Questions
- Should the calendar show completed sessions in a muted/different style for context?
- Weekly or daily view as default?
- Should the session detail view show the AI's rationale for that session?

## Dependencies
- **Product dependencies**: UC-003 (plan source), UC-004 (navigation), UC-007 (plan updates), UC-011 (export)
- **Technical dependencies**: PostgreSQL, calendar UI component, frontend framework
- **Legal / Privacy constraints**: None specific (all local data)
