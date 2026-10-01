# F-03 — Missing Content Security Policy

## Finding ID
F-03

## Title
Content Security Policy Header Not Set

## Severity
Medium

## Status
Confirmed

## Target
http://127.0.0.1:3000

## Affected Resource
GET /

## Description
The main Juice Shop application response does not define a Content
Security Policy through either the Content-Security-Policy or
Content-Security-Policy-Report-Only HTTP response headers.

The finding was identified by OWASP ZAP and independently validated
using curl.

## Evidence

Validation command:

curl.exe -I http://127.0.0.1:3000/

Response:

HTTP/1.1 200 OK

Security headers observed included:

X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'

However, the response did not contain:

Content-Security-Policy

or:

Content-Security-Policy-Report-Only

## Security Impact

Content Security Policy provides an additional browser-enforced security
layer that can restrict the sources from which scripts, styles, frames,
images and other resources may be loaded.

Without an appropriate CSP, the application lacks this additional layer
of defense against certain content-injection and cross-site scripting
scenarios.

The absence of CSP alone does NOT demonstrate that the application is
vulnerable to cross-site scripting.

No XSS vulnerability is claimed by this finding.

## Recommendation

1. Define a restrictive Content Security Policy appropriate for the
   application's required resources.
2. Begin with Content-Security-Policy-Report-Only during policy development
   where appropriate.
3. Identify and explicitly allow only required script, style, image,
   font, frame and connection sources.
4. Avoid unsafe-inline and unsafe-eval unless there is a documented
   requirement.
5. Test the policy against all application functionality before enforcing
   it.
6. Monitor CSP violations after deployment and refine the policy as
   required.

## Retest

Pending remediation.

## Scope

Testing was restricted to the intentionally vulnerable local OWASP Juice
Shop instance at 127.0.0.1:3000.

No third-party systems were tested.

