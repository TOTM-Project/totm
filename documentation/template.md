# UC-XXX — [Name]

## Metadata
| Key | Value |
|-----|-------|
| ID | UC-XX |
| Version | 0.1 |
| Status | Draft / Validated / Needs Review |
| Priority | High / Medium / Low |
| Primary Persona | |
| Business Goal | |

## Summary
<!-- 2-4 sentences: what the user wants to accomplish and why this use case exists in the MVP. -->

## Actors
- **Primary actor**:
- **Secondary actors**:
- **External systems**:

## Trigger
<!-- The event that initiates this use case. -->

## Preconditions
- [ ] 
- [ ] 
- [ ] 

## Expected Outcomes
### On Success
- 
- 

### On Failure
- 
- 

## Input Data Schema
| Field | Type | Required | Validation Rules | Example |
|-------|------|----------|-----------------|---------|
|  |  | Yes/No |  |  |
|  |  | Yes/No |  |  |

## Happy Path (Step-by-Step)
```
1. USER: ...
2. SYSTEM: ...
3. USER: ...
4. SYSTEM: ...
5. SYSTEM: confirms ...
```

## Alternative Flows
### A1. Missing Data
```
1. SYSTEM detects missing required field(s).
2. SYSTEM returns error with field list and expected format.
3. USER resubmits with corrected data.
```

### A2. Inconsistent Data
```
1. SYSTEM detects data inconsistency (describe rule).
2. SYSTEM returns error with explanation.
3. USER corrects and resubmits.
```

## Error Flows
### E1. Internal System Error
```
1. SYSTEM encounters an unexpected failure.
2. SYSTEM logs error details.
3. SYSTEM returns generic error message to USER.
4. No data is persisted (transaction rolled back).
```

### E2. External Integration Unavailable
```
1. SYSTEM fails to reach external service after N retries.
2. SYSTEM returns degraded response or queues for retry.
3. USER is notified of partial/delayed processing.
```

## Business Rules
<!-- Numbered list of rules the implementation MUST enforce. -->
- **BR1**:
- **BR2**:
- **BR3**:

## MVP Scope
### Included
- 

### Excluded (Post-MVP)
- 

### Temporary Assumptions
- 

## Non-Functional Requirements
- **Expected usage frequency**:
- **Number of affected users**:
- **Expected load/volume**:

## Acceptance Criteria
<!-- These must be verifiable. Write them as testable assertions. -->
- [ ] GIVEN ... WHEN ... THEN ...
- [ ] GIVEN ... WHEN ... THEN ...
- [ ] GIVEN ... WHEN ... THEN ...

## Success Metrics
- 
- 
- 

## Open Questions
- 
- 

## Dependencies
- **Product dependencies** (other use cases / features):
- **Technical dependencies** (APIs, services, libraries):
- **Legal / Privacy constraints**: