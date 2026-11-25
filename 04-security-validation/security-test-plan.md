# SecureWallet Security Test Plan

| Document Attribute | Details |
|---|---|
| Project | SecureWallet |
| Document Type | Application Security Test Plan |
| Version | 1.0 |
| Status | Planned |
| Application | SecureWallet REST API |
| Framework | FastAPI |
| Database | SQLite |
| Primary Testing Approach | Manual security testing, automated testing, and static analysis |
| Related Documents | System Architecture, Data Flow Diagram, STRIDE Analysis, Threat Register, Security Mitigations |

---

## 1. Purpose

This document defines the security testing approach for the SecureWallet application.

The purpose is to verify whether the security controls identified during architecture design and threat modelling are implemented correctly and whether the application can resist common attacks against authentication, authorization, wallet operations, transaction processing, data storage, and application availability.

The test plan translates the identified threats into practical security test scenarios and establishes the evidence required to demonstrate whether each security control works as intended.

Testing will take place throughout application development rather than being postponed until the end of the project.

## 2. Security Testing Objectives

The objectives are to:

1. Verify that users can only authenticate using valid credentials.
2. Confirm that passwords and authentication tokens are handled securely.
3. Verify that customers cannot access another customer's information or perform actions on their behalf.
4. Confirm that administrative functionality is restricted to authorized administrators.
5. Protect wallet balances and transaction records against unauthorized modification.
6. Verify that transfers are processed accurately and consistently.
7. Identify common API vulnerabilities, including injection, broken access control, and excessive data exposure.
8. Verify that sensitive information is not unnecessarily exposed through API responses, logs, or database files.
9. Assess whether resource-intensive requests can affect application availability.
10. Collect reproducible evidence demonstrating the effectiveness of implemented security controls.

## 3. Scope

### 3.1 In Scope

The following application functionality and security areas are included:

| Area | Functionality / Security Requirement |
|---|---|
| Registration | Customer account creation and input validation |
| Authentication | Login, password verification, and token issuance |
| Token security | JWT signature validation, expiration, and claims validation |
| Authorization | Role-based access and object-level access control |
| Account management | Customer account management and administrator operations |
| Wallet management | Wallet creation and balance retrieval |
| Wallet funding | Simulated wallet funding |
| Transfers | Transfers between SecureWallet customers |
| Transaction history | Retrieval and protection of transaction records |
| Database security | Access control, query safety, and data protection |
| Audit logging | Recording security-sensitive operations |
| Availability | Rate limiting, request validation, and resource protection |
| Secure development | Static analysis, dependency checks, and secret detection |

### 3.2 Out of Scope

The following are excluded from this project's testing scope:

- Integration with real banks or external financial institutions.
- Real payment processing and settlement.
- Production banking infrastructure.
- Testing against systems or networks that the project does not own or have permission to assess.
- Real customer financial information.
- Formal regulatory certification or independent penetration-testing certification.

Wallet funding and transfers are simulated application functions. No real money is moved.

## 4. Test Environment

Testing will be performed in a controlled local development environment.

| Component | Planned Environment |
|---|---|
| Operating System | Kali Linux |
| Application Framework | FastAPI |
| Programming Language | Python |
| Database | SQLite |
| ORM | SQLAlchemy |
| Authentication | Signed JWT access tokens |
| Password Hashing | Argon2id |
| Automated Testing | Pytest |
| API Testing | Postman |
| Static Analysis | Bandit and Semgrep |
| Dependency Analysis | pip-audit |
| Secret Detection | Gitleaks |
| Version Control | Git |

### 4.1 Environment Requirements

Before security testing begins:

- The application must run successfully in the local development environment.
- Database tables and required application components must be available.
- Test accounts must be created using synthetic information.
- At least two customer accounts must exist for authorization and transfer testing.
- An administrator test account must be available for privileged-operation testing.
- Test data must be resettable so that test scenarios can be repeated.
- No production credentials, personal information, or real financial data may be used.

## 5. Test Identities and Roles

The following identities will be used to verify authentication and authorization boundaries.

