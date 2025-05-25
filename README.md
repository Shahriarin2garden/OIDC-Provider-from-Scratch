# Flask OIDC Provider

> **Enterprise-grade OpenID Connect (OIDC) server with Flask**

A modular, extensible identity provider fully compliant with OIDC 1.0. Designed for production and easy integration, it offers secure authentication flows, dynamic client onboarding, JWT lifecycle management, and built‑in observability.

---

## Table of Contents

1. [Overview](#overview)
2. [Core Concepts](#core-concepts)
3. [Features](#features)
4. [Architecture](#architecture)
5. [Data Flow](#data-flow)
6. [Step-by-Step Implementation](#step-by-step-implementation)
7. [Repository Layout](#repository-layout)
8. [Setup & Configuration](#setup--configuration)
9. [Security Best Practices](#security-best-practices)
10. [Usage Examples](#usage-examples)
11. [API Reference](#api-reference)
12. [Deployment & Scaling](#deployment--scaling)
13. [Testing](#testing)
14. [Contributing](#contributing)
15. [License](#license)

---

## Overview

Flask OIDC Provider is a turnkey identity solution, offering:

* **Secure Authentication**: Authorization Code Flow with PKCE and optional Implicit/Hybrid.
* **Token Management**: Issue, introspect, refresh, and revoke JWTs.
* **Dynamic Client Registration**: Programmatic client onboarding.
* **Standards Compliance**: OIDC discovery and JWKS for metadata and key distribution.
* **Extensible Persistence**: Pluggable adapters for in-memory, Redis, or SQL stores.
* **Observability**: Built‑in Prometheus metrics and structured logging.

Ideal for microservices, SPAs, and mobile applications requiring federated identity.

---

## Core Concepts

1. **OpenID Connect (OIDC)**: Adds an identity layer to OAuth 2.0.
2. **PKCE (RFC 7636)**: Mitigates authorization code interception for public clients.
3. **JSON Web Tokens (JWT)**: Self‑contained tokens with claims, signed for integrity.
4. **Discovery & JWKS**: `.well-known/openid-configuration` and `/jwks` endpoints enable client auto‑configuration and key verification.
5. **Dynamic Client Registration**: REST API for registering and updating clients at runtime.

---

## Features

* 🔒 **Authorization Code + PKCE**: Secure server‑side flow with optional PKCE.
* 🔁 **Refresh & Revoke**: Long‑lived sessions via refresh tokens and immediate token revocation.
* 📜 **Discovery & JWKS**: Automatic metadata and public key distribution.
* 🛠️ **Dynamic Registration**: `/register` endpoint for client onboarding.
* ⚙️ **Custom Claims**: Extend `UserInfo` with application‑specific attributes.
* 🌍 **Pluggable Stores**: In‑memory, Redis, PostgreSQL/MySQL adapters.
* 📊 **Metrics & Logs**: Prometheus metrics and structured logs.
* 🧪 **Test Suite & CI**: 90%+ coverage with Pytest and GitHub Actions.

---

## Architecture

```mermaid
flowchart LR
  subgraph API Layer
    A[/authorize, /token, /userinfo, /register, /.well-known/]
  end
  subgraph Logic Layer
    B[AuthService]
    C[PKCEService]
    D[TokenService]
    E[ClientService]
  end
  subgraph Persistence Layer
    F[Store Interface]
    G[Redis / SQL / In-Memory]
  end

  A -->|auth| B
  B -->|challenge| C
  B -->|code| F
  A -->|token| D
  D -->|jwt| F
  A -->|client mgmt| E
  E -->|metadata| F
  F --> G
```

**Layer Responsibilities**

* **API Layer**: Exposes endpoints, validates inputs, and handles HTTP interactions.
* **Logic Layer**: Implements core business workflows—authentication, PKCE, token issuance, and client registration.
* **Persistence Layer**: Abstracts storage; adapters allow seamless backend swapping.

---

## Data Flow

### Sequence Diagram

```mermaid
sequenceDiagram
    participant UA as User Agent
    participant API as API Layer
    participant Auth as AuthService
    participant PKCE as PKCEService
    participant Token as TokenService
    participant Store as Persistence

    UA->>API: GET /authorize?client_id&redirect_uri&code_challenge
    API->>Auth: validateClient()
    Auth->>PKCE: generateChallenge()
    Auth->>Store: saveAuthCode()
    Auth-->>API: auth_code
    API-->>UA: 302 Redirect with auth_code

    UA->>API: POST /token (code, code_verifier)
    API->>Token: fetchAuthCode()
    Token->>PKCE: verifyVerifier()
    Token->>Store: createTokens()
    Token-->>API: {access_token, id_token, refresh_token}
    API-->>UA: JSON response

    UA->>API: GET /userinfo (Authorization: Bearer)
    API->>Token: validateAccessToken()
    Token-->>API: user_claims
    API-->>UA: JSON claims
```

### Step-by-Step

1. **Initiate Authorization**: Client requests `/authorize` with `code_challenge`.
2. **User Login**: AuthService validates credentials and stores code challenge.
3. **Issue Code**: AuthService generates an authorization code and persists it.
4. **Token Exchange**: Client posts `code_verifier` to `/token`; TokenService verifies and issues JWTs.
5. **UserInfo**: Client uses access token to fetch claims from `/userinfo`.
6. **Token Management**: `/introspect` and `/revoke` endpoints manage token validity.

---

## Step-by-Step Implementation

1. **Clone & Install**

   ```bash
   git clone https://github.com/your-org/flask-oidc-provider.git
   cd flask-oidc-provider
   python3 -m venv venv && source venv/bin/activate
   pip install -r requirements.txt
   ```

2. **Configure Environment**

   ```bash
   cp .env.example .env
   # Edit .env per your environment
   ```

3. **Run Migrations** (if using SQL)

   ```bash
   flask db upgrade
   ```

4. **Start Server**

   ```bash
   flask run --host=0.0.0.0 --port=5000
   ```

5. **Verify**

   ```bash
   curl GET http://localhost:5000/.well-known/openid-configuration
   ```

---

## Repository Layout

```text
flask-oidc-provider/
├── app.py                # Application entrypoint
├── config.py             # Configuration management
├── models.py             # Data models and persistence interfaces
├── auth/                 # Authentication and PKCE services
│   ├── auth.py
│   ├── pkce.py
│   └── discovery.py
├── token/                # Token issuance and JWT handling
│   ├── token.py
│   └── jwt.py
├── client/               # Dynamic client registration logic
│   └── client.py
├── persistence/          # Storage adapters (Redis, SQL, In-memory)
│   ├── redis.py
│   └── sql.py
├── templates/            # Jinja2 templates (login, consent)
├── static/               # Frontend assets
├── certs/                # Cryptographic key pairs
├── jwks.json             # Public JSON Web Key Set
├── requirements.txt      # Python dependencies
├── Dockerfile            # Docker container spec
├── docker-compose.yml    # Development environment setup
└── tests/                # Unit and integration tests
```

---

## Setup & Configuration

**.env file**:

```ini
FLASK_ENV=development
SECRET_KEY=your-secret
ISSUER_URL=https://auth.example.com
PRIVATE_KEY_PATH=certs/private.pem
PUBLIC_KEY_PATH=certs/public.pem
DATABASE_URL=postgresql://user:pass@localhost/db
REDIS_URL=redis://localhost:6379/0
TOKEN_EXPIRY=3600
REFRESH_EXPIRY=86400
```

> **Note**: Use a secure vault in production.

---

## Security Best Practices

* Enforce HTTPS and HSTS.
* Whitelist and validate `redirect_uri`.
* Rate-limit `/token`, `/register`, `/introspect`, and `/revoke`.
* Rotate keys via the JWKS endpoint.
* Mask sensitive data in logs.

---

## Usage Examples

### Dynamic Client Registration

```bash
curl -X POST $ISSUER_URL/register \
  -H 'Content-Type: application/json' \
  -d '{
    "client_name": "MyApp",
    "redirect_uris": ["https://myapp.com/callback"]
  }'
```

### Authorization Code Flow with PKCE

```bash
VERIFIER=$(openssl rand -base64 32 | tr -d '=+/')
CHALLENGE=$(printf '%s' "$VERIFIER" | openssl dgst -sha256 -binary | openssl base64 | tr -d '=+/')

# 1. User Redirect
open "$ISSUER_URL/authorize?client_id=ID&response_type=code&redirect_uri=<uri>&code_challenge=$CHALLENGE&state=xyz"

# 2. Token Exchange
curl -X POST $ISSUER_URL/token \
  -H 'Content-Type:application/x-www-form-urlencoded' \
  -d "grant_type=authorization_code&code=<code>&redirect_uri=<uri>&code_verifier=$VERIFIER"
```

---

## API Reference

| Method | Path                                | Purpose                     |
| ------ | ----------------------------------- | --------------------------- |
| POST   | `/register`                         | Register or update a client |
| GET    | `/.well-known/openid-configuration` | OIDC Discovery metadata     |
| GET    | `/jwks`                             | JSON Web Key Set            |
| GET    | `/authorize`                        | Start auth code flow        |
| POST   | `/token`                            | Exchange code for tokens    |
| GET    | `/userinfo`                         | Retrieve user claims        |
| POST   | `/introspect`                       | Validate a token            |
| POST   | `/revoke`                           | Revoke a token              |

---

## Deployment & Scaling

* **Docker Compose**: `docker-compose up --build`
* **Gunicorn**: `gunicorn -w4 app:app`
* **Horizontal Scaling**: Stateless app behind LB; clustered DB/Redis.
* **Monitoring**: Scrape `/metrics` for Prometheus; integrate with Grafana.

---

## Testing

```bash
pytest --cov-report=term-missing --cov=.
```

> Ensure coverage ≥ 90% on all PRs.

---

## Contributing

1. Fork the repository.
2. Create a feature branch.
3. Implement changes and add tests.
4. Open a PR; maintainers will review.

---

## License

MIT © Your Organization. See [LICENSE](./LICENSE) for details.
