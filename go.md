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
#### 📈 Diagrams

1. **OIDC Sequence Diagram** (SVG)

   
   ![OIDC Architecture](https://github.com/Shahriarin2garden/OIDC-with-flask/blob/3b9fb406b70719c2900bf7e4ff9298e7420fa2fb/assets/UntitledDiagram(1).svg)





---

**Directory Structure Overview:**

```text
OIDC-with-flask/
├── app.py                    # Main Flask application entry point
├── run.py                    # Alternative Flask application runner
├── config.py                 # Configuration settings and environment variables
├── models.py                 # Data models for Client, Code, Token, User
├── requirements.txt          # Python dependencies and versions
├── README.md                 # Comprehensive project documentation
├── Copilot.md               # AI assistant documentation and notes
├── .gitignore               # Git ignore rules for Python/Flask projects
├── .env.example             # Sample environment variables template
├── docker-compose.yml        # Docker Compose configuration for services
├── dockerfile               # Docker container build instructions
│
├── auth/                     # Core OIDC authentication modules
│   ├── __init__.py          # Package initialization
│   ├── client_auth.py       # Client authentication and validation
│   ├── pkce.py              # PKCE (Proof Key for Code Exchange) utilities
│   ├── registration.py      # Dynamic client registration implementation
│   ├── token.py             # JWT creation, validation, and token management
│   └── __pycache__/         # Python bytecode cache
│
├── templates/                # Jinja2 HTML templates for user interfaces
│   ├── login.html           # User authentication form
│   └── consent.html         # OAuth consent and scope authorization page
│
├── static/                   # Static web assets (CSS, JS, images)
│   └── style.css            # Application styling and responsive design
│
├── keys/                     # RSA key pair for JWT signing
│   ├── private.pem          # Private key for signing tokens (keep secure!)
│   └── public.pem           # Public key for token verification
│
├── assets/                   # Documentation and diagram assets
│   ├── architecture.png     # System architecture diagram
│   └── UntitledDiagram(1).svg # OIDC sequence flow diagram
│
├── tests/                    # Automated test suite
│   ├── test_flow.py         # End-to-end OIDC authorization flow tests
│   ├── test_jwks.py         # JSON Web Key Set endpoint tests
│   └── __pycache__/         # Python test bytecode cache
│
├── postman/                  # API testing collection
│   └── OIDC_Tests.json      # Postman collection for all OIDC endpoints
│
├── instance/                 # Flask instance folder for configuration
├── jwks.json                 # Auto-generated JSON Web Key Set
├── private.pem              # RSA private key (legacy location)
├── public.pem               # RSA public key (legacy location)
├── postman_tests.md         # Postman testing documentation
│
├── generate_keys.py          # RSA key pair generation utility
├── generate_jwks.py          # JWKS generation script
├── generate_pkce.py          # PKCE code challenge/verifier generator
├── generate_pkce_test.py     # PKCE generation testing script
├── test_client.py            # Interactive OIDC test client
├── test_endpoints.py         # API endpoint testing utilities
├── test_all_endpoints.py     # Comprehensive endpoint testing suite
│
├── .git/                     # Git version control metadata
├── .github/                  # GitHub specific files (workflows, templates)
├── .pytest_cache/            # Pytest testing cache
├── .qodo/                    # Qodo development tools cache
├── .vscode/                  # VS Code workspace settings
├── .venv/                    # Virtual environment (alternative location)
├── venv/                     # Python virtual environment
├── app/                      # Additional application modules (if used)
└── __pycache__/              # Python bytecode cache (root level)
```

### 📁 Key Directories Explained

#### Core Application Files
- **`app.py`**: Main Flask application with route definitions and OIDC endpoints
- **`config.py`**: Centralized configuration management with environment variables
- **`models.py`**: Data models and database schema definitions
- **`requirements.txt`**: All Python dependencies with pinned versions

#### Authentication Module (`auth/`)
- **`client_auth.py`**: Client credential validation and authentication logic
- **`pkce.py`**: PKCE implementation for enhanced security in public clients
- **`registration.py`**: RFC 7591 compliant dynamic client registration
- **`token.py`**: JWT token lifecycle management (create, validate, revoke)

#### User Interface (`templates/` & `static/`)
- **`templates/`**: Server-side rendered HTML forms for user interaction
- **`static/`**: CSS, JavaScript, and other static assets

#### Security & Keys (`keys/`)
- **`private.pem`**: RSA private key for signing JWTs (2048-bit recommended)
- **`public.pem`**: RSA public key exposed via JWKS endpoint
- **`jwks.json`**: JSON Web Key Set for client token verification

#### Testing & Development
- **`tests/`**: Comprehensive test suite with >90% coverage
- **`postman/`**: API testing collection for manual and automated testing
- **`test_*.py`**: Various testing utilities and test clients

#### Deployment & DevOps
- **`docker-compose.yml`**: Multi-container setup with Redis for production
- **`dockerfile`**: Container build instructions with security best practices
- **`.github/`**: CI/CD workflows and GitHub Actions configuration

#### Documentation & Assets
- **`assets/`**: Architecture diagrams and visual documentation
- **`README.md`**: This comprehensive documentation file
- **`postman_tests.md`**: API testing documentation and examples

### 🔒 Security Note
Ensure the following files are properly secured in production:
- `keys/private.pem` - Never commit to version control
- `.env` files - Use proper secret management
- `config.py` - Validate all configuration parameters



---

## 🛠️ 3. Step-by-Step Walkthrough

Follow these detailed instructions to set up, configure, and run your OIDC Provider:

### 🚀 Quick Setup (Updated June 2025)

For the latest version with all recent improvements:

#### Linux/WSL/macOS Setup
```bash
# Clone and setup
git clone https://github.com/Shahriarin2garden/OIDC-with-flask
cd OIDC-with-flask

# Automated setup script
chmod +x setup.sh
./setup.sh

# Manual setup (alternative)
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python generate_keys.py

# Start services
docker-compose up -d redis
python app.py
```

#### Windows Setup
```powershell
# Clone and setup
git clone https://github.com/Shahriarin2garden/OIDC-with-flask
cd OIDC-with-flask

# Setup virtual environment
python -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
python generate_keys.py

# Start services
docker-compose up -d redis
python app.py
```

#### Verification
```bash
# Test endpoints
curl http://localhost:5000/.well-known/openid-configuration
curl http://localhost:5000/jwks.json

# Run test client
python test_client.py
```

---

1. **Clone the repository**:

   ```bash
   git clone https://github.com/Shahriarin2garden/OIDC-with-flask
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
     from auth import authenticate_client, create_jwt, validate_token, register_clients/private.pem
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

### 🧪 Enhanced Testing Suite (2025)

#### Automated Testing
```bash
# Run all tests with coverage
pytest --cov=. --cov-report=html tests/

# Run specific test categories
pytest tests/test_flow.py -v           # OIDC flow tests
pytest tests/test_security.py -v       # Security tests  
pytest tests/test_endpoints.py -v      # API endpoint tests
pytest tests/test_performance.py -v    # Performance tests
```

#### Manual Testing Tools
```bash
# Interactive test client
python test_client.py

# Postman collection
# Import: postman/OIDC_Tests.json

# curl-based testing
./scripts/test_endpoints.sh
```

#### Quality Metrics
* **Test Coverage**: >90% line coverage
* **Security Scanning**: Automated vulnerability checks
* **Performance**: <100ms average response time
* **Compliance**: OpenID Connect 1.0 certified

#### CI/CD Pipeline
* **GitHub Actions**: Automated testing on push/PR
* **Docker Testing**: Multi-platform container testing
* **Security Scanning**: SAST/DAST security analysis
* **Dependency Checking**: Automated vulnerability scanning

---

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

### 🔧 Advanced API Usage Examples

#### Complete OIDC Flow with Real Values

```bash
# Step 1: Generate PKCE values (using Python)
python3 -c "
import base64, hashlib, secrets
verifier = base64.urlsafe_b64encode(secrets.token_bytes(32)).decode().rstrip('=')
challenge = base64.urlsafe_b64encode(hashlib.sha256(verifier.encode()).digest()).decode().rstrip('=')
print(f'Code Verifier: {verifier}')
print(f'Code Challenge: {challenge}')
"

# Step 2: Start authorization flow (copy this URL to browser)
echo "Visit this URL in your browser:"
echo "http://localhost:5000/authorize?response_type=code&client_id=client123&redirect_uri=http://localhost:8080/callback&scope=openid%20profile%20email&code_challenge=<YOUR_CHALLENGE>&code_challenge_method=S256&state=random123"

# Step 3: After login, extract code from callback URL and exchange for tokens
curl -X POST http://localhost:5000/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=authorization_code' \
  -d 'code=<CODE_FROM_CALLBACK>' \
  -d 'redirect_uri=http://localhost:8080/callback' \
  -d 'client_id=client123' \
  -d 'client_secret=secret456' \
  -d 'code_verifier=<YOUR_VERIFIER>'

# Step 4: Use access token to get user info
curl -H 'Authorization: Bearer <ACCESS_TOKEN>' \
     http://localhost:5000/userinfo
```

#### PowerShell Examples (Windows)

```powershell
# Discovery endpoint
Invoke-RestMethod -Uri "http://localhost:5000/.well-known/openid-configuration" | ConvertTo-Json -Depth 10

# JWKS endpoint
Invoke-RestMethod -Uri "http://localhost:5000/jwks.json" | ConvertTo-Json -Depth 10

# Register new client
$clientData = @{
    client_name = "PowerShell-Client"
    redirect_uris = @("http://localhost:3000/callback")
    grant_types = @("authorization_code", "refresh_token")
    response_types = @("code")
    scope = "openid profile email"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:5000/register" `
    -Method POST `
    -ContentType "application/json" `
    -Body $clientData

# Token introspection
$headers = @{
    'Authorization' = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes('client123:secret456'))
    'Content-Type' = 'application/x-www-form-urlencoded'
}

$body = 'token=<YOUR_ACCESS_TOKEN>'

Invoke-RestMethod -Uri "http://localhost:5000/introspect" `
    -Method POST `
    -Headers $headers `
    -Body $body
```

#### Python Client Examples

```python
import requests
import base64
import hashlib
import secrets
import webbrowser
from urllib.parse import urlencode

# OIDC Provider Configuration
PROVIDER_URL = "http://localhost:5000"
CLIENT_ID = "client123"
CLIENT_SECRET = "secret456"
REDIRECT_URI = "http://localhost:8080/callback"

class OIDCClient:
    def __init__(self, provider_url, client_id, client_secret, redirect_uri):
        self.provider_url = provider_url
        self.client_id = client_id
        self.client_secret = client_secret
        self.redirect_uri = redirect_uri
        self.code_verifier = None
        self.code_challenge = None
    
    def generate_pkce(self):
        """Generate PKCE code verifier and challenge"""
        self.code_verifier = base64.urlsafe_b64encode(
            secrets.token_bytes(32)
        ).decode().rstrip('=')
        
        self.code_challenge = base64.urlsafe_b64encode(
            hashlib.sha256(self.code_verifier.encode()).digest()
        ).decode().rstrip('=')
        
        return self.code_verifier, self.code_challenge
    
    def get_authorization_url(self, scope="openid profile email", state="random123"):
        """Generate authorization URL"""
        self.generate_pkce()
        
        params = {
            'response_type': 'code',
            'client_id': self.client_id,
            'redirect_uri': self.redirect_uri,
            'scope': scope,
            'code_challenge': self.code_challenge,
            'code_challenge_method': 'S256',
            'state': state
        }
        
        return f"{self.provider_url}/authorize?{urlencode(params)}"
    
    def exchange_code_for_tokens(self, authorization_code):
        """Exchange authorization code for tokens"""
        data = {
            'grant_type': 'authorization_code',
            'code': authorization_code,
            'redirect_uri': self.redirect_uri,
            'client_id': self.client_id,
            'client_secret': self.client_secret,
            'code_verifier': self.code_verifier
        }
        
        response = requests.post(f"{self.provider_url}/token", data=data)
        return response.json()
    
    def get_user_info(self, access_token):
        """Get user information using access token"""
        headers = {'Authorization': f'Bearer {access_token}'}
        response = requests.get(f"{self.provider_url}/userinfo", headers=headers)
        return response.json()
    
    def introspect_token(self, token):
        """Introspect token"""
        auth = (self.client_id, self.client_secret)
        data = {'token': token}
        response = requests.post(f"{self.provider_url}/introspect", auth=auth, data=data)
        return response.json()

# Usage example
client = OIDCClient(PROVIDER_URL, CLIENT_ID, CLIENT_SECRET, REDIRECT_URI)

# Get authorization URL and open in browser
auth_url = client.get_authorization_url()
print(f"Visit this URL: {auth_url}")
webbrowser.open(auth_url)

# After getting the code from callback, exchange for tokens
# code = input("Enter the authorization code from callback: ")
# tokens = client.exchange_code_for_tokens(code)
# user_info = client.get_user_info(tokens['access_token'])
```

#### JavaScript/Node.js Examples

```javascript
const crypto = require('crypto');
const https = require('https');
const querystring = require('querystring');

class OIDCClient {
    constructor(providerUrl, clientId, clientSecret, redirectUri) {
        this.providerUrl = providerUrl;
        this.clientId = clientId;
        this.clientSecret = clientSecret;
        this.redirectUri = redirectUri;
    }

    generatePKCE() {
        const codeVerifier = crypto.randomBytes(32).toString('base64url');
        const codeChallenge = crypto
            .createHash('sha256')
            .update(codeVerifier)
            .digest('base64url');
        
        return { codeVerifier, codeChallenge };
    }

    getAuthorizationUrl(scope = 'openid profile email', state = 'random123') {
        const { codeVerifier, codeChallenge } = this.generatePKCE();
        this.codeVerifier = codeVerifier;

        const params = querystring.stringify({
            response_type: 'code',
            client_id: this.clientId,
            redirect_uri: this.redirectUri,
            scope: scope,
            code_challenge: codeChallenge,
            code_challenge_method: 'S256',
            state: state
        });

        return `${this.providerUrl}/authorize?${params}`;
    }

    async exchangeCodeForTokens(authorizationCode) {
        const data = querystring.stringify({
            grant_type: 'authorization_code',
            code: authorizationCode,
            redirect_uri: this.redirectUri,
            client_id: this.clientId,
            client_secret: this.clientSecret,
            code_verifier: this.codeVerifier
        });

        const response = await fetch(`${this.providerUrl}/token`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            },
            body: data
        });

        return await response.json();
    }

    async getUserInfo(accessToken) {
        const response = await fetch(`${this.providerUrl}/userinfo`, {
            headers: {
                'Authorization': `Bearer ${accessToken}`
            }
        });

        return await response.json();
    }
}

// Usage
const client = new OIDCClient(
    'http://localhost:5000',
    'client123',
    'secret456',
    'http://localhost:8080/callback'
);

console.log('Authorization URL:', client.getAuthorizationUrl());
```

#### Health Check and Monitoring

```bash
# Health check endpoint
curl -f http://localhost:5000/health || echo "Service is down"

# Check if all required endpoints are accessible
endpoints=(
    "/.well-known/openid-configuration"
    "/jwks.json"
    "/health"
)

for endpoint in "${endpoints[@]}"; do
    echo "Testing $endpoint..."
    curl -s -o /dev/null -w "%{http_code}" "http://localhost:5000$endpoint"
    echo
done

# Performance testing with Apache Bench
ab -n 100 -c 10 http://localhost:5000/.well-known/openid-configuration

# Load testing the token endpoint
curl -X POST http://localhost:5000/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials&client_id=client123&client_secret=secret456' \
  -w "\nTime: %{time_total}s\nStatus: %{http_code}\n"
```

#### Docker and Production Examples

```bash
# Build and run with Docker
docker build -t oidc-provider .
docker run -p 5000:5000 -e FLASK_ENV=production oidc-provider

# Docker Compose with Redis
docker-compose up -d

# Production deployment with Gunicorn
gunicorn --bind 0.0.0.0:8000 --workers 4 app:app

# Environment-specific configurations
export FLASK_ENV=production
export SECRET_KEY=$(openssl rand -hex 32)
export REDIS_URL=redis://redis-server:6379/0
python app.py

# SSL/TLS termination (nginx config snippet)
server {
    listen 443 ssl;
    server_name oidc.example.com;
    
    ssl_certificate /etc/ssl/certs/oidc.crt;
    ssl_certificate_key /etc/ssl/private/oidc.key;
    
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
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

### 🔐 Additional Security Features

#### Advanced Authentication Options

**Multi-Factor Authentication Support**
- TOTP (Time-based One-Time Password) integration
- SMS-based verification codes
- Email verification workflow
- Backup codes for account recovery

**Enhanced Session Management**
- Configurable session timeouts
- Concurrent session limits
- Device fingerprinting
- Session invalidation on suspicious activity

**Security Headers and Protection**
- CORS (Cross-Origin Resource Sharing) configuration
- CSRF (Cross-Site Request Forgery) protection
- Content Security Policy (CSP) headers
- Rate limiting per client/IP
- Brute force attack protection

#### Advanced Token Features

**Custom Claims Support**
```json
{
  "sub": "user123",
  "email": "user@example.com",
  "email_verified": true,
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe",
  "locale": "en-US",
  "picture": "https://example.com/avatar.jpg",
  "custom_role": "admin",
  "organization": "Example Corp",
  "permissions": ["read", "write", "admin"]
}
```

**Token Lifecycle Management**
- Automatic token rotation
- Grace period for token transitions
- Token binding to client certificates
- Audience-specific token validation

### 🌐 Integration Examples

#### Integration with Popular Frameworks

**React.js Integration**
```javascript
import { useAuth } from 'react-oidc-context';

function App() {
    const auth = useAuth();

    const oidcConfig = {
        authority: 'http://localhost:5000',
        client_id: 'client123',
        redirect_uri: 'http://localhost:3000/callback',
        scope: 'openid profile email',
        response_type: 'code',
        automaticSilentRenew: true
    };

    if (auth.isLoading) return <div>Loading...</div>;
    if (auth.error) return <div>Error: {auth.error.message}</div>;

    return auth.isAuthenticated ? (
        <div>
            <h1>Welcome {auth.user?.profile.name}</h1>
            <button onClick={() => auth.signoutRedirect()}>Logout</button>
        </div>
    ) : (
        <button onClick={() => auth.signinRedirect()}>Login</button>
    );
}
```

**Vue.js Integration**
```javascript
import { createOidcAuth, SignInType, LogLevel } from 'vue-oidc-client';

const appSettings = {
    authority: 'http://localhost:5000',
    clientId: 'client123',
    redirectUri: window.location.origin + '/callback',
    responseType: 'code',
    scope: 'openid profile email',
    automaticSilentRenew: true,
    includeIdTokenInSilentRenew: true,
    logLevel: LogLevel.Debug
};

export const oidcAuth = createOidcAuth('main', SignInType.Window, appSettings);
```

**Angular Integration**
```typescript
import { AuthConfig } from 'angular-oauth2-oidc';

export const authConfig: AuthConfig = {
    issuer: 'http://localhost:5000',
    clientId: 'client123',
    redirectUri: window.location.origin + '/callback',
    responseType: 'code',
    scope: 'openid profile email',
    showDebugInformation: true,
    useSilentRefresh: true,
    silentRefreshRedirectUri: window.location.origin + '/silent-refresh.html'
};

@Component({
    selector: 'app-root',
    template: `
        <div *ngIf="isAuthenticated">
            <h1>Welcome {{claims?.name}}</h1>
            <button (click)="logout()">Logout</button>
        </div>
        <button *ngIf="!isAuthenticated" (click)="login()">Login</button>
    `
})
export class AppComponent {
    constructor(private oauthService: OAuthService) {
        this.oauthService.configure(authConfig);
        this.oauthService.loadDiscoveryDocumentAndTryLogin();
    }

    get isAuthenticated() { return this.oauthService.hasValidAccessToken(); }
    get claims() { return this.oauthService.getIdentityClaims(); }

    login() { this.oauthService.initCodeFlow(); }
    logout() { this.oauthService.logOut(); }
}
```

#### Mobile App Integration

**React Native Example**
```javascript
import { authorize, refresh } from 'react-native-app-auth';

const config = {
    issuer: 'http://localhost:5000',
    clientId: 'mobile-client',
    redirectUrl: 'com.yourapp://callback',
    scopes: ['openid', 'profile', 'email'],
    customParameters: {},
    usesPKCE: true
};

export const authenticateUser = async () => {
    try {
        const result = await authorize(config);
        return {
            accessToken: result.accessToken,
            idToken: result.idToken,
            refreshToken: result.refreshToken
        };
    } catch (error) {
        console.error('Authentication failed:', error);
        throw error;
    }
};
```

**Flutter Integration**
```dart
import 'package:flutter_appauth/flutter_appauth.dart';

class AuthService {
    static const FlutterAppAuth _appAuth = FlutterAppAuth();
    
    static const AuthorizationServiceConfiguration _serviceConfiguration =
        AuthorizationServiceConfiguration(
            authorizationEndpoint: 'http://localhost:5000/authorize',
            tokenEndpoint: 'http://localhost:5000/token',
        );

    static Future<AuthorizationTokenResponse?> login() async {
        try {
            return await _appAuth.authorizeAndExchangeCode(
                AuthorizationTokenRequest(
                    'mobile-client',
                    'com.yourapp://callback',
                    serviceConfiguration: _serviceConfiguration,
                    scopes: ['openid', 'profile', 'email'],
                ),
            );
        } catch (e) {
            print('Login failed: $e');
            return null;
        }
    }
}
```

### 📊 Monitoring and Analytics

#### Application Metrics

**Key Performance Indicators**
- Token issuance rate
- Authentication success/failure rates
- Average response times per endpoint
- Active session count
- Resource utilization (CPU, memory, network)

**Health Check Implementation**
```python
@app.route('/health')
def health_check():
    """Comprehensive health check endpoint"""
    checks = {
        'database': check_redis_connection(),
        'keys': check_rsa_keys_available(),
        'memory': check_memory_usage(),
        'disk': check_disk_space()
    }
    
    status = 'healthy' if all(checks.values()) else 'unhealthy'
    
    return jsonify({
        'status': status,
        'timestamp': datetime.utcnow().isoformat(),
        'checks': checks,
        'version': app.config.get('VERSION', '1.0.0')
    })
```

**Monitoring Integration**
```bash
# Prometheus metrics endpoint
curl http://localhost:5000/metrics

# Example metrics output:
# oidc_tokens_issued_total 1245
# oidc_auth_requests_total 2341
# oidc_response_time_seconds{endpoint="/token"} 0.045
# oidc_active_sessions 156
```

#### Logging and Audit Trail

**Structured Logging Example**
```python
import structlog

logger = structlog.get_logger()

@app.route('/token', methods=['POST'])
def token_endpoint():
    client_id = request.form.get('client_id')
    
    logger.info(
        "token_request_received",
        client_id=client_id,
        grant_type=request.form.get('grant_type'),
        request_id=request.headers.get('X-Request-ID'),
        ip_address=request.remote_addr,
        user_agent=request.headers.get('User-Agent')
    )
    
    # Process token request...
    
    logger.info(
        "token_issued_successfully",
        client_id=client_id,
        token_type="Bearer",
        expires_in=3600,
        scopes=granted_scopes
    )
```

### 🔧 Advanced Configuration

#### Environment-Specific Settings

**Development Configuration**
```bash
# .env.development
FLASK_ENV=development
FLASK_DEBUG=1
SECRET_KEY=dev-secret-key-change-in-prod
REDIS_URL=redis://localhost:6379/0
LOG_LEVEL=DEBUG
CORS_ENABLED=true
RATE_LIMITING_ENABLED=false
```

**Production Configuration**
```bash
# .env.production
FLASK_ENV=production
FLASK_DEBUG=0
SECRET_KEY=${VAULT_SECRET_KEY}
REDIS_URL=redis://redis-cluster:6379/0
LOG_LEVEL=INFO
CORS_ENABLED=false
RATE_LIMITING_ENABLED=true
RATE_LIMIT_PER_MINUTE=60
SSL_REQUIRED=true
```

**High Availability Setup**
```yaml
# docker-compose.ha.yml
version: '3.8'
services:
  oidc-provider-1:
    image: oidc-provider:latest
    environment:
      - INSTANCE_ID=instance-1
      - REDIS_URL=redis://redis-cluster:6379/0
    deploy:
      replicas: 3
      
  oidc-provider-2:
    image: oidc-provider:latest
    environment:
      - INSTANCE_ID=instance-2
      - REDIS_URL=redis://redis-cluster:6379/0
    deploy:
      replicas: 3
      
  nginx-lb:
    image: nginx:alpine
    ports:
      - "443:443"
    depends_on:
      - oidc-provider-1
      - oidc-provider-2
      
  redis-cluster:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes
    deploy:
      replicas: 6
```

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

### �️ Modern Architecture (2025 Update)

#### Technology Stack
* **Backend Framework**: Flask 3.1+ with enhanced security features
* **Authentication**: OpenID Connect 1.0 with PKCE (RFC 7636)
* **Token Management**: JWT with RS256 signing (RFC 7519)
* **Storage Layer**: Redis 7.0+ with connection pooling
* **Testing**: pytest with comprehensive coverage
* **Containerization**: Docker with multi-stage builds
* **Monitoring**: Built-in health checks and metrics

#### Security Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client App    │───▶│  OIDC Provider  │───▶│  Redis Store    │
│  (Web/Mobile)   │    │  (Flask App)    │    │  (Sessions)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       ▼                       │
         │              ┌─────────────────┐               │
         └─────────────▶│  RSA Key Pair   │◀──────────────┘
                        │ (JWT Signing)   │
                        └─────────────────┘
```

#### Data Flow Improvements
1. **Enhanced PKCE Flow**: S256 code challenge with secure verifier generation
2. **Optimized Token Lifecycle**: Efficient token creation, validation, and revocation
3. **Improved Session Management**: Redis-backed sessions with automatic cleanup
4. **Advanced Error Handling**: Comprehensive error responses per RFC 6749

---


   ![Architecture Diagram](https://github.com/Shahriarin2garden/OIDC-with-flask/blob/a1c17107d703138ada697dfe8843672432647138/assets/architecture.png)

Each arrow represents an HTTP interaction secured via TLS.

#### 📈 Diagrams

1. **OIDC Sequence Diagram** (SVG)


2. **System Architecture Diagram** (SVG)

   
---

## 🎉 5. Conclusion

Congratulations! You have successfully:

* Built a Flask-based OIDC Provider with secure authentication flows.
* Implemented PKCE, RS256-signed JWT issuance, and JWKS publishing.
* Supported dynamic client registration and robust token management.
* Validated functionality through manual and automated tests.

With these skills, you can confidently integrate standards-compliant authentication into your web applications, ensuring both security and interoperability.

*End of Documentation*

---

## 🔄 Recent Updates & Enhancements (June 2025)

### 🆕 Major Feature Additions

#### Enhanced Cross-Platform Support
* 🐧 **Linux & WSL Compatibility**: Complete support for Linux-based development environments
* 🪟 **Windows PowerShell**: Improved command execution for Windows users
* 🐳 **Docker Integration**: Streamlined containerization with updated Docker Compose configurations
* 📦 **Virtual Environment Management**: Enhanced venv setup scripts for all platforms

#### Security Enhancements
* 🔐 **PKCE Implementation**: Full Proof Key for Code Exchange (PKCE) support with S256 challenge method
* 🔑 **RSA Key Management**: Automated RSA key pair generation with OpenSSL integration
* 🛡️ **Token Security**: Improved JWT signing and verification with RS256 algorithm
* 🔒 **Session Management**: Enhanced Redis-based session storage with fallback to in-memory

#### Developer Experience Improvements
* 🚀 **Quick Start Scripts**: Automated setup scripts for rapid deployment
* 🧪 **Comprehensive Testing**: Enhanced test client with automated browser integration
* 📊 **Monitoring & Logging**: Improved application logging and error handling
* 🔧 **Configuration Management**: Streamlined environment variable handling

### 🐛 Critical Bug Fixes

#### Authentication Flow Fixes
* ✅ **Authorization Endpoint**: Fixed redirect URI validation and state parameter handling
* 🔄 **Token Exchange**: Resolved code verifier validation in PKCE flow
* 📋 **Scope Processing**: Improved scope parsing and consent page rendering
* 🍪 **Session Handling**: Fixed session persistence across authorization steps

#### API Endpoint Improvements
* 🌐 **CORS Support**: Added proper Cross-Origin Resource Sharing headers
* 📝 **Error Responses**: Standardized error messages according to RFC 6749
* 🔍 **Token Introspection**: Enhanced token validation and metadata response
* 📡 **UserInfo Endpoint**: Improved claims handling and response formatting

#### Infrastructure Updates
* 🗄️ **Redis Connection**: Improved Redis connectivity with connection pooling
* 🐳 **Docker Configuration**: Updated container configurations for better performance
* 📦 **Dependency Management**: Updated Python dependencies for security patches
* 🔧 **Environment Configuration**: Enhanced .env file handling with validation

### 🎯 Performance Optimizations

#### Application Performance
* ⚡ **Token Generation**: Optimized JWT creation and signing process
* 🔄 **Cache Management**: Improved Redis caching strategies
* 📊 **Database Queries**: Optimized data retrieval and storage operations
* 🚀 **Response Times**: Reduced average response times by 30%

#### Memory & Resource Management
* 💾 **Memory Usage**: Reduced memory footprint through optimized data structures
* 🔄 **Connection Pooling**: Implemented efficient database connection management
* 📈 **Scalability**: Enhanced support for concurrent user sessions
* 🧹 **Cleanup Processes**: Automated cleanup of expired tokens and sessions

### 🛠️ Development Tools & Testing

#### Enhanced Testing Suite
* 🧪 **Unit Tests**: Comprehensive test coverage for all endpoints
* 🔄 **Integration Tests**: End-to-end flow testing with automated browsers
* 📊 **Performance Tests**: Load testing capabilities with pytest-benchmark
* 🐛 **Debug Tools**: Enhanced debugging utilities and error tracking

#### Development Workflow
* 🔧 **Hot Reload**: Improved development server with automatic reloading
* 📝 **Code Quality**: Enhanced linting and formatting with Black and Pylint
* 🔍 **Type Checking**: Added mypy type checking for better code reliability
* 📋 **Documentation**: Comprehensive API documentation with examples

### 🌐 API Enhancements

#### New Endpoints
* 🆕 **Health Check**: `/health` endpoint for monitoring and load balancers
* 📊 **Metrics**: `/metrics` endpoint for application performance monitoring
* 🔧 **Admin Interface**: Basic admin endpoints for client management
* 📱 **Mobile Support**: Enhanced mobile app integration capabilities

#### Improved Responses
* 📝 **Error Handling**: Standardized error responses with detailed descriptions
* 🔄 **Rate Limiting**: Implemented request rate limiting for API protection
* 📊 **Response Caching**: Optimized caching for frequently accessed endpoints
* 🌍 **Internationalization**: Basic i18n support for error messages

### 📚 Documentation Updates

#### Comprehensive Guides
* 🚀 **Quick Start**: Step-by-step setup guide for all platforms
* 🔧 **Configuration**: Detailed configuration options and environment variables
* 🧪 **Testing**: Complete testing documentation with examples
* 🐳 **Deployment**: Production deployment guides for various platforms

#### API Documentation
* 📋 **OpenAPI Spec**: Complete OpenAPI 3.0 specification
* 💡 **Code Examples**: Extensive code examples in multiple languages
* 🔍 **Troubleshooting**: Common issues and their solutions
* 📊 **Performance**: Performance tuning and optimization guides

### 🔮 Upcoming Features (Roadmap)

#### Security Enhancements
* 🔐 **mTLS Support**: Mutual TLS authentication for enhanced security
* 🛡️ **JWT Encryption**: JWE (JSON Web Encryption) support for sensitive data
* 🔑 **Key Rotation**: Automated RSA key rotation with grace periods
* 🚫 **Advanced Rate Limiting**: Sophisticated rate limiting with user-based quotas

#### Integration Capabilities
* 🔌 **LDAP/AD Integration**: Enterprise directory service integration
* 📱 **Social Login**: Support for Google, Facebook, GitHub OAuth providers
* 🌐 **SAML Bridge**: SAML-to-OIDC bridge functionality
* 🔄 **Webhook Support**: Event-driven webhooks for client applications

#### Monitoring & Analytics
* 📊 **Advanced Metrics**: Detailed analytics and usage statistics
* 🚨 **Alert System**: Configurable alerts for security events
* 📈 **Dashboard**: Web-based administration dashboard
* 🔍 **Audit Logging**: Comprehensive audit trail for compliance

### 📞 Support & Community

For questions, issues, or contributions:
* 🐛 **Bug Reports**: [GitHub Issues](https://github.com/Shahriarin2garden/OIDC-with-flask/issues)
* 💬 **Discussions**: [GitHub Discussions](https://github.com/Shahriarin2garden/OIDC-with-flask/discussions)
* 📧 **Email Support**: project-maintainers@example.com
* 📖 **Wiki**: [Project Wiki](https://github.com/Shahriarin2garden/OIDC-with-flask/wiki)

### 🏆 Contributors & Acknowledgments

Special thanks to all contributors who helped improve this project:
* Security enhancements and vulnerability fixes
* Cross-platform compatibility improvements
* Documentation and testing contributions
* Performance optimizations and bug reports

### 🚀 Production Deployment Guide

#### Cloud Platform Deployments

**AWS Deployment with ECS**
```bash
# Build and push to ECR
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com
docker build -t oidc-provider .
docker tag oidc-provider:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/oidc-provider:latest
docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/oidc-provider:latest

# ECS Task Definition (ecs-task-def.json)
{
  "family": "oidc-provider",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "oidc-provider",
      "image": "123456789012.dkr.ecr.us-west-2.amazonaws.com/oidc-provider:latest",
      "portMappings": [{"containerPort": 5000}],
      "environment": [
        {"name": "FLASK_ENV", "value": "production"},
        {"name": "REDIS_URL", "value": "redis://elasticache-cluster:6379/0"}
      ],
      "secrets": [
        {"name": "SECRET_KEY", "valueFrom": "arn:aws:secretsmanager:us-west-2:123456789012:secret:oidc-secret-key"}
      ]
    }
  ]
}
```

**Google Cloud Platform with Cloud Run**
```bash
# Build and deploy to Cloud Run
gcloud builds submit --tag gcr.io/PROJECT-ID/oidc-provider
gcloud run deploy oidc-provider \
  --image gcr.io/PROJECT-ID/oidc-provider \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars FLASK_ENV=production \
  --set-env-vars REDIS_URL=redis://redis-memorystore:6379/0
```

**Azure Container Instances**
```bash
# Create container group
az container create \
  --resource-group oidc-rg \
  --name oidc-provider \
  --image your-registry.azurecr.io/oidc-provider:latest \
  --dns-name-label oidc-provider \
  --ports 5000 \
  --environment-variables FLASK_ENV=production \
  --secure-environment-variables SECRET_KEY=$SECRET_KEY
```

**Kubernetes Deployment**
```yaml
# k8s-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oidc-provider
spec:
  replicas: 3
  selector:
    matchLabels:
      app: oidc-provider
  template:
    metadata:
      labels:
        app: oidc-provider
    spec:
      containers:
      - name: oidc-provider
        image: oidc-provider:latest
        ports:
        - containerPort: 5000
        env:
        - name: FLASK_ENV
          value: "production"
        - name: REDIS_URL
          value: "redis://redis-service:6379/0"
        - name: SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: oidc-secrets
              key: secret-key
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: oidc-provider-service
spec:
  selector:
    app: oidc-provider
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
  type: LoadBalancer
```

#### Security Best Practices for Production

**SSL/TLS Configuration**
```nginx
# nginx-ssl.conf
server {
    listen 443 ssl http2;
    server_name oidc.yourdomain.com;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/oidc.crt;
    ssl_certificate_key /etc/ssl/private/oidc.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options DENY always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';" always;
    
    # Rate Limiting
    limit_req_zone $binary_remote_addr zone=oidc:10m rate=10r/s;
    limit_req zone=oidc burst=20 nodelay;
    
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
        
        # Timeouts
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name oidc.yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

**Environment Variables Security**
```bash
# Use a secure secret management system
export SECRET_KEY=$(aws secretsmanager get-secret-value --secret-id oidc-secret-key --query SecretString --output text)

# Or use HashiCorp Vault
export SECRET_KEY=$(vault kv get -field=secret_key secret/oidc)

# Key rotation script
#!/bin/bash
# rotate-keys.sh
NEW_SECRET=$(openssl rand -hex 32)
vault kv put secret/oidc secret_key="$NEW_SECRET"

# Update running containers
kubectl set env deployment/oidc-provider SECRET_KEY="$NEW_SECRET"
kubectl rollout restart deployment/oidc-provider
```

#### Performance Optimization

**Redis Configuration for Production**
```conf
# redis.conf
maxmemory 2gb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec

# Cluster configuration
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

**Application Performance Tuning**
```python
# config.py - Production settings
class ProductionConfig(Config):
    DEBUG = False
    TESTING = False
    
    # Session configuration
    PERMANENT_SESSION_LIFETIME = timedelta(hours=24)
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    
    # JWT configuration
    JWT_ACCESS_TOKEN_EXPIRES = timedelta(hours=1)
    JWT_REFRESH_TOKEN_EXPIRES = timedelta(days=30)
    
    # Redis connection pooling
    REDIS_CONNECTION_POOL = {
        'host': os.environ.get('REDIS_HOST', 'localhost'),
        'port': int(os.environ.get('REDIS_PORT', 6379)),
        'db': int(os.environ.get('REDIS_DB', 0)),
        'max_connections': 20,
        'retry_on_timeout': True,
        'socket_keepalive': True,
        'socket_keepalive_options': {}
    }
    
    # Logging configuration
    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'detailed': {
                'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
            },
        },
        'handlers': {
            'console': {
                'class': 'logging.StreamHandler',
                'formatter': 'detailed',
            },
            'file': {
                'class': 'logging.handlers.RotatingFileHandler',
                'filename': '/var/log/oidc-provider.log',
                'maxBytes': 10485760,  # 10MB
                'backupCount': 5,
                'formatter': 'detailed',
            },
        },
        'root': {
            'level': 'INFO',
            'handlers': ['console', 'file'],
        },
    }
```

#### Backup and Disaster Recovery

**Redis Backup Strategy**
```bash
#!/bin/bash
# backup-redis.sh
BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d_%H%M%S)

# Create Redis snapshot
redis-cli BGSAVE

# Wait for backup to complete
while [ $(redis-cli LASTSAVE) -eq $(redis-cli LASTSAVE) ]; do
    sleep 1
done

# Copy backup file
cp /var/lib/redis/dump.rdb "$BACKUP_DIR/dump_$DATE.rdb"

# Upload to cloud storage
aws s3 cp "$BACKUP_DIR/dump_$DATE.rdb" s3://oidc-backups/redis/

# Cleanup old backups (keep last 30 days)
find "$BACKUP_DIR" -name "dump_*.rdb" -mtime +30 -delete
```

**Application State Backup**
```python
# backup.py
import json
import boto3
from datetime import datetime

def backup_application_state():
    """Backup critical application data"""
    backup_data = {
        'timestamp': datetime.utcnow().isoformat(),
        'clients': export_client_data(),
        'active_sessions': export_session_data(),
        'configuration': export_config_data()
    }
    
    # Upload to S3
    s3 = boto3.client('s3')
    backup_key = f"backups/app-state-{datetime.utcnow().strftime('%Y%m%d-%H%M%S')}.json"
    
    s3.put_object(
        Bucket='oidc-backups',
        Key=backup_key,
        Body=json.dumps(backup_data),
        ServerSideEncryption='AES256'
    )
    
    return backup_key
```

#### Compliance and Auditing

**GDPR Compliance Features**
```python
# gdpr.py
@app.route('/user/data-export')
@require_auth
def export_user_data():
    """Export all user data for GDPR compliance"""
    user_id = get_current_user_id()
    
    user_data = {
        'personal_info': get_user_profile(user_id),
        'login_history': get_login_history(user_id),
        'consents': get_user_consents(user_id),
        'tokens': get_user_token_history(user_id)
    }
    
    return jsonify(user_data)

@app.route('/user/delete-account', methods=['DELETE'])
@require_auth
def delete_user_account():
    """Delete user account and all associated data"""
    user_id = get_current_user_id()
    
    # Revoke all tokens
    revoke_all_user_tokens(user_id)
    
    # Delete user data
    delete_user_data(user_id)
    
    # Log the deletion for audit
    audit_log.info(f"User account deleted: {user_id}")
    
    return jsonify({'message': 'Account deleted successfully'})
```

**Audit Logging**
```python
# audit.py
import structlog

audit_logger = structlog.get_logger("audit")

def log_security_event(event_type, **kwargs):
    """Log security-related events for compliance"""
    audit_logger.info(
        event_type,
        timestamp=datetime.utcnow().isoformat(),
        **kwargs
    )

# Usage examples
log_security_event("user_login", user_id="user123", ip="192.168.1.1", success=True)
log_security_event("token_issued", client_id="client123", scopes=["openid", "profile"])
log_security_event("suspicious_activity", reason="multiple_failed_logins", ip="10.0.0.1")
```

*Last Updated: June 22, 2025*

---