| Identity | Description | Expected Access |
|---|---|---|
| Visitor | Unauthenticated user | Public registration and login functionality only |
| Customer A | Standard customer account | Own wallet, account information, and transaction history |
| Customer B | Another standard customer account | Own wallet, account information, and transaction history |
| Administrator | Privileged account | Authorized administrative functionality |

Customer A and Customer B are particularly important for testing object-level authorization.

For example, a valid Customer A token must not allow access to Customer B's wallet simply because Customer A knows or changes the wallet identifier.

## 6. Testing Methodology

Testing will combine manual API security testing, automated application tests, and security scanning.

### 6.1 Authentication Testing

Verify that the application correctly identifies users and protects account credentials.

Activities include:

- Test registration with valid and invalid inputs.
- Test login with valid and invalid credentials.
- Verify that passwords are stored using Argon2id hashes rather than plaintext.
- Examine authentication responses for unnecessary account information.
- Test whether repeated login attempts are appropriately restricted.
- Verify that authentication failures do not disclose sensitive implementation details.

Related threats: TH-01, TH-04, TH-05, TH-20.

### 6.2 JWT and Session Security Testing

Verify that authentication tokens cannot be forged, modified, or accepted after expiration.

Activities include:

- Authenticate and inspect the issued JWT.
- Modify token claims and verify signature validation.
- Test expired tokens.
- Test malformed tokens and invalid signatures.
- Test missing or unexpected authentication claims.
- Verify that the application rejects unsupported signing algorithms.
- Confirm that protected endpoints reject requests without valid tokens.
- Verify that sensitive information is not included in token payloads.

Related threats: TH-02, TH-03, TH-20.

### 6.3 Authorization and Access Control Testing

Verify that authenticated users can only perform operations permitted by their roles and ownership.

Activities include:

- Attempt to access administrator-only endpoints using a customer account.
- Attempt to submit modified role values during registration or account updates.
- Attempt to change another customer's account information.
- Attempt to perform operations using another customer's identifier.
- Verify that administrative privileges are assigned and checked on the server.
- Confirm that customers cannot access administrator information or functionality.

Related threats: TH-03, TH-06, TH-07, TH-21, TH-23, TH-25.

### 6.4 Broken Object-Level Authorization Testing

Verify that object identifiers cannot be manipulated to bypass ownership checks.

Activities include:

- Authenticate as Customer A.
- Request Customer A's wallet and transaction history.
- Change the wallet or transaction identifier to one belonging to Customer B.
- Attempt to retrieve or modify Customer B's records.
- Repeat the checks across relevant endpoints and HTTP methods.

Expected security property: possession of a valid token must not automatically grant access to every object in the application.

Related threats: TH-07, TH-09, TH-12.

### 6.5 Wallet and Transaction Integrity Testing

Verify that wallet balances and transaction operations cannot be manipulated to produce unauthorized financial changes.

Activities include:

- Attempt transfers with modified amounts.
- Submit zero, negative, excessively large, and invalid amounts.
- Attempt transfers when the sender has insufficient funds.
- Verify that the sender's balance decreases by the correct amount.
- Verify that the recipient's balance increases by the same amount.
- Test whether a failed transfer leaves both balances unchanged.
- Attempt to manipulate sender identifiers in transfer requests.
- Test repeated requests and transaction idempotency where implemented.
- Test concurrent transfers to identify possible race conditions.
- Verify that transaction history accurately reflects completed operations.

Related threats: TH-07, TH-08, TH-10, TH-11, TH-13.

### 6.6 Wallet Funding Testing

Verify that wallet funding cannot be performed by unauthorized users or manipulated through client-controlled input.

Activities include:

- Attempt funding without authentication.
- Attempt to fund another customer's wallet.
- Modify wallet identifiers and funding amounts.
- Submit invalid or negative funding amounts.
- Verify that successful funding updates the correct wallet and creates an appropriate transaction record.
- Verify that failed funding attempts do not change wallet balances.

Related threat: TH-14.

### 6.7 Database and Injection Testing

Verify that application inputs cannot be used to execute unauthorized database operations.

Activities include:

- Submit SQL injection test strings through relevant input fields.
- Test registration, login, account lookup, and transaction-related inputs.
- Verify that database queries use safe ORM operations or parameterized statements.
- Inspect database permissions and ensure the application uses only the privileges it requires.
- Verify that database files and backups are not unnecessarily exposed.
- Confirm that sensitive records are not returned through unauthorized API responses.

