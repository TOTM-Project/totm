# UC-010 — Edit Personal Data

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-010 |
| Version | 0.1 |
| Status | Draft |
| Priority | Medium |
| Primary Persona | User |
| Business Goal | Allow users to update their profile to keep the training plan relevant |

## Summary
In a "Profile" menu accessible from the dashboard, the user can modify the personal data originally collected during the initialization questionnaire (UC-001). Changes to profile data trigger a training plan adaptation to reflect the updated information.

## Actors
- **Primary actor**: User
- **Secondary actors**: None
- **External systems**: PostgreSQL database

## Trigger
The user navigates to the "Profile" section from the dashboard (UC-004).

## Preconditions
- [ ] The user has completed the initialization questionnaire (UC-001)
- [ ] User profile data exists in the database
- [ ] The database is accessible

## Expected Outcomes
### On Success
- Updated profile data is persisted in the database (UC-002)
- The training plan adaptation (UC-007) is triggered to reflect the changes
- The user sees a confirmation of the update

### On Failure
- Invalid data is rejected with validation messages
- The original data remains unchanged on failure
- The user can correct and retry

## Input Data Schema

### Athlete Profile
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| full_name | String | Yes | Max 100 characters | John Doe |
| age | Integer | Yes | 18 ≤ age ≤ 100 | 33 |
| date_of_birth | Date | Yes | Valid past date | 1994-03-15 |
| sex | Enum | Yes | Male / Female / Other | Male |
| location | String | Yes | Max 200 characters | Lyon, France |
| timezone | String | Yes | Valid IANA timezone | Europe/Paris |
| occupation | String | No | Max 200 characters | Software Engineer |
| family_responsibilities | String | No | Max 500 characters | Two young children |
| preferred_contact_method | Enum | No | Email / SMS / In-App | In-App |
| communication_style | Enum | Yes | Detailed / Concise / Encouraging / Direct | Direct |

### Goal and Timeline
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| main_goal_event | String | Yes | Max 200 characters | UTMB CCC |
| goal_distance | Float | Yes | > 0, in km | 101.0 |
| goal_date | Date | Yes | Future date | 2027-08-25 |
| goal_terrain | String | No | Max 300 characters | Alpine trails, 6100m D+ |
| goal_type | Enum | Yes | Finish / Time Target / Placement / Return to Running | Finish |
| secondary_events | String | No | Max 500 characters | Trail 50K in June as qualifier |
| goal_motivation | String | No | Max 1000 characters | Bucket list race |

### Running Background
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| years_running | Float | Yes | ≥ 0 | 6.0 |
| years_trail_ultra | Float | No | ≥ 0 | 3.0 |
| race_results | String | No | Max 1000 characters | 10K: 40min, Marathon: 3h25, Trail 50K: 6h |
| weekly_mileage | Float | Yes | ≥ 0, in km | 55.0 |
| longest_recent_run | Float | No | ≥ 0, in km | 35.0 |
| easy_run_pace | String | No | Max 50 characters | 5:15-5:45 min/km |
| long_run_pace | String | No | Max 50 characters | 5:45-6:15 min/km |
| effort_tracking_method | Enum | Yes | Pace / Heart Rate / Power / Perceived Effort | Heart Rate |

### Current Training
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| recent_training_summary | String | Yes | Max 1000 characters | 5 runs/week, 2 quality sessions... |
| training_days_per_week | Integer | Yes | 1–7 | 5 |
| workout_types | Array of String | No | Speed / Hill / Long Run / Doubles | [Speed, Hill, Long Run] |
| cross_training | String | No | Max 500 characters | Cycling, ski touring in winter |
| strength_mobility | String | No | Max 500 characters | Strength 3x/week, yoga 1x/week |
| preferred_long_run_day | Enum | No | Mon–Sun | Sunday |
| unavailable_days | Array of Enum | No | Mon–Sun | [Tuesday] |
| limited_time_days | Array of Enum | No | Mon–Sun | [Monday, Friday] |

### Health and Injury History
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| current_pain_concerns | String | No | Max 1000 characters | None currently |
| past_injuries | String | No | Max 1000 characters | IT band 2024, Achilles tendinitis 2025 |
| past_surgeries | String | No | Max 1000 characters | None |
| medications | String | No | Max 500 characters | None |
| exercise_tolerance_notes | String | No | Max 500 characters | None |
| medical_clearance | Boolean | Yes | true / false | true |

### Recovery and Lifestyle
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| avg_sleep_hours | Float | Yes | 3 ≤ x ≤ 14 | 8.0 |
| stress_level | Enum | Yes | Low / Moderate / High / Very High | Low |
| recovery_ability | Enum | Yes | Poor / Average / Good / Excellent | Excellent |
| schedule_irregularities | String | No | Max 500 characters | None |
| life_constraints | String | No | Max 500 characters | Flexible remote work |

### Trail and Ultra Specifics
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| trail_access | Enum | No | None / Limited / Good / Excellent | Excellent |
| hill_access | Enum | No | None / Limited / Good / Excellent | Excellent |
| altitude_exposure | String | No | Max 200 characters | Live at 600m, regular access to 2000m+ |
| technical_terrain_experience | Enum | No | None / Beginner / Intermediate / Advanced | Advanced |
| hiking_ability_steep | Enum | No | Weak / Average / Strong | Strong |
| downhill_tolerance | Enum | No | Low / Moderate / High | High |
| pole_experience | Enum | No | None / Beginner / Comfortable / Expert | Expert |

### Strengths and Limits
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|--------|
| strengths | String | Yes | Max 1000 characters | Strong climber, good endurance |
| weaknesses | String | Yes | Max 1000 characters | Nutrition management on ultras |
| consistency_derailers | String | No | Max 500 characters | Overtraining tendency |
| improvement_priorities | String | Yes | Max 500 characters | Long-distance nutrition strategy |
| goal_concerns | String | No | Max 500 characters | Cut-off times on CCC |

## Happy Path (Step-by-Step)
```
1. USER: Clicks on "Profile" in the navigation menu.
2. SYSTEM: Fetches current user profile data from the database.
3. SYSTEM: Displays the profile form pre-filled with current values (all sections).
4. USER: Modifies one or more fields across any section (e.g., updates goal, changes training days, updates health info).
5. USER: Submits the updated profile.
6. SYSTEM: Validates all input fields.
7. SYSTEM: Persists the updated data to the database (UC-002).
8. SYSTEM: Triggers training plan adaptation (UC-007).
9. SYSTEM: Displays a success confirmation message.
10. USER: Returns to the dashboard or stays on profile page.
```

## Alternative Flows
### A1. No Changes Made
```
1. USER opens the profile form but submits without making changes.
2. SYSTEM detects no fields were modified.
3. SYSTEM does not trigger a database write or plan adaptation.
4. SYSTEM informs the user ("No changes detected").
```

### A2. Invalid Data Submitted
```
1. USER modifies fields with invalid values (e.g., age = 0).
2. SYSTEM validates and detects errors.
3. SYSTEM highlights invalid fields with error messages.
4. USER corrects the data and resubmits.
```

### A3. Training Days Changed
```
1. USER changes their available training days.
2. SYSTEM persists the update.
3. SYSTEM triggers plan adaptation (UC-007) which must reschedule future sessions to the new days.
```

## Error Flows
### E1. Database Write Failure
```
1. SYSTEM fails to persist the updated data.
2. SYSTEM displays an error message.
3. Original data remains unchanged.
4. USER can retry submission.
```

### E2. Plan Adaptation Failure
```
1. Data is persisted successfully but plan adaptation (UC-007) fails.
2. SYSTEM informs the user that data was saved but plan update is pending.
3. SYSTEM schedules a retry for plan adaptation.
```

## Business Rules
- **BR1**: All fields that were required in UC-001 remain required in profile editing.
- **BR2**: Profile changes must trigger training plan adaptation (UC-007).
- **BR3**: If no fields are changed, no database write or adaptation is triggered.
- **BR4**: The system must keep the updated_at timestamp current on every save.

## MVP Scope
### Included
- Pre-filled form with current profile data
- Edit all fields from the initialization questionnaire
- Validation identical to UC-001
- Persistence via UC-002
- Trigger plan adaptation via UC-007

### Excluded (Post-MVP)
- Profile photo upload
- Change history / audit log
- Account deletion
- Password/authentication management

### Temporary Assumptions
- Same form layout as the initialization questionnaire
- No confirmation dialog before saving (immediate save on submit)
- Single local user; no access control needed

## Non-Functional Requirements
- **Expected usage frequency**: Occasionally (a few times per month)
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: Single read + single write per update

## Acceptance Criteria
- [ ] GIVEN an existing profile WHEN the user accesses Profile THEN the form is pre-filled with current data
- [ ] GIVEN valid updated data WHEN the user submits THEN the data is persisted and plan adaptation is triggered
- [ ] GIVEN invalid data WHEN the user submits THEN validation errors are shown and no data is saved
- [ ] GIVEN no changes WHEN the user submits THEN no database write or adaptation occurs
- [ ] GIVEN a training days change WHEN adaptation runs THEN future sessions are rescheduled to the new days

## Success Metrics
- Profile updates persist correctly 100% of the time
- Plan adaptation triggers within 30 seconds of profile save

## Open Questions
- Should there be a confirmation dialog before saving changes?
- Should certain fields (like sex) be non-editable after initialization?

## Dependencies
- **Product dependencies**: UC-001 (initial data), UC-002 (persistence), UC-004 (navigation), UC-007 (plan adaptation)
- **Technical dependencies**: PostgreSQL, form validation library, frontend framework
- **Legal / Privacy constraints**: User data stored locally only
