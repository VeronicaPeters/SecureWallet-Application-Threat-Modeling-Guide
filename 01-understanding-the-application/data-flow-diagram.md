# SecureWallet Data Flow Diagram (DFD)

| Document Attribute | Details |
|---|---|
| Project | SecureWallet |
| Document Type | Data Flow Diagram Documentation |
| Version | 1.0 |
| Status | Diagram exported; documentation complete |
| Modelling Tool | OWASP Threat Dragon |
| Threat Modelling Methodology | STRIDE |
| Related Documents | System Architecture, Threat Register, STRIDE Analysis, Security Mitigations, Security Test Plan |

---

## 1. Purpose

This document describes the SecureWallet Data Flow Diagram (DFD).

The DFD provides a visual representation of how information moves through the application, which components process that information, where application data is stored, and where trust boundaries exist.

It supports the application's threat-modelling process by identifying points where data enters the system, crosses security boundaries, is processed, and is stored.

The diagram is modelled using OWASP Threat Dragon and supports the STRIDE threat analysis documented separately.

## 2. DFD Scope

The diagram represents the logical data flows associated with the SecureWallet application.

It covers:

- External users interacting with the API.
- Authentication and authorization.
- Customer account management.
- Wallet and transaction processing.
- Application data stores.
- Audit logging.
- The developer's source-code development and security-scanning workflow.

The diagram represents the intended application architecture. It does not claim that all components or security controls have already been implemented.

## 3. External Entities

External entities represent people or systems outside the application boundary that interact with SecureWallet.

| Entity | Description | Main Interaction |
|---|---|---|
| Visitor | An individual who has not authenticated | Registration and login requests |
| Customer | An authenticated standard user | Wallet operations, transfers, and transaction history |
| Administrator | An authenticated privileged user | Authorized customer account management |
| Developer | Individual responsible for developing and maintaining the application | Source-code changes and development activities |

The Developer is included to represent the development-side security workflow. The developer does not directly access the running application through the IDE process shown in the diagram.

## 4. Application Processes

Processes represent components that receive, validate, or transform information.

### 4.1 SecureWallet API

The main application entry point.

Responsibilities:

- Receives requests from visitors, customers, and administrators.
- Validates incoming request structures.
- Routes requests to the relevant application services.
- Returns responses to clients.
- Enforces application-level request handling and security requirements.

### 4.2 Authentication and Authorization

Responsible for verifying user identity and determining whether the requested operation is permitted.

Responsibilities:

- Validates customer credentials.
- Issues authentication tokens after successful authentication.
- Validates incoming access tokens.
- Determines the authenticated user and their permissions.
- Supports role-based and ownership-based access decisions.
- Records relevant authentication and authorization events.

Authentication answers: **Who are you?**

Authorization answers: **What are you allowed to do?**

### 4.3 Account Management Service

Responsible for customer account operations and authorized administrative account management.

Responsibilities:

- Creates and manages customer account records.
- Processes permitted account updates.
- Supports authorized administrator operations.
- Enforces account-related business rules.
- Records relevant account-management events.

### 4.4 Wallet and Transaction Service

Responsible for wallet operations and transaction processing.

Responsibilities:

- Creates and retrieves wallet information.
- Retrieves wallet balances.
- Processes simulated wallet funding.
- Processes transfers between customers.
- Retrieves transaction history.
- Maintains transaction and balance integrity.
- Records relevant wallet and transaction events.

### 4.5 Developer IDE / Security Scanning Process

Represents the developer's local development environment and its security-scanning capability.

Responsibilities:

- Supports application source-code development.
- Allows security analysis of application code.
- Produces code-analysis findings for developer review.

The IDE and scanning workflow is separate from the running SecureWallet API. The diagram does not represent the developer as an application user or the scanner as an API runtime component.

## 5. Data Stores

Data stores represent information retained by the application.

| Data Store | Description | Example Information |
|---|---|---|
| User Records | Customer identity and account information | User identifiers, account details, password hashes, assigned roles |
| Wallet Records | Current wallet state | Wallet identifiers, owner identifiers, current balances |
| Transaction Records | Historical wallet and transaction events | Transaction identifiers, sender, recipient, amount, transaction status, timestamps |
| Audit Logs | Security and accountability records | Authentication events, authorization events, account changes, administrative actions, transaction events |

### 5.1 Wallet Records vs. Transaction Records

These are separate logical data stores because they serve different purposes.

