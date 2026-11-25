# SecureWallet – System Architecture

**Project:** SecureWallet API
**Document:** System Architecture
**Version:** 1.0
**Assessment stage:** Design and implementation
**Date:** 22 November 2026

## 1. Overview

SecureWallet is a fictional digital wallet application developed as an Application Security portfolio project.

The application provides REST API functionality for customer registration, authentication, wallet management, simulated wallet funding, transfers between customers and transaction history.

The project is designed to demonstrate secure application development, API security, threat modelling and practical security testing.

SecureWallet is a controlled learning environment. It does not connect to real banking infrastructure or process real financial transactions.

## 2. Architecture Objectives

The architecture is designed around the following security objectives:

- Protect customer credentials and account information.
- Authenticate users before granting access to protected functionality.
- Enforce authorization on the server side.
- Prevent customers from accessing other customers' wallets and transaction records.
- Protect wallet balances and transaction integrity.
- Maintain reliable records of financial operations and administrative actions.
- Reduce exposure to common API and application security vulnerabilities.
- Support security testing throughout development.

## 3. Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| Backend framework | FastAPI | Provides REST API endpoints |
| Programming language | Python | Application development |
| Database | SQLite | Stores application records during local development |
| ORM | SQLAlchemy | Database interaction and data management |
| Authentication | JWT | Represents authenticated user sessions |
| Password hashing | Argon2id | Secure password hashing |
| Testing framework | Pytest | Automated application and security testing |
| API testing | Postman | Manual API testing |
| Threat modelling | OWASP Threat Dragon | Data flow and threat modelling |
| Version control | Git | Tracks source code and documentation |

## 4. High-Level Architecture

SecureWallet follows a layered application architecture.

The main components are:

### 4.1 Client

The client represents an external application or user interacting with SecureWallet.

It sends HTTP requests to the API and receives responses.

The client must not be trusted to determine its own identity, privileges, wallet ownership or transaction permissions.

### 4.2 SecureWallet API

The API is the main entry point into the application.

It receives client requests, validates incoming data and coordinates the appropriate application functionality.

It exposes operations for:

- Customer registration.
- Customer login.
- Wallet creation.
- Wallet balance retrieval.
- Simulated wallet funding.
- Customer-to-customer transfers.
- Transaction history retrieval.
- Administrative account management.

### 4.3 Authentication and Authorization

This component verifies user identity and determines whether the authenticated user is permitted to perform a requested operation.

Authentication uses securely hashed passwords and signed JWT access tokens.

Authorization is enforced server-side through role-based access control and object-level ownership checks.

A valid token alone does not grant permission to access every resource.

### 4.4 Account Management Service

This component manages customer account information and administrative account operations.

It is responsible for appropriate access restrictions when creating, retrieving or modifying account information.

Privileged account operations must be restricted to authorized administrators.

### 4.5 Wallet and Transaction Service

This component handles wallet-related functionality and transaction processing.

Its responsibilities include:

- Creating and managing wallet records.
- Retrieving wallet balances.
- Processing simulated funding operations.
- Initiating transfers between customers.
- Recording transaction details.
- Enforcing balance and transaction integrity.

Financial operations must be validated and processed safely to prevent unauthorized balance changes and overspending.

### 4.6 Database

SQLite provides persistent storage during local development. SQLAlchemy is used to interact with the database.

The main logical data stores are:

| Data store | Purpose |
|---|---|
| User Records | Customer account details, password hashes and assigned roles |
| Wallet Records | Wallet ownership and current wallet balances |
| Transaction Records | Historical funding and transfer activities |
| Audit Logs | Relevant authentication, administrative and security events |

Wallet Records represent the current state of a wallet, while Transaction Records preserve the history of financial operations. They serve different purposes and should remain logically distinct.

## 5. Authentication and Authorization Flow

1. A customer submits registration details.
2. The application validates the submitted information.
3. The customer's password is hashed using Argon2id before storage.
4. The customer submits login credentials.
5. The application verifies the submitted password against the stored password hash.
6. Following successful authentication, the application issues a signed JWT access token.
7. The client presents the token when accessing protected endpoints.
8. The application validates the token signature, expiry and required claims.
9. The application checks the user's permissions before performing the requested operation.

The application must never rely on a client-supplied user ID or role to establish authorization.

## 6. Wallet and Transaction Processing

Wallet operations must be controlled by server-side business logic.

For a transfer:

1. The authenticated customer initiates a transfer request.
2. The application identifies the sender from the authenticated context.
3. The application validates the destination wallet and verifies the required ownership and access conditions.
4. The transfer amount is validated.
5. The application checks that the sender has sufficient funds.
6. The debit and credit operations are performed within an appropriate database transaction.
7. The transaction outcome is recorded.
8. The application returns the result to the client.

The transfer implementation must prevent partial updates, unauthorized transfers and overspending caused by concurrent requests.

## 7. Trust Boundaries

The architecture contains two primary trust boundaries.

### 7.1 External Client and Application Boundary

This separates external clients from the SecureWallet application.

All incoming requests must be treated as untrusted.

Security controls include:

- Input validation.
- Authentication.
- Authorization.
- Request-size restrictions.
- Rate limiting.
- Secure error handling.

### 7.2 Application and Database Boundary

This separates application processing from persistent data storage.

Database operations must be performed through controlled application components.

Security controls include:

- Restricted database access.
- Least-privilege permissions.
- Safe SQLAlchemy operations.
- Protection of sensitive records.
- Appropriate transaction handling.

## 8. Security Design Considerations

The following principles guide the implementation:

| Security principle | Application |
|---|---|
| Least privilege | Users and application components receive only the permissions required |
| Defense in depth | Multiple security controls protect sensitive functionality |
| Server-side authorization | Permissions are checked by the application, not trusted to the client |
| Secure password storage | Passwords are hashed using Argon2id |
| Secure session handling | JWT signatures, expiry and required claims are validated |
| Input validation | Incoming data is validated before processing |
| Object-level authorization | Customers can access only resources they are authorized to use |
| Financial integrity | Wallet updates and transfers are processed safely |
| Auditability | Relevant security and administrative events are recorded |
| Secure error handling | Responses avoid exposing sensitive internal information |

## 9. Development and Security Testing

The application will be developed and tested in a controlled local environment.

Security testing will cover:

- Authentication and token validation.
- Broken access control and BOLA/IDOR.
- Privilege escalation.
- Input validation and SQL injection.
- Wallet balance manipulation.
- Unauthorized funding and transfers.
- Concurrent transaction handling.
- Rate limiting and denial-of-service resilience.
- Audit logging.
- Static code analysis and dependency security.

Automated tests will be implemented using Pytest, supported by manual API testing with Postman and appropriate security analysis tools.

## 10. Architecture Scope and Limitations

This architecture describes a local development implementation of a fictional wallet API.

The initial implementation uses SQLite for simplicity and reproducibility.

It does not represent a production banking architecture and does not include real payment-provider integrations, external settlement systems or real-money processing.

Additional controls and architectural changes may be required before a system of this nature could be deployed in a production financial environment.

## 11. Architecture Status

The architecture provides the design foundation for SecureWallet development.

The implementation will be assessed continuously against the documented security requirements, threat register and security test plan.
