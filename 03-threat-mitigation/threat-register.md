# SecureWallet – Threat Register

**Project:** SecureWallet API
**Document:** Security Threat Register
**Version:** 1.0
**Assessment stage:** Design-stage threat assessment
**Date:** 22 November 2026

## 1. Purpose

This register documents the security threats identified during the SecureWallet threat-modelling exercise. It records the relevant STRIDE category, affected components, attack scenarios, inherent risk ratings, proposed security controls and evidence required to verify those controls.

The register will be updated as security controls are implemented and tested.

## 2. Risk assessment methodology

Risk is calculated using:

**Risk score = Likelihood × Impact**

Both likelihood and impact are rated from 1 to 5.

| Score | Likelihood / Impact |
|---|---|
| 1 | Very Low |
| 2 | Low |
| 3 | Moderate |
| 4 | High |
| 5 | Very High |

### Risk classification

| Score | Risk level |
|---|---|
| 1–4 | Low |
| 5–9 | Medium |
| 10–16 | High |
| 17–25 | Critical |

**Important:** Inherent risk represents the risk before security controls are applied. Residual risk represents the risk remaining after controls have been implemented and their effectiveness verified.

A proposed control does not automatically reduce the residual risk. Residual risk remains pending until implementation and testing provide sufficient evidence.

## 3. Threat register

The table below consolidates all 25 identified threats. Each row represents one threat, and each column captures a specific part of its assessment and treatment.

