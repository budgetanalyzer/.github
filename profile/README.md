# Budget Analyzer

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Production-grade microservices security infrastructure and ReactJS frontend built with AI-assisted development**

---

## Quickstart

**Prerequisites**: VS Code with [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers), Docker

```bash
git clone https://github.com/budgetanalyzer/workspace
```

Open in VS Code → "Reopen in Container" → Follow [Getting Started](https://github.com/budgetanalyzer/orchestration/blob/main/docs/development/getting-started.md)

## Background

This project started as a simple re-fresh of my spring boot microservices skills after a 2 year sabbatical.  I wanted to solve the relatively simple problem of reconciling multiple bank accounts in multiple currencies, so I figured I'd do a quick microservice + ReactJS frontend and go find a consulting role as I've been doing the last few years.  I was shocked to finish that in a couple weeks after gettting comfortable using Claude Code, so I expanded the scope of the project significantly.  This is a full production grade best practices session-based edge authorization with OAuth2 implementation for an auditable compliance-oriented financial application.  But really it's just an AI sandbox.  And now I'm excited to go back to building stuff and am looking to work with people that get what this is.

✨ *Using **Claude Code**, we rapidly expanded from a basic app to a full enterprise-grade security architecture. The AI didn't just write code—it helped design systems, document decisions, and implement patterns that would typically require a dedicated team.*   (*Claude wrote that*)

Once I realized that I could use AI as a peer collaborator, I worked with Claude and Codex to build a system an order of magnitude more advanced than my initial plan.  I wanted to see how agents perform in real production environments, so I created my own "Pet Store" here in budgetanalyzer and built a fully deployed local dev/production parity web application demonstrating modern patterns for application security.  The budgetanalyzer app and organaization are just the vehicle to keep myself honest about production grade complexity that we tend to hand wave for demo projects. 

## Vision

We're building a **pluggable security and authorization infrastructure** that any company can adopt.

The goal isn't just a budget app. It's a reusable foundation that demonstrates:
- Production-ready OAuth2/OIDC authentication
- Server-side session management (opaque sessions in Redis)
- Per-request session validation at the gateway
- Permissions-based access control
- Defense-in-depth security layers

Once these patterns mature, this becomes a template for enterprise applications—drop in your business logic and inherit battle-tested security.

But we're also in parallel figuring out how to code with AI.  The project quickly became meta.  For example we want to avoid vendor lock-in for AI service providers.  We should be able to swap out and experiment with Codex and Claude and whatever comes next.  The criticism I keep hearing about using AI for development is that it's great for greenfield projects like this, but it breaks for a real company with a large complex codebase.  My hypothesis is that we simply haven't quite formalized the best practices for doing those migrations, but I think it's possible to introduce these AI development tools into a mature system by starting on leaf nodes and working your way up documenting functionality.  For example create a AGENTS.md file in a single microservice or piece of the architecture and focus on getting it working well there, and then expand outwards.  I think it's a mistake to try to use a top-down approach introducing these tools.

## Architecture

```mermaid
flowchart TB
    subgraph Clients
        Browser[Browser]
    end

    subgraph Ingress["Istio Ingress Gateway (:443)"]
        direction LR
        SSL[SSL Termination]
        EXT["ext_authz (inline on /api/*)"]
        THROTTLE[Auth-Path Throttling]
    end

    subgraph Auth["Auth Layer"]
        SG[Session Gateway<br/>OAuth2 / Sessions<br/>Port 8081]
        EA[ext_authz<br/>Session Validation<br/>Redis-backed]
    end

    subgraph Routing["NGINX API Gateway"]
        direction LR
        ROUTE[Routing + Rate Limiting]
    end

    subgraph Services["Backend Microservices"]
        TS[Transaction Service]
        CS[Currency Service]
        PS[Permission Service]
    end

    Redis[(Redis<br/>Session Store)]

    Browser -->|HTTPS| Ingress
    Ingress -->|"/auth/*, /oauth2/*, /login/oauth2/*, /logout"| SG
    Ingress -->|"/api/* (ext_authz approved)"| Routing
    Ingress -->|"/, /login (frontend)"| Routing
    EXT -.->|Session Lookup| EA
    EA -->|Session Lookup| Redis
    SG -->|Session Write| Redis
    Routing -->|mTLS| Services

    style SG fill:#e1f5fe,color:#0d47a1
    style EA fill:#f3e5f5,color:#4a148c
    style Ingress fill:#fff3e0,color:#e65100
```

### Security Layers (Defense in Depth)

| Layer | Component | Responsibility |
|-------|-----------|----------------|
| 1 | **Istio Ingress Gateway** | SSL termination, ext_authz enforcement, auth-path throttling, ingress routing |
| 2 | **Session Gateway** | OAuth2 flows, HTTP-only cookies, session management |
| 3 | **ext_authz** | Per-request session validation via Redis, header injection |
| 4 | **Backend Services** | mTLS-enforced access (Istio STRICT), data-level authorization |

### Key Security Benefits

- **Tokens never exposed to browser** — Immune to XSS token theft
- **Instant session revocation** — Redis key delete terminates access immediately
- **Identity provider abstraction** — Swap Auth0/Okta/Keycloak without client changes
- **Pluggable design** — Security infrastructure meant for reuse across organizations

## AI-Assisted Development

This project demonstrates what's achievable when AI augments development:

- **Architecture design** — Security patterns, component responsibilities, data flows
- **Implementation** — Services, configurations, and integrations
- **Documentation** — Living docs that stay current with the code
- **Code review** — Pattern consistency and security considerations

✨ *The rapid expansion from simple app to enterprise architecture was only possible through AI assistance. This isn't just a showcase—it's a proof point for AI-augmented software development.*

## Technology Stack

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React, TypeScript, Vite |
| **Backend** | Spring Boot, Java 24+, Gradle |
| **Gateway** | Istio Ingress Gateway, NGINX |
| **Auth** | OAuth2/OIDC, Auth0 |
| **Infrastructure** | Kubernetes (Kind), Tilt, Istio, Calico, Docker, PostgreSQL, Redis, RabbitMQ |

## Live Development in Kubernetes

No tradeoff between development speed and production fidelity. Edit code locally — changes reach the running Kubernetes pod in seconds without image rebuilds or pod restarts, while Istio mTLS, Calico network policies, ext_authz session validation, and TLS-encrypted infrastructure stay active around it.

| Stack | Inner-loop mechanism |
|-------|---------------------|
| **Java (Spring Boot)** | Gradle compiles on the host, Tilt syncs the JAR into the pod, process restarts — seconds |
| **React (Vite)** | Tilt syncs source files, Vite HMR hot-patches the browser — sub-second |
| **Shared library** | `service-common` change cascades to all downstream services automatically |

Most teams choose: fast local dev (unfaithful) or real Kubernetes (slow rebuilds). This setup gives both. Details in the [live development pipeline docs](https://github.com/budgetanalyzer/orchestration/blob/main/docs/development/local-environment.md#live-development-pipeline).

## Repositories

| Repository | Purpose |
|------------|---------|
| [orchestration](https://github.com/budgetanalyzer/orchestration) | Tilt + Kind development environment, Istio ingress, ext_authz, NGINX configuration |
| [session-gateway](https://github.com/budgetanalyzer/session-gateway) | OAuth2 authentication service, session management, ext_authz validation |
| [transaction-service](https://github.com/budgetanalyzer/transaction-service) | Financial transactions, accounts, and analytics API |
| [currency-service](https://github.com/budgetanalyzer/currency-service) | Currency management and exchange rates — **Demo service showcasing advanced microservice patterns** |
| [permission-service](https://github.com/budgetanalyzer/permission-service) | Role management and access delegation (RBAC) |
| [budget-analyzer-web](https://github.com/budgetanalyzer/budget-analyzer-web) | React frontend with multi-currency support |
| [service-common](https://github.com/budgetanalyzer/service-common) | Shared Java library for all backend services |
| [checkstyle-config](https://github.com/budgetanalyzer/checkstyle-config) | Shared checkstyle rules for Java services |
| [basic-repository-template](https://github.com/budgetanalyzer/basic-repository-template) | Template for creating new services |
| [workspace](https://github.com/budgetanalyzer/workspace) | **Start here** — Devcontainer entry point, single clone to get everything |

> **Note:** The `currency-service` serves as our reference implementation. It demonstrates generic patterns commonly needed in production microservices—patterns we're fleshing out to be reusable across services.

---

## License

MIT

---


