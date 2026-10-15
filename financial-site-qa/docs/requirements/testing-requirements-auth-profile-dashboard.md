# Testing Requirements: User Registration, Login, Profile and Dashboard

| Field | Value |
|---|---|
| Document ID | TR-AUTH-001 |
| Version | 0.1 (draft) |
| Status | Draft for review |
| Owner | QA Lead (TBC) |
| Related requirement | Financial Website, Part 1, Phase 1 scope: "User registration, login and a simple profile or dashboard" |
| Last updated | 2026-10-15 |

> **How to use this document:** Every requirement has a unique ID (e.g. `REG-003`). Each test case written in the repository must reference at least one requirement ID, so coverage can be traced. Items marked **[TBC]** are assumptions to be confirmed with the product owner before test cases are finalised.

---

## 1. Purpose and Scope

### 1.1 Purpose
Define what must be tested for the user account features of the financial website, so that test cases, automation scripts and test reports can be derived consistently and kept under version control.

### 1.2 In Scope
- User registration (sign-up)
- Email verification
- Login and logout
- Session management
- Forgot / reset password and change password
- Profile view and edit
- Dashboard (saved calculations, bookmarks, alerts, recent activity)
- Account deletion / data export (if included in Phase 1) **[TBC]**
- Non-functional testing of the above: security, performance, accessibility, compatibility, usability

### 1.3 Out of Scope
- Public pages, blog, calculators and market data widgets (separate test plans)
- Admin panel and content management
- Payments, KYC, bank or brokerage linking (not in Phase 1)
- Third-party identity provider internals (only the integration is tested)
- Native mobile apps

---

## 2. Assumptions and Configurable Rules

These are default assumptions used to write the acceptance criteria. Update this table first if any change; dependent requirements are then revised.

| Ref | Assumption | Default Value | Status |
|---|---|---|---|
| A-01 | Login identifier | Email address | [TBC] |
| A-02 | Email verification required before first login | Yes | [TBC] |
| A-03 | Password minimum length | 12 characters | [TBC] |
| A-04 | Password complexity | At least 1 uppercase, 1 lowercase, 1 digit, 1 special character | [TBC] |
| A-05 | Maximum password length | 128 characters | [TBC] |
| A-06 | Failed login lockout | 5 consecutive failures, 15 minute lock | [TBC] |
| A-07 | Verification link validity | 24 hours | [TBC] |
| A-08 | Password reset link validity | 60 minutes, single use | [TBC] |
| A-09 | Session idle timeout | 30 minutes | [TBC] |
| A-10 | "Remember me" duration | 30 days | [TBC] |
| A-11 | Social login (Google, Apple) | Not in Phase 1 | [TBC] |
| A-12 | Multi-factor authentication | Optional, post Phase 1 | [TBC] |
| A-13 | CAPTCHA on registration and login | Yes, after repeated failures | [TBC] |
| A-14 | Languages | English only | [TBC] |
| A-15 | Terms and Privacy consent required at sign-up | Yes | [TBC] |

---

## 3. Test Environments, Data and Tools

### 3.1 Environments

| Environment | Purpose | Data |
|---|---|---|
| Local / Dev | Developer and unit testing | Synthetic |
| QA / Test | Functional, regression, automation | Synthetic, resettable |
| Staging | UAT, performance, pre-release security checks | Synthetic, production-like |
| Production | Smoke tests only, read-only checks | Dedicated test accounts only |

### 3.2 Test Data Rules
- Never use real personal data or real financial data in any environment.
- Use a controlled email domain or mail-catcher (e.g. a test inbox service) so verification and reset emails can be read.
- Maintain a seed set of accounts (see section 9).
- Each automated test creates or resets its own data and must not depend on another test's state.

### 3.3 Browsers and Devices (initial matrix) **[TBC]**

| Category | Coverage |
|---|---|
| Desktop browsers | Latest 2 versions of Chrome, Firefox, Edge, Safari |
| Mobile browsers | Chrome on Android, Safari on iOS (latest 2 OS versions) |
| Viewports | 360 px, 768 px, 1280 px, 1920 px wide |

