# 🌐 OIDC Provider from Scratch

[![Build Status](https://img.shields.io/github/actions/workflow/status/your-org/oidc-provider-architecture/ci.yml?branch=main)](https://github.com/your-org/oidc-provider-architecture/actions)
[![Coverage Status](https://img.shields.io/codecov/c/gh/your-org/oidc-provider-architecture)](https://codecov.io/gh/your-org/oidc-provider-architecture)
[![License](https://img.shields.io/github/license/your-org/oidc-provider-architecture)](./LICENSE)

> A from-zero, modular OpenID Connect Authorization Server built with Node.js, featuring clear architectural layers and extensible components.

---

## 📋 Table of Contents

* [🔍 Overview](#-overview)
* [🚀 Getting Started](#-getting-started)

  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
  * [Environment](#environment)
* [📂 Project Structure](#-project-structure)
* [⚙️ Configuration](#️-configuration)
* [🏗️ Architecture](#️-architecture)
* [👩‍💻 Usage](#-usage)
* [🛠️ Development](#️-development)

  * [Testing](#testing)
  * [Linting & Formatting](#linting--formatting)
  * [CI/CD](#cidc)
* [🤝 Contributing](#-contributing)
* [📄 License](#-license)

---

## 🔍 Overview

This repository implements an OpenID Connect Authorization Server from scratch using Node.js and the `oidc-provider` library. It emphasizes:

* **Modularity:** Clean separation between adapters, controllers, services, and interactions.
* **Extensibility:** Swap out persistence layers (Memory, Redis, MongoDB) without touching business logic.
* **Security:** RSA-signed JWTs, robust session management, and OIDC best practices.
* **Test Coverage:** Comprehensive unit and integration tests covering all critical flows.

---

## 🚀 Getting Started

### Prerequisites

* Node.js >= 16.x
* npm or Yarn
* OpenSSL (for generating RSA key pairs)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/oidc-provider-architecture.git
cd oidc-provider-architecture

# Install dependencies
npm install
# or
yarn install
```

### Environment

Copy `.env.example` to `.env` and configure:

```ini
PORT=4000
ISSUER_URL=http://localhost:4000
SESSION_SECRET=your-long-secret
JWT_SIGNING_KEY=./keys/private.pem
JWT_PUBLIC_KEY=./keys/public.pem
```

Generate RSA key pairs:

```bash
openssl genrsa -out keys/private.pem 2048
openssl rsa -in keys/private.pem -pubout -out keys/public.pem
```

---

## 📂 Project Structure

```text
├── .github/              # CI workflows, issue/pr templates
├── keys/                 # RSA key pair for signing JWTs
├── src/                  # Application source code
│   ├── config/           # OIDC provider configuration
│   ├── adapters/         # Persistence adapters (Memory, Redis, MongoDB)
│   ├── controllers/      # Auth & consent handlers
│   ├── interactions/     # HTML views for login & consent
│   ├── models/           # Domain entities (Account, Session)
│   ├── services/         # Business logic (user/token services)
│   ├── utils/            # Logger, error handling
│   └── index.js          # Express bootstrap & provider mount
├── tests/                # Unit & integration tests
├── .env.example          # Environment variable template
├── package.json          # npm scripts & dependencies
├── README.md             # This file
└── LICENSE               # MIT License
```

---

## ⚙️ Configuration

All OIDC settings reside in `src/config/oidcConfig.js`, including:

* **clients:** Registered OIDC clients and metadata
* **features:** PKCE, introspection, revocation
* **formats:** Token formats (e.g., JWT)
* **interactions:** URLs for login & consent prompts
* **jwks:** Public keys for JWT verification

---

## 🏗️ Architecture

### Component Breakdown

1. **Express Server** (`src/index.js`)

   * Parses requests, logs traffic, serves static interactions, and mounts `/oidc` routes.
2. **OIDC-Provider Core**

   * Managed via the `oidc-provider` library: protocol handling, grants, token issuance, interaction flow.
3. **Persistence Adapters** (`src/adapters/`)

   * Implements `upsert()`, `find()`, `destroy()`, etc., for Memory, Redis, MongoDB.
4. **Controllers** (`src/controllers/`)

   * `authController.js` handles login validation.
   * `consentController.js` records scope approvals.
5. **Interactions** (`src/interactions/`)

   * `login.html` and `consent.html` provide minimal UIs for end users.
6. **Services** (`src/services/`)

   * `userService.js` (registration, credential verification using bcrypt).
   * `tokenService.js` (JWT issuance with RSA keys).
7. **Models** (`src/models/`)

   * `Account.js` and `Session.js` represent domain entities.
8. **Utilities** (`src/utils/logger.js`)

   * Winston-based structured logging.
9. **RSA Keys** (`keys/`)

   * `private.pem` and `public.pem` used for JWT signing & JWKS.

---

## 👩‍💻 Usage

Start the server in development mode:

```bash
npm run dev
# or
yarn dev
```

Access OIDC endpoints at `http://localhost:4000/oidc`.

---

## 🛠️ Development

### Testing

* **Unit tests:** `npm run test:unit`
* **Integration tests:** `npm run test:integration`
* **Coverage report:** `npm run test -- --coverage`

### Linting & Formatting

* **ESLint:** `npm run lint`
* **Prettier:** formatting is automatic if you integrate it

### CI/CD

GitHub Actions in `.github/workflows/ci.yml` runs lint, tests, and uploads coverage to Codecov.

---

## 🤝 Contributing

Please refer to [CONTRIBUTING.md](.github/CONTRIBUTING.md) for guidelines on contributing, branching, and PR workflow.

---

## 📄 License

This project is licensed under the MIT License. See [LICENSE](./LICENSE) for details.
