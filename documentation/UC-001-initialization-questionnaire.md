# UC-001 — Initialization Questionnaire

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-001 |
| Version | 0.1 |
| Status | Draft |
| Priority | High |
| Primary Persona | New User |
| Business Goal | Collect user context to generate a personalized training plan |

## Summary
A new user fills out an onboarding questionnaire at first login to provide personal and athletic information. This data is used to give the AI coach enough context to generate a first personalized running training plan.

## Actors
- **Primary actor**: User
- **Secondary actors**: None
- **External systems**: AI Training Plan Generator (UC-003)

## Trigger
The user logs in for the first time.

## Preconditions
- [ ] The user has created an account and is authenticated
- [ ] The user has not previously completed the initialization questionnaire
- [ ] The application backend and database are available

## Expected Outcomes
### On Success
- The user's personal and athletic data is persisted in the database
- The system triggers the AI training plan generation (UC-003)

### On Failure
- The user is informed of the error and can retry submitting the questionnaire
- No partial data is persisted

## Input Data Schema

### Athlete Profile
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| full_name | String | Yes | Max 100 characters | John Doe |
| age | Integer | Yes | 18 ≤ age ≤ 100 | 32 |
| date_of_birth | Date | Yes | Valid past date | 1994-03-15 |
| sex | Enum | Yes | Male / Female / Other | Male |
| location | String | Yes | Max 200 characters | Lyon, France |
| timezone | String | Yes | Valid IANA timezone | Europe/Paris |
| occupation | String | No | Max 200 characters | Software Engineer |
| family_responsibilities | String | No | Max 500 characters | Two young children |
| preferred_contact_method | Enum | No | Email / SMS / In-App | In-App |
| communication_style | Enum | Yes | Detailed / Concise / Encouraging / Direct | Encouraging |

### Goal and Timeline
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| main_goal_event | String | Yes | Max 200 characters | Trail du Mont Blanc |
| goal_distance | Float | Yes | > 0, in km | 42.0 |
| goal_date | Date | Yes | Future date | 2026-09-15 |
| goal_terrain | String | No | Max 300 characters | Mountain trails, 2500m elevation gain |
| goal_type | Enum | Yes | Finish / Time Target / Placement / Return to Running | Time Target |
| secondary_events | String | No | Max 500 characters | Local 10K in July as prep race |
| goal_motivation | String | No | Max 1000 characters | Personal challenge after injury recovery |

### Running Background
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| years_running | Float | Yes | ≥ 0 | 5.5 |
| years_trail_ultra | Float | No | ≥ 0 | 2.0 |
| race_results | String | No | Max 1000 characters | 10K: 42min, Half: 1h35, Marathon: 3h30 |
| weekly_mileage | Float | Yes | ≥ 0, in km | 45.0 |
| longest_recent_run | Float | No | ≥ 0, in km | 28.0 |
| easy_run_pace | String | No | Max 50 characters | 5:30-6:00 min/km |
| long_run_pace | String | No | Max 50 characters | 6:00-6:30 min/km |
| effort_tracking_method | Enum | Yes | Pace / Heart Rate / Power / Perceived Effort | Heart Rate |

### Current Training
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| recent_training_summary | String | Yes | Max 1000 characters | 4 runs/week, 1 long run, 1 interval... |
| training_days_per_week | Integer | Yes | 1–7 | 4 |
| workout_types | Array of String | No | Speed / Hill / Long Run / Doubles | [Speed, Long Run] |
| cross_training | String | No | Max 500 characters | Cycling 2x/week, hiking on weekends |
| strength_mobility | String | No | Max 500 characters | Bodyweight strength 2x/week |
| preferred_long_run_day | Enum | No | Mon–Sun | Saturday |
| unavailable_days | Array of Enum | No | Mon–Sun | [Tuesday, Thursday] |
| limited_time_days | Array of Enum | No | Mon–Sun | [Monday, Friday] |

### Health and Injury History
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| current_pain_concerns | String | No | Max 1000 characters | Mild left knee soreness after long runs |
| past_injuries | String | No | Max 1000 characters | IT band syndrome 2024, resolved |
| past_surgeries | String | No | Max 1000 characters | None |
| medications | String | No | Max 500 characters | None |
| exercise_tolerance_notes | String | No | Max 500 characters | None |
| medical_clearance | Boolean | Yes | true / false | true |

### Recovery and Lifestyle
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| avg_sleep_hours | Float | Yes | 3 ≤ x ≤ 14 | 7.5 |
| stress_level | Enum | Yes | Low / Moderate / High / Very High | Moderate |
| recovery_ability | Enum | Yes | Poor / Average / Good / Excellent | Good |
| schedule_irregularities | String | No | Max 500 characters | Business travel once a month |
| life_constraints | String | No | Max 500 characters | Childcare mornings, work 9-18h |

### Trail and Ultra Specifics
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| trail_access | Enum | No | None / Limited / Good / Excellent | Good |
| hill_access | Enum | No | None / Limited / Good / Excellent | Good |
| altitude_exposure | String | No | Max 200 characters | Live at 400m, train up to 1500m |
| technical_terrain_experience | Enum | No | None / Beginner / Intermediate / Advanced | Intermediate |
| hiking_ability_steep | Enum | No | Weak / Average / Strong | Strong |
| downhill_tolerance | Enum | No | Low / Moderate / High | Moderate |
| pole_experience | Enum | No | None / Beginner / Comfortable / Expert | Comfortable |

