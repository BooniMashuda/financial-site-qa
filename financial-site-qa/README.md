# Financial Site QA

Test requirements and test assets for the financial website (Phase 1: user registration, login, profile and dashboard).

## Contents
- `docs/requirements/testing-requirements-auth-profile-dashboard.md` - master testing requirements (TR-AUTH-001)
- `tests/manual/` - manual test cases, one folder per module
- `tests/automation/` - API, E2E, performance, security and accessibility tests
- `tests/data/` - test data (no secrets or real personal data)
- `reports/` - traceability matrix and run reports

## Conventions
- Test case ID: `TC-<MODULE>-<REQ number>-<sequence>`, e.g. `TC-LOG-004-02`
- Every test case references at least one requirement ID
- Changes go through pull requests
- Never commit secrets, real personal data or production data