**Wallet Records** represent the current state of a wallet, including its available balance.

**Transaction Records** represent the history of operations that affect wallets.

For example, after a successful transfer:

- The sender's Wallet Record reflects the reduced balance.
- The recipient's Wallet Record reflects the increased balance.
- The Transaction Record captures the transfer details.

The records are related but are not interchangeable. Keeping transaction history separate supports traceability, reconciliation, and investigation.

The logical separation does not require separate physical database servers. The records may be implemented as separate tables within the same database.

## 6. Trust Boundaries

Trust boundaries identify points where information crosses between environments with different levels of trust.

The diagram contains two primary trust boundaries.

### 6.1 External Client - Application Trust Boundary

This boundary separates external clients from the SecureWallet application.

It includes interactions from:

- Visitor to SecureWallet API.
- Customer to SecureWallet API.
- Administrator to SecureWallet API.

Data crossing this boundary must be treated as untrusted.

Security considerations include:

- Authentication and token validation.
- Request validation.
- Authorization checks.
- Protection against injection and malformed input.
- Enforcement of object ownership.
- Secure communication over HTTPS in a deployed environment.

A valid request format does not automatically mean that the requester is authorized to perform the requested operation.

### 6.2 Restricted Database Trust Boundary

This boundary separates application processing components from persistent data stores.

Security considerations include:

- Restricting database access to authorized application components.
- Using safe ORM operations or parameterized queries.
- Applying least-privilege database permissions.
- Preventing unauthorized record access or modification.
- Maintaining data integrity.
- Protecting sensitive information.
- Recording security-relevant operations.

The application must not trust client-controlled identifiers or values simply because they are passed to a database operation.

## 7. Data Flow Descriptions

This section explains the named data flows represented in the DFD.

### 7.1 External Client and API Flows

| Source | Destination | Data Flow | Purpose |
|---|---|---|---|
| Visitor | SecureWallet API | Registration Request / Login Request | Submits registration details or authentication credentials |
| SecureWallet API | Visitor | Registration Response / Authentication Response | Returns the outcome of registration or authentication |
| Customer | SecureWallet API | Authenticated API Request | Requests an authorized wallet, account, or transaction operation |
| SecureWallet API | Customer | API Response | Returns the result of the requested operation |
| Administrator | SecureWallet API | Administrative Request | Requests an administrative operation |
| SecureWallet API | Administrator | Administrative Response | Returns the result of the administrative operation |

### 7.2 API and Authentication / Authorization Flows

| Source | Destination | Data Flow | Purpose |
|---|---|---|---|
| SecureWallet API | Authentication and Authorization | Authentication / Authorization Request | Requests identity verification or permission evaluation |
| Authentication and Authorization | SecureWallet API | Authentication / Authorization Result | Returns the authentication or authorization decision |
| Authentication and Authorization | User Records | User Lookup Request | Retrieves account information required for authentication or authorization |
| User Records | Authentication and Authorization | User Account Data | Returns relevant stored account information |

The API relies on the authentication and authorization process to determine the identity and permissions associated with a request.

The client must not be allowed to establish its own trusted identity or privilege level by submitting arbitrary user identifiers or role values.

### 7.3 Account Management Flows

| Source | Destination | Data Flow | Purpose |
|---|---|---|---|
| SecureWallet API | Account Management Service | Account Operation Request | Requests an account-related operation |
| Account Management Service | User Records | Account Data Operation | Creates, retrieves, or updates authorized account information |
| User Records | Account Management Service | Account Data Result | Returns the outcome or relevant account data |
| Account Management Service | SecureWallet API | Account Operation Result | Returns the account operation result |

Account-related operations must enforce authorization before sensitive records are accessed or changed.

### 7.4 Wallet and Transaction Flows

| Source | Destination | Data Flow | Purpose |
|---|---|---|---|
| SecureWallet API | Wallet and Transaction Service | Wallet / Transaction Request | Requests a wallet or transaction operation |
| Wallet and Transaction Service | Wallet Records | Wallet Data Operation | Retrieves or updates wallet information |
| Wallet Records | Wallet and Transaction Service | Wallet Data Result | Returns the current wallet data |
| Wallet and Transaction Service | Transaction Records | Transaction Data Operation | Creates or retrieves transaction records |
| Transaction Records | Wallet and Transaction Service | Transaction Data Result | Returns transaction information or operation results |
| Wallet and Transaction Service | SecureWallet API | Wallet / Transaction Result | Returns the result of the requested operation |