Related threats: TH-15, TH-16, TH-17, TH-19.

### 6.8 Audit Logging and Accountability Testing

Verify that security-sensitive operations produce useful audit records.

Activities include:

- Verify that successful and failed authentication attempts are recorded.
- Verify that account changes and administrative actions are logged.
- Verify that wallet funding and transfer events are recorded.
- Confirm that logs contain sufficient information to investigate activity.
- Confirm that passwords, full access tokens, and other sensitive secrets are not logged.
- Verify that unauthorized users cannot modify or delete audit records through application endpoints.

Related threats: TH-11, TH-14, TH-22.

### 6.9 Availability and Resource Protection Testing

Verify that application endpoints have reasonable protections against excessive or abusive requests.

Activities include:

- Test repeated login requests.
- Test repeated transaction requests.
- Submit oversized or malformed request bodies.
- Test endpoints that perform database queries with potentially expensive inputs.
- Verify that request size limits and validation are enforced.
- Verify rate limiting where implemented.
- Assess whether administrative operations can be abused to affect application availability.

All availability testing will be conducted locally and at a controlled rate. Destructive denial-of-service testing is excluded.

Related threats: TH-05, TH-13, TH-18, TH-24.

### 6.10 Static Analysis and Dependency Testing

Automated security tools will be used to identify weaknesses in the source code and its dependencies.

| Tool | Purpose |
|---|---|
| Bandit | Identify common Python security issues |
| Semgrep | Detect insecure coding patterns and application security weaknesses |
| pip-audit | Identify known vulnerabilities in installed Python dependencies |
| Gitleaks | Detect accidentally committed credentials and secrets |
| Pytest | Execute automated functional and security-focused tests |

Findings must be reviewed rather than treated as confirmed vulnerabilities automatically. False positives and contextual findings will be assessed before remediation decisions are made.

## 7. Security Test Cases

The following test cases define the planned verification activities.

The expected results describe the intended secure behaviour. Test results will be updated only after the corresponding tests have actually been executed.

### 7.1 Authentication and Token Security

| Test ID | Related Threat | Test Scenario | Expected Result |
|---|---|---|---|
| ST-01 | TH-01 | Login using valid credentials | Authentication succeeds and a valid token is issued |
| ST-02 | TH-01 | Login using an incorrect password | Authentication is rejected |
| ST-03 | TH-01 | Inspect stored customer passwords | Passwords are stored as Argon2id hashes |
| ST-04 | TH-04 | Compare responses for unknown users and incorrect passwords | Responses do not unnecessarily disclose account existence |
| ST-05 | TH-05 | Submit repeated login attempts | Appropriate rate limiting or abuse protection is enforced |
| ST-06 | TH-02 | Replay a valid token before expiration | Access follows the application's documented token policy |
| ST-07 | TH-02 | Submit an expired token | Request is rejected |
| ST-08 | TH-03 | Modify JWT claims without a valid signature | Request is rejected |
| ST-09 | TH-03 | Submit a malformed token or unsupported signing algorithm | Request is rejected |
| ST-10 | TH-20 | Attempt protected operations without valid authentication | Access is denied |

### 7.2 Authorization and Access Control

| Test ID | Related Threat | Test Scenario | Expected Result |
|---|---|---|---|
| ST-11 | TH-06 | Customer submits an administrator role during registration | Customer cannot assign themselves administrator privileges |
| ST-12 | TH-07 | Customer A attempts an operation using Customer B's identity | Server derives identity from authenticated context and rejects unauthorized access |
| ST-13 | TH-09 | Customer A requests Customer B's wallet | Access is denied |
| ST-14 | TH-12 | Customer A requests Customer B's transaction history | Access is denied |
| ST-15 | TH-21 | Customer attempts to modify their own role or privileges | Privilege modification is rejected |
| ST-16 | TH-23 | Customer attempts to retrieve administrator-only information | Access is denied |
| ST-17 | TH-25 | Customer attempts to access an administrator-only endpoint | Access is denied |
| ST-18 | TH-03 | User attempts to manipulate authorization-related token claims | Modified or invalid claims are rejected |

