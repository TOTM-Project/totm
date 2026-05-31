# UC-004 — Dashboard

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-004 |
| Version | 0.1 |
| Status | Draft |
| Priority | High |
| Primary Persona | User |
| Business Goal | Provide an at-a-glance view of recent activity, upcoming training, and AI interaction |

## Summary
The dashboard is the main screen users see from their second login onwards. It displays three tiles: a 7-day training summary chart (distance, elevation, duration), a detailed view of the next scheduled session with an import button, and a chat interface to interact with the AI coach.

## Actors
- **Primary actor**: User
- **Secondary actors**: AI Coach (chat tile)
- **External systems**: PostgreSQL database, AI service

## Trigger
The user logs in (after having completed the initialization questionnaire).

## Preconditions
- [ ] The user has completed the initialization questionnaire (UC-001)
- [ ] A training plan exists in the database (UC-003 completed)
- [ ] The application backend and database are available

## Expected Outcomes
### On Success
- The dashboard displays with three functional tiles
- Training data from the last 7 days is rendered in a chart
- The next scheduled session is shown with details and an import button
- The AI chat interface is functional

### On Failure
- If training data is unavailable, the chart tile shows an empty state with a message
- If no next session exists, the session tile shows appropriate messaging
- If the AI service is down, the chat tile shows a degraded state

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| training_history | Array | No | Last 7 days of recorded sessions | [{date, distance, elevation, duration}] |
| next_session | Object | No | Next planned session from the plan | {date, type, duration, description} |
| chat_messages | Array | No | Conversation history | [{role, content, timestamp}] |

## Happy Path (Step-by-Step)
```
1. USER: Logs in to the application.
2. SYSTEM: Detects the user has completed onboarding (questionnaire done).
3. SYSTEM: Fetches the last 7 days of training data from the database.
4. SYSTEM: Fetches the next scheduled training session from the plan.
5. SYSTEM: Renders the dashboard with three tiles:
   - Tile 1: Chart with daily distance, elevation, and duration for the last 7 days.
   - Tile 2: Next session details (type, duration, description) + "Import Data" button.
   - Tile 3: AI chat interface.
6. USER: Views their training summary and upcoming session.
```

## Alternative Flows
### A1. No Training Data Recorded Yet
```
1. SYSTEM detects no training data exists for the last 7 days.
2. SYSTEM renders the chart tile with an empty state ("No data yet — complete your first session!").
3. Remaining tiles render normally.
```

### A2. No Upcoming Session
```
1. SYSTEM detects no future sessions in the training plan.
2. SYSTEM renders the session tile with a message ("All sessions completed — plan regeneration needed").
3. Remaining tiles render normally.
```

### A3. User Clicks Import Button
```
1. USER clicks the "Import Data" button on the next session tile.
2. SYSTEM navigates to the training data import flow (UC-005).
```

### A4. User Sends a Chat Message
```
1. USER types a message in the AI chat tile.
2. SYSTEM sends the message to the AI service (UC-007).
3. SYSTEM displays the AI response in the chat tile.
```

## Error Flows
### E1. Database Unavailable
```
1. SYSTEM fails to fetch data from PostgreSQL.
2. SYSTEM displays an error banner on the dashboard.
3. SYSTEM shows empty states in affected tiles.
```

### E2. AI Service Unavailable
```
1. SYSTEM fails to connect to the AI service for chat.
2. SYSTEM displays "AI temporarily unavailable" in the chat tile.
3. Other tiles continue to function normally.
```

## Business Rules
- **BR1**: The chart must show exactly 7 days (today and 6 previous days), with zero values for days without data.
- **BR2**: The "next session" is the earliest future session in the training plan that has not been completed.
- **BR3**: The chat interface must maintain conversation context within the current session.
- **BR4**: The dashboard is only accessible after the initialization questionnaire is completed.

## MVP Scope
### Included
- 7-day training chart (distance, elevation, duration)
- Next session tile with details and import button
- AI chat tile for direct interaction
- Navigation to other sections (History, Training, Profile)

### Excluded (Post-MVP)
- Customizable tile layout
- Weekly/monthly chart views
- Push notifications for upcoming sessions
- Performance trend analysis widgets

### Temporary Assumptions
- Chart library is chosen and integrated (e.g., Chart.js, Recharts)
- Single-page application with client-side routing
- AI chat uses the same LLM as plan generation

## Non-Functional Requirements
- **Expected usage frequency**: Every login (daily for active users)
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: Dashboard load < 2 seconds including data fetch

## Acceptance Criteria
- [ ] GIVEN a user with training data WHEN they access the dashboard THEN a chart displays distance, elevation, and duration for the last 7 days
- [ ] GIVEN a user with a training plan WHEN they access the dashboard THEN the next scheduled session is displayed with type, duration, and description
- [ ] GIVEN the next session tile WHEN the user clicks "Import Data" THEN they are navigated to the import flow (UC-005)
- [ ] GIVEN the AI chat tile WHEN the user sends a message THEN they receive a response from the AI coach
- [ ] GIVEN a new user who has not completed the questionnaire WHEN they try to access the dashboard THEN they are redirected to the questionnaire (UC-001)

## Success Metrics
- Dashboard loads in under 2 seconds
- Users interact with the dashboard daily when actively training

## Open Questions
- Should the chart support switching between distance/elevation/duration views or show all simultaneously?
- Should chat history persist across sessions or reset each time?
- What chart library should be used?

## Dependencies
- **Product dependencies**: UC-001 (prerequisite), UC-003 (training plan data), UC-005 (import navigation), UC-007 (AI chat), UC-008 (history nav), UC-009 (training nav), UC-010 (profile nav)
- **Technical dependencies**: PostgreSQL, charting library, AI service API, frontend framework
- **Legal / Privacy constraints**: None specific (all local data)