### 3.4 Suggested Tooling (not mandatory)
- UI automation: Playwright or Cypress
- API testing: Postman/Newman or REST-assured
- Accessibility: axe, Lighthouse, manual screen-reader checks
- Performance: k6 or JMeter
- Security: OWASP ZAP, manual review against OWASP ASVS
- Test management: Markdown/CSV in this repository, optionally synced to a tracker

---

## 4. Functional Requirements and Acceptance Criteria

**Priority:** P1 = critical (blocks release), P2 = high, P3 = medium, P4 = low.
**Test types:** F = functional, N = negative, B = boundary, S = security, U = usability, A = accessibility, API = API level, E2E = end to end.

### 4.1 Registration (REG)

| ID | Requirement | Acceptance Criteria | Types | Priority |
|---|---|---|---|---|
| REG-001 | A visitor can open the registration page from the header and from the login page. | Link is visible and navigates to the registration form. | F, U | P2 |
| REG-002 | Form collects: full name, email, password, confirm password, consent checkbox. | All fields present, labelled, and mandatory fields marked. | F, A | P1 |
| REG-003 | Successful registration creates an account in "unverified" state. | With valid data, account is created, success message shown, no session granted if A-02 applies. | F, E2E | P1 |
| REG-004 | Email must be valid and unique. | Invalid format rejected with a clear message. Duplicate email does not create a second account. | F, N | P1 |
| REG-005 | Duplicate email response must not reveal account existence. | Message is generic (e.g. "If this email can be registered, you will receive an email"); response time and content equivalent for new and existing emails. | S | P1 |
| REG-006 | Password policy is enforced client-side and server-side. | Passwords violating A-03 to A-05 are rejected by the server even if client validation is bypassed. | F, N, B, S, API | P1 |
| REG-007 | Confirm password must match. | Mismatch blocks submission with an inline error. | F, N | P2 |
| REG-008 | Password strength indicator guides the user. | Indicator updates as the user types and is announced to assistive technology. | F, U, A | P3 |
| REG-009 | Consent to Terms and Privacy Policy is mandatory. | Submission blocked when unchecked. Consent timestamp and policy version are stored. | F, N | P1 |
| REG-010 | Name and email fields are sanitised. | Script tags, HTML and SQL fragments are rejected or safely encoded; nothing executes or breaks the page. | S, N | P1 |
| REG-011 | Leading and trailing whitespace is trimmed from the email; the email is case-insensitive. | `User@Test.com` and `user@test.com` are treated as the same account. | F, B | P2 |
| REG-012 | Registration form protects against automated abuse. | Rate limiting and/or CAPTCHA triggers after repeated submissions from one source (A-13). | S | P2 |
| REG-013 | Double submission is handled. | Double-click or rapid resubmit creates only one account and one email. | F, N | P2 |
| REG-014 | Welcome / verification email is sent. | Email arrives within 1 minute (staging), has correct sender, subject, branding and a working link. | F, E2E | P1 |
| REG-015 | Registration errors preserve entered data (except passwords). | After a validation error, name and email remain populated. | U | P3 |

### 4.2 Email Verification (VER)

| ID | Requirement | Acceptance Criteria | Types | Priority |
|---|---|---|---|---|
| VER-001 | Verification link activates the account. | Valid link sets state to "verified" and shows confirmation. | F, E2E | P1 |
| VER-002 | Verification link expires after A-07. | Expired link shows a clear message and offers to resend. | F, B | P1 |
| VER-003 | Verification link is single-use. | Reusing a link does not error unsafely and shows "already verified". | F, N | P2 |
| VER-004 | Tampered or invalid token is rejected. | No account state changes; generic error shown. | S, N | P1 |
| VER-005 | User can request a new verification email. | New email sent; the previous link is invalidated; resend is rate limited. | F, S | P2 |
| VER-006 | Unverified users cannot access protected pages. | Login attempt shows "verify your email" with a resend option. | F, S | P1 |

