# SecureWallet – Security Mitigations

**Project:** SecureWallet API
**Document:** Security Mitigations
**Version:** 1.0
**Assessment stage:** Design and implementation planning
**Date:** 22 November 2026

## 1. Purpose

This document describes the security controls proposed to address the threats identified during the SecureWallet threat-modelling exercise.

It translates the identified risks into practical application security requirements and implementation guidance.

Each mitigation is mapped to the relevant threat IDs in the Threat Register to maintain traceability between identified threats, security controls and subsequent verification activities.

A proposed mitigation is not considered effective until it has been implemented and supported by appropriate testing evidence.

## 2. Security Control Objectives

The security controls are intended to achieve the following objectives:

- Prevent unauthorized access to customer and administrator accounts.
- Protect authentication credentials and access tokens.
- Enforce authorization at both role and resource levels.
- Prevent unauthorized wallet balance and transaction modifications.
- Protect customer records and transaction information.
- Maintain reliable transaction and administrative audit records.
- Reduce the likelihood of application and database availability disruptions.
- Minimize the impact of compromised application components or accounts.

## 3. Authentication and Password Security

**Related threats:** TH-01, TH-04, TH-05, TH-20

### Security requirements

1. Passwords must never be stored in plaintext.
2. Passwords must be hashed using Argon2id.
3. Password verification must use the password-hashing library rather than direct string comparison against stored credentials.
4. Login responses must not reveal whether a particular email address is registered.
5. Authentication attempts must be subject to appropriate rate limiting.
6. Administrator accounts should receive stronger authentication protections, with MFA considered for privileged access.
7. Authentication failures must be handled without exposing internal application details.

### Implementation guidance

- Use a maintained Argon2id implementation.
- Apply password-length and input-validation requirements.
- Use generic authentication failure messages.
- Implement login throttling or rate limiting.
- Monitor repeated authentication failures.
- Avoid logging submitted passwords or password hashes.

### Verification evidence

- Passwords are stored as Argon2id hashes.
- Correct credentials authenticate successfully.
- Incorrect credentials are rejected.
- Registered and unregistered email addresses produce appropriately generic login failures.
- Excessive authentication attempts are restricted.
- Sensitive credentials do not appear in application logs.

## 4. JWT and Token Security

**Related threats:** TH-02, TH-03, TH-20

### Security requirements

1. Access tokens must be cryptographically signed.
2. The application must validate token signatures before accepting authenticated requests.
3. Token expiry must be enforced.
4. Required claims must be validated.
5. Unsupported signing algorithms and invalid signatures must be rejected.
6. Client-supplied token claims must never be trusted without cryptographic validation.
7. Complete access tokens must not be written to application logs.
8. Sensitive information must not be stored in JWT payloads.

### Implementation guidance

- Use a trusted signing key maintained outside source control.
- Configure an explicit supported signing algorithm.
- Set appropriate access-token lifetimes.
- Validate expiry and required claims.
- Keep token verification logic centralized.
- Avoid exposing signing keys through application errors or repository files.
- Consider token revocation or rotation mechanisms where appropriate.

A signed JWT provides integrity protection but is not automatically encrypted. Its payload should therefore be treated as readable by anyone who obtains the token.

### Verification evidence

- Valid signed tokens are accepted.
- Expired tokens are rejected.
- Modified token payloads are rejected when their signatures are invalid.
- Tokens signed using unsupported algorithms are rejected.
- Missing or invalid required claims are rejected.
- Signing keys and complete tokens are absent from source control and application logs.

## 5. Authorization and Access Control

**Related threats:** TH-03, TH-06, TH-07, TH-09, TH-12, TH-19, TH-21, TH-23, TH-25

### Security requirements

1. Authorization must be enforced by the backend.
2. Customers must not be able to assign themselves administrator privileges.
3. Administrator-only operations must require administrator authorization.
4. Customers must only access wallets they are authorized to use.
5. Customers must only retrieve their permitted transaction records.
6. Object identifiers must never be treated as proof of authorization.
7. Privileged account attributes must not be freely modifiable through customer-controlled requests.
8. Database and application permissions must follow least-privilege principles.

### Implementation guidance

