# Job Portal — Unauthenticated IDOR in Job Seeker Biodata

## Summary

An **Insecure Direct Object Reference (IDOR)** and missing authorization vulnerability was identified in the Job Portal application. The endpoint `/jobportal/Employer/ViewBiodata.php` allows unauthenticated users to access job seekers' personal information by modifying the `JobSeekId` parameter in the URL.

The application returns personal information associated with the supplied identifier without requiring the user to log in.

By changing the `JobSeekId` value, an attacker can retrieve biodata belonging to different job seekers, including names, addresses, email addresses, phone numbers, educational qualifications, dates of birth, and links to uploaded resumes.

- **Vulnerability class:** Broken Access Control / Unauthenticated IDOR
- **Primary CWE:** CWE-862 — Missing Authorization
- **Secondary CWE:** CWE-639 — Authorization Bypass Through User-Controlled Key
- **Related CWE:** CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor
- **OWASP:** A01:2021 — Broken Access Control
- **Affected endpoint:** `GET /jobportal/Employer/ViewBiodata.php?JobSeekId=<id>`
- **Authentication required:** No
- **Impact:** Unauthorized disclosure of job seekers' personal information
- **Test environment:** `http://192.168.56.101/jobportal/`
- **Version:** Not independently verified; tested local installation
- **CVE:** Not assigned

## Steps to reproduce

1. Open a new private browser window to ensure that no authenticated session is active.
2. Navigate directly to the vulnerable endpoint:

   `http://192.168.56.101/jobportal/Employer/ViewBiodata.php?JobSeekId=4`

3. Observe that the application displays the biodata associated with job seeker ID `4`.
4. Modify the `JobSeekId` parameter from `4` to `6`:

   `http://192.168.56.101/jobportal/Employer/ViewBiodata.php?JobSeekId=6`

5. Reload the page.
6. Observe that the application displays a different job seeker's personal information without requiring authentication.
7. Compare the responses to confirm that the displayed records correspond to different user identifiers.

## Proof-of-Concept Requests

### Request 1 — Job Seeker ID 4

```http
GET /jobportal/Employer/ViewBiodata.php?JobSeekId=4 HTTP/1.1
Host: 192.168.56.101
Connection: close
```

**Observed result:**

The application displays the biodata of job seeker ID `4`, including:

- Full name
- Residential address and city
- Email address
- Mobile phone number
- Educational qualifications
- Gender
- Date of birth
- Resume access link

### Request 2 — Job Seeker ID 6

```http
GET /jobportal/Employer/ViewBiodata.php?JobSeekId=6 HTTP/1.1
Host: 192.168.56.101
Connection: close
```

**Observed result:**

The application displays another job seeker's personal information, corresponding to `JobSeekId=6`.

The URL parameter is the only application-level identifier changed between the two requests.

The supplied screenshots demonstrate that different job seeker profiles are accessible through direct identifier manipulation.

## Expected Result

The application should reject requests from unauthenticated users attempting to access protected job seeker biodata.

Expected behavior:

- Anonymous requests should return `401 Unauthorized` or redirect to the login page.
- Authenticated users should only be permitted to access job seeker information for which they have appropriate authorization.
- Changing the `JobSeekId` parameter should not bypass access control.
- Sensitive personal information should not be exposed through unrestricted endpoints.

## Observed Result

During testing, the application returned two different job seeker profiles based on the supplied `JobSeekId` parameter.

| Parameter | Observed behavior |
|---|---|
| `JobSeekId=4` | Displays personal information belonging to job seeker ID 4 |
| `JobSeekId=6` | Displays personal information belonging to job seeker ID 6 |

Both pages were accessible through the same endpoint.

According to the test conditions, the requests were performed without an authenticated session.

The application did not require the user to authenticate before displaying the requested biodata.

The rendered pages also disclosed a PHP warning:

```text
Warning: Undefined array key "Status"
in C:\xampp\htdocs\jobportal\Employer\ViewBiodata.php
on line 192
```

This warning additionally exposes an internal application file path and source code line number. It is a separate information disclosure issue and is not required to exploit the authorization vulnerability.

## Root Cause

The vulnerability results from insufficient authorization enforcement when accessing job seeker records.

The application accepts a user-controlled `JobSeekId` parameter and uses it to select the requested biodata.

The observed behavior indicates that the endpoint does not enforce an effective authentication requirement before returning the selected personal information.

The root cause is consistent with missing server-side authorization checks.

The precise PHP implementation of `ViewBiodata.php` was not reviewed as part of this proof of concept.

## Impact

Successful exploitation allows unauthorized users to access personal information belonging to multiple job seekers.


### Screenshot 4.png — Job Seeker ID 4

**File:** `4.png`

The screenshot demonstrates access to a job seeker's biodata using:

`/jobportal/Employer/ViewBiodata.php?JobSeekId=4`

### Screenshot 6.png — Job Seeker ID 6

**File:** `6.png`

The screenshot demonstrates access to a different job seeker's biodata after changing the identifier to:

`/jobportal/Employer/ViewBiodata.php?JobSeekId=6`