### 4.3 Login (LOG)

| ID | Requirement | Acceptance Criteria | Types | Priority |
|---|---|---|---|---|
| LOG-001 | Verified user logs in with valid credentials. | User lands on the dashboard; session is established. | F, E2E | P1 |
| LOG-002 | Invalid credentials are rejected with a generic message. | The message does not reveal whether the email or the password was wrong. | F, N, S | P1 |
| LOG-003 | Empty fields are validated. | Inline errors appear; no request is sent for empty email. | F, N | P2 |
| LOG-004 | Account lockout after repeated failures (A-06). | The Nth failure locks the account for the configured period; a correct password during lock is still rejected; the lock expires correctly. | F, B, S | P1 |
| LOG-005 | Lockout notification. | User is informed (on-screen and/or by email) without exposing sensitive details. | F, S | P3 |
| LOG-006 | Show/hide password toggle works. | Toggle reveals and masks the password; the default state is masked. | F, U, A | P3 |
| LOG-007 | "Remember me" persists the session for A-10. | Session survives browser restart when ticked and does not when unticked. | F | P2 |
| LOG-008 | Redirect after login. | User returns to the originally requested protected page, or the dashboard by default. Open-redirect attempts are blocked. | F, S | P2 |
| LOG-009 | Already logged-in users visiting login/registration are redirected. | Redirected to the dashboard. | F | P3 |
| LOG-010 | Login is protected against brute force and credential stuffing. | Rate limiting per IP and per account; CAPTCHA after repeated failures (A-13). | S | P1 |
| LOG-011 | Credentials are never exposed in URLs, logs or error messages. | Inspect URL, network traffic, server and browser logs. | S | P1 |
| LOG-012 | Login page is served over HTTPS only. | HTTP redirects to HTTPS; cookies flagged Secure. | S | P1 |
| LOG-013 | Case sensitivity rules. | Email is case-insensitive; password is case-sensitive. | F, B | P2 |
| LOG-014 | Browser autofill and password managers work. | Fields use correct `autocomplete` attributes. | U | P3 |
| LOG-015 | Concurrent sessions behaviour **[TBC]**. | Behaviour matches the decision (allow multiple, or limit and notify). | F | P3 |

### 4.4 Logout and Session Management (SES)

| ID | Requirement | Acceptance Criteria | Types | Priority |
|---|---|---|---|---|
| SES-001 | User can log out from any page. | Session is destroyed server-side; the user lands on a public page. | F, S | P1 |
| SES-002 | Back button after logout does not reveal protected content. | Protected pages are not served from cache; the user is asked to log in. | S, F | P1 |
| SES-003 | Idle timeout (A-09). | After inactivity the next action redirects to login with a clear message; unsaved-work warning shown where relevant. | F, B | P1 |
| SES-004 | Session cookies are secure. | `HttpOnly`, `Secure`, `SameSite` attributes set; session ID changes after login (no fixation). | S | P1 |
| SES-005 | Session invalidated after password change or reset. | Other active sessions are terminated. | S | P1 |
| SES-006 | Logout in one tab is reflected in other tabs. | Other tabs require login on the next action. | F | P3 |
| SES-007 | Direct access to protected URLs without a session. | Redirect to login; no data in the response body. | S, F | P1 |
| SES-008 | CSRF protection on state-changing requests. | Requests without a valid token are rejected. | S | P1 |

### 4.5 Forgot, Reset and Change Password (PWD)