### 7.3 Wallet and Transaction Integrity

| Test ID | Related Threat | Test Scenario | Expected Result |
|---|---|---|---|
| ST-19 | TH-08 | Modify a transfer amount before submitting the request | Server validates the amount and enforces transaction rules |
| ST-20 | TH-08 | Submit zero or negative transfer amounts | Invalid transfers are rejected |
| ST-21 | TH-08 | Submit an excessively large transfer amount | Amount and business-rule validation are enforced |
| ST-22 | TH-10 | Attempt a transfer exceeding the available balance | Transfer is rejected and balances remain unchanged |
| ST-23 | TH-10 | Execute concurrent transfers against the same wallet | Balance integrity is maintained and overspending is prevented |
| ST-24 | TH-10 | Cause a transfer operation to fail partway through processing | Database transaction is rolled back without partial balance changes |
| ST-25 | TH-07 | Attempt to specify another customer as the sender | Sender identity is derived from the authenticated user |
| ST-26 | TH-11 | Complete a transfer and inspect its transaction record | An accurate transaction record is created |
| ST-27 | TH-13 | Submit repeated transaction requests | Appropriate request controls prevent abusive usage |
| ST-28 | TH-14 | Attempt to fund a wallet without authentication | Funding request is rejected |
| ST-29 | TH-14 | Attempt to fund another customer's wallet | Unauthorized funding is rejected |
| ST-30 | TH-14 | Submit an invalid or negative funding amount | Funding request is rejected without changing balances |

### 7.4 Database and Input Security

| Test ID | Related Threat | Test Scenario | Expected Result |
|---|---|---|---|
| ST-31 | TH-16 | Submit SQL injection payloads through application inputs | Inputs cannot alter query structure or execute unauthorized SQL |
| ST-32 | TH-15 | Attempt unauthorized access to database records | Access is restricted to authorized application operations |
| ST-33 | TH-17 | Inspect database and backup file handling | Database files and backups are not unnecessarily exposed |
| ST-34 | TH-19 | Inspect database access privileges | Application database access follows least privilege |
| ST-35 | TH-18 | Submit inputs designed to trigger expensive database queries | Validation and resource controls limit unnecessary processing |

### 7.5 Logging and Availability

| Test ID | Related Threat | Test Scenario | Expected Result |
|---|---|---|---|
| ST-36 | TH-11 | Inspect logs after a completed transaction | Relevant transaction activity is recorded |
| ST-37 | TH-22 | Inspect logs after an administrative action | Administrative activity is attributable and recorded |
| ST-38 | TH-05 | Send repeated authentication requests | Abuse controls operate as designed |
| ST-39 | TH-13 | Send repeated transaction requests | Request controls operate as designed |
| ST-40 | TH-24 | Test administrative operations with repeated requests | Appropriate authorization and resource controls are enforced |

### 7.6 Secure Development and Automated Scanning

| Test ID | Related Threat | Test Scenario | Expected Result |
|---|---|---|---|
| ST-41 | Multiple | Run Bandit against the Python source code | Findings are reviewed and relevant issues are addressed |
| ST-42 | Multiple | Run Semgrep against the application | Relevant insecure coding patterns are investigated |
| ST-43 | TH-15, TH-16, TH-17 | Run database and input security-focused code reviews | Unsafe query and data-handling patterns are identified |
| ST-44 | Multiple | Run pip-audit against installed dependencies | Vulnerable dependencies are assessed and remediated where appropriate |
| ST-45 | TH-01, TH-02, TH-03 | Run Gitleaks against the repository | No real credentials or secrets are committed |

## 8. Evidence Collection

Each executed test should have enough evidence for another person to understand what was tested and reproduce the result.

Acceptable evidence includes:

- Postman request and response screenshots.
- HTTP status codes and response bodies.
- Pytest execution output.
- Relevant application logs.
- Database records before and after a test.
- Bandit and Semgrep scan output.
- pip-audit findings.
- Gitleaks scan results.
- Relevant code references showing the implemented security control.

Evidence must not contain real passwords, private signing keys, full access tokens, or other secrets.

Where appropriate, evidence should include:

1. Test ID.
2. Related threat ID.
3. Test date.
4. Test setup and preconditions.
5. Request or action performed.
6. Actual result.
7. Expected result.
8. Pass/fail status.
9. Relevant screenshot, output, or code reference.
10. Any remediation required.

## 9. Test Result Classification

Each test case will be assigned one of the following statuses.

| Status | Meaning |
|---|---|
| Not Started | Testing has not begun |
| In Progress | Test execution is underway |
| Passed | Actual behaviour matches the expected secure behaviour |
| Failed | Actual behaviour does not meet the expected security requirement |
| Blocked | Test cannot proceed because a dependency or prerequisite is unavailable |
| Not Applicable | Test does not apply to the implemented functionality, with justification recorded |

A test must not be marked Passed solely because the corresponding security control has been documented or planned.

## 10. Defect Handling and Remediation

When a test identifies a security weakness:

1. Record the finding and reference its test case and related threat ID.
2. Describe the affected component and the conditions required to reproduce the issue.
3. Capture relevant evidence without exposing sensitive information.
4. Assess the potential impact and likelihood.
5. Prioritize remediation according to risk.
6. Implement the corrective security control.
7. Repeat the original test.
8. Perform relevant regression tests to ensure the change has not introduced another weakness.
9. Update the test result and threat register.

Critical and high-risk issues should be prioritized before lower-risk findings.

## 11. Testing Prioritization

Testing priority will be informed by the inherent risk ratings in the threat register.

| Priority | Risk Level | Testing Focus |
|---|---|---|
| P1 | Critical | Credential compromise, transfer manipulation, unauthorized wallet access, and balance integrity |
| P2 | High | Token security, privilege escalation, transaction history access, SQL injection, and database security |
| P3 | Medium | Account enumeration and administrative availability concerns |
| P4 | Low | Additional hardening and lower-impact security improvements |

The prioritization does not mean lower-priority tests can be ignored. All applicable test cases should be addressed before the project is considered complete.

## 12. Entry Criteria

Security testing may begin when:

- The relevant application functionality has been implemented.
- The application starts successfully in the local environment.
- Required test accounts and data are available.
- Relevant API endpoints are accessible.
- The test environment is isolated and controlled.
- The corresponding security requirements have been identified.

## 13. Exit Criteria

The security testing phase can be considered complete when:

- All applicable planned test cases have been executed.
- Critical and high-risk security findings have been addressed or explicitly documented as unresolved.
- Failed tests have been retested after remediation.
- Relevant regression testing has been completed.
- Static analysis and dependency scanning have been performed.
- Evidence has been collected for executed tests.
- Outstanding risks and limitations have been documented.
- The threat register reflects the actual implementation and verification status.

Completion of the test plan does not automatically mean that the application is free from vulnerabilities.

## 14. Limitations

This is a local application security project and its testing scope is intentionally limited.

The following limitations apply:

- SQLite is used for local development and does not represent every production database deployment.
- The project does not integrate with real banking or payment systems.
- Testing does not establish compliance with financial-sector regulatory requirements.
- Automated scanners cannot identify every application security weakness.
- Passing a defined test case does not prove that a vulnerability is impossible.
- Testing results are limited to the application version and environment in which they were obtained.

## 15. Relationship to Other Security Documents

This test plan is part of the SecureWallet security documentation set.

| Document | Relationship |
|---|---|
| System Architecture | Defines the application components and their responsibilities |
| Data Flow Diagram | Identifies data movement, actors, processes, data stores, and trust boundaries |
| Threat Register | Records identified threats, risk ratings, and recommended controls |
| STRIDE Analysis | Categorizes threats using the six STRIDE threat categories |
| Security Mitigations | Defines the controls intended to address identified threats |
| Security Test Plan | Defines how the controls and security requirements will be verified |

The threat register identifies what could go wrong. The security mitigations document explains how the application intends to address those threats. This test plan defines how we will verify that the controls work.

## 16. Current Testing Status

<badge>Planned — testing not yet executed</badge>

The test cases in this document represent the intended security verification activities.

No test results are claimed at this stage. As SecureWallet functionality is implemented, the test cases will be executed and their actual results, evidence, and remediation status will be recorded.

The document will evolve alongside the application.
