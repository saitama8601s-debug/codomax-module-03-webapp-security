# codomax-module-03-webapp-security
Module 03: Web Application Security — Learning Notes & Vulnerability Report
Intern Name: Akshat Dave
Program: Codomax Cybersecurity Virtual Internship
Module: Module 03 (Day 11 – Day 15)
1. Web Application Architecture, HTTP Lifecycle & Session Management
Modern web applications operate on a stateless client-server model over HTTP/HTTPS:
HTTP Request/Response Lifecycle: A browser (client) sends an HTTP request containing a method (GET, POST, PUT, DELETE), headers, and an optional body; the web server processes the logic, interacts with backend databases/APIs, and returns a status code (200 OK, 302 Redirect, 401 Unauthorized, 403 Forbidden, 500 Server Error) with response headers and payload.
Cookies & Sessions: Because HTTP is stateless, servers issue a unique Session ID stored in a browser cookie or JSON Web Token (JWT) after login. Secure cookies must enforce HttpOnly (blocks JavaScript/XSS access), Secure (transmits only over HTTPS), and SameSite=Strict (mitigates CSRF).
Authentication & Authorization: Authentication verifies identity (credentials, MFA), while Authorization enforces Role-Based Access Control (RBAC) on every server-side endpoint to ensure users cannot access resources belonging to other accounts.
2. OWASP Top 10 Vulnerabilities Analysis
Vulnerability Category
Root Cause & Impact
Primary Remediation
 
A01: Broken Access Control
Missing server-side ownership checks (e.g., IDOR via URL parameter tampering) allow unauthorized data access or privilege escalation.
Enforce deny-by-default server-side RBAC and validate object ownership on every API/page request.
A02: Cryptographic Failures
Transmitting data over plaintext HTTP or storing passwords with weak/unsalted hashes (MD5/SHA1) exposes sensitive data.
Enforce TLS 1.2/1.3 (HTTPS + HSTS) and hash passwords using strong adaptive algorithms (Argon2id, bcrypt).
A03: Injection (SQLi & XSS)
Untrusted user input concatenated directly into SQL queries (SQL Injection) or rendered unescaped in HTML (Cross-Site Scripting).
Use parameterized queries / prepared statements for SQL; apply context-aware output encoding and Content Security Policy (CSP) for XSS.
A07: Authentication Failures
Weak password policies, lack of rate limiting against credential stuffing, or predictable session tokens enable account takeover.
Implement Multi-Factor Authentication (MFA), account lockout/rate limiting, and regenerate session IDs upon login.

3. Secure Web Application Lab Walkthrough (Identify → Test → Fix → Document)
Lab Environment: Local sandboxed web application (Node.js/Express + SQLite test portal) evaluated in an isolated test environment.
Lab Finding 1: SQL Injection in Login Endpoint
Identify: Login form concatenated raw POST parameters directly into a SQL string.
Test: Submitting ' OR '1'='1' -- bypassed authentication logic in the sandboxed login query.
Fix: Replaced string concatenation with parameterized prepared statements.
// Vulnerable Code (Before)
const query = "SELECT * FROM users WHERE username = '" + req.body.user + "'";

// Remediated Secure Code (After)
const stmt = db.prepare("SELECT * FROM users WHERE username = ?");
const user = stmt.get(req.body.user);
Lab Finding 2: Reflected Cross-Site Scripting (XSS) in Search Parameter
Identify: Search results page reflected the ?q= query parameter directly into the DOM without sanitization.
Test: Supplying a harmless test payload (<script>console.log('XSS-Test')</script>) executed script in the sandbox browser console.
Fix: Enforced HTML entity encoding on output rendering and added a strict Content-Security-Policy header.
Lab Finding 3: Insecure Direct Object Reference (IDOR) on User Profile API
Identify: Endpoint GET /api/profile?id=102 returned profile records solely based on the numeric query ID without checking session ownership.
Test: Changing id=102 to id=101 in an intercepted proxy request exposed another test user's record.
Fix: Added server-side session verification comparing req.session.userId === requestedId before returning records (returning 403 Forbidden on mismatch).
4. Summary Remediation Checklist
Validate and sanitize all client-supplied inputs on the server side using strict allowlists.
Use parameterized queries/ORMs for all database operations.
Enforce HttpOnly, Secure, and SameSite cookie flags alongside HTTPS across all routes.
Deploy security response headers (Content-Security-Policy, X-Content-Type-Options: nosniff, Strict-Transport-Security).