- Implement role-based access control (RBAC).
- Assign the customer role server-side during registration.
- Derive the authenticated user ID from the validated token.
- Apply ownership checks to every wallet and transaction operation.
- Restrict role changes to authorized administrative functionality.
- Avoid mass assignment of privileged fields.
- Apply default-deny authorization.
- Perform authorization checks on every administrative endpoint.

Authentication answers the question, "Who are you?"

Authorization answers the question, "What are you allowed to do?"

A successfully authenticated customer must not automatically receive access to every application resource.

### Verification evidence

- Customers cannot modify their own role through registration or profile requests.
- Customers cannot invoke administrator-only endpoints successfully.
- Customers cannot access another customer's wallet.
- Customers cannot retrieve another customer's transaction history.
- Attempts to supply another customer's identity as the sender are rejected.
- Unauthorized privilege changes are rejected and appropriately recorded.

## 6. Wallet and Transaction Integrity

**Related threats:** TH-07, TH-08, TH-09, TH-10, TH-11, TH-13, TH-14

### Security requirements

1. Transaction amounts must be validated server-side.
2. Zero, negative and invalid monetary values must be rejected.
3. The sender must be derived from the authenticated user context.
4. Wallet ownership must be verified before performing wallet operations.
5. Transfers must require sufficient available balance.
6. Debit and credit operations must be performed atomically.
7. Failed transactions must not leave partially updated wallet balances.
8. Repeated transaction submissions must be considered when implementing idempotency protections.
9. Funding operations must be authorized and recorded.
10. Transaction records must preserve relevant details of completed operations.

### Implementation guidance

- Use an appropriate decimal or integer-based representation for monetary values.
- Validate amount boundaries and transaction limits.
- Perform balance checks within the appropriate database transaction.
- Use conditional updates or appropriate locking to protect against concurrent overspending.
- Ensure both sides of a transfer succeed or fail together.
- Generate unique transaction identifiers.
- Record transaction status and timestamps.
- Prevent customers from directly modifying wallet balances.
- Clearly identify funding as simulated functionality rather than verified external payment settlement.

### Verification evidence

- Negative and zero transfer amounts are rejected.
- Excessive amounts are rejected according to configured limits.
- Customers cannot transfer from wallets they do not own.
- Insufficient balances prevent transfers.
- Concurrent transfer tests do not result in overspending.
- Failed transfers do not partially modify wallet balances.
- Funding requests cannot bypass authorization or amount validation.
- Transaction records correctly represent successful and failed operations.

## 7. Database and Data Protection

**Related threats:** TH-15, TH-16, TH-17, TH-19

### Security requirements

1. Database access must be restricted to authorized application components.
2. Database queries must be constructed safely.
3. Database accounts must not have unnecessary privileges.
4. Database files and backups must not be publicly exposed.
5. Sensitive information must be protected from unauthorized access.
6. Database errors must not reveal internal implementation details.

### Implementation guidance

- Use SQLAlchemy ORM operations and parameterized queries safely.
- Avoid unsafe dynamic SQL construction.
- Validate incoming data before database operations.
- Apply least-privilege database permissions.
- Protect SQLite database files using appropriate operating-system permissions.
- Exclude database files and backups from source control.
- Protect backups and restrict access to their storage locations.
- Avoid exposing database services unnecessarily.

### Verification evidence

- SQL injection tests do not result in unauthorized database operations.
- Database access permissions are appropriately restricted.
- Database files are not exposed through public application routes.
- Backups are excluded from source control.
- Unauthorized database operations are denied.
- Database errors do not expose sensitive internal information.

## 8. Audit Logging and Accountability

**Related threats:** TH-11, TH-14, TH-22

### Security requirements

The application must maintain appropriate records of relevant security-sensitive operations.

Audit events should capture, where applicable:

- Event identifier.
- Initiating user or administrator.
- Event type.
- Affected account or resource.
- Timestamp.
- Operation outcome.
- Relevant transaction identifier.

### Implementation guidance

Record important events such as:

- Successful and failed authentication attempts.
- Account creation.
- Privilege changes.
- Administrative account-management operations.
- Wallet funding operations.
- Transfer initiation and completion.
- Relevant authorization failures.

Audit logs must be protected against unauthorized modification.

Passwords, complete JWTs and unnecessary sensitive customer information must not be recorded.

### Verification evidence

- Successful transactions generate appropriate records.
- Failed transactions are represented accurately.
- Administrative actions generate audit events.
- Relevant event details are present.
- Unauthorized users cannot modify audit records.
- Sensitive credentials are absent from logs.