| ID | Requirement | Acceptance Criteria | Types | Priority |
|---|---|---|---|---|
| PWD-001 | "Forgot password" request. | Entering any email shows the same generic confirmation; an email is sent only if the account exists. | F, S | P1 |
| PWD-002 | Reset link validity and single use (A-08). | Expired or used links are rejected. | F, B, S | P1 |
| PWD-003 | New password follows the policy. | Same rules as REG-006; the new password cannot equal the current one **[TBC]**. | F, N | P1 |
| PWD-004 | Reset confirmation. | User receives an email notification that the password was changed. | F | P2 |
| PWD-005 | Reset requests are rate limited. | Excessive requests are throttled. | S | P2 |
| PWD-006 | Tokens are unguessable and stored hashed. | Tokens have sufficient entropy; DB review shows no plaintext tokens. | S | P1 |
| PWD-007 | Change password from the profile (logged in). | Requires the current password; fails with a wrong current password; success message and email notification. | F, N, S | P1 |
| PWD-008 | Locked account can recover via reset **[TBC]**. | Behaviour matches the decision. | F | P3 |
| PWD-009 | Passwords are stored using a strong adaptive hash. | Verified through code and config review (bcrypt, scrypt or Argon2); never reversible. | S | P1 |

### 4.6 Profile (PRO)

| ID | Requirement | Acceptance Criteria | Types | Priority |
|---|---|---|---|---|
| PRO-001 | Logged-in user can view their profile. | Displays name, email, member-since date, and preferences. | F | P1 |
| PRO-002 | User can edit name and preferences. | Changes persist and display immediately after saving. | F, E2E | P1 |
| PRO-003 | Profile field validation. | Name length (1 to 100) **[TBC]**, allowed characters including Unicode names, and rejection of markup. | F, N, B, S | P2 |
| PRO-004 | Changing the email requires re-verification. | New email is pending until verified; the old email is notified; login continues with the old one until confirmed. | F, S | P1 |
| PRO-005 | Preferences: currency, country, newsletter opt-in, alert opt-in. | Selections are saved and applied across the site. | F | P2 |
| PRO-006 | Users can only access their own profile. | Changing the user ID in a URL or API call returns 403/404 (no IDOR). | S, API | P1 |
| PRO-007 | Unsaved changes warning. | Navigating away with edits prompts the user. | U | P4 |
| PRO-008 | Concurrent edits (two tabs/devices). | Last-write or conflict behaviour is as designed; no data corruption. | F | P3 |
| PRO-009 | Optional avatar upload **[TBC]**. | Only allowed types and sizes accepted; files are scanned; EXIF stripped. | F, S | P3 |
| PRO-010 | Newsletter unsubscribe from the profile and from email. | Both paths take effect immediately and are consistent. | F | P2 |
| PRO-011 | Account deletion **[TBC]**. | Requires re-authentication and confirmation; data removed or anonymised per policy; user logged out; confirmation email; same email can re-register or not, per policy. | F, S | P1 |
| PRO-012 | Personal data export **[TBC]**. | User can request a copy of their data in a readable format. | F | P3 |

### 4.7 Dashboard (DSH)

| ID | Requirement | Acceptance Criteria | Types | Priority |
|---|---|---|---|---|
| DSH-001 | Dashboard is available only to authenticated users. | Unauthenticated access redirects to login. | F, S | P1 |
| DSH-002 | Shows a personalised greeting and navigation to profile, saved items and settings. | Displays the correct user name; links work. | F | P2 |
| DSH-003 | Saved calculations list. | Shows the user's saved calculations with date and type; user can open, rename and delete. | F, E2E | P1 |
| DSH-004 | Bookmarked articles list. | Bookmarks added from articles appear; removal updates immediately. | F, E2E | P2 |
| DSH-005 | Alerts and notifications (rates or article alerts) **[TBC]**. | User can create, edit, pause and delete alerts; the correct data is displayed. | F | P3 |
| DSH-006 | Empty state. | A new user sees helpful guidance, not blank space or errors. | F, U | P2 |
| DSH-007 | Data isolation. | A user never sees another user's data, including after logout/login on a shared device. | S | P1 |
| DSH-008 | Data accuracy. | Saved calculation inputs and results match what was saved to the stored precision, with correct currency and number formats. | F | P1 |
| DSH-009 | Pagination, sorting and filtering for lists above 20 items. | Works correctly with 0, 1, 20, 21 and 500 items. | F, B | P3 |
| DSH-010 | Error handling when backend services fail. | Friendly message and retry; no stack traces; the rest of the dashboard remains usable. | F, N | P2 |
| DSH-011 | Responsive layout. | Usable at all viewports in 3.3 without horizontal scrolling. | F, U | P2 |
| DSH-012 | Disclaimer visibility. | A financial information disclaimer is visible where calculations or rates appear. | F | P2 |

