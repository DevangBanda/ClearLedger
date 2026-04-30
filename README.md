<div align="center">

```
 ██████╗██╗     ███████╗ █████╗ ██████╗ ██╗     ███████╗██████╗  ██████╗ ███████╗██████╗
██╔════╝██║     ██╔════╝██╔══██╗██╔══██╗██║     ██╔════╝██╔══██╗██╔════╝ ██╔════╝██╔══██╗
██║     ██║     █████╗  ███████║██████╔╝██║     █████╗  ██║  ██║██║  ███╗█████╗  ██████╔╝
██║     ██║     ██╔══╝  ██╔══██║██╔══██╗██║     ██╔══╝  ██║  ██║██║   ██║██╔══╝  ██╔══██╗
╚██████╗███████╗███████╗██║  ██║██║  ██║███████╗███████╗██████╔╝╚██████╔╝███████╗██║  ██║
 ╚═════╝╚══════╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝╚══════╝╚═════╝  ╚═════╝ ╚══════╝╚═╝  ╚═╝
```

### **Enterprise-Grade Full-Stack Digital Banking Platform**
*Microservices · Event-Driven · Cloud-Native · Production-Ready*

---

[![Java](https://img.shields.io/badge/Java-17-f89820?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-6db33f?style=for-the-badge&logo=springboot&logoColor=white)](https://spring.io/)
[![React](https://img.shields.io/badge/React_+_Vite-Frontend-61dafb?style=for-the-badge&logo=react&logoColor=black)](https://reactjs.org/)
[![Kafka](https://img.shields.io/badge/Apache_Kafka-Event_Driven-231f20?style=for-the-badge&logo=apachekafka)](https://kafka.apache.org/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestrated-326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ed?style=for-the-badge&logo=docker&logoColor=white)](docker-compose.yml)
[![License](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)](LICENSE)

</div>

---

## 🏦 Overview

**ClearLedger** is a production-inspired enterprise banking platform built from the ground up — not a CRUD demo.

It demonstrates how modern FinTech companies architect digital banking at scale: 9 independently deployable microservices, event-driven communication via Kafka, RS256 JWT security, live Razorpay payment integration, full observability with Prometheus + Grafana, and complete Kubernetes orchestration with HPA auto-scaling.

> Built to showcase end-to-end ownership across backend engineering, distributed systems, DevOps, and cloud-native architecture in a single cohesive project.

---

## ✅ What's Built

| | Capability | Implementation |
|---|---|---|
| ✅ | **Bank-Grade Security** | JWT RS256 asymmetric signing · RBAC (Admin/Manager/User) · Redis token store · BCrypt · OTP |
| ✅ | **Event-Driven Architecture** | Apache Kafka — transaction events, notification triggers, audit trails |
| ✅ | **Live Payment Gateway** | Razorpay SDK — real money top-up with signature verification |
| ✅ | **9 Microservices** | Independently deployable, each owns its own DB schema and domain |
| ✅ | **Service Discovery** | Netflix Eureka server + Feign Clients for sync inter-service calls |
| ✅ | **Full Observability** | Prometheus metrics scraping + Grafana dashboards per service |
| ✅ | **One-Command Local Stack** | Docker Compose — full platform up including infra in one command |
| ✅ | **Kubernetes Orchestration** | HPA auto-scaling · rolling deployments · liveness/readiness probes |
| ✅ | **Redis Caching** | Low-latency reads · session management · per-IP rate limiting |
| ✅ | **React Frontend** | Role-based UI · real-time transfers · loan wizard · Razorpay modal |

## 🔜 Roadmap

| | Planned Feature |
|---|---|
| 🔜 | **Fraud Detection Service** — ML-based risk scoring + AML rule engine |
| 🔜 | **Distributed Tracing** — Zipkin / Jaeger across all service hops |
| 🔜 | **Config Server** — centralized external configuration management |
| 🔜 | **Circuit Breaker** — Resilience4j fallbacks per service |
| 🔜 | **Elasticsearch** — full-text audit log search + transaction history queries |

---

## 🏗️ System Architecture

```
                        ┌─────────────────────────────────────┐
                        │           CLIENT LAYER               │
                        │     React + Vite SPA (port 9000)    │
                        │  Customer Portal · Admin Dashboard   │
                        └──────────────────┬──────────────────┘
                                           │ HTTPS
                   ┌───────────────────────▼───────────────────────┐
                   │            API GATEWAY  :8080                  │
                   │  JWT Validation · Rate Limiting · Routing      │
                   └──┬──────┬──────┬──────┬──────┬──────┬────────┘
                      │      │      │      │      │      │
             ┌────────▼┐ ┌───▼───┐ ┌▼─────┐ ┌────▼┐ ┌───▼───┐ ┌──▼──────┐
             │  Auth   │ │ User  │ │Acct. │ │ Txn │ │ Loan  │ │Payment  │
             │  :8081  │ │ :8082 │ │:8083 │ │:8084│ │ :8086 │ │  :8089  │
             └────┬────┘ └───────┘ └──────┘ └──┬──┘ └───────┘ └─────────┘
                  │                             │
             ┌────▼──────────────┐    ┌─────────▼──────────────────────┐
             │   KYC  :8087      │    │     Apache Kafka Event Bus      │
             │ Doc Verification  │    │ txn · notification · audit      │
             └───────────────────┘    └────────────┬───────────────────┘
                                                   │
             ┌─────────────────────────────────────▼──────────────────┐
             │          Notification :8085  ·  Admin :8088            │
             │     Email/SMS Alerts         System Controls           │
             └────────────────────────────────────────────────────────┘
                                           │
          ┌────────────────────────────────┼───────────────────────────┐
          │                                │                           │
  ┌───────▼────────┐              ┌────────▼───────┐          ┌────────▼───────┐
  │  MySQL  :3306  │              │  Redis  :6379  │          │ Eureka  :8761  │
  │  Per-service   │              │  Cache · Tokens│          │  Service Reg.  │
  │  schema        │              │  Rate Limits   │          │  + Discovery   │
  └────────────────┘              └────────────────┘          └────────────────┘
          │
  ┌───────▼────────────────────────────────────────────────────────────┐
  │                    Observability Layer                              │
  │          Prometheus  :9090  ·  Grafana  :3000                      │
  │          Spring Boot Actuator — /health · /metrics · /info         │
  └────────────────────────────────────────────────────────────────────┘
```

---

## 🧩 Microservices

### Built

| Service | Port | Responsibility |
|---|---|---|
| **API Gateway** | `8080` | Single entry point — JWT validation, rate limiting, intelligent routing |
| **Auth Service** | `8081` | Login, registration, JWT RS256 issuance, refresh token lifecycle |
| **User Service** | `8082` | User profiles, KYC management, account linking |
| **Account Service** | `8083` | Bank account creation, balance management, account types |
| **Transaction Service** | `8084` | Fund transfers, transaction validation, ledger history |
| **Notification Service** | `8085` | Email and SMS alerts triggered by Kafka events |
| **Loan Service** | `8086` | Loan applications, EMI calculations, repayment lifecycle |
| **KYC Service** | `8087` | Identity verification, document upload, KYC approval workflow |
| **Payment Service** | `8089` | Razorpay integration — account top-up and payment verification |
| **Admin Service** | `8088` | Admin dashboard, user management, system-wide controls |
| **Eureka Server** | `8761` | Service discovery and health registry |

### Planned

| Service | Purpose |
|---|---|
| **Fraud Detection** | ML rule engine, AML checks, risk scoring per transaction |
| **Config Server** | Centralized external configuration for all services |
| **Audit Service** | Immutable audit log with Elasticsearch-backed search |

---

## 🔐 Security Model

```
┌──────────────────────────────────────────────────────────────────┐
│                       SECURITY LAYERS                            │
├──────────────────────────────────────────────────────────────────┤
│  1. JWT RS256          Asymmetric sign (private) / verify (pub)  │
│  2. RBAC               ADMIN · MANAGER · USER role hierarchy     │
│  3. Redis Token Store  Stateless sessions with instant revoke    │
│  4. BCrypt Hashing     Salted password storage                   │
│  5. OTP Verification   Required for high-value transfers         │
│  6. Rate Limiting      API Gateway — per-IP request throttling   │
└──────────────────────────────────────────────────────────────────┘
```

**Why RS256?** Unlike HS256 (symmetric), RS256 uses a **private key to sign** and a **public key to verify** — only the Auth Service can issue tokens while all other services validate them independently. True zero-trust architecture.

---

## 🖥️ Frontend (React + Vite)

**Stack:** React · Vite · Tailwind CSS · Axios · Context API · React Router

<p align="center">
  <img src="assets/img.png" alt="ClearLedger Dashboard" />
</p>

**Features:**
- JWT-based auth with silent token refresh
- Role-based UI — Admin controls hidden from standard users
- Real-time fund transfers with OTP confirmation
- Transaction history with filtering and pagination
- Loan application wizard with live EMI preview
- Razorpay payment modal for account top-up
- Fully responsive across all screen sizes

---

## 📊 Monitoring & Observability

<p align="center">
  <img src="assets/first.png" alt="Grafana Dashboard" />
  <img src="assets/se.png" alt="Prometheus Metrics" />
</p>

| Tool | Role |
|---|---|
| **Prometheus** | Scrapes metrics from Spring Boot Actuator on all services |
| **Grafana** | Real-time dashboards, alerting, per-service visualizations |
| **Spring Actuator** | Exposes `/health`, `/metrics`, `/info` per service |

**Tracked Metrics:**
- Request throughput and latency (p50, p95, p99) per service
- HTTP error rates and 5xx spikes
- Kafka consumer lag per topic
- JVM heap, GC pauses, thread pool saturation
- MySQL query performance and connection pool usage
- Redis cache hit/miss ratio
- Business metrics: transactions/sec, active loans, KYC approval rate

---

## 🐳 DevOps & Deployment

### Docker Compose — Local / Dev

The full platform starts with a single command:

```bash
git clone https://github.com/DevangBanda/ClearLedger.git
cd ClearLedger
docker-compose up -d
```

Starts automatically:
- All 11 microservices
- MySQL (schema auto-init)
- Redis
- Apache Kafka + Zookeeper
- Prometheus + Grafana
- Eureka Server

> First startup takes 3–5 minutes as images pull and services initialize.

### Kubernetes — Production

```bash
# Deploy all manifests
kubectl apply -f K8s/

# Watch pods come up
kubectl get pods -n clearledger --watch

# Scale a service
kubectl scale deployment transaction-service --replicas=3 -n clearledger
```

**K8s Features:**
- HPA auto-scaling — services scale under load automatically
- Rolling deployments — zero-downtime updates
- Liveness and readiness probes — automatic restart of unhealthy pods
- Kubernetes Secrets — credentials never in plain config
- Prometheus + Grafana deployed in-cluster

---

## 🛠️ Tech Stack

### Backend
| Technology | Version | Role |
|---|---|---|
| Java | 17 | Core language |
| Spring Boot | 3.x | Microservice framework |
| Spring Security | 6.x | Auth and RBAC |
| Spring Cloud Gateway | Latest | API Gateway |
| Spring Cloud Eureka | Latest | Service discovery |
| Spring Cloud OpenFeign | Latest | Sync inter-service HTTP |
| Apache Kafka | Latest | Async event streaming |
| MySQL | 8.x | Primary relational database |
| Redis | 7.x | Cache and token store |
| Razorpay Java SDK | Latest | Payment gateway |

### Frontend
| Technology | Role |
|---|---|
| React | UI framework |
| Vite | Build tool and dev server |
| Tailwind CSS | Utility-first styling |
| Axios | HTTP client |
| React Router | Client-side routing |
| Context API | Global state |

### DevOps & Infra
| Technology | Role |
|---|---|
| Docker | Service containerization |
| Docker Compose | Local orchestration |
| Kubernetes | Production orchestration |
| Prometheus | Metrics collection |
| Grafana | Dashboards and alerting |
| Spring Boot Actuator | Metrics endpoint exposure |

---

## 🚀 Quick Start

**Prerequisites:**
```
Docker & Docker Compose
Git
kubectl + K8s cluster (optional, for K8s deployment only)
```

```bash
# 1. Clone
git clone https://github.com/DevangBanda/ClearLedger.git
cd ClearLedger

# 2. Start everything
docker-compose up -d

# 3. Open the frontend
open http://localhost:9000
```

---

## 🌐 Service URLs

| Service | URL |
|---|---|
| **Frontend** | http://localhost:9000 |
| **API Gateway** | http://localhost:8080 |
| **Auth Service** | http://localhost:8081 |
| **User Service** | http://localhost:8082 |
| **Account Service** | http://localhost:8083 |
| **Transaction Service** | http://localhost:8084 |
| **Notification Service** | http://localhost:8085 |
| **Loan Service** | http://localhost:8086 |
| **KYC Service** | http://localhost:8087 |
| **Admin Service** | http://localhost:8088 |
| **Payment Service** | http://localhost:8089 |
| **Eureka Dashboard** | http://localhost:8761 |
| **Grafana** | http://localhost:3000 |
| **Prometheus** | http://localhost:9090 |

---

## 👨‍💻 What This Project Demonstrates

| Domain | Evidence |
|---|---|
| **Backend Engineering** | 9 Spring Boot microservices with clean domain boundaries and owned schemas |
| **Distributed Systems** | Kafka event bus, Eureka discovery, Feign inter-service calls, Redis across services |
| **Security Engineering** | RS256 JWT, RBAC, Redis session store, OTP flows, BCrypt, rate limiting |
| **Payment Integration** | Real Razorpay SDK — order creation, payment capture, signature verification |
| **Event-Driven Design** | Kafka producers/consumers with DLQ handling and retry semantics |
| **DevOps** | Full Docker Compose stack + complete K8s manifests with HPA and probes |
| **Observability** | Prometheus scraping + Grafana dashboards wired to Spring Actuator |
| **Frontend Engineering** | React + Vite + Tailwind with JWT auth, role-based routing, real-time UI |
| **System Design** | End-to-end ownership across infra, backend, frontend, and monitoring |

---

<div align="center">

**Built with precision. Designed for scale. Ready for production.**

*ClearLedger — Enterprise Digital Banking, End to End.*

</div>
