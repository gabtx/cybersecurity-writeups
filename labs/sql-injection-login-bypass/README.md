# SQL Injection | Authentication Bypass

**Platform:** PortSwigger Web Security Academy  
**Lab:** SQL injection vulnerability allowing login bypass  
**Difficulty:** Apprentice  
**Status:** Solved  
**Tool:** Burp Suite Community Edition  
**Date:** 2026-10-21

## Objective

Demonstrate how a SQL injection vulnerability in a login form can allow authentication as the `administrator` user without knowing their password.

## Environment and scope

This exercise was performed exclusively in the intentionally vulnerable PortSwigger Web Security Academy laboratory.

## Methodology

1. Opened the lab and accessed the login page.
2. Used Burp Suite Proxy > HTTP history to capture the `POST /login` request.
3. Sent the request to Burp Repeater to inspect the form fields (`csrf`, `username`, and `password`) and the session cookie.
4. Tested an SQL comment injection in the username field: `administrator'--` (URL encoded as `administrator%27--`), leaving the password empty.
5. Preserved the valid CSRF token and session context in the request.
6. Observed an HTTP `302 Found` response with `Location: /my-account?id=administrator`.
7. Followed up with a `GET /my-account?id=administrator` request using the authenticated session.
8. Confirmed the account page displayed `Your username is: administrator` and the lab status showed `Solved`.

### Why it works

In a vulnerable query assembled through string concatenation, the injected apostrophe can terminate the username string and the SQL comment sequence can cause the password check to be ignored. The exact backend SQL query was not available for inspection; this describes the mechanism illustrated by the lab.

## Evidence

- Burp Repeater showed a redirect to `/my-account?id=administrator`.
- The rendered response displayed **Your username is: administrator**.
- PortSwigger displayed **Congratulations, you solved the lab!** and **LAB Solved**.

> Screenshot: add the sanitized Burp Repeater screenshot to `evidence/sql-injection-login-bypass-solved.png` and embed it here. Avoid publishing active session cookies or CSRF tokens.

## Security impact

An attacker could bypass authentication and gain unauthorized access to a privileged account, potentially exposing sensitive information or administrative functions.

## Remediation

- Use parameterized queries / prepared statements for all database interactions.
- Avoid constructing SQL by concatenating user-controlled strings.
- Enforce least privilege for database accounts.
- Add automated tests covering SQL injection attempts against authentication.
- Treat input validation and monitoring as defense in depth, not replacements for parameterized queries.

## Learning outcome

Practiced capturing login traffic, editing HTTP requests in Burp Repeater, interpreting redirects and session state, and verifying a successful authentication bypass.

**Reference:** [PortSwigger lab](https://portswigger.net/web-security/sql-injection/lab-login-bypass)