---

## 5. Field Validation Matrix

Use this table to generate boundary and negative tests (equivalence partitioning and boundary value analysis).

| Field | Required | Min | Max | Valid Examples | Invalid / Edge Examples |
|---|---|---|---|---|---|
| Full name | Yes | 1 | 100 [TBC] | `Anna Smith`, `José Müller`, `李 雷`, `O'Brien-Lee` | empty, 101 chars, only spaces, `<script>`, emoji-only |
| Email | Yes | 6 | 254 | `a.b+tag@example.co` | no `@`, `a@b`, spaces, 255 chars, double dots, unicode domain edge cases |
| Password | Yes | 12 | 128 | meets A-03/A-04 | 11 chars, 129 chars, no digit, no special char, common password (`Password123!`), contains email or name |
| Confirm password | Yes | = password | = password | identical | one char different, empty |
| Consent checkbox | Yes | checked | n/a | checked | unchecked |
| Current password (change) | Yes | n/a | n/a | correct value | wrong, empty |
| Currency | No | n/a | n/a | from list | value not in list (API tampering) |
| Country | No | n/a | n/a | from list | value not in list |

---

## 6. Non-Functional Test Requirements

### 6.1 Security (SEC)

| ID | Requirement | Verification Method |
|---|---|---|
| SEC-001 | All traffic uses TLS 1.2 or higher; HSTS enabled. | TLS scan, header inspection |
| SEC-002 | Protection against OWASP Top 10 risks in auth flows (injection, broken authentication, broken access control, XSS, CSRF, security misconfiguration). | Manual and automated scanning |
| SEC-003 | Security headers present (CSP, X-Content-Type-Options, X-Frame-Options or frame-ancestors, Referrer-Policy). | Header scan |
| SEC-004 | No user enumeration via registration, login, reset, or response timing. | Manual and scripted comparison |
| SEC-005 | Authorization checks on every API endpoint. | API tests with other users' IDs and tokens |
| SEC-006 | Sensitive data is not logged, cached or exposed in URLs. | Log review, network inspection |
| SEC-007 | Error messages do not leak stack traces, SQL or server details. | Fault injection |
| SEC-008 | Dependencies are free of known critical vulnerabilities. | Dependency scan in CI |
| SEC-009 | Penetration test of authentication flows before go-live. | External or internal pen test report |
| SEC-010 | Audit log records registration, login success/failure, password changes, email changes and deletions, with timestamp and source, and excludes secrets. | Log inspection |

### 6.2 Performance (PERF) **[targets TBC]**

| ID | Requirement | Target |
|---|---|---|
| PERF-001 | Login page load on 4G mobile | Under 3 seconds |
| PERF-002 | Login API response, 95th percentile at normal load | Under 500 ms |
| PERF-003 | Registration API response, 95th percentile | Under 800 ms (excluding email delivery) |
| PERF-004 | Dashboard load, 95th percentile | Under 2 seconds |
| PERF-005 | Load test at expected peak (e.g. 100 concurrent users) | Error rate under 1% |
| PERF-006 | Stress and spike tests | Graceful degradation, no data loss, recovery without manual intervention |
| PERF-007 | Password hashing cost does not enable denial of service | Rate limiting holds under attack simulation |

### 6.3 Accessibility (ACC) - target WCAG 2.2 AA

| ID | Requirement |
|---|---|
| ACC-001 | All forms are fully keyboard operable with a logical tab order and visible focus. |
| ACC-002 | Every input has a programmatically associated label; errors are linked to fields and announced to screen readers. |
| ACC-003 | Colour contrast meets AA; error states do not rely on colour alone. |
| ACC-004 | CAPTCHA has an accessible alternative. |
| ACC-005 | Session timeout warning is perceivable and allows extension. |
| ACC-006 | Verified with automated tools plus manual testing using at least one screen reader. |

