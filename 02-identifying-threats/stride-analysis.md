# SecureWallet – STRIDE Threat Analysis

**Project:** SecureWallet API
**Document:** STRIDE Threat Analysis
**Version:** 1.0
**Assessment stage:** Design-stage threat assessment
**Date:** 22 November 2026

## 1. Introduction

This document records the STRIDE threat analysis conducted for the SecureWallet API.

STRIDE is a threat-modelling methodology used to identify security threats across an application's architecture and data flows.

The methodology examines six categories of threats:

- Spoofing
- Tampering
- Repudiation
- Information Disclosure
- Denial of Service
- Elevation of Privilege

The analysis considers the SecureWallet actors, application processes, data stores, API operations and trust boundaries identified during the design-stage threat-modelling exercise.

The purpose is to understand how an attacker could misuse the application, identify the potential security consequences and establish appropriate security controls.

## 2. SecureWallet Architecture Scope

The STRIDE analysis considers the following components.

### 2.1 Actors

| Actor | Description |
|---|---|
| Visitor | An unauthenticated user who may register or attempt to authenticate |
| Customer | An authenticated user who can access permitted wallet functionality |
| Administrator | A privileged user responsible for authorized account-management operations |
| Developer | A person responsible for developing and maintaining the application |

### 2.2 Application Processes

| Process | Responsibility |
|---|---|
| SecureWallet API | Receives client requests and coordinates application operations |
| Authentication and Authorization | Verifies user identity and enforces access permissions |
| Account Management Service | Handles customer account and administrative account-management operations |
| Wallet and Transaction Service | Handles wallet operations, simulated funding and customer transfers |

### 2.3 Data Stores

| Data store | Description |
|---|---|
| User Records | Stores customer account information, password hashes and assigned roles |
| Wallet Records | Stores wallet ownership and current wallet balances |
| Transaction Records | Stores historical funding and transfer activities |
| Audit Logs | Stores relevant security, authentication and administrative events |

### 2.4 Trust Boundaries

Two primary trust boundaries are considered:

**External Client and Application Boundary**

Separates external clients from the SecureWallet application. Requests crossing this boundary are untrusted and require appropriate validation, authentication and authorization.

**Application and Database Boundary**

Separates application processing from persistent data storage. Database operations must be controlled and performed through authorized application components.

## 3. STRIDE Categories

### 3.1 Spoofing

**Definition**

Spoofing occurs when an attacker pretends to be another user, system or trusted identity to gain unauthorized access or perform actions under that identity.

**How it applies to SecureWallet**

An attacker may attempt to impersonate a customer or administrator by obtaining credentials, stealing access tokens or manipulating authentication-related information.

**Relevant components**

- SecureWallet API
- Authentication and Authorization
- Account Management Service
- Wallet and Transaction Service

**Identified threats**

| Threat ID | Threat | Attack scenario |
|---|---|---|
| TH-01 | Stolen Customer Credentials | An attacker obtains customer credentials and authenticates as the legitimate customer |
| TH-02 | Stolen Access Token Replay | An attacker obtains and reuses a valid customer's access token |
| TH-07 | Attacker Initiates a Transfer as Another Customer | An attacker attempts to initiate a transfer under another customer's identity |
| TH-20 | Administrator Impersonation | An attacker obtains administrator credentials or a valid administrator token |

**Potential impact**

- Unauthorized access to customer accounts.
- Unauthorized wallet operations.
- Unauthorized transfers.
- Compromise of administrative functionality.
- Loss of customer trust.

**Recommended security controls**

- Hash passwords using Argon2id.
- Validate credentials securely.
- Use signed JWT access tokens.
- Validate token signatures, expiry and required claims.
- Use short-lived access tokens.
- Protect token storage and handling.
- Apply login rate limiting.
- Consider MFA, particularly for privileged accounts.
- Derive user identity from the authenticated server-side context.

**Verification approach**

Test authentication using valid and invalid credentials, expired tokens, invalid signatures, stolen-token scenarios and attempts to perform operations using another customer's identity.

### 3.2 Tampering

**Definition**

Tampering occurs when an attacker modifies data, requests, messages or application resources without authorization.

**How it applies to SecureWallet**

An attacker may intercept API requests and manipulate transaction amounts, wallet information, account data or other values that influence application behaviour.

**Relevant components**

- SecureWallet API
- Wallet and Transaction Service
- Account Management Service
- Wallet Records
- Transaction Records
- Database

**Identified threats**

| Threat ID | Threat | Attack scenario |
|---|---|---|
| TH-07 | Attacker Initiates a Transfer as Another Customer | An attacker manipulates transfer details to impersonate another sender |
| TH-08 | Transfer Amount Manipulation | A customer modifies the transfer amount before submitting the request |
| TH-09 | Unauthorized Access to Another Customer's Wallet | A customer attempts to modify another customer's wallet |
| TH-10 | Overspending and Insufficient Balance Control | Concurrent transfers exploit inadequate balance checks or transaction handling |
| TH-14 | Unauthorized Wallet Funding | A customer manipulates a funding operation to increase a wallet balance improperly |
| TH-16 | SQL Injection | Crafted input is interpreted as part of a database query |

