# Unified Wellness Ecosystem - Architecture Overview

## 1. Problem Statement

A unified platform combining fitness tracking, e-commerce, crypto wallets, and personalized recommendations. It enables token-based incentives and decentralized identity management for a holistic, next-generation digital wellness experience.

---

## 2. System Goals

- Multi-tenant, scalable, and secure platform  
- Support for decentralized IDs (DIDs) and crypto wallets  
- Event-driven architecture with real-time processing  
- Personalized analytics and ML-driven features  

---

## 3. Functional Requirements

- User registration, authentication, and profile management  
- Crypto wallet creation and management  
- Fitness tracking with token-based rewards  
- Order placement, payment processing, and notifications  
- Intelligent recommendation engine  

---

## 4. Non-Functional Requirements

- High availability and fault tolerance  
- Strong observability and logging  
- API rate limiting and tenant isolation  
- CI/CD pipeline automation  

---

## 5. High-Level Architecture

- **Microservices** architecture with **gRPC** and **Kafka** for internal communication  
- **Service Mesh** (e.g., Istio) to manage secure traffic between services  
- **API Gateway** (e.g., Kong, Apigee) to control and secure external access  
- Centralized **Auth Service**  
- **Redis** for caching and **distributed object storage** for media/large files  

---

## 6. Component Design

- **AuthService**: OAuth2, JWT-based auth, mTLS, and policy enforcement  
- **WalletService**: Crypto wallet balances and smart contract interactions  
- **FitnessService**: Activity tracking and reward computation  
- **MiningService**: Token minting through interaction with BlockchainNode  

---

## 7. Data Model

- **Relational DB (PostgreSQL)**: Users, Orders, Wallets  
- **Document DB (MongoDB)**: Analytics data, Decentralized Identifiers (DIDs)  
- **Distributed Storage (IPFS/Filecoin)**: Media and large file storage  

---

## 8. API Design

- Internal communication via **gRPC**  
- External APIs exposed through **REST** and **GraphQL** via API Gateway  
- Secured with OAuth2 and JWT, with rate limits per tenant  

