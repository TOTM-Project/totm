# UC-006 — Store Running Data in PostgreSQL

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-006 |
| Version | 0.1 |
| Status | Draft |
| Priority | High |
| Primary Persona | System |
| Business Goal | Persist parsed running session data for history tracking and AI plan adaptation |

## Summary
The system reads a parsed .FIT file and persists the extracted running data into the PostgreSQL database. This enables the user to maintain a training history and provides the AI with objective performance data to adapt the training plan.

## Actors
- **Primary actor**: System
- **Secondary actors**: None
- **External systems**: PostgreSQL database (Docker container)

## Trigger
A .FIT file has been successfully parsed during the import flow (UC-005).

## Preconditions
- [ ] The .FIT file has been parsed and data is available in a structured format
- [ ] The target training session exists in the database
- [ ] The PostgreSQL database is accessible
- [ ] The perceived effort data from the questionnaire is available

## Expected Outcomes
### On Success
- All extracted running metrics are persisted and linked to the session
- Perceived effort data is stored alongside objective metrics
- The training plan adaptation (UC-007) is triggered

### On Failure
- The transaction is rolled back — no partial data is saved
- The calling use case (UC-005) is notified of the failure

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| session_id | UUID | Yes | Must reference existing planned session | 550e8400-... |
| total_distance | Float | Yes | > 0, in meters | 10523.5 |
| total_duration | Integer | Yes | > 0, in seconds | 3245 |
| total_elevation_gain | Float | No | ≥ 0, in meters | 156.3 |
| average_pace | Float | Yes | > 0, in min/km | 5.12 |
| average_heart_rate | Integer | No | 40–220 bpm | 152 |
| max_heart_rate | Integer | No | 40–220 bpm | 178 |
| calories | Integer | No | ≥ 0 | 620 |
| start_time | Timestamp | Yes | Valid datetime | 2026-05-30T07:30:00Z |
| perceived_difficulty | Enum | Yes | Very Easy / Easy / Moderate / Hard / Very Hard | Moderate |
| perceived_fatigue | Integer | Yes | 1–10 | 6 |
| notes | String | No | Max 1000 chars | Felt good overall |
| recorded_at | Timestamp | Yes | Auto-generated | 2026-05-31T10:00:00Z |

## Happy Path (Step-by-Step)
```
1. SYSTEM: Receives parsed .FIT data and perceived effort data from UC-005.
2. SYSTEM: Opens a database transaction.
3. SYSTEM: Inserts the running session record with all metrics.
4. SYSTEM: Links the record to the planned training session.
5. SYSTEM: Marks the planned session as "completed" in the training plan.
6. SYSTEM: Commits the transaction.
7. SYSTEM: Triggers training plan adaptation (UC-007).
8. SYSTEM: Returns success to UC-005.
```

## Alternative Flows
### A1. Overwrite Existing Data
```
1. SYSTEM detects the target session already has recorded data (user confirmed overwrite).
2. SYSTEM deletes the existing running record within the same transaction.
3. SYSTEM inserts the new data.
4. SYSTEM commits transaction.
```

### A2. Partial FIT Data
```
1. SYSTEM detects some optional fields are missing from the .FIT file (e.g., no heart rate data).
2. SYSTEM persists available data with NULL for missing optional fields.
3. SYSTEM proceeds normally.
```

## Error Flows
### E1. Database Write Failure
```
1. SYSTEM encounters an error during INSERT.
2. SYSTEM rolls back the transaction.
3. SYSTEM logs error details.
4. SYSTEM returns error to UC-005.
```

### E2. Referential Integrity Error
```
1. SYSTEM attempts to link data to a non-existent session_id.
2. SYSTEM rolls back and logs the integrity error.
3. SYSTEM returns specific error to UC-005.
```

## Business Rules
- **BR1**: All data writes must be atomic (all or nothing within a transaction).
- **BR2**: The planned session must be marked as "completed" when data is recorded.
- **BR3**: After successful persistence, UC-007 must be triggered for plan adaptation.
- **BR4**: Optional metrics (heart rate, calories) may be NULL if not present in .FIT file.

## MVP Scope
### Included
- Storage of core running metrics (distance, duration, elevation, pace)
- Storage of optional metrics (heart rate, calories) when available
- Storage of perceived effort data
- Session status update (planned → completed)
- Trigger to UC-007

### Excluded (Post-MVP)
- Storage of GPS track/route data
- Lap-by-lap detailed splits
- Power data storage
- Data aggregation/rollup tables

### Temporary Assumptions
- Single table for running sessions with all metrics
- No data compression or archiving needed at MVP scale
- GPS/route data is discarded after metric extraction

## Non-Functional Requirements
- **Expected usage frequency**: 2–5 times per week
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: Single record per import; total records < 1000 in first year

## Acceptance Criteria
- [ ] GIVEN parsed .FIT data WHEN the system persists it THEN all required metrics are stored and retrievable
- [ ] GIVEN a successful persist WHEN the session is saved THEN the planned session status changes to "completed"
- [ ] GIVEN a database failure WHEN persistence is attempted THEN no partial data is saved (full rollback)
- [ ] GIVEN a successful persist WHEN data is committed THEN UC-007 (plan adaptation) is triggered
- [ ] GIVEN a .FIT file without heart rate data WHEN persisted THEN heart rate fields are NULL but other data is saved

## Success Metrics
- Zero data loss on valid imports
- Write operation completes in < 1 second

## Open Questions
- Should we store raw .FIT file as a blob for potential re-parsing later?
- What indexes are needed for efficient historical queries?

## Dependencies
- **Product dependencies**: UC-005 (data source), UC-007 (triggered after persistence), UC-008 (reads historical data)
- **Technical dependencies**: PostgreSQL, Docker, database migration tool
- **Legal / Privacy constraints**: Health-related data stored locally only