**Potential impact**

- Unauthorized balance changes.
- Financial integrity violations.
- Unauthorized modification of account information.
- Incorrect transaction records.
- Database data exposure or modification.

**Recommended security controls**

- Validate all sensitive values on the server.
- Never trust client-supplied sender identities or privileged attributes.
- Enforce wallet ownership checks.
- Reject zero, negative and invalid transaction amounts.
- Use appropriate decimal or integer-based monetary representation.
- Perform balance checks and updates atomically.
- Use SQLAlchemy ORM operations and parameterized queries safely.
- Prevent unauthorized direct modification of wallet balances.
- Record transaction outcomes and relevant security events.

**Verification approach**

Modify API requests to test transfer amounts, sender IDs, wallet IDs and funding values. Test negative values, zero values, excessive values, SQL injection payloads and concurrent transfers.

### 3.3 Repudiation

**Definition**

Repudiation occurs when a user denies performing an action and the application cannot provide sufficient evidence to establish what happened.

**How it applies to SecureWallet**

A customer may dispute a transfer or an administrator may deny performing a sensitive account-management operation.

If the application does not maintain reliable transaction and audit records, it may be difficult to establish who performed an operation and whether it succeeded.

**Relevant components**

- Wallet and Transaction Service
- Account Management Service
- Transaction Records
- Audit Logs

**Identified threats**

| Threat ID | Threat | Attack scenario |
|---|---|---|
| TH-11 | Customer Disputes a Transaction Due to Inadequate Audit Records | Incomplete transaction and audit records prevent the application from establishing who initiated a transfer or its outcome |
| TH-22 | Administrator Denies an Administrative Action | An administrator denies performing an account-management operation because reliable audit evidence is unavailable |

**Potential impact**

- Inability to investigate disputed transactions.
- Weak accountability for administrative actions.
- Incomplete incident investigations.
- Reduced confidence in transaction records.
- Difficulty identifying unauthorized activity.

**Recommended security controls**

- Assign unique identifiers to transactions.
- Record the initiating user's identity.
- Record transaction amounts and timestamps.
- Record transaction status and operation outcomes.
- Record relevant authentication and administrative events.
- Restrict access to audit records.
- Protect audit records against unauthorized modification.
- Avoid recording passwords, complete access tokens or other unnecessary secrets.

**Verification approach**

Perform successful and failed transactions and administrative operations. Confirm that the corresponding records contain the necessary identities, timestamps, operation details and outcomes.

Also verify that unauthorized users cannot modify audit records.

### 3.4 Information Disclosure

**Definition**

Information Disclosure occurs when sensitive information is exposed to individuals or systems that are not authorized to access it.

**How it applies to SecureWallet**

An attacker may exploit weak authorization, excessive API responses, database exposure or inadequate error handling to retrieve customer information, wallet balances, transaction histories or other sensitive data.

**Relevant components**

- SecureWallet API
- Authentication and Authorization
- Account Management Service
- Wallet and Transaction Service
- User Records
- Wallet Records
- Transaction Records
- Database

**Identified threats**

| Threat ID | Threat | Attack scenario |
|---|---|---|
| TH-04 | Account Enumeration Through Login Responses | An attacker compares authentication responses to identify registered customer email addresses |
| TH-09 | Unauthorized Access to Another Customer's Wallet | A customer retrieves another customer's wallet information |
| TH-12 | Unauthorized Access to Another Customer's Transaction History | A customer accesses transaction records belonging to another customer |
| TH-15 | Unauthorized Database Access | An attacker gains unauthorized database access and retrieves stored information |
| TH-17 | Exposed Database Backups | An attacker obtains an exposed database backup containing sensitive records |
| TH-23 | Customer Accesses Administrator Information | A customer accesses administrative information through an insufficiently protected endpoint |

**Potential impact**

- Exposure of customer information.
- Disclosure of wallet balances and transaction histories.
- Exposure of administrative information.
- Compromise of stored application data.
- Privacy violations and loss of customer trust.

**Recommended security controls**

- Enforce authentication and authorization on protected endpoints.
- Implement object-level authorization for wallet and transaction resources.
- Apply role-based access control to administrative functionality.
- Return only information required for each operation.
- Use generic authentication error messages.
- Protect database files and backups.
- Restrict database access.
- Avoid exposing sensitive internal information in API error responses.

**Verification approach**

Use customer and administrator accounts to test access to protected endpoints. Attempt to retrieve other customers' wallets, transaction histories and administrative information.

Review API responses, database permissions, backup locations and error messages for unintended disclosure.

### 3.5 Denial of Service

**Definition**

Denial of Service occurs when an attacker makes an application or its resources unavailable or significantly reduces their ability to serve legitimate users.

**How it applies to SecureWallet**

An attacker may overwhelm authentication, transaction or account-management endpoints through excessive requests or resource-intensive operations.

Poorly controlled database queries may also consume application resources and affect availability.

**Relevant components**

- SecureWallet API
- Authentication and Authorization
- Wallet and Transaction Service
- Account Management Service
- Database

**Identified threats**

