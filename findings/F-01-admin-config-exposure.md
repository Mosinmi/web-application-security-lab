# F-01 — Unauthenticated Administrative Configuration Exposure

## Finding ID
F-01

## Title
Unauthenticated Access to Administrative Application Configuration

## Severity
High

## Status
Confirmed

## Target
http://127.0.0.1:3000

## Affected Endpoint
GET /rest/admin/application-configuration

## Description
The administrative application configuration endpoint returned a large
configuration object without requiring authentication.

The endpoint was accessed directly from the local Juice Shop instance
without supplying authentication credentials or a session cookie.

## Evidence

Request:

GET /rest/admin/application-configuration

Response:

HTTP/1.1 200 OK

The response returned an application configuration object containing
server, application, challenge, social-media, OAuth and other configuration
information.

The endpoint also returned HTTP 200 when accessed without a Cookie header.

Validation request:

curl.exe -i -H "Cookie:" http://127.0.0.1:3000/rest/admin/application-configuration

Result:

HTTP/1.1 200 OK

## Security Impact

Administrative configuration information should generally not be exposed
to unauthenticated users when it contains information intended for
administrative functionality.

The observed response disclosed internal application configuration and
implementation details. Depending on the information exposed and how it
is consumed, such disclosure can assist further reconnaissance and
targeted attacks.

This assessment does not claim that the disclosed configuration alone
provides administrative privileges.

## Proof of Concept

The issue can be reproduced with:

curl.exe -i http://127.0.0.1:3000/rest/admin/application-configuration

No authentication credentials are required.

## Recommendation

1. Require authentication for administrative configuration endpoints.
2. Enforce authorization checks server-side for administrative resources.
3. Return only configuration values required by unauthenticated clients.
4. Avoid exposing internal implementation details through public API
   responses.
5. Review all endpoints under the /rest/admin namespace for equivalent
   authorization weaknesses.
6. Add automated authorization tests to prevent regression.

## Retest

Pending remediation.

## Scope

Testing was restricted to the intentionally vulnerable local OWASP Juice
Shop instance at 127.0.0.1:3000.

No third-party systems were tested.