## 9. Availability and Resource Protection

**Related threats:** TH-05, TH-13, TH-18, TH-24

### Security requirements

1. Sensitive endpoints must have appropriate request limits.
2. Expensive operations must be controlled.
3. Requests must have sensible size and processing limits.
4. Database queries must avoid unnecessary resource consumption.
5. Administrative operations must not be allowed to consume unlimited resources.

### Implementation guidance

- Apply endpoint-specific rate limiting.
- Validate request bodies and parameters.
- Implement pagination for transaction-history retrieval.
- Apply appropriate query and request timeouts.
- Restrict unnecessary bulk operations.
- Monitor unusual request patterns.
- Use database indexes where appropriate.

### Verification evidence

- Excessive login requests are restricted.
- Repeated transaction requests are appropriately limited.
- Large requests are handled safely.
- Transaction-history pagination works correctly.
- Resource-intensive administrative operations are controlled.
- Normal application operations remain functional under controlled testing.

## 10. Secure Development Practices

**Related threats:** TH-01 to TH-25, as applicable

Security must be considered throughout application development rather than introduced only after functionality is complete.

The project will incorporate the following practices:

| Practice | Purpose |
|---|---|
| Input validation | Prevent invalid or unexpected data from entering application workflows |
| Secure error handling | Avoid exposing sensitive implementation information |
| Static code analysis | Identify potentially insecure code patterns |
| Dependency security checks | Identify vulnerable third-party packages |
| Secret scanning | Detect credentials or sensitive values accidentally committed to source control |
| Automated security tests | Verify security requirements continuously |
| Code review | Identify unsafe logic and missing authorization checks |
| Git version control | Track implementation changes and security-related improvements |

Tools planned for the project include:

- Bandit or Semgrep for static analysis.
- Pytest for automated testing.
- Gitleaks for secret scanning.
- pip-audit for Python dependency vulnerability checks.
- Postman for manual API testing.

## 11. Security Control Traceability

The following table connects major security controls to the threats they address.

| Security control | Related threat IDs |
|---|---|
| Argon2id password hashing | TH-01 |
| Login rate limiting | TH-01, TH-05 |
| JWT signature and expiry validation | TH-02, TH-03, TH-20 |
| Generic authentication errors | TH-04 |
| Server-side role assignment | TH-06 |
| Server-derived sender identity | TH-07 |
| Transfer amount validation | TH-08 |
| Wallet ownership enforcement | TH-09 |
| Atomic transfers and balance enforcement | TH-10 |
| Transaction and audit records | TH-11, TH-22 |
| Transaction ownership enforcement | TH-12 |
| Transaction endpoint rate limiting | TH-13 |
| Authorized simulated funding | TH-14 |
| Database access restrictions | TH-15, TH-19 |
| Safe parameterized database operations | TH-16 |
| Backup protection | TH-17 |
| Pagination and query limits | TH-18 |
| Administrator authentication protection | TH-20 |
| Restricted privilege modification | TH-21 |
| Administrative endpoint authorization | TH-23, TH-25 |
| Administrative operation limits | TH-24 |

## 12. Implementation and Verification Status

The controls in this document represent proposed security requirements and implementation guidance.

Their status must be updated as development and testing progress.

| Status | Meaning |
|---|---|
| Planned | Control has been identified but implementation has not started |
| Implemented | Control has been added to the application |
| Verified | Testing provides evidence that the control works as intended |
| Needs improvement | Testing identified a weakness or incomplete implementation |

A control must not be marked verified solely because corresponding code exists.

Verification requires appropriate evidence, such as successful automated tests, manual security testing or configuration review.

## 13. Assessment Limitations

SecureWallet is a fictional wallet API developed for educational and portfolio purposes.

The proposed controls are intended to support a secure local implementation. They do not represent a complete production banking security architecture.

The effectiveness of each control will depend on its implementation, configuration and verification.

Additional security requirements may be identified during development and testing.

## 14. Current Status

**Security mitigation planning:** Documented.

**Threat coverage:** Mitigations mapped to the identified SecureWallet threats.

**Implementation status:** Pending implementation and verification.

**Next activities:** Implement the controls, collect testing evidence and update the Threat Register with verified residual risk assessments.
