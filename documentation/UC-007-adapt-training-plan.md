# UC-007 — Adapt Training Plan

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-007 |
| Version | 0.1 |
| Status | Draft |
| Priority | High |
| Primary Persona | System |
| Business Goal | Keep the training plan aligned with the user's actual performance and evolving needs |

## Summary
The system uses AI to analyze the user's recent training results and/or direct chat messages to adapt the training plan. This can be triggered automatically after importing training data (UC-006) or manually via the AI chat on the dashboard (UC-004). The adapted plan replaces future sessions while preserving completed history.

## Actors
- **Primary actor**: System
- **Secondary actors**: User (via chat interaction)
- **External systems**: LLM / AI Service

## Trigger
- Automatic: After successful data persistence in UC-006
- Manual: User sends a message via the AI chat tile on the dashboard (UC-004)

## Preconditions
- [ ] A training plan exists in the database
- [ ] Training history is available (for automatic adaptation)
- [ ] The AI service / LLM is accessible
- [ ] User profile data is available

## Expected Outcomes
### On Success
- Future training sessions are updated based on AI analysis
- Past (completed) sessions remain unchanged
- The user can see the updated plan on the dashboard and in the calendar (UC-009)

### On Failure
- The current plan remains unchanged
- The user is informed that adaptation failed
- The system logs the error for debugging

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| user_profile | Object | Yes | Full profile from UC-001/UC-002 (athlete profile, goal, running background, current training, health, recovery, trail specifics, strengths) | {full_name, goal_date, weekly_mileage, stress_level, ...} |
| training_history | Array | Yes | Completed sessions with metrics | [{session, metrics, effort}...] |
| current_plan | Array | Yes | Remaining planned sessions | [{date, type, duration}...] |
| user_message | String | No | Max 2000 chars (for chat trigger) | "I want to increase my long run distance" |

## Happy Path (Step-by-Step)
```
1. SYSTEM: Collects user profile, training history, and current plan from the database.
2. SYSTEM: Constructs an AI prompt with performance context and current plan state.
3. SYSTEM: If triggered by chat, includes the user's message in the prompt.
4. SYSTEM: Sends the prompt to the LLM/AI service.
5. SYSTEM: Receives the adapted plan in structured format.
6. SYSTEM: Validates the adapted plan structure.
7. SYSTEM: Replaces future (non-completed) sessions in the database.
8. SYSTEM: Confirms the adaptation to the user (dashboard update or chat response).
```

## Alternative Flows
### A1. No Significant Change Needed
```
1. SYSTEM sends data to AI for analysis.
2. AI determines current plan is appropriate given recent results.
3. SYSTEM keeps the plan unchanged.
4. SYSTEM informs the user ("Your plan is on track — no changes needed").
```

### A2. Chat-Triggered Modification
```
1. USER sends a specific request via chat ("Skip next week's long run").
2. SYSTEM includes the request in the AI prompt.
3. AI generates a modified plan respecting the user's request.
4. SYSTEM updates future sessions and confirms via chat response.
```

### A3. AI Suggests Reducing Load
```
1. AI detects high fatigue levels or underperformance in recent sessions.
2. AI generates a plan with reduced volume/intensity.
3. SYSTEM updates the plan and explains the reasoning to the user.
```

## Error Flows
### E1. AI Service Unavailable
```
1. SYSTEM fails to reach the AI service.
2. SYSTEM keeps the current plan unchanged.
3. SYSTEM informs the user of the temporary issue.
4. SYSTEM schedules a retry on next trigger.
```

### E2. Invalid AI Response
```
1. SYSTEM receives an unparseable or invalid plan from the AI.
2. SYSTEM retries with adjusted prompt (max 3 retries).
3. If all retries fail, current plan is preserved.
4. SYSTEM logs the issue and informs the user.
```

## Business Rules
- **BR1**: Completed sessions must never be modified during adaptation.
- **BR2**: Adapted sessions must respect the user's available training days (from `training_days_per_week`, `unavailable_days`, `preferred_long_run_day`).
- **BR3**: The AI must consider the user's perceived effort data, not just objective metrics.
- **BR4**: Chat-triggered adaptations must still validate against safety constraints (no unreasonable volumes).
- **BR5**: The system must keep a record of plan versions for potential rollback.

## MVP Scope
### Included
- Automatic adaptation after each training data import
- Manual adaptation via AI chat messages
- Replacement of future sessions only
- AI analysis of both objective metrics and perceived effort

### Excluded (Post-MVP)
- Plan version history with rollback
- User approval step before applying changes
- Adaptation based on external factors (weather, travel)
- Multi-week trend analysis

### Temporary Assumptions
- The same LLM is used for both plan generation and adaptation
- Adaptation replaces all future sessions (no partial updates)
- No user approval required before applying AI changes

## Non-Functional Requirements
- **Expected usage frequency**: 2–5 times per week (after each import) + ad-hoc chat requests
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: 1 AI call per adaptation; response time < 30s

## Acceptance Criteria
- [ ] GIVEN new training data imported WHEN adaptation runs THEN future sessions are updated while completed sessions remain unchanged
- [ ] GIVEN a user chat message requesting a change WHEN adaptation runs THEN the plan reflects the user's request
- [ ] GIVEN high perceived fatigue in recent sessions WHEN adaptation runs THEN the AI reduces upcoming session intensity
- [ ] GIVEN the AI service is unavailable WHEN adaptation is triggered THEN the current plan is preserved and the user is informed
- [ ] GIVEN adapted sessions WHEN they are persisted THEN they respect the user's schedule constraints (unavailable_days, training_days_per_week, limited_time_days)

## Success Metrics
- Plan adaptation completes in < 30 seconds
- Users perceive adapted plans as appropriate to their current fitness level

## Open Questions
- Should the user be asked to confirm plan changes before they are applied?
- How many past sessions should be included in the AI context?
- Should there be a limit on how often adaptation can be triggered?

## Dependencies
- **Product dependencies**: UC-004 (chat trigger), UC-006 (data trigger), UC-003 (initial plan), UC-009 (displays updated plan)
- **Technical dependencies**: LLM API, PostgreSQL, structured prompt engineering
- **Legal / Privacy constraints**: Training data sent to LLM; prefer local model if possible