### 6.4 Compatibility (COMP)
- Core flows pass on the browser and device matrix in 3.3.
- Works with browser password managers and autofill.
- Works with cookies disabled (graceful message) and with common privacy extensions **[TBC]**.

### 6.5 Usability (USE)
- A new user can complete registration and verification in under 3 minutes without help (usability session, 5 participants minimum).
- Error messages are specific enough to fix the problem, except where security requires generic wording.
- Consistent wording, branding and button placement across flows.

### 6.6 Email and Notification (EML)
- Emails render correctly in major clients (Gmail, Outlook, Apple Mail) and on mobile.
- Correct sender, SPF, DKIM and DMARC alignment; links use HTTPS and the correct environment domain.
- Plain-text fallback exists; unsubscribe is present on marketing emails only.
- Emails never contain passwords.

### 6.7 Compliance and Privacy (CMP) **[region TBC]**
- Consent captured and stored with timestamp and policy version.
- Privacy Policy and Terms are linked on registration.
- Data subject rights (access, correction, deletion) are testable where required by applicable law (e.g. GDPR, local data protection law).
- Cookie consent behaves correctly before and after login.
- Retention rules for unverified accounts (e.g. purge after 7 days) **[TBC]**.

### 6.8 Localization (LOC)
- English only in Phase 1, but the UI tolerates long strings and Unicode names, and number, date and currency formats come from user preferences.

### 6.9 Reliability and Recovery (REL)
- Behaviour when the database, email provider, or CAPTCHA provider is down: user-friendly error, no partial accounts, retry safe.
- Idempotency of verification and reset operations.
- Backup and restore preserves accounts and credentials integrity (verified in staging).

---

## 7. Test Approach

| Level | Focus | Automation Target |
|---|---|---|
| Unit | Validation rules, token generation, hashing, policy logic | 80%+ coverage of auth modules **[TBC]** |
| API / Integration | Endpoints, auth, authorization, rate limiting, database effects, email triggers | All P1 and P2 requirements |
| UI / E2E | Registration to verification to login to dashboard; reset flow; profile edits | All P1 flows, headless in CI |
| Security | Automated scans plus manual exploratory testing | Scan on every release candidate |
| Performance | Load and stress on auth endpoints | Before each major release |
| Accessibility | Automated checks in CI plus manual audit | Automated on every build |
| Exploratory | Session-based charters per module | Each sprint |
| UAT | Business acceptance on staging | Before go-live |

### 7.1 Regression Pack
- **Smoke (under 5 minutes):** register, verify, login, view dashboard, logout. Runs on every deployment, including production (with dedicated test accounts).
- **Core regression:** all P1 and P2 cases. Runs nightly and before release.
- **Full regression:** all cases. Runs before each release.

---

## 8. Entry and Exit Criteria

### 8.1 Entry Criteria
- Requirements and assumptions in section 2 are confirmed or the open items are logged.
- Build deployed to the QA environment, and the smoke test passes.
- Test data, mail-catcher and accounts are ready.

### 8.2 Exit Criteria (release readiness)
- 100% of P1 test cases executed and passed.
- At least 95% of P2 test cases passed, with the remainder having accepted workarounds.
- No open Critical or High defects; Medium defects have a documented decision.
- Security scan and penetration test findings of High severity or above are resolved.
- Accessibility audit shows no blocking AA failures in auth flows.
- Performance targets in section 6.2 are met.
- Traceability matrix shows every requirement mapped to at least one executed test.

### 8.3 Defect Severity

| Severity | Definition | Example |
|---|---|---|
| Critical | Security breach, data loss, or the feature is unusable | User sees another user's dashboard |
| High | Major flow broken, no workaround | Cannot log in after verification |
| Medium | Flow works with a workaround or has a significant defect | Reset email delayed beyond SLA |
| Low | Cosmetic or minor | Misaligned error text |

---

