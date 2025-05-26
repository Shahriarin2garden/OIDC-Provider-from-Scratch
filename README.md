# 🔐 Flask OIDC Provider

**A Robust, Standards‑Compliant OpenID Connect (OIDC) Provider Built with Flask**

---

## 📑 Table of Contents

1. [Introduction](#🚀-1-introduction)
2. [Task Description](#🎯-2-task-description)
3. [Step-by-Step Walkthrough](#🛠️-3-step-by-step-walkthrough)

   * [3.1 Clone & Set Up the Project](#31-clone--set-up-the-project)
   * [3.2 Environment Configuration](#32-environment-configuration)
   * [3.3 Generate RSA Keys and JWKS](#🔑-33-generate-rsa-keys-and-jwks)
   * [3.4 Start the OIDC Provider](#▶️-34-start-the-oidc-provider)
   * [3.5 Dynamic Client Registration](#🤝-35-dynamic-client-registration)
   * [3.6 Authorization Code Flow with PKCE](#🔄-36-authorization-code-flow-with-pkce)
   * [3.7 Run Automated Tests](#✅-37-run-automated-tests)
   * [3.8 API Usage Examples](#📡-38-api-usage-examples)
   * [3.9 API Endpoint Reference](#📋-39-api-endpoint-reference)
4. [Additional Sections](#🔒-4-additional-sections)

   * [4.1 Security Considerations](#⚙️-41-security-considerations)
   * [4.2 Troubleshooting Tips](#🐞-42-troubleshooting-tips)
   * [4.3 Architecture Overview](#🏛️-43-architecture-overview)
5. [Conclusion](#🎉-5-conclusion)

---

## 🚀 1. Introduction

Welcome to the **Flask OIDC Provider** lab. In this tutorial, you will build a fully featured OpenID Connect (OIDC) Provider from the ground up using Python and Flask. By following these instructions, you will:

* Understand core concepts of OAuth 2.0 and OIDC.
* Configure secure token issuance and key management.
* Implement PKCE-enhanced authorization flows.
* Register and manage clients dynamically.
* Protect and expose user information via standardized endpoints.

This provider is designed for production readiness, offering modular storage backends, configurable security policies, and automated testing—ensuring your implementation is both scalable and maintainable.

### 📖 1.1 What is OAuth 2.0?

OAuth 2.0 is an industry-standard framework for delegated authorization. It decouples resource owners (users) from clients (applications), allowing users to grant limited access without sharing credentials. Key components include:

* **Authorization Server**: Issues tokens after authenticating the user.
* **Resource Server**: Hosts protected APIs, validating tokens.
* **Client**: Third-party application requesting access.
* **Grant Types**: Methods clients use to obtain tokens (e.g., authorization code, client credentials, implicit, resource owner password).

### 🔍 1.2 What is OpenID Connect?

OpenID Connect (OIDC) extends OAuth 2.0 by introducing an identity layer. It standardizes how clients verify user identity and obtain user profile information via JSON Web Tokens (JWTs). Core additions include:

* **ID Token**: JWT containing authentication information and user claims.
* **UserInfo Endpoint**: Returns additional user profile data.
* **Discovery Endpoint**: Provides metadata about supported flows and endpoints.
* **JWKS Endpoint**: Publishes public keys for token verification.

### ⚖️ 1.3 Key Differences

| Feature     | OAuth 2.0                | OpenID Connect (OIDC)      |
| ----------- | ------------------------ | -------------------------- |
| Purpose     | Authorization            | Authentication + Identity  |
| Token Types | Access Token             | Access + ID Token          |
| Metadata    | Not standardized         | Standardized discovery API |
| Format      | Any opaque token (often) | JSON Web Token (JWT)       |

### 🎁 1.4 Benefits of OpenID Connect

* **Unified Protocol**: Single framework for authN and authZ.
* **Interoperability**: Widely supported across platforms.
* **Scalability**: Stateless JWTs reduce server load.
* **Extensibility**: Hooks for custom claims, session management, and more.

---

## 🎯 2. Task Description

In this lab, your goal is to implement a production-ready OIDC Provider with the following capabilities:

1. **PKCE-Enhanced Authorization Code Flow** (`auth/pkce.py`)

   * Protect confidential and public clients against interception attacks.
2. **RS256 JWT Issuance & JWKS Endpoint** (`auth/token.py`)

   * Sign tokens with RSA keys and expose public keys via `/jwks.json`.
3. **Dynamic Client Registration** (`/register` in `auth/registration.py`)

   * Allow third-party apps to self-register, receive credentials, and configure redirect URIs.
4. **Token Management Endpoints**

   * `/token`: Issue access, ID, and refresh tokens.
   * `/introspect`: Validate token status and metadata.
   * `/revoke`: Invalidate tokens on demand.
5. **UserInfo Endpoint** (`/userinfo` in `auth/token.py`)

   * Return standardized user claims (e.g., `sub`, `email`, `name`).
6. **Modular Storage Backends**

   * **In-memory** (`store/memory.py`) for quick prototyping.
   * **Redis** (`store/redis_store.py`) for production-scale and persistence.

**Directory Structure Overview:**

```text
flask-oidc-provider/
├── app.py                    # Initializes Flask app and registers routes
├── config.py                 # Loads environment variables and defaults
├── models.py                 # Defines data models for Client, Code, Token
├── auth/                     # Core authentication logic and endpoints
│   ├── pkce.py               # PKCE utilities: challenge & verifier handling
│   ├── token.py              # JWT creation, introspection, userinfo logic
│   └── registration.py       # Dynamic client registration implementation
├── store/                    # Storage layer options
│   ├── memory.py             # Volatile, in-memory store for quick tests
│   └── redis_store.py        # Redis-backed store for persistence
├── templates/                # User-facing HTML templates
│   ├── login.html            # Secure login UI
│   └── consent.html          # User consent and scope grant UI
├── static/                   # Front-end assets (CSS, JS)
├── keys/                     # RSA key pair management
│   ├── private.pem           # Private key for signing JWTs
│   └── public.pem            # Public key for JWKS endpoint
├── jwks.json                 # Auto-generated JSON Web Key Set
├── requirements.txt          # Python dependencies
├── Dockerfile                # Container instructions
├── docker-compose.yml        # Compose file for app + Redis
├── .env.example              # Sample environment variables
├── tests/                    # Automated tests with ≥80% coverage
│   ├── test_flow.py          # End-to-end authorization flow tests
│   └── test_endpoints.py     # Unit tests for error and edge cases
├── .github/                  # CI/CD workflows (GitHub Actions)
│   └── workflows/ci.yml
├── CONTRIBUTING.md           # Contribution guidelines
├── CODE_OF_CONDUCT.md        # Community standards
└── README.md                 # Project documentation (this file)
```

**Expected Deliverables:**

* **Draw\.io diagrams** placed in `/docs/*.svg` illustrating data flows.
* **Annotated screenshots** under `/docs/screenshots/` highlighting key UI interactions.
* **Automated test suite** in `tests/` with coverage reports in `/reports`.

---

## 🛠️ 3. Step-by-Step Walkthrough

Follow these detailed instructions to set up, configure, and run your OIDC Provider:

### 3.1 Clone & Set Up the Project

1. **Clone the repository**:

   ```bash
   git clone https://github.com/your-org/flask-oidc-provider.git
   cd flask-oidc-provider
   ```
2. **Create and activate a virtual environment**:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate   # On Windows: .venv\Scripts\activate
   ```
3. **Install dependencies**:

   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

### 3.2 Environment Configuration

1. **Copy example environment file**:

   ```bash
   cp .env.example .env
   ```
2. \*\*Open \*\*\`\` and configure the following variables:

   ```ini
   FLASK_ENV=development         # Switch to production in live environments
   ISSUER_URL=http://localhost:5000
   SECRET_KEY=<your-secret>      # Used for session management and CSRF
   PRIVATE_KEY_PATH=keys/private.pem
   PUBLIC_KEY_PATH=keys/public.pem
   REDIS_URL=redis://localhost:6379/0  # Optional: remove for in-memory mode
   ```
3. **Tips:**

   * Use strong, randomly generated `SECRET_KEY`.
   * In production, secure `.env` and avoid checking it into Git.

### 🔑 3.3 Generate RSA Keys and JWKS

1. **Generate RSA key pair** (2048-bit recommended):

   ```bash
   openssl genrsa -out keys/private.pem 2048
   openssl rsa -in keys/private.pem -pubout -out keys/public.pem
   ```
2. **Generate JWKS** via the application:

   ```bash
   python app.py --generate-jwks
   ```
3. \*\*Confirm \*\*\`\` is populated with key IDs (`kid`) and algorithms.

### ▶️ 3.4 Start the OIDC Provider

* **In‑Memory Mode (default)**:

  ```bash
  flask run --host=0.0.0.0 --port=5000
  ```

  * Fast startup, ideal for development.

* **With Redis Backend**:

  ```bash
  docker-compose up --build
  ```

  * Redis provides durable storage of codes, tokens, and client data.

**Verify service**:

```bash
curl http://localhost:5000/.well-known/openid-configuration
```

Expect JSON with endpoints such as `authorization_endpoint`, `token_endpoint`, etc.

### 🤝 3.5 Dynamic Client Registration

Programmatically register clients with your provider:

```bash
curl -X POST http://localhost:5000/register \
  -H 'Content-Type: application/json' \
  -d '{
    "client_name": "spa-app",
    "redirect_uris": ["http://localhost:3000/callback"],
    "grant_types": ["authorization_code"],
    "response_types": ["code"],
    "scope": "openid profile email"
  }'
```

**Response (201 Created)**:

```json
{
  "client_id": "abc123",
  "client_secret": "shh-its-a-secret",
  "redirect_uris": ["http://localhost:3000/callback"]
}
```

### 🔄 3.6 Authorization Code Flow with PKCE

1. **Initiate authorization request** (client/browser):

   ```http
   GET /authorize?response_type=code
     &client_id=abc123
     &redirect_uri=http://localhost:3000/callback
     &scope=openid%20email%20profile
     &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
     &code_challenge_method=S256
   ```
2. **User Login & Consent**:

   * User enters credentials on `login.html`.
   * Upon success, user reviews scopes in `consent.html` and accepts.
3. **Exchange code for tokens**:

   ```bash
   curl -X POST http://localhost:5000/token \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -d 'grant_type=authorization_code'
     -d 'code=<auth_code>'
     -d 'redirect_uri=http://localhost:3000/callback'
     -d 'client_id=abc123'
     -d 'code_verifier=<original_verifier>'
   ```
4. **Inspect tokens**:

   * `access_token`: Use for API calls.
   * `id_token`: JWT with user identity claims.
   * `refresh_token`: Obtain new access tokens.
5. **Access protected resource**:

   ```bash
   curl -H 'Authorization: Bearer <access_token>' http://localhost:5000/userinfo
   ```

### ✅ 3.7 Run Automated Tests

Execute full test suite:

```bash
pytest --maxfail=1 --disable-warnings -q
```

* **Coverage report**: `/reports/index.html` provides line-by-line insights.
* Ensure coverage ≥80% before merging to `main`.

### 📡 3.8 API Usage Examples

Below are extended examples demonstrating common interactions with your OIDC Provider. Replace placeholders (`<...>`) with actual values as needed:

```bash
# 1) Fetch OIDC Discovery Metadata
curl http://localhost:5000/.well-known/openid-configuration

# 2) Fetch JWKS to verify token signatures
curl http://localhost:5000/jwks.json

# 3) Register a new client dynamically
curl -X POST http://localhost:5000/register \
  -H 'Content-Type: application/json' \
  -d '{
      "client_name": "mobile-app",
      "redirect_uris": ["com.myapp://callback"],
      "grant_types": ["authorization_code", "refresh_token"],
      "response_types": ["code"],
      "scope": "openid profile email"
  }'

# 4) Initiate Authorization Code Flow with PKCE
curl -X GET "http://localhost:5000/authorize?response_type=code\
  &client_id=<client_id>\
  &redirect_uri=<redirect_uri>\
  &scope=openid%20email%20profile\
  &code_challenge=<code_challenge>\
  &code_challenge_method=S256"

# 5) Exchange Authorization Code for Tokens
curl -X POST http://localhost:5000/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=authorization_code' \
  -d 'code=<authorization_code>' \
  -d 'redirect_uri=<redirect_uri>' \
  -d 'client_id=<client_id>' \
  -d 'code_verifier=<code_verifier>'

# 6) Refresh Access Token using Refresh Token
curl -X POST http://localhost:5000/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=refresh_token' \
  -d 'refresh_token=<refresh_token>' \
  -d 'client_id=<client_id>' \
  -d 'client_secret=<client_secret>'

# 7) Introspect an Access Token
curl -X POST http://localhost:5000/introspect \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -u '<client_id>:<client_secret>' \
  -d 'token=<access_token>'

# 8) Revoke a Refresh Token or Access Token
curl -X POST http://localhost:5000/revoke \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -u '<client_id>:<client_secret>' \
  -d 'token=<refresh_token>'

# 9) Retrieve UserInfo Claims
curl -H 'Authorization: Bearer <access_token>' \
     http://localhost:5000/userinfo
```

### 📋 3.9 API Endpoint Reference

| Method | Path                                | Description                 | Auth               |
| ------ | ----------------------------------- | --------------------------- | ------------------ |
| POST   | `/register`                         | Dynamic client registration | n/a                |
| GET    | `/.well-known/openid-configuration` | OIDC discovery metadata     | n/a                |
| GET    | `/jwks.json`                        | JSON Web Key Set            | n/a                |
| GET    | `/authorize`                        | Begin login & consent flow  | n/a                |
| POST   | `/authorize`                        | Submit user credentials     | n/a                |
| POST   | `/consent`                          | Submit user consent         | Session cookie     |
| POST   | `/token`                            | Exchange code for tokens    | Basic/Auth or body |
| POST   | `/revoke`                           | Revoke tokens               | Basic/Auth or body |
| POST   | `/introspect`                       | Introspect token validity   | Basic/Auth         |
| GET    | `/userinfo`                         | Retrieve user claims        | Bearer Token       |

---

### 📢 3.10 API Endpoint Sample Responses

Below are the detailed JSON responses for each endpoint shown above:

#### `/register` (201 Created)

```json
{
  "client_id": "abc123",
  "client_secret": "shh-its-a-secret",
  "redirect_uris": ["http://localhost:3000/callback"],
  "grant_types": ["authorization_code"],
  "response_types": ["code"],
  "scope": "openid profile email"
}
```

---

#### `/.well-known/openid-configuration` (200 OK)

```json
{
  "issuer": "http://localhost:5000",
  "authorization_endpoint": "http://localhost:5000/authorize",
  "token_endpoint": "http://localhost:5000/token",
  "userinfo_endpoint": "http://localhost:5000/userinfo",
  "jwks_uri": "http://localhost:5000/jwks.json",
  "scopes_supported": ["openid","profile","email"],
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code","refresh_token"]
}
```

---

#### `/jwks.json` (200 OK)

```json
{
  "keys": [
    {"kty": "RSA", "use": "sig", "kid": "2025-05-27-001", "alg": "RS256", "n": "...modulus...", "e": "AQAB"}
  ]
}
```

---

#### `/authorize` (302 Redirect)

* **Successful**: Redirects to `?code=<auth_code>`.
* **Error** example:

```json
{
  "error": "access_denied",
  "error_description": "User denied consent"
}
```

---

#### `/token` (200 OK)

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "def456",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

---

#### `/introspect` (200 OK)

```json
{
  "active": true,
  "scope": "openid profile email",
  "client_id": "abc123",
  "username": "jdoe",
  "token_type": "access_token",
  "exp": 1716912000
}
```

---

#### `/revoke` (200 OK)

* **Successful**: HTTP 200 with empty body.

---

#### `/userinfo` (200 OK)

```json
{
  "sub": "user-123",
  "email": "jdoe@example.com",
  "name": "John Doe",
  "email_verified": true
}
```

---

## 🔒 4. Additional Sections

### ⚙️ 4.1 Security Considerations

* **Token Rotation & Replay Prevention**:

  * Issue one-time-use refresh tokens.
  * Maintain nonce tracking to prevent replay attacks.
* **RSA Key Lifecycle**:

  * Securely store keys in dedicated vaults (e.g., AWS KMS).
  * Rotate keys periodically and update JWKS accordingly.
* **Input Validation & Sanitization**:

  * Validate all query parameters and JSON payloads.
  * Use strict content security policies to prevent XSS.

### 🐞 4.2 Troubleshooting Tips

* **Common Errors**:

  * 400 Bad Request: Missing or malformed parameters.
  * 401 Unauthorized: Invalid client credentials or expired tokens.
  * 500 Internal Server Error: Inspect `app.log` for stack traces.
* **Debugging Tools**:

  * Use Postman or HTTPie for step-by-step request inspection.
  * Enable Flask debug mode during development (never in prod).

### 🏛️ 4.3 Architecture Overview

Below is a high-level overview of the data and control flow:

```
+-------------------+      +---------------------+
|    OIDC Client    | ---> | /authorize Endpoint |
+-------------------+      +---------------------+
        |                           |
        v                           v
   [Login UI]                [Consent Page]
        |                           |
        v                           v
+-------------------+      +---------------------+
| Authorization Code| ---> | /token Endpoint     |
+-------------------+      +---------------------+
        |                           |
        v                           v
 Access + ID + Refresh         /userinfo Endpoint
       Tokens
```

Each arrow represents an HTTP interaction secured via TLS.

#### 📈 Diagrams

1. **OIDC Sequence Diagram** (SVG)

   ```markdown
   ![OIDC Sequence Diagram](docs/diagrams/oidc-sequence.svg)
   ```

2. **System Architecture Diagram** (SVG)

   ```markdown
   ![Architecture Diagram](docs/diagrams/architecture.svg)
   ```

---

## 🎉 5. Conclusion

Congratulations! You have successfully:

* Built a Flask-based OIDC Provider with secure authentication flows.
* Implemented PKCE, RS256-signed JWT issuance, and JWKS publishing.
* Supported dynamic client registration and robust token management.
* Validated functionality through manual and automated tests.

With these skills, you can confidently integrate standards-compliant authentication into your web applications, ensuring both security and interoperability.

*End of Documentation — clear, consistent, and educational.*
