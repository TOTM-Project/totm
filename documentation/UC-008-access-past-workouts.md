# UC-008 — Access Past Workouts

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-008 |
| Version | 0.1 |
| Status | Draft |
| Priority | Medium |
| Primary Persona | User |
| Business Goal | Allow users to review their training history and AI analysis of past sessions |

## Summary
In a "History" menu accessible from the dashboard, the user can browse their past recorded training sessions. Each entry shows the objective metrics (from .FIT data), perceived effort, and the AI's analysis/feedback on that session.

## Actors
- **Primary actor**: User
- **Secondary actors**: None
- **External systems**: PostgreSQL database

## Trigger
The user navigates to the "History" section from the dashboard (UC-004).

## Preconditions
- [ ] The user has at least one recorded training session (UC-005/UC-006 completed at least once)
- [ ] The database is accessible

## Expected Outcomes
### On Success
- A list of past recorded sessions is displayed in reverse chronological order
- Each session shows key metrics and the AI's feedback
- The user can select a session for detailed view

### On Failure
- If no data exists, an empty state is shown with guidance
- If the database is unavailable, an error message is displayed

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| session_id | UUID | Yes | Existing completed session | 550e8400-... |
| date | Date | Yes | Past date | 2026-05-30 |
| distance | Float | Yes | In km | 10.5 |
| duration | Integer | Yes | In minutes | 54 |
| elevation | Float | No | In meters | 156 |
| average_pace | Float | Yes | In min/km | 5.12 |
| perceived_difficulty | Enum | Yes | Effort level | Moderate |
| ai_feedback | String | No | AI-generated analysis | "Good session, pace consistent..." |

## Happy Path (Step-by-Step)
```
1. USER: Clicks on "History" in the navigation menu.
2. SYSTEM: Fetches all completed sessions from the database.
3. SYSTEM: Displays a scrollable list of sessions (date, distance, duration, difficulty).
4. USER: Selects a specific session.
5. SYSTEM: Displays the full session detail view with:
   - All recorded metrics (distance, duration, elevation, pace, heart rate)
   - Perceived effort data
   - AI analysis/feedback for that session
6. USER: Reviews the information and returns to the list or dashboard.
```

## Alternative Flows
### A1. No Past Sessions
```
1. SYSTEM detects no completed sessions in the database.
2. SYSTEM displays an empty state ("No sessions recorded yet. Import your first training data from the dashboard!").
```

### A2. Session Without AI Feedback
```
1. USER selects a session that has no AI feedback generated yet.
2. SYSTEM displays metrics and effort data normally.
3. AI feedback section shows "Analysis pending" or generates it on demand.
```

## Error Flows
### E1. Database Unavailable
```
1. SYSTEM fails to fetch session data.
2. SYSTEM displays an error message with a retry option.
```

### E2. Corrupted Session Data
```
1. SYSTEM encounters invalid data for a session record.
2. SYSTEM displays available data and flags the issue.
3. SYSTEM logs the data integrity error.
```

## Business Rules
- **BR1**: Sessions are displayed in reverse chronological order (most recent first).
- **BR2**: Only sessions with recorded data (via UC-005) appear in the history — planned-but-not-completed sessions do not.
- **BR3**: AI feedback should be available for each completed session.
- **BR4**: The user cannot modify past session data from the history view.

## MVP Scope
### Included
- Chronological list of past recorded sessions
- Detailed view for each session with all metrics
- Display of perceived effort data
- Display of AI analysis/feedback per session

### Excluded (Post-MVP)
- Filtering and search within history
- Comparison between sessions
- Export of history data
- Charts/graphs showing progress over time in history view

### Temporary Assumptions
- AI feedback is generated at import time and stored alongside session data
- Simple list view without pagination (MVP scale < 100 sessions)
- No data editing from the history view

## Non-Functional Requirements
- **Expected usage frequency**: A few times per week
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: List view with < 100 sessions in first year

## Acceptance Criteria
- [ ] GIVEN a user with recorded sessions WHEN they access History THEN all completed sessions are listed in reverse chronological order
- [ ] GIVEN a session in the list WHEN the user selects it THEN full metrics, effort, and AI feedback are displayed
- [ ] GIVEN no recorded sessions WHEN the user accesses History THEN an appropriate empty state message is shown
- [ ] GIVEN a session detail view WHEN displayed THEN it shows distance, duration, elevation, pace, heart rate (if available), and perceived effort
- [ ] GIVEN the history list WHEN it loads THEN only sessions with imported data are shown (not planned sessions)

## Success Metrics
- History list loads in under 1 second
- All completed sessions with imported data appear in the history

## Open Questions
- Should AI feedback be generated at import time or on-demand when viewing history?
- Should there be pagination or infinite scroll for the list?

## Dependencies
- **Product dependencies**: UC-004 (navigation), UC-005/UC-006 (data source)
- **Technical dependencies**: PostgreSQL, frontend framework, list/detail UI components
- **Legal / Privacy constraints**: None specific (all local data)