## The code for the mermaidchart! To visualize you can go to the [link](https://www.mermaidchart.com/app/projects/fc0cec76-197e-4826-9880-db1dffc48d7c/diagrams/a56fbabe-0c2e-4d71-8bc1-4b0e900f6037/version/v0.1/edit) also you can see the current [screenshort here](https://drive.google.com/file/d/1190e6weBLzwHtCgpEQSUE4Hp3GIK9Weq/view?usp=sharing)
```
graph LR
  %% Legend
  subgraph Legend
    style Legend fill:#f9f,stroke:#333,stroke-width:1px
    A[REST/gRPC] --- B[Kafka/Event Bus]
  end

  %% Clients
  ClientApps["Client Apps<br/>(Web, Mobile, Wallet)"]
  APIGateway["API Gateway<br/>(OAuth2, JWT, Rate Limit, Multi-Tenant)"]

  %% Central Auth System
  AuthService["Central Auth Service<br/>(OAuth2, JWT, mTLS gRPC, OPA Policies)"]
  SecretsVault["Secrets Manager<br/>(Vault)"]

  %% Service Mesh
  ServiceMesh["Service Mesh<br/>(Istio/Linkerd, mTLS, Policy, Retry)"]

  %% Observability Stack
  subgraph Observability
    Prometheus[Prometheus]
    Grafana[Grafana]
    Loki[Loki/EFK]
    Jaeger[Jaeger Tracing]
  end

  %% Microservices
  UserService["User Service<br/>(Profiles, Preferences)"]
  IdentityService["Identity Service<br/>(Decentralized ID - DID, DIDComm)"]
  WalletService["Wallet Service<br/>(Crypto Wallet, Token Balances)"]
  EcommerceService["E-Commerce Service<br/>(Catalog, Orders, Payments)"]
  FitnessService["Fitness Service<br/>(Workouts, Nutrition, Rewards)"]
  MiningService["Mining Service<br/>(Activity Tracking, Token Minting)"]
  NotificationService["Notification Service<br/>(Push, Email, SMS, Cron Jobs)"]
  EmailService["Email Service<br/>(Batch Emails, Reports via Cron)"]
  AnalyticsService["Analytics Service<br/>(Event Sourcing, ML, Dashboards)"]
  RecommendationService["Recommendation Service<br/>(Personalized Fitness & Products)"]
  BlockchainNode["Blockchain Node<br/>(Smart Contracts, Token Txn)"]
  DistributedStorage["Distributed Storage<br/>(IPFS/Filecoin)"]
  RedisCache["Redis/Memcached Cache"]
  FeatureStore["Feature Store<br/>(Feast or custom)"]

  %% Kafka Event Bus
  Kafka["Kafka Event Bus"]

  %% Databases
  Postgres["PostgreSQL<br/>(Relational DB)"]
  MongoDB["MongoDB<br/>(Document DB)"]

  %% CI/CD & DevOps
  CICD["CI/CD Pipeline<br/>(ArgoCD, GitHub Actions, Helm, K8s)"]
  Chaos["Chaos Testing<br/>(Litmus)"]

  %% Client -> API Gateway -> Auth
  ClientApps -->|REST/gRPC| APIGateway
  APIGateway -->|Auth Token Validation| AuthService
  AuthService --- SecretsVault

  %% Auth grants access to microservices
  AuthService ---|Policy Check| UserService
  AuthService --- IdentityService
  AuthService --- WalletService
  AuthService --- EcommerceService
  AuthService --- FitnessService
  AuthService --- MiningService
  AuthService --- NotificationService
  AuthService --- EmailService
  AuthService --- AnalyticsService
  AuthService --- RecommendationService

  %% gRPC synchronous calls (solid arrows)
  APIGateway -->|gRPC| UserService
  APIGateway -->|gRPC| EcommerceService
  APIGateway -->|gRPC| FitnessService
  APIGateway -->|gRPC| WalletService
  APIGateway -->|gRPC| MiningService
  APIGateway -->|gRPC| NotificationService

  UserService -->|gRPC| IdentityService
  FitnessService -->|gRPC| WalletService
  MiningService -->|gRPC| BlockchainNode
  WalletService -->|gRPC| BlockchainNode

  %% Async event-driven communication (dashed arrows)
  EcommerceService -.->|Kafka Event: OrderCreated| Kafka
  FitnessService -.->|Kafka Event: FitnessReward| Kafka
  MiningService -.->|Kafka Event: TokenMinted| Kafka
  NotificationService -.->|Kafka Event: NotificationQueued| Kafka
  EmailService -.->|Kafka Event: EmailBatchReady| Kafka

  Kafka -.-> EcommerceService
  Kafka -.-> FitnessService
  Kafka -.-> MiningService
  Kafka -.-> NotificationService
  Kafka -.-> EmailService
  Kafka -.-> AnalyticsService
  Kafka -.-> RecommendationService

  %% Cron jobs for Notification & Email Services
  subgraph CronJobs["Scheduled Jobs"]
    direction TB
    NotificationCron["Notification Cron Jobs<br/>(reminders, challenges)"]
    EmailCron["Email Cron Jobs<br/>(batch emails, reports)"]
  end

  NotificationCron --> NotificationService
  EmailCron --> EmailService

  %% Caching & Storage
  UserService --- RedisCache
  EcommerceService --- RedisCache
  FitnessService --- RedisCache
  WalletService --- RedisCache
  NotificationService --- RedisCache
  EcommerceService --- DistributedStorage
  FitnessService --- DistributedStorage

  %% Data stores
  UserService --- Postgres
  EcommerceService --- Postgres
  WalletService --- Postgres
  IdentityService --- MongoDB
  AnalyticsService --- MongoDB

  %% Feature Store & ML
  AnalyticsService --- FeatureStore
  RecommendationService --- FeatureStore

  %% Real-time notification
  NotificationService --->|"FCM|WEBSOCKET|SNS"| ClientApps

  %% Observability connections
  UserService --- Prometheus
  EcommerceService --- Prometheus
  AuthService --- Prometheus
  NotificationService --- Jaeger
  MiningService --- Loki

  %% DevOps
  CICD --- APIGateway
  CICD --- UserService
  Chaos --- FitnessService
  Chaos --- WalletService

  %% Service Mesh handles all internal service traffic
  ServiceMesh --- UserService
  ServiceMesh --- WalletService
  ServiceMesh --- EcommerceService
  ServiceMesh --- FitnessService
  ServiceMesh --- MiningService

```
