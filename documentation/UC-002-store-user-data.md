# UC-002 — Store User Common Data in PostgreSQL

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-002 |
| Version | 0.1 |
| Status | Draft |
| Priority | High |
| Primary Persona | System |
| Business Goal | Persist user profile data locally for use across sessions |

## Summary
The system stores the user's common data (personal information and preferences) in a locally containerized PostgreSQL database. This occurs after the initialization questionnaire (UC-001) or when the user modifies their profile data (UC-010). The data must persist between application sessions.

## Actors
- **Primary actor**: System
- **Secondary actors**: None
- **External systems**: PostgreSQL database (Docker container)

## Trigger
The user completes the initialization questionnaire (UC-001) or updates their personal data (UC-010).

## Preconditions
- [ ] The PostgreSQL Docker container is running and accessible
- [ ] The user data has been validated by the calling use case (UC-001 or UC-010)
- [ ] The database schema is initialized

## Expected Outcomes
### On Success
- User data is persisted in the PostgreSQL database
- The AI training plan generation (UC-003) is triggered if this is the initial save

### On Failure
- The transaction is rolled back and no partial data is saved
- The calling use case is notified of the failure

## Input Data Schema

### System Fields
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| user_id | UUID | Yes | Valid UUID format | 550e8400-e29b-41d4-a716-446655440000 |
| created_at | Timestamp | Yes | Auto-generated on INSERT | 2026-05-31T10:30:00Z |
| updated_at | Timestamp | Yes | Auto-generated on every write | 2026-05-31T10:30:00Z |

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
1. SYSTEM: Receives validated user data from UC-001 or UC-010.
2. SYSTEM: Opens a database transaction.
3. SYSTEM: Inserts or updates the user record in the users table.
4. SYSTEM: Commits the transaction.
5. SYSTEM: Returns success status to the calling use case.
6. SYSTEM: If initial save, triggers UC-003 (AI training plan generation).
```

## Alternative Flows
### A1. Update Existing Record
```
1. SYSTEM detects user_id already exists in the database.
2. SYSTEM performs an UPDATE instead of INSERT.
3. SYSTEM updates the updated_at timestamp.
4. SYSTEM commits transaction and returns success.
```

### A2. Schema Migration Required
```
1. SYSTEM detects database schema version mismatch.
2. SYSTEM runs pending migrations before data operation.
3. SYSTEM proceeds with normal insert/update flow.
```

## Error Flows
### E1. Database Connection Failure
```
1. SYSTEM fails to connect to PostgreSQL container.
2. SYSTEM logs connection error with details.
3. SYSTEM returns error to calling use case.
4. No data is persisted.
```

### E2. Constraint Violation
```
1. SYSTEM encounters a database constraint violation.
2. SYSTEM rolls back the transaction.
3. SYSTEM logs the constraint details.
4. SYSTEM returns specific error to calling use case.
```

## Business Rules
- **BR1**: All database operations must be wrapped in transactions.
- **BR2**: The updated_at field must be set on every write operation.
- **BR3**: On initial save (INSERT), the system must trigger UC-003 after successful commit.
- **BR4**: On update, the system must notify UC-007 to re-evaluate the training plan.

## MVP Scope
### Included
- Single-user data storage in PostgreSQL
- Insert and update operations with transaction safety
- Docker-containerized database instance
- Automatic trigger to UC-003 on initial save

### Excluded (Post-MVP)
- Multi-user support with authentication
- Data encryption at rest
- Backup and restore functionality
- Cloud synchronization

### Temporary Assumptions
- Single local user; no multi-tenancy
- Database runs in a Docker container on the same machine
- No data migration from external sources

## Non-Functional Requirements
- **Expected usage frequency**: Once at initialization, then occasionally on profile updates
- **Number of affected users**: 1 (local single-user application)
- **Expected load/volume**: Low; single writes on user action

## Acceptance Criteria
- [ ] GIVEN valid user data from UC-001 WHEN the system persists it THEN the data is retrievable from PostgreSQL with all fields intact
- [ ] GIVEN the database is unavailable WHEN a write is attempted THEN the system returns an error and no partial data is saved
- [ ] GIVEN an existing user record WHEN the user updates their data via UC-010 THEN the record is updated (not duplicated) and updated_at is refreshed
- [ ] GIVEN a successful initial save WHEN the transaction commits THEN UC-003 is triggered

## Success Metrics
- Zero data loss on valid submissions
- Database write latency < 500ms for single-user operations

## Open Questions
- Should we use a migration tool (e.g., Flyway, Alembic) for schema management?
- What is the retention policy for user data?

## Dependencies
- **Product dependencies**: UC-001 (input source), UC-003 (triggered on initial save), UC-010 (input source for updates)
- **Technical dependencies**: PostgreSQL 15+, Docker, ORM or database driver
- **Legal / Privacy constraints**: Data stored locally only; no external transmission
