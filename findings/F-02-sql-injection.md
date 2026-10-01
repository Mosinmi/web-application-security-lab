# F-02 — SQL Injection in Product Search

## Finding ID
F-02

## Title
SQL Injection in Product Search Parameter

## Severity
High

## Status
Confirmed

## Target
http://127.0.0.1:3000

## Affected Endpoint
GET /rest/products/search?q=

## Vulnerable Parameter
q

## Description
The product search endpoint processes user-controlled input through the
q query parameter.

Controlled malformed input caused the application to return a SQLite
syntax error, demonstrating that the supplied input reaches the database
query processing layer without adequate parameterization.

The finding was initially identified by OWASP ZAP and subsequently
independently reproduced using curl.

## Baseline Request

Request:

GET /rest/products/search?q=apple

Result:

HTTP/1.1 200 OK

The application returned product records for the search term.

## Validation Request

Request:

GET /rest/products/search?q=%27%28

Where:

%27 = '
%28 = (

Result:

HTTP/1.1 500 Internal Server Error

The response disclosed:

SQLITE_ERROR: near "(" : syntax error

The application also returned an Express error page identifying the
OWASP Juice Shop application.

## Evidence

Observed response:

HTTP/1.1 500 Internal Server Error

Observed database error:

SQLITE_ERROR: near "(" : syntax error

OWASP ZAP identified:

Parameter: q
Attack: '(
Evidence: HTTP/1.1 500 Internal Server Error
Risk: High
Initial confidence: Low

The finding was manually validated independently using curl.

## Security Impact

SQL injection can allow attacker-controlled input to influence database
queries.

The demonstrated impact in this assessment is:

- Database query error generation
- SQL error disclosure
- Exposure of database implementation details

This assessment did NOT demonstrate:

- Database extraction
- Authentication bypass
- Data modification
- Data deletion
- Remote code execution

These impacts are therefore not claimed as demonstrated results.

## Recommendation

1. Replace dynamically constructed SQL queries with parameterized or
   prepared statements.
2. Treat all client-supplied search parameters as untrusted input.
3. Implement appropriate server-side input validation.
4. Do not return raw database errors to clients.
5. Configure production error handling to return generic error responses.
6. Review other search and filtering endpoints for similar query
   construction patterns.
7. Add automated security tests for SQL injection regression.

## Retest

Pending remediation.

## Scope

Testing was restricted to the intentionally vulnerable local OWASP Juice
Shop instance at 127.0.0.1:3000.

No third-party systems were tested.

