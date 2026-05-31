# UC-005 — Import Training Data

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-005 |
| Version | 0.1 |
| Status | Draft |
| Priority | High |
| Primary Persona | User |
| Business Goal | Allow users to import actual running session data for tracking and AI feedback |

## Summary
From the dashboard, the user can import training data for a specific scheduled session. The user uploads a .FIT file from their running watch and fills out a short questionnaire about perceived effort. This provides the system with objective and subjective data to track progress and allow AI feedback.

## Actors
- **Primary actor**: User
- **Secondary actors**: None
- **External systems**: File system (for .FIT file upload)

## Trigger
The user clicks the "Import Data" button on the next session tile in the dashboard (UC-004).

## Preconditions
- [ ] The user has an active training plan with at least one session
- [ ] The user has a .FIT file from a running watch/device
- [ ] The target session exists in the database

## Expected Outcomes
### On Success
- The .FIT file is parsed and training metrics are extracted
- The user's perceived effort is recorded
- Data is linked to the specific training session
- The system triggers data persistence (UC-006)

### On Failure
- Invalid .FIT files are rejected with a clear error message
- The user can retry the upload with a different file
- No partial data is persisted

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| fit_file | File (.fit) | Yes | Valid .FIT format, max 50MB | activity_2026-05-30.fit |
| session_id | UUID | Yes | Must reference an existing planned session | 550e8400-e29b-41d4-a716-446655440000 |
| perceived_difficulty | Enum | Yes | Very Easy / Easy / Moderate / Hard / Very Hard | Moderate |
| perceived_fatigue | Integer | Yes | 1–10 scale | 6 |
| notes | String | No | Max 1000 characters | Felt good, slightly tired at the end |

## Happy Path (Step-by-Step)
```
1. USER: Clicks "Import Data" on the dashboard session tile.
2. SYSTEM: Displays the import form with file upload and effort questionnaire.
3. USER: Selects a .FIT file from their file system.
4. SYSTEM: Validates the file format (.FIT extension and valid structure).
5. USER: Fills in the perceived difficulty and fatigue questionnaire.
6. USER: Optionally adds free-text notes.
7. USER: Submits the import form.
8. SYSTEM: Parses the .FIT file and extracts training metrics.
9. SYSTEM: Links the data to the target session and persists it (UC-006).
10. SYSTEM: Displays a success confirmation with session summary.
11. SYSTEM: Redirects the user to the dashboard.
```

## Alternative Flows
### A1. Invalid File Format
```
1. USER selects a non-.FIT file or a corrupted .FIT file.
2. SYSTEM detects the invalid format during validation.
3. SYSTEM displays an error ("Invalid file format. Please upload a valid .FIT file.").
4. USER selects a different file and retries.
```

### A2. File Too Large
```
1. USER selects a .FIT file exceeding the size limit.
2. SYSTEM rejects the file with a size error message.
3. USER selects a smaller or more recent file.
```

### A3. Session Already Has Data
```
1. USER tries to import data for a session that already has recorded data.
2. SYSTEM warns the user that existing data will be overwritten.
3. USER confirms the overwrite or cancels.
```

## Error Flows
### E1. FIT File Parsing Error
```
1. SYSTEM fails to parse the .FIT file content.
2. SYSTEM logs the parsing error with file details.
3. SYSTEM displays a user-friendly error message.
4. No data is persisted.
```

### E2. Database Persistence Failure
```
1. SYSTEM successfully parses the file but fails to persist data.
2. SYSTEM rolls back the transaction.
3. SYSTEM displays an error and asks the user to retry.
```

## Business Rules
- **BR1**: Each import must be linked to a specific planned training session.
- **BR2**: The perceived effort questionnaire is mandatory — file upload alone is insufficient.
- **BR3**: Only .FIT format files are accepted in the MVP.
- **BR4**: If data already exists for a session, the user must explicitly confirm overwrite.

## MVP Scope
### Included
- .FIT file upload and parsing
- Perceived effort questionnaire (difficulty + fatigue)
- Data linked to a specific session
- Persistence to PostgreSQL (UC-006)

### Excluded (Post-MVP)
- Drag-and-drop file upload
- Support for .GPX, .TCX, or other formats
- Automatic sync from Garmin/Strava APIs
- Batch import of multiple sessions

### Temporary Assumptions
- .FIT file parsing uses an existing library (e.g., fitparse for Python)
- The user manually exports .FIT files from their watch/app
- One .FIT file corresponds to one training session

## Non-Functional Requirements
- **Expected usage frequency**: After each training session (2–5 times per week)
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: Single file upload, typical .FIT file < 5MB

## Acceptance Criteria
- [ ] GIVEN a valid .FIT file WHEN the user uploads it THEN training metrics (distance, duration, elevation, pace) are extracted and displayed
- [ ] GIVEN the effort questionnaire WHEN the user submits all required fields THEN the data is persisted alongside the .FIT metrics
- [ ] GIVEN an invalid file WHEN the user tries to upload it THEN a clear error message is shown and no data is saved
- [ ] GIVEN a session with existing data WHEN the user imports new data THEN they are warned and must confirm overwrite
- [ ] GIVEN a successful import WHEN data is persisted THEN the dashboard chart updates to reflect the new data

## Success Metrics
- Successful import rate > 95% for valid .FIT files
- Import flow completion time < 2 minutes (including questionnaire)

## Open Questions
- Which .FIT parsing library should be used?
- Should we show a preview of parsed data before final submission?
- What specific metrics should be extracted from the .FIT file?

## Dependencies
- **Product dependencies**: UC-004 (trigger), UC-006 (data persistence), UC-007 (plan adaptation trigger)
- **Technical dependencies**: .FIT file parsing library, PostgreSQL, file upload handling
- **Legal / Privacy constraints**: Training data stored locally only
