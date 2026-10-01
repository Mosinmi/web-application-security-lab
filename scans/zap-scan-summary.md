# OWASP ZAP Vulnerability Assessment

## Target

http://127.0.0.1:3000

## Tool

OWASP ZAP 2.17.0

## Assessment Type

Passive reconnaissance followed by active vulnerability scanning.

## Confirmed Findings

### F-01 — Unauthenticated Administrative Configuration Exposure

Endpoint:

/rest/admin/application-configuration

The endpoint returned application configuration data without requiring
authentication.

### F-02 — SQL Injection

Endpoint:

/rest/products/search?q=

Parameter:

q

Controlled malformed input caused:

HTTP 500 Internal Server Error

SQLite error:

SQLITE_ERROR: near "(" : syntax error

The finding was identified by ZAP and independently reproduced using curl.

### F-03 — Missing Content Security Policy

Endpoint:

/

The main application response did not contain either:

Content-Security-Policy

or:

Content-Security-Policy-Report-Only

The finding was identified by ZAP and independently validated using curl.

## Scanner Observations Not Treated as Confirmed Vulnerabilities

- Wildcard CORS on public resources
- Missing anti-clickjacking header on Socket.IO response
- Missing X-Content-Type-Options on Socket.IO response
- Session ID in Socket.IO URL
- Private IP disclosure
- Unix timestamp disclosure
- Suspicious comments
- User-Agent Fuzzer observation
- Modern Web Application identification

These observations were not promoted to confirmed vulnerabilities without
sufficient evidence of meaningful security impact.
