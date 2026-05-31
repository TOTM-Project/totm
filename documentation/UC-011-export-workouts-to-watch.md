# UC-011 — Export Workouts to Watch

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-011 |
| Version | 0.1 |
| Status | Draft |
| Priority | Medium |
| Primary Persona | System |
| Business Goal | Enable users to transfer planned workouts to their running watch for guided training |

## Summary
Future planned training sessions can be exported as .FIT files. Users download these files to their computer and transfer them to their sports watch via the watch's companion application. This allows guided, structured workouts on the watch during training sessions.

## Actors
- **Primary actor**: User
- **Secondary actors**: None
- **External systems**: File system, sports watch companion app (Garmin Connect, etc.)

## Trigger
The user clicks "Export to .FIT" on a session detail view from the dashboard (UC-004) or the training calendar (UC-009).

## Preconditions
- [ ] A training plan exists with future sessions
- [ ] The target session has sufficient detail to generate a structured .FIT workout (type, duration, targets)
- [ ] The application can generate valid .FIT workout files

## Expected Outcomes
### On Success
- A valid .FIT workout file is generated and downloaded to the user's computer
- The file is compatible with standard sports watch platforms (Garmin, etc.)
- The user can transfer the file to their watch via the companion app

### On Failure
- If workout data is insufficient for .FIT generation, the user is informed
- If file generation fails, an error message is displayed
- No corrupted files are delivered to the user

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
| session_id | UUID | Yes | Existing planned session | 550e8400-... |
| session_type | Enum | Yes | Easy Run / Intervals / Long Run / Recovery / Tempo | Intervals |
| planned_duration | Integer | Yes | > 0, in minutes | 45 |
| planned_distance | Float | No | In km | 8.0 |
| workout_steps | Array | Yes | At least 1 step | [{type: "warmup", duration: 10}, ...] |
| target_pace | Float | No | In min/km | 5.00 |
| target_heart_rate_zone | Integer | No | 1–5 | 3 |

## Happy Path (Step-by-Step)
```
1. USER: Navigates to a future session detail (from UC-004 or UC-009).
2. USER: Clicks the "Export to .FIT" button.
3. SYSTEM: Retrieves the session details from the database.
4. SYSTEM: Converts the session structure into .FIT workout format.
5. SYSTEM: Generates the .FIT file with appropriate workout steps.
6. SYSTEM: Triggers a file download to the user's computer.
7. USER: Downloads the file.
8. USER: Transfers the .FIT file to their watch via companion app (outside the application).
```

## Alternative Flows
### A1. Simple Session (No Structured Steps)
```
1. SYSTEM detects the session is a simple run (e.g., "Easy Run 45min") without structured intervals.
2. SYSTEM generates a basic .FIT workout with single-step duration target.
3. File is downloaded normally.
```

### A2. Interval Session
```
1. SYSTEM detects the session has structured intervals (warmup, work, recovery, cooldown).
2. SYSTEM generates a multi-step .FIT workout with appropriate step types and targets.
3. File is downloaded normally.
```

### A3. Missing Target Data
```
1. SYSTEM detects the session lacks specific targets (pace, HR zone).
2. SYSTEM generates a .FIT workout with duration-only targets.
3. SYSTEM informs the user that targets are generic.
```

## Error Flows
### E1. FIT File Generation Error
```
1. SYSTEM encounters an error during .FIT file generation.
2. SYSTEM logs the error details.
3. SYSTEM displays an error message to the user.
4. No file is downloaded.
```

### E2. Session Data Insufficient
```
1. SYSTEM detects the session has no meaningful data to export.
2. SYSTEM informs the user that this session cannot be exported.
3. No file is generated.
```

## Business Rules
- **BR1**: Only future (non-completed) sessions can be exported.
- **BR2**: The .FIT file must comply with the Garmin FIT SDK specification for workout files.
- **BR3**: Each workout step must have at least a duration or distance target.
- **BR4**: Interval workouts must include warmup and cooldown steps.
- **BR5**: The exported file name should follow the pattern: `TOTM_{session_type}_{date}.fit`.

## MVP Scope
### Included
- Export individual sessions as .FIT workout files
- Support for basic session types (easy run, intervals, long run, tempo)
- File download to local computer
- Compatible with Garmin .FIT workout format

### Excluded (Post-MVP)
- Direct wireless transfer to watch
- Batch export of multiple sessions
- Export to other formats (.ZWO, .ERG)
- Integration with Garmin Connect API for direct upload
- Custom workout step editor

### Temporary Assumptions
- Users manually transfer .FIT files via USB or companion app
- FIT SDK or compatible library is available for file generation
- Only Garmin-compatible format in MVP (most widely supported)

## Non-Functional Requirements
- **Expected usage frequency**: 2–5 times per week (once per upcoming session)
- **Number of affected users**: 1 (local single-user)
- **Expected load/volume**: Single file generation per export; files < 1KB typically

## Acceptance Criteria
- [ ] GIVEN a future session with workout details WHEN the user clicks Export THEN a valid .FIT file is downloaded
- [ ] GIVEN an interval session WHEN exported THEN the .FIT file contains correct warmup, work, recovery, and cooldown steps
- [ ] GIVEN a simple session WHEN exported THEN the .FIT file contains a single step with duration target
- [ ] GIVEN a completed session WHEN the user tries to export THEN the export option is disabled or hidden
- [ ] GIVEN an exported .FIT file WHEN loaded into Garmin Connect THEN it is recognized as a valid workout

## Success Metrics
- 100% of exported .FIT files are valid and importable into Garmin Connect
- Export process completes in under 2 seconds

## Open Questions
- Which .FIT SDK/library should be used for file generation?
- Should we support watch-specific customizations (e.g., Garmin vs. Suunto formats)?
- Should exported sessions be marked visually in the calendar?

## Dependencies
- **Product dependencies**: UC-003 (plan data), UC-004 (export trigger), UC-009 (export trigger)
- **Technical dependencies**: FIT SDK / .FIT file generation library, file download mechanism
- **Legal / Privacy constraints**: None specific (local file generation)