| Threat ID | Threat | Attack scenario |
|---|---|---|
| TH-05 | Login Endpoint Abuse | Repeated authentication requests overwhelm the login endpoint or support password guessing |
| TH-13 | Transaction Endpoint Abuse | Repeated transaction requests degrade API availability |
| TH-18 | Expensive Queries Affect Application Availability | Resource-intensive queries consume database and application resources |
| TH-24 | Administrative Operation Abuse Affects Availability | Repeated administrative operations degrade application availability |

**Potential impact**

- Reduced API availability.
- Increased application response times.
- Database resource exhaustion.
- Failed legitimate customer requests.
- Disruption of wallet operations.

**Recommended security controls**

- Implement endpoint-specific rate limiting.
- Apply request validation and size restrictions.
- Use appropriate request and query timeouts.
- Implement pagination for transaction history.
- Restrict expensive operations.
- Monitor unusual request patterns.
- Apply sensible limits to administrative operations.

**Verification approach**

Perform controlled rate-limit tests, submit unusually large requests, test pagination and assess the application's response to repeated or resource-intensive operations.

Testing must remain within the controlled local environment.

### 3.6 Elevation of Privilege

**Definition**

Elevation of Privilege occurs when a user gains permissions or access beyond those assigned to their legitimate role.

**How it applies to SecureWallet**

A customer may attempt to become an administrator, modify their role, access privileged endpoints or manipulate authorization decisions.

An application or database account may also possess excessive permissions that increase the impact of a compromise.

**Relevant components**

- SecureWallet API
- Authentication and Authorization
- Account Management Service
- User Records
- Database

**Identified threats**

| Threat ID | Threat | Attack scenario |
|---|---|---|
| TH-03 | Manipulation of Token Claims | An attacker modifies JWT claims to obtain unauthorized privileges |
| TH-06 | Customer Manipulates Their Role to Become an Administrator | A customer submits a modified role field to obtain administrator privileges |
| TH-19 | Excessive Database Account Permissions | An application or compromised database account has permissions beyond its operational requirements |
| TH-21 | Unauthorized Privilege Modification | A customer attempts to change their own or another user's privileges |
| TH-25 | Customer Accesses an Administrator-Only Endpoint | An authenticated customer invokes an endpoint reserved for administrators |

**Potential impact**

- Unauthorized administrative access.
- Privilege escalation.
- Unauthorized customer account management.
- Unauthorized access to privileged functionality.
- Increased impact of application or database compromise.

**Recommended security controls**

- Implement server-side role-based access control.
- Apply default-deny authorization.
- Assign customer roles on the server during registration.
- Restrict privilege changes to authorized administrative operations.
- Prevent mass assignment of privileged fields.
- Validate JWT signatures and claims.
- Apply authorization checks to every administrative endpoint.
- Follow least-privilege principles for database accounts.

**Verification approach**

Attempt to modify role fields, manipulate JWT claims, access administrator-only endpoints using customer accounts and perform unauthorized privilege changes.

Review database permissions to confirm that application accounts cannot perform unnecessary privileged operations.

## 4. STRIDE Summary

The following table summarizes the six STRIDE categories and their corresponding threats.

| STRIDE category | Main security concern | Threat IDs |
|---|---|---|
| Spoofing | Impersonation of customers or administrators | TH-01, TH-02, TH-07, TH-20 |
| Tampering | Unauthorized modification of requests, balances and data | TH-07, TH-08, TH-09, TH-10, TH-14, TH-16 |
| Repudiation | Inability to establish accountability for actions | TH-11, TH-22 |
| Information Disclosure | Unauthorized access to sensitive information | TH-04, TH-09, TH-12, TH-15, TH-17, TH-23 |
| Denial of Service | Disruption or degradation of application availability | TH-05, TH-13, TH-18, TH-24 |
| Elevation of Privilege | Unauthorized access to higher-level permissions | TH-03, TH-06, TH-19, TH-21, TH-25 |

Some threats are associated with more than one STRIDE category because an attack may have multiple security consequences. The primary category is maintained in the threat register, while secondary categories are identified where relevant.

## 5. Relationship to the Threat Register

This STRIDE analysis supports the SecureWallet Threat Register.

The Threat Register provides the individual threat assessments, including:

- Threat descriptions and attack scenarios.
- Affected application components.
- Likelihood and impact ratings.
- Inherent risk scores.
- Recommended security controls.
- Verification evidence.
- Residual risk and implementation status.

The STRIDE analysis provides the broader classification and reasoning behind those identified threats.

Both documents should be updated when significant architecture changes, new functionality or security testing results introduce additional threats.

## 6. Assessment Limitations

This analysis represents a design-stage assessment of a fictional wallet API.

The threats describe potential attack scenarios based on the proposed architecture and functionality. They are not all confirmed vulnerabilities.

The effectiveness of the recommended controls must be established through implementation, code review and security testing.

## 7. Current Status

**STRIDE analysis:** Initial design-stage analysis completed.

**Threat coverage:** Six STRIDE categories assessed against the 25 threats in the SecureWallet Threat Register.

**Next activities:** Implement the identified security controls, perform security testing and update the threat assessments based on observed results.
