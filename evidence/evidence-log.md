# Project 4 — Evidence Log

## Scope

Target:
http://127.0.0.1:3000

Application:
OWASP Juice Shop

Assessment date:
2026-10-01

All testing was restricted to the local intentionally vulnerable Juice Shop
instance.

---

# Evidence E-01 — Nmap Port Discovery

Command:

nmap -p 3000 127.0.0.1

Observed:

3000/tcp open

Purpose:

Confirmed that the local web application was reachable on TCP port 3000.

---

# Evidence E-02 — Application Endpoint Discovery

Source:

Chrome DevTools Network / Fetch-XHR

Observed endpoints included:

/rest/admin/application-version
/rest/admin/application-configuration
/api/Challenges/
/rest/languages
/api/Quantitys/
/rest/products/search?q=
/socket.io/
/assets/i18n/en.json

Purpose:

Established the application's HTTP/API attack surface.

---

# Evidence E-03 — Unauthenticated Administrative Configuration

Endpoint:

GET /rest/admin/application-configuration

Request:

No authentication credentials.

Observed:

HTTP/1.1 200 OK

The response returned a large JSON configuration object.

Validation:

curl.exe -i -H "Cookie:" http://127.0.0.1:3000/rest/admin/application-configuration

Result:

HTTP/1.1 200 OK

Finding:

F-01 — Unauthenticated Administrative Configuration Exposure

---

# Evidence E-04 — SQL Injection Baseline

Endpoint:

GET /rest/products/search?q=

Normal request:

GET /rest/products/search?q=apple

Observed:

HTTP/1.1 200 OK

Product records were returned.

Purpose:

Established normal application behavior before vulnerability validation.

---

# Evidence E-05 — SQL Injection Validation

Endpoint:

GET /rest/products/search?q=

Validation request:

GET /rest/products/search?q=%27%28

Observed:

HTTP/1.1 500 Internal Server Error

Database error:

SQLITE_ERROR: near "(" : syntax error

The finding was initially identified by OWASP ZAP and independently
reproduced using curl.

Finding:

F-02 — SQL Injection in Product Search

---

# Evidence E-06 — Missing Content Security Policy

Endpoint:

GET /

Validation command:

curl.exe -I http://127.0.0.1:3000/

Observed:

HTTP/1.1 200 OK

Present security headers included:

X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'

The following headers were not observed:

Content-Security-Policy
Content-Security-Policy-Report-Only

Finding:

F-03 — Missing Content Security Policy

---

# Evidence E-07 — CORS Observation

Endpoint:

/scripts.js

Validation request used an Origin header:

Origin: https://example-attacker.test

Observed:

Access-Control-Allow-Origin: *

No Access-Control-Allow-Credentials: true header was observed.

Assessment:

Wildcard CORS was observed on a publicly accessible resource, but no
authenticated/private-data exposure through CORS was demonstrated.

This is therefore retained as an observation rather than a confirmed
high-impact vulnerability.

---

# Evidence E-08 — Scanner Findings Rejected or Deprioritized

The following ZAP alerts were reviewed and not promoted to confirmed
vulnerabilities:

- Missing anti-clickjacking header on Socket.IO response
- Missing X-Content-Type-Options on Socket.IO response
- Session ID in Socket.IO URL
- Private IP disclosure
- Unix timestamp disclosure
- Suspicious comments
- Modern web application identification

Reason:

The available evidence did not demonstrate meaningful exploitable impact
for these observations within the assessment scope.

---

# Evidence Handling

Screenshots should be stored in:

evidence/screenshots/

Screenshots should use descriptive filenames corresponding to the evidence
IDs where possible.