| ID | Threat | STRIDE | Affected component(s) | Attack scenario | Likelihood | Impact | Inherent risk | Recommended security controls | Verification evidence | Residual risk | Status |
|---|---|---|---|---|---:|---:|---|---|---|---|---|
| TH-01 | Stolen Customer Credentials | Spoofing | Authentication & Authorization; SecureWallet API | An attacker obtains a customer's credentials through phishing, credential reuse, malware or another compromise and authenticates as that customer to access their wallet. | 4 | 5 | **20 – Critical** | Hash passwords using Argon2id; enforce appropriate password requirements; implement login rate limiting and account protection; consider MFA; use HTTPS; return generic authentication errors. | Password-hashing tests; authentication tests; login rate-limit tests; HTTPS configuration review. | Pending verification | Open – controls not yet verified |
| TH-02 | Stolen Access Token Replay | Spoofing | Authentication & Authorization; SecureWallet API | An attacker obtains a valid customer's access token and reuses it to access protected API endpoints while the token remains valid. | 3 | 5 | **15 – High** | Use short-lived access tokens; validate signatures, expiry and relevant claims; avoid insecure token storage; never log complete tokens; implement revocation or token rotation where appropriate. | Expired-token tests; invalid-signature tests; token-reuse tests; token-handling review. | Pending verification | Open – controls not yet verified |
| TH-03 | Manipulation of Token Claims | Elevation of Privilege | Authentication & Authorization; SecureWallet API | An attacker modifies JWT claims, such as the user ID or role, to impersonate another user or obtain administrator privileges. | 2 | 5 | **10 – High** | Validate JWT signatures using a trusted signing key; validate expiry and required claims; reject invalid signatures and unsupported algorithms; never trust client-supplied roles or identities; keep sensitive information out of JWT payloads. | Tests using modified JWT payloads, invalid signatures, expired tokens and altered role claims. | Pending verification | Open – controls not yet verified |
| TH-04 | Account Enumeration Through Login Responses | Information Disclosure | Authentication & Authorization; SecureWallet API | An attacker submits different email addresses and compares login responses to determine which email addresses are registered with SecureWallet. | 4 | 2 | **8 – Medium** | Return generic login failure messages; avoid exposing whether an account exists; ensure error responses do not reveal unnecessary account information. | Compare responses for registered and unregistered email addresses, including response messages and relevant status codes. | Pending verification | Open – controls not yet verified |
| TH-05 | Login Endpoint Abuse | Denial of Service | SecureWallet API; Authentication & Authorization | An attacker repeatedly submits authentication requests to overwhelm the login endpoint or conduct large-scale password guessing. | 4 | 3 | **12 – High** | Implement rate limiting; apply appropriate login attempt restrictions; use request validation and sensible request-size limits; monitor repeated authentication failures. | Automated tests demonstrating that excessive login requests are restricted and normal requests remain functional. | Pending verification | Open – controls not yet verified |
| TH-06 | Customer Manipulates Their Role to Become an Administrator | Elevation of Privilege | Authentication & Authorization; Account Management Service | A customer attempts to modify a role field during registration, profile updates or another API request to assign themselves administrator privileges. | 3 | 5 | **15 – High** | Assign customer roles on the server; prevent customers from updating privileged account attributes; enforce RBAC; apply default-deny authorization. | Registration and profile-update tests containing manipulated role fields; tests confirming customers cannot access administrator functions. | Pending verification | Open – controls not yet verified |
| TH-07 | Attacker Initiates a Transfer as Another Customer | Spoofing (secondary: Tampering) | Authentication & Authorization; Wallet & Transaction Service | An attacker manipulates a transfer request to make it appear that another customer initiated the transaction, or uses another customer's compromised identity to initiate an unauthorized transfer. | 3 | 5 | **15 – High** | Derive sender identity from the authenticated server-side context; never trust a sender ID supplied by the client; enforce object-level authorization; validate source-wallet ownership. | Tests submitting another customer's ID as sender and attempting transfers from wallets the authenticated customer does not own. | Pending verification | Open – controls not yet verified |
| TH-08 | Transfer Amount Manipulation | Tampering | SecureWallet API; Wallet & Transaction Service | A customer intercepts or modifies a transfer request to change the intended transfer amount, for example changing ₦5,000 to ₦50,000. | 4 | 5 | **20 – Critical** | Validate transfer amounts on the server; reject zero, negative and invalid amounts; apply transaction limits; use appropriate decimal or integer-based monetary representation; perform balance checks and wallet updates atomically. | API tests using altered amounts, negative values, zero values, excessive values and boundary values. | Pending verification | Open – controls not yet verified |
| TH-09 | Unauthorized Access to Another Customer's Wallet | Information Disclosure (secondary: Tampering) | SecureWallet API; Wallet & Transaction Service; Wallet Records | A customer changes a wallet identifier in an API request to access another customer's wallet balance or attempt to modify that wallet. | 4 | 5 | **20 – Critical** | Enforce object-level authorization on every wallet operation; verify wallet ownership using the authenticated user's identity; do not rely on unpredictable IDs as authorization; apply default-deny access rules. | BOLA/IDOR tests attempting to read and modify wallets belonging to other customers. | Pending verification | Open – controls not yet verified |
| TH-10 | Overspending and Insufficient Balance Control | Tampering | Wallet & Transaction Service; Wallet Records; Transaction Records | A customer initiates multiple concurrent transfers. Due to inadequate balance checks or transaction handling, multiple requests may succeed even though the wallet does not have enough funds to cover them. | 4 | 5 | **20 – Critical** | Perform balance checks on the server; use atomic database transactions; use conditional balance updates or appropriate locking; ensure transfers cannot partially update wallet balances; implement idempotency protections where appropriate. | Concurrent transfer tests; insufficient-balance tests; tests confirming failed transfers do not alter wallet balances. | Pending verification | Open – controls not yet verified |
| TH-11 | Customer Disputes a Transaction Due to Inadequate Audit Records | Repudiation | Wallet & Transaction Service; Transaction Records; Audit Logs | A customer disputes a transfer, but SecureWallet cannot establish who initiated it, when it occurred or whether it succeeded because transaction and audit records are incomplete. | 3 | 4 | **12 – High** | Maintain transaction records with unique identifiers; record initiating user, amount, timestamp and transaction status; record relevant security events; protect audit records against unauthorized modification. | Transaction-history tests; audit-log inspection; tests confirming successful and failed operations are recorded appropriately. | Pending verification | Open – controls not yet verified |
| TH-12 | Unauthorized Access to Another Customer's Transaction History | Information Disclosure | SecureWallet API; Wallet & Transaction Service; Transaction Records | A customer changes a transaction or wallet identifier in an API request to retrieve another customer's transaction history. | 4 | 4 | **16 – High** | Enforce ownership checks on transaction-history requests; derive customer identity from the authenticated context; apply object-level authorization; return only information the requesting customer is permitted to view. | Tests attempting to access transaction records belonging to other customers. | Pending verification | Open – controls not yet verified |
| TH-13 | Transaction Endpoint Abuse | Denial of Service | SecureWallet API; Wallet & Transaction Service | An attacker repeatedly submits transaction requests or other resource-intensive operations to degrade API availability and prevent legitimate customers from using the wallet service. | 3 | 4 | **12 – High** | Apply rate limits to sensitive endpoints; validate request sizes and input values; set appropriate request timeouts; monitor unusual transaction-request patterns. | Rate-limit tests; invalid-request tests; basic load or stress testing. | Pending verification | Open – controls not yet verified |
| TH-14 | Unauthorized Wallet Funding | Tampering | SecureWallet API; Wallet & Transaction Service; Wallet Records; Transaction Records | A customer manipulates a funding request or invokes a funding operation without the required authorization, causing their wallet balance to increase improperly. | 3 | 5 | **15 – High** | Restrict funding operations to authorized users and processes; validate funding amounts server-side; prevent customers from directly modifying wallet balances; record funding operations in transaction records and audit logs; distinguish simulated funding from verified external payment settlement. | Tests for unauthorized funding, manipulated amounts and direct balance modification attempts. | Pending verification | Open – controls not yet verified |
| TH-15 | Unauthorized Database Access | Information Disclosure | Database; User Records; Wallet Records; Transaction Records | An attacker gains unauthorized access to the database and retrieves customer information, wallet balances or transaction records. | 2 | 5 | **10 – High** | Restrict database access to authorized application components; apply least-privilege permissions; protect database files and deployment environments; avoid publicly exposing database services; protect sensitive data and backups. | Database permission review; file-permission checks; tests confirming unauthorized access is denied. | Pending verification | Open – controls not yet verified |
| TH-16 | SQL Injection | Tampering (secondary: Information Disclosure) | SecureWallet API; Account Management Service; Wallet & Transaction Service; Database | An attacker submits crafted input that is interpreted as part of a database query, potentially allowing unauthorized data retrieval or modification. | 3 | 5 | **15 – High** | Use SQLAlchemy ORM operations and parameterized queries safely; avoid unsafe dynamic SQL construction; validate and constrain user input; apply least-privilege database access; return generic errors without exposing database internals. | SQL injection tests against relevant API inputs; code review; static-analysis results. | Pending verification | Open – controls not yet verified |
| TH-17 | Exposed Database Backups | Information Disclosure | Database; User Records; Wallet Records; Transaction Records | An attacker obtains an exposed or improperly stored database backup containing customer details, wallet information or transaction history. | 2 | 5 | **10 – High** | Store backups outside publicly accessible locations; restrict backup access; encrypt backups where appropriate; exclude database backups from source control; establish backup retention and disposal practices. | Repository secret/file checks; backup-location review; access-permission checks. | Pending verification | Open – controls not yet verified |
| TH-18 | Expensive Queries Affect Application Availability | Denial of Service | SecureWallet API; Wallet & Transaction Service; Database | An attacker submits requests that trigger expensive database queries or retrieve excessive amounts of data, consuming resources and degrading service availability. | 3 | 4 | **12 – High** | Implement pagination for transaction history; limit request and response sizes; add appropriate database indexes; apply query and request timeouts; restrict expensive operations. | Pagination tests; query-performance checks; tests using unusually large request parameters. | Pending verification | Open – controls not yet verified |
| TH-19 | Excessive Database Account Permissions | Elevation of Privilege | Database; SecureWallet API | The application or a compromised database account has excessive permissions, allowing an attacker to perform operations beyond what is required for normal application functionality. | 3 | 4 | **12 – High** | Apply least-privilege database permissions; separate administrative and application database responsibilities where appropriate; avoid unnecessary privileged database accounts; review database permissions periodically. | Database permission review; tests confirming restricted accounts cannot perform unauthorized operations. | Pending verification | Open – controls not yet verified |
| TH-20 | Administrator Impersonation | Spoofing | Authentication & Authorization; SecureWallet API; Account Management Service | An attacker obtains administrator credentials or a valid administrator token and uses it to impersonate an authorized administrator. | 3 | 5 | **15 – High** | Apply strong authentication to administrator accounts; consider MFA for privileged users; validate administrator tokens and claims; use short-lived access tokens and appropriate revocation mechanisms; monitor privileged authentication activity. | Administrator authentication tests; invalid-token tests; privileged-access log review. | Pending verification | Open – controls not yet verified |
| TH-21 | Unauthorized Privilege Modification | Elevation of Privilege | Account Management Service; Authentication & Authorization; User Records | A customer exploits an account-management function or improperly protected user record to modify their privileges or another user's role. | 3 | 5 | **15 – High** | Restrict role modification to authorized administrative operations; enforce server-side authorization checks; prevent mass assignment of privileged fields; record privilege changes in audit logs. | Tests attempting unauthorized role changes; tests confirming privilege changes are restricted and logged. | Pending verification | Open – controls not yet verified |
| TH-22 | Administrator Denies an Administrative Action | Repudiation | Account Management Service; Audit Logs; User Records | An administrator performs a sensitive operation, such as disabling a customer account, and later denies carrying out the action because the system lacks reliable audit evidence. | 3 | 4 | **12 – High** | Record administrator identity and actions; record timestamps, affected accounts and operation outcomes; protect audit logs from unauthorized modification; restrict access to audit records. | Tests confirming administrative actions generate appropriate audit events and unauthorized users cannot alter audit records. | Pending verification | Open – controls not yet verified |
| TH-23 | Customer Accesses Administrator Information | Information Disclosure | SecureWallet API; Account Management Service; User Records | A customer accesses an administrative endpoint or response that exposes administrator-only information, such as account-management data or privileged user details. | 3 | 4 | **12 – High** | Enforce role-based access control; apply authorization checks to every administrative endpoint; return only information required for each operation; avoid exposing sensitive administrative information in error messages. | Customer-role tests against administrative endpoints; checks for excessive information in API responses. | Pending verification | Open – controls not yet verified |
| TH-24 | Administrative Operation Abuse Affects Availability | Denial of Service | SecureWallet API; Account Management Service | An authorized or compromised administrator repeatedly performs resource-intensive account-management operations, degrading the availability of the application. | 2 | 3 | **6 – Medium** | Apply sensible request limits to administrative operations; restrict bulk operations where not required; monitor unusual administrative activity; use pagination and timeouts for account-management queries. | Administrative endpoint limit tests; review of resource-intensive operations. | Pending verification | Open – controls not yet verified |
| TH-25 | Customer Accesses an Administrator-Only Endpoint | Elevation of Privilege | SecureWallet API; Authentication & Authorization; Account Management Service | An authenticated customer directly invokes an administrator-only API endpoint, bypassing restrictions that should prevent access to privileged functionality. | 3 | 5 | **15 – High** | Enforce role-based authorization on the server; apply default-deny access rules; never rely on frontend visibility or hidden buttons for authorization; test privileged endpoints using customer-level accounts. | API tests confirming customer accounts receive an authorization failure when invoking administrator-only endpoints. | Pending verification | Open – controls not yet verified |