### Strengths and Limits
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| strengths | String | Yes | Max 1000 characters | Good endurance base, mental toughness |
| weaknesses | String | Yes | Max 1000 characters | Speed work, downhill running |
| consistency_derailers | String | No | Max 500 characters | Motivation drops in winter |
| improvement_priorities | String | Yes | Max 500 characters | Improve vertical gain capacity |
| goal_concerns | String | No | Max 500 characters | Worried about the distance being too much |

## Happy Path (Step-by-Step)
```
1. USER: Opens the application for the first time after account creation.
2. SYSTEM: Detects first login and displays the initialization questionnaire as a multi-section form.
3. USER: Fills in athlete profile (name, age, sex, location, communication preferences).
4. USER: Fills in goal and timeline (target race, distance, date, goal type).
5. USER: Fills in running background (experience, race results, weekly mileage, pacing).
6. USER: Fills in current training (days, workout types, cross-training, schedule constraints).
7. USER: Fills in health and injury history (current concerns, past injuries, medical clearance).
8. USER: Fills in recovery and lifestyle (sleep, stress, constraints).
9. USER: Fills in trail/ultra specifics (terrain access, skills).
10. USER: Fills in strengths and limits (strengths, weaknesses, priorities).
11. USER: Submits the questionnaire.
12. SYSTEM: Validates all required input fields.
13. SYSTEM: Persists user data to the PostgreSQL database (UC-002).
14. SYSTEM: Triggers AI training plan generation (UC-003).
15. SYSTEM: Redirects user to the dashboard with a confirmation message.
```

## Alternative Flows
### A1. Missing Data
```
1. SYSTEM detects missing required field(s).
2. SYSTEM highlights the missing fields and displays validation messages.
3. USER fills in the missing data and resubmits.
```

### A2. Invalid Data
```
1. SYSTEM detects out-of-range or invalid values (e.g., age = 5).
2. SYSTEM returns error with explanation of valid ranges.
3. USER corrects the data and resubmits.
```

## Error Flows
### E1. Internal System Error
```
1. SYSTEM encounters an unexpected failure during data persistence.
2. SYSTEM logs error details.
3. SYSTEM returns generic error message to USER.
4. No data is persisted (transaction rolled back).
```

### E2. Database Unavailable
```
1. SYSTEM fails to connect to PostgreSQL after retries.
2. SYSTEM displays an error message asking the user to try again later.
3. No data is persisted.
```

## Business Rules
- **BR1**: The questionnaire must be completed in full (all required fields) before any training plan can be generated.
- **BR2**: A user can only complete the initialization questionnaire once; subsequent modifications go through the profile editing flow (UC-010).
- **BR3**: At least one training day must be available (training_days_per_week ≥ 1).
- **BR4**: The goal date must be in the future.
- **BR5**: Medical clearance must be confirmed (medical_clearance = true) before plan generation.

## MVP Scope
### Included
- Simple form with all required fields
- Client-side and server-side validation
- Data persistence to PostgreSQL
- Trigger to AI plan generation

### Excluded (Post-MVP)
- Multi-step wizard with progress indicator
- Auto-save of partial questionnaire
- Integration with external fitness platforms for pre-filling data

### Temporary Assumptions
- The application runs locally with a Docker-containerized PostgreSQL instance
- No multi-user authentication system (single local user)

## Non-Functional Requirements
- **Expected usage frequency**: Once per user (at first login)
- **Number of affected users**: All new users
- **Expected load/volume**: 1 request (local single-user application)

## Acceptance Criteria
- [ ] GIVEN a new user at first login WHEN the questionnaire is displayed THEN all sections (athlete profile, goal, running background, current training, health, recovery, trail specifics, strengths) are present
- [ ] GIVEN a user submits a valid questionnaire WHEN all required fields pass validation THEN data is persisted in the database and the AI plan generation is triggered
- [ ] GIVEN a user submits an incomplete questionnaire WHEN required fields are missing THEN an error is displayed and no data is saved
- [ ] GIVEN a user who has already completed the questionnaire WHEN they log in again THEN they are redirected to the dashboard (UC-004) instead of the questionnaire
- [ ] GIVEN a user without medical clearance WHEN they submit the questionnaire THEN they are informed that medical clearance is required before plan generation

## Success Metrics
- 100% of new users complete the questionnaire before accessing the dashboard
- Zero data loss on valid questionnaire submissions

## Open Questions
- Should we support metric and imperial units for distance and pace?
- Should the questionnaire be split into multiple pages/steps or presented as a single long form?
- Should trail/ultra-specific fields be shown conditionally based on the goal type?

## Dependencies
- **Product dependencies**: UC-002 (data persistence), UC-003 (AI plan generation)
- **Technical dependencies**: PostgreSQL database, Docker environment
- **Legal / Privacy constraints**: User data stored locally only (no cloud transmission in MVP)