The wallet and transaction service must preserve the relationship between wallet balances and transaction records.

Transfers must be processed consistently so that a partial failure cannot leave wallet balances in an incorrect state.

### 7.5 Audit Logging Flows

| Source | Destination | Data Flow | Purpose |
|---|---|---|---|
| Authentication and Authorization | Audit Logs | Authentication and Authorization Events | Records relevant authentication and access-control activity |
| Account Management Service | Audit Logs | Account Audit Events | Records account changes and administrative operations |
| Wallet and Transaction Service | Audit Logs | Transaction Audit Events | Records wallet funding and transaction activity |

Audit events should contain sufficient context to support investigation and accountability without storing passwords, full access tokens, or unnecessary sensitive information.

### 7.6 Development and Security Scanning Flows

| Source | Destination | Data Flow | Purpose |
|---|---|---|---|
| Developer | Developer IDE / Security Scanning Process | Source Code Changes | Developer creates or modifies application code |
| Developer IDE / Security Scanning Process | Developer | Code Analysis Results | Returns security scanning findings for developer review |

This workflow represents security checks performed during development.

It is separate from the application's runtime request-processing flows.

## 8. Security Considerations Identified by the DFD

The DFD helps identify security concerns at different stages of data movement.

| DFD Area | Security Concern | Relevant Threats |
|---|---|---|
| External Client / API Boundary | Credential theft and unauthorized requests | TH-01, TH-02, TH-05 |
| Authentication and Authorization | Token manipulation and privilege escalation | TH-03, TH-06, TH-20, TH-21 |
| Account Management | Unauthorized account access and administrative actions | TH-06, TH-21, TH-22, TH-23, TH-25 |
| Wallet and Transaction Processing | Amount manipulation, unauthorized transfers, and overspending | TH-07, TH-08, TH-10, TH-14 |
| Wallet Records | Unauthorized access or modification of wallet balances | TH-09, TH-10 |
| Transaction Records | Unauthorized transaction history access and inadequate traceability | TH-11, TH-12 |
| Database Boundary | Injection, unauthorized database access, and excessive privileges | TH-15, TH-16, TH-19 |
| Audit Logs | Missing or insufficient accountability records | TH-11, TH-22 |
| Resource-intensive Operations | Excessive requests and application availability risks | TH-05, TH-13, TH-18, TH-24 |

The DFD is a foundation for threat identification. The threat register and STRIDE analysis provide the detailed attack scenarios, risk ratings, and security controls.

## 9. DFD Modelling Assumptions

The following assumptions apply to this diagram:

1. SecureWallet is a simulated wallet application and does not process real money.
2. The application exposes functionality through a REST API.
3. Authentication and authorization are represented as a logical process.
4. Account management and wallet/transaction processing are represented as separate logical services.
5. User, wallet, transaction, and audit information are represented as separate logical data stores.
6. These logical data stores may be implemented within one SQLite database.
7. The application is responsible for validating client input and enforcing authorization.
8. The Developer IDE represents development activity and is not part of the running application.
9. The diagram represents the intended logical design and may evolve as implementation decisions are finalized.

## 10. Relationship to STRIDE

The DFD provides the system elements and data flows used during STRIDE threat analysis.

Each major process, data store, external entity, and trust boundary provides a point for considering potential threats.

| STRIDE Category | Example DFD Security Concern |
|---|---|
| Spoofing | An attacker impersonates a customer or administrator |
| Tampering | An attacker modifies transfer amounts or wallet records |
| Repudiation | A user denies performing a transaction or administrative operation |
| Information Disclosure | A customer accesses another customer's wallet or transaction information |
| Denial of Service | Repeated or expensive requests affect application availability |
| Elevation of Privilege | A customer obtains administrator-level access |

The detailed threat scenarios and associated controls are maintained in `../threat-model/stride-analysis.md` and `../threat-model/threat-register.md`.

## 11. Diagram Export

The DFD was created in OWASP Threat Dragon.

**Diagram image:**
![SecureWallet Data Flow Diagram](securewallet-dfd.png)

## 12. Current Status

The logical DFD structure and its supporting documentation have been defined.

The diagram includes:

- Four external entities.
- Five logical processes, including the development and scanning workflow.
- Four logical data stores.
- Two primary trust boundaries.
- Client, authentication, account-management, wallet, transaction, audit, and development data flows.