## 4. Risk summary

The initial assessment identifies 25 threats.

| Risk level | Number of threats |
|---|---:|
| Critical | 4 |
| High | 19 |
| Medium | 2 |
| Low | 0 |
| **Total** | **25** |

### Critical threats requiring priority attention

| Threat ID | Threat | Score |
|---|---|---:|
| TH-01 | Stolen Customer Credentials | 20 |
| TH-08 | Transfer Amount Manipulation | 20 |
| TH-09 | Unauthorized Access to Another Customer's Wallet | 20 |
| TH-10 | Overspending and Insufficient Balance Control | 20 |

These threats are prioritized because they could lead to unauthorized account access, financial manipulation, exposure of customer financial information or loss of wallet integrity.

## 5. Implementation and verification tracking

The following implementation sequence will guide the next phase of the project:

1. **Authentication:** Password hashing, login validation, JWT signing and verification, token expiry.
2. **Authorization:** Role-based access control, object-level authorization and server-side identity enforcement.
3. **Financial integrity:** Amount validation, wallet ownership checks, atomic transfers, balance enforcement and idempotency.
4. **Database security:** Safe SQLAlchemy usage, restricted access and protected backups.
5. **Auditability:** Authentication, administrative and transaction audit events.
6. **Availability:** Rate limiting, pagination, request limits and timeouts.
7. **Secure development:** Static analysis, dependency checks, secret scanning and security tests.

For each implemented control, record:

- The threat IDs it addresses.
- The implementation location in the source code.
- The relevant test case.
- The test result.
- Any remaining limitations.
- The updated residual risk rating.

## 6. Assessment limitations

This register represents an initial design-stage assessment of a fictional wallet API. The likelihood and impact ratings are preliminary estimates based on the proposed functionality and architecture.

They are not confirmed vulnerabilities or results from a completed penetration test.

The register must be reviewed as the application architecture evolves and as security testing produces new evidence.

**Current overall status:** Initial threat identification and inherent risk assessment completed. Security control implementation and verification pending.