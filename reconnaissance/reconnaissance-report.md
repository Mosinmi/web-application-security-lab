# Project 4 — Reconnaissance

## Target

OWASP Juice Shop running locally.

Target URL:
http://127.0.0.1:3000

Scope:
Local intentionally vulnerable OWASP Juice Shop instance only.

## Environment

- Windows 10 Pro 22H2
- Docker Desktop
- OWASP Juice Shop
- Nmap 7.99
- OWASP ZAP 2.17.0
- Chrome DevTools

## Nmap Results

### Port Scan

Command:

nmap -p 3000 127.0.0.1

Result:

3000/tcp open

### Service Detection

Command:

nmap -sV -p 3000 127.0.0.1

Result:

3000/tcp open ppp?

The service fingerprint was inconclusive. The application was independently
identified as OWASP Juice Shop through browser and HTTP reconnaissance.

## Application Reconnaissance

The application was accessed through:

http://localhost:3000

Chrome DevTools Network inspection identified the following application
endpoints:

/socket.io/
/assets/i18n/en.json
/rest/admin/application-version
/rest/admin/application-configuration
/api/Challenges/
/rest/languages
/api/Quantitys/
/rest/products/search
/rest/products/search?q=
/rest/products/search?q=apple

Additional application namespaces discovered through OWASP ZAP:

/api/
/api/Challenges
/api/Feedbacks
/api/Quantitys

/rest/
/rest/admin
/rest/languages
/rest/products
/rest/user

## Application Version

GET /rest/admin/application-version

Response:

{"version":"20.2.0"}

## Product Search

GET /rest/products/search?q=apple

Response:

HTTP 200 OK

The endpoint returned product records including product IDs, names,
descriptions and prices.

## Security Headers Observed

The main application response included:

X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'

The main application response did not include:

Content-Security-Policy

## Reconnaissance Conclusion

The application exposes multiple REST and API namespaces and uses HTTP,
WebSocket/Socket.IO and client-side JavaScript resources.

Further vulnerability assessment was performed using OWASP ZAP against the
local Juice Shop instance.
