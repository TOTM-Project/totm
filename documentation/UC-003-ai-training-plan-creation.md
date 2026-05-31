# UC-003 — AI Training Plan Creation

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-003 |
| Version | 0.1 |
| Status | Draft |
| Priority | High |
| Primary Persona | System |
| Business Goal | Generate a personalized running training plan using AI based on user profile |

## Summary
An AI specialized in running coaching analyzes the user's stored profile data and generates a tailored training program. This occurs automatically after the user's data is first saved (UC-002). The plan must match the user's experience level, goals, and available training days.

## Actors
- **Primary actor**: System
- **Secondary actors**: None
- **External systems**: LLM / AI Service (running coaching specialist)

## Trigger
Successful initial data persistence in UC-002.

## Preconditions
- [ ] User profile data is available in the database (UC-002 completed)
- [ ] The AI service / LLM is accessible
- [ ] Training plan schema is defined in the database

## Expected Outcomes
### On Success
- A multi-week training plan is generated and stored in the database
- Each session includes: date, type (easy run, intervals, long run, etc.), duration, distance, and instructions
- The user is redirected to the dashboard (UC-004) showing the new plan

### On Failure
- The system logs the AI generation error
- The user is informed that plan generation failed and can retry
- No incomplete plan is persisted

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| age | Integer | Yes | From user profile | 32 |
| weight | Float | Yes | From user profile | 72.5 |
| sex | Enum | Yes | From user profile | Male |
| running_experience | Enum | Yes | From user profile | Intermediate |
| running_goal | String | Yes | From user profile | Run a marathon in under 4 hours |
| training_days | Array of Enum | Yes | From user profile | [Monday, Wednesday, Saturday] |

## Happy Path (Step-by-Step)
```
1. SYSTEM: Retrieves user profile data from the database.
2. SYSTEM: Constructs a structured prompt for the AI with user context.
3. SYSTEM: Sends the prompt to the LLM/AI service.
4. SYSTEM: Receives the generated training plan in structured format.
5. SYSTEM: Validates the plan structure (correct dates, valid session types, etc.).
6. SYSTEM: Persists the training plan sessions to the database.
7. SYSTEM: Notifies the UI that the plan is ready.
```

## Alternative Flows
### A1. AI Returns Invalid Structure
```
1. SYSTEM receives a response that doesn't match the expected plan schema.
2. SYSTEM retries the generation with an adjusted prompt (max 3 retries).
3. If successful on retry, continues with normal flow.
4. If all retries fail, triggers error flow E1.
```

### A2. Partial Plan Generation
```
1. SYSTEM receives a valid but incomplete plan (fewer weeks than expected).
2. SYSTEM persists the available plan.
3. SYSTEM flags the plan as partial and schedules a completion attempt.
```

## Error Flows
### E1. AI Service Unavailable
```
1. SYSTEM fails to reach the AI service after 3 retries.
2. SYSTEM logs the failure with timestamps and error details.
3. SYSTEM displays an error to the user with a "Retry" button.
4. No plan data is persisted.
```

### E2. Internal Processing Error
```
1. SYSTEM encounters an error while parsing or persisting the plan.
2. SYSTEM rolls back any partial database writes.
3. SYSTEM logs error details.
4. SYSTEM informs the user of the failure.
```

## Business Rules
- **BR1**: The training plan must only schedule sessions on the user's selected training days.
- **BR2**: The plan must respect progressive overload principles (gradual increase in volume).
- **BR3**: The plan must be appropriate for the user's declared experience level.
- **BR4**: Each session must have a clear type, target duration, and description.
- **BR5**: The initial plan should cover at least 4 weeks.

## MVP Scope
### Included
- AI-generated training plan based on user profile
- Structured plan with individual sessions (date, type, duration, distance, description)
- Plan storage in PostgreSQL
- Single generation at initialization

### Excluded (Post-MVP)
- Plan regeneration based on performance feedback
- Multiple plan options for the user to choose from
- Integration with periodization models
- Support for non-running activities

### Temporary Assumptions
- The LLM is accessed via API (e.g., OpenAI, local model)
- The plan is generated in a single request/response cycle
- Plan structure is returned as JSON from the AI

## Non-Functional Requirements
- **Expected usage frequency**: Once at initialization, then on plan regeneration (UC-007)
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: 1 AI call per plan generation; response time < 30s

## Acceptance Criteria
- [ ] GIVEN a completed user profile WHEN the AI generates a plan THEN the plan contains at least 4 weeks of training sessions
- [ ] GIVEN the user selected 3 training days WHEN a plan is generated THEN sessions are only scheduled on those 3 days
- [ ] GIVEN a beginner user WHEN a plan is generated THEN session intensity and volume are appropriate for beginners
- [ ] GIVEN the AI service is unavailable WHEN plan generation is triggered THEN the user sees an error message and can retry
- [ ] GIVEN a generated plan WHEN it is persisted THEN each session has a date, type, duration, and description

## Success Metrics
- Plan generated successfully on first attempt in > 90% of cases
- Generated plans align with the user's stated goal and experience level

## Open Questions
- Which LLM provider/model should be used?
- Should the prompt include example plans for few-shot learning?
- What is the maximum acceptable generation time before showing a timeout?

## Dependencies
- **Product dependencies**: UC-002 (data source), UC-004 (displays the plan), UC-007 (plan adaptation)
- **Technical dependencies**: LLM API (OpenAI / local model), PostgreSQL, structured output parsing
- **Legal / Privacy constraints**: User data sent to LLM must be handled per privacy policy; prefer local models if possible