## 9. Seed Test Data

| Account Key | State | Purpose |
|---|---|---|
| `user_new` | Does not exist | Registration happy path |
| `user_unverified` | Registered, not verified | Verification and blocked login |
| `user_verified` | Verified, empty dashboard | Login, empty state |
| `user_populated` | Verified, 25 saved calculations, 30 bookmarks | Pagination, sorting, data accuracy |
| `user_locked` | Locked after failures | Lockout tests |
| `user_expired_token` | Pending verification with an expired token | Expiry tests |
| `user_other` | Verified, own data | Data isolation and IDOR tests |

Credentials for these accounts are stored in the team's secret manager, never in this repository.

**Reusable test strings**
- XSS: `<script>alert(1)</script>`, `"><img src=x onerror=alert(1)>`
- SQL injection: `' OR '1'='1`, `admin'--`
- Unicode: `Zoë Œuvre`, `Мария`, `محمد`, `田中太郎`
- Whitespace and length: single space, 101-character name, 255-character email

---

## 10. Templates

### 10.1 Test Case Template

```markdown
---
id: TC-REG-003-01
title: Register with valid details creates an unverified account
requirement_ids: [REG-003, REG-014]
module: Registration
type: Functional
priority: P1
automation: Automated | Manual | Planned
preconditions: Email user_new@example.test does not exist
---

## Steps
1. Open /register.
2. Enter a valid name, email and password; confirm password; tick consent.
3. Submit.

## Test Data
Name: Anna Smith; Email: user_new@example.test; Password: <valid per policy>

## Expected Result
- Success message displayed.
- Account created in "unverified" state.
- Verification email received within 1 minute.
- No session is created.

## Postconditions
Delete the test account.
```

### 10.2 Traceability Matrix (CSV example)

```csv
requirement_id,test_case_ids,coverage_status,last_result,last_run_date
REG-003,"TC-REG-003-01;TC-REG-003-02",Covered,Pass,2026-10-15
REG-004,"TC-REG-004-01;TC-REG-004-02;TC-REG-004-03",Covered,Fail,2026-10-15
```

### 10.3 Defect Report Template

```markdown
**Title:**
**Environment / Build:**
**Linked requirement / test case:**
**Severity / Priority:**
**Steps to reproduce:**
**Expected result:**
**Actual result:**
**Attachments (screenshots, HAR, logs):**
```

---

## 11. Repository Structure and Conventions

```text
/docs
  /requirements
    testing-requirements-auth-profile-dashboard.md   <- this file
/tests
  /manual
    /registration
    /verification
    /login
    /session
    /password
    /profile
    /dashboard
  /automation
    /api
    /e2e
    /performance
    /security
    /accessibility
  /data
    seed-accounts.md          <- keys only, no secrets
    test-strings.json
/reports
  traceability-matrix.csv
  /runs
/.github (or CI config)
  workflows/
```

**Conventions**
- **Test case ID:** `TC-<MODULE>-<REQ number>-<sequence>`, e.g. `TC-LOG-004-02`.
- **Branching:** change requirements and tests via pull request; at least one QA and one product/dev reviewer.
- **Commit messages:** reference requirement IDs, e.g. `test(login): add lockout boundary cases for LOG-004`.
- **Versioning:** update the version and "last updated" fields in this file on every change; use Git tags for release baselines (e.g. `tests-v1.0`).
- **Changed requirement:** mark affected test cases for review (label `needs-review`) in the same pull request.
- **No secrets, real personal data or production data** may be committed.

---

## 12. Open Items to Resolve Before Test Case Writing

1. Confirm all **[TBC]** assumptions in section 2.
2. Confirm the target region and applicable privacy and consent regulations.
3. Decide on social login and MFA timing.
4. Decide on account deletion and data export for Phase 1.
5. Confirm performance targets and expected concurrent users.
6. Confirm the browser and device matrix.
7. Confirm the email provider and the test inbox approach.
8. Confirm the automation tool stack and CI platform.

---

## 13. Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-10-15 | | Initial draft |
