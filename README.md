# 🛒 EShop Microservices
​
A reference **e-commerce backend** built on **.NET 8** following modern microservices architecture. It demonstrates **DDD, CQRS, Vertical Slice & Clean Architecture**, **event-driven communication**, **gRPC**, distributed **caching**, and a **YARP API Gateway** — each service owning its own database and communicating asynchronously over a message broker.
​
![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-12-239120?logo=csharp&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)
​
---
​
## 📑 Table of Contents
​
- [Overview](#-overview)
- [Architecture](#-architecture)
- [Microservices](#-microservices)
- [Tech Stack](#-tech-stack)
- [Patterns & Practices](#-patterns--practices)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Service Endpoints & Ports](#-service-endpoints--ports)
- [API Gateway Routes](#-api-gateway-routes)
- [Configuration](#-configuration)
- [Roadmap](#-roadmap)
- [License](#-license)
​
---
​
## 🔎 Overview
​
EShop Microservices splits an online-shop backend into small, independently deployable services. Each service is responsible for a single business capability, owns its own data store (**database-per-service**), and exposes its own API. Synchronous, high-performance calls use **gRPC**, while cross-service workflows (e.g. checkout) are handled asynchronously through **RabbitMQ** integration events. A **YARP API Gateway** is the single entry point that routes external traffic to the right service.
​
**Key capabilities**
​
- 🧱 Independent microservices with isolated databases
- ⚡ High-performance inter-service calls via **gRPC**
- 📨 Asynchronous, event-driven workflows via **RabbitMQ + MassTransit**
- 🚪 Centralized routing, transforms & **rate limiting** via **YARP API Gateway**
- 🗄️ Redis distributed caching (cache-aside / decorator pattern)
- 🧩 Multiple architectural styles (Vertical Slice + Clean/DDD) in one solution
- 🐳 Fully containerized with **Docker Compose**
​
---
​
## 🏗 Architecture
​
```
mermaid
flowchart TB
    Client([Client])
    GW["YARP API Gateway"]
​
    subgraph Catalog["Catalog.API · Vertical Slice + CQRS"]
        C[(PostgreSQL / Marten)]
    end
    subgraph Basket["Basket.API · Vertical Slice + CQRS"]
        B[(PostgreSQL / Marten)]
        R[(Redis Cache)]
    end
    subgraph Discount["Discount.Grpc"]
        D[(SQLite)]
    end
    subgraph Ordering["Ordering · DDD + Clean Architecture"]
        O[(SQL Server)]
    end
    MQ["RabbitMQ<br>(MassTransit)"]
​
    Client --> GW
    GW -->|/catalog-service| Catalog
    GW -->|/basket-service| Basket
    GW -->|/ordering-service| Ordering
​
    Basket -- gRPC --> Discount
    Basket -->|BasketCheckout event| MQ
    MQ -->|consume| Ordering
```
​
- **Database per service** — Catalog & Basket use PostgreSQL (via Marten as a document store), Discount uses SQLite, Ordering uses SQL Server.
- **Sync communication** — Basket calls Discount over **gRPC** to apply product discounts.
- **Async communication** — On checkout, Basket publishes a `BasketCheckout` integration event to **RabbitMQ**; Ordering consumes it to create an order.
- **API Gateway** — YARP reverse proxy routes external requests and applies a fixed-window rate limiter to the ordering route.
​
---
​
## 🧩 Microservices
​
| Service | Responsibility | Architecture | Database | Communication |
| --- | --- | --- | --- | --- |
| **Catalog.API** | Product catalog management | Vertical Slice + CQRS | PostgreSQL (Marten) | REST via Gateway |
| **Basket.API** | Shopping cart & checkout | Vertical Slice + CQRS | PostgreSQL (Marten) + Redis | REST, gRPC client, publishes events |
| **Discount.Grpc** | Product discount/coupon engine | Minimal gRPC service | SQLite (EF Core) | gRPC server |
| **Ordering** | Order placement & fulfilment | DDD + Clean Architecture (API / Application / Domain / Infrastructure) | SQL Server (EF Core) | REST, consumes events |
| **YarpApiGateway** | Single entry point / reverse proxy | YARP | — | HTTP routing + rate limiting |
​
---
​
## 🛠 Tech Stack
​
**Platform:** .NET 8 · C# 12 · ASP.NET Core
​
**Data:** PostgreSQL · Marten · Redis · SQL Server · SQLite · Entity Framework Core
​
**Communication:** gRPC · RabbitMQ · MassTransit · YARP Reverse Proxy
​
**Libraries & Patterns:** MediatR (CQRS) · Carter (minimal API endpoints) · FluentValidation · Mapster · Scrutor (decorator/DI)
​
**Infrastructure:** Docker · Docker Compose
​
---
​
## 🧠 Patterns & Practices
​
- **CQRS** with MediatR — commands & queries handled in dedicated handlers.
- **Vertical Slice Architecture** (Catalog, Basket) — each feature is a self-contained slice.
- **Domain-Driven Design + Clean Architecture** (Ordering) — `Domain`, `Application`, `Infrastructure`, `API` layers with domain & integration events.
- **Cross-cutting MediatR pipeline behaviors** — validation & logging behaviors in `BuildingBlocks`.
- **Decorator pattern** (Scrutor) — Redis caching layered transparently over the basket repository.
- **Event-Driven Communication** — integration events shared via `BuildingBlocks.Messaging`.
- **Gateway Routing Pattern** — YARP routes/clusters/transforms + **rate limiting**.
- **Global exception handling & health checks** across services.
​
---
​
## 📁 Project Structure
​
```
src/
├── ApiGateways/
│   └── YarpApiGateway/            # YARP reverse proxy (routes, clusters, rate limiting)
├── BuildingBlocks/
│   ├── BuildingBlocks/            # Shared CQRS abstractions, behaviors, exceptions, pagination
│   └── BuildingBlocks.Messaging/  # MassTransit config + integration events
├── Services/
│   ├── Catalog/Catalog.API/       # Vertical Slice + CQRS, PostgreSQL/Marten
│   ├── Basket/Basket.API/         # Vertical Slice + CQRS, Redis cache, gRPC client, RabbitMQ
│   ├── Discount/Discount.Grpc/    # gRPC service, SQLite + EF Core
│   └── Ordering/
│       ├── Ordering.API/          # Endpoints / composition root
│       ├── Ordering.Application/  # CQRS handlers, behaviors
│       ├── Ordering.Domain/       # Entities, value objects, domain events
│       └── Ordering.Infrastructure/ # EF Core (SQL Server), event consumers
├── docker-compose.yml
└── docker-compose.override.yml
```
​
---
​
## 🚀 Getting Started
​
### Prerequisites
​
- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (with Docker Compose)
- A trusted HTTPS dev certificate (for running the containers over HTTPS):
  ```bash
  dotnet dev-certs https --trust
  ```
​
### Run with Docker Compose (recommended)
​
```bash
# clone
git clone https://github.com/MAAF1/EShopMicroservices.git
cd EShopMicroservices/src
​
# build & start everything (services + databases + RabbitMQ + Redis)
docker compose up -d --build
```
​
This spins up all microservices plus their backing services (PostgreSQL ×2, SQL Server, Redis, RabbitMQ).
​
### Stop
​
```bash
docker compose down          # add -v to also remove database volumes
```
​
### Run a single service locally
​
```bash
cd src/Services/Catalog/Catalog.API
dotnet run
```
​
> 💡 When running individual services outside Docker, update the connection strings in each service's `appsettings.Development.json` to point at `localhost` with the mapped ports below.
​
---
​
## 🔌 Service Endpoints & Ports
​
Host ports as mapped in `docker-compose.override.yml`:
​
| Service | HTTP | HTTPS | Swagger |
| --- | --- | --- | --- |
| Catalog.API | `http://localhost:6000` | `https://localhost:6060` | `/swagger` |
| Basket.API | `http://localhost:6001` | `https://localhost:6061` | `/swagger` |
| Discount.Grpc | `http://localhost:6002` | `https://localhost:6062` | gRPC (no Swagger) |
| Ordering.API | `http://localhost:6003` | `https://localhost:6063` | `/swagger` |
| **YarpApiGateway** | _see gateway launch settings_ | _see gateway launch settings_ | — |
​
### Backing services
​
| Component | Host Port | Credentials / DB |
| --- | --- | --- |
| Catalog DB (PostgreSQL) | `5432` | `postgres` / `postgres` · db `CatalogDb` |
| Basket DB (PostgreSQL) | `5433` | `postgres` / `postgres` · db `BasketDb` |
| Ordering DB (SQL Server) | `1433` | `sa` / `SwN12345678` · db `OrderDb` |
| Redis (distributed cache) | `6379` | — |
| RabbitMQ | `5672` (broker) · `15672` (management UI) | `guest` / `guest` |
​
> ⚠️ The credentials above are **development defaults** from the compose file. Change them before any non-local deployment.
​
---
​
## 🚪 API Gateway Routes
​
All external traffic goes through the **YARP API Gateway**, which forwards to the internal services:
​
| Public path (via Gateway) | Routed to | Notes |
| --- | --- | --- |
| `/catalog-service/{**catch-all}` | Catalog.API | — |
| `/basket-service/{**catch-all}` | Basket.API | — |
| `/ordering-service/{**catch-all}` | Ordering.API | Fixed-window **rate limiting** applied |
​
> Discount is an internal **gRPC** service consumed by Basket and is intentionally **not** exposed through the gateway.
​
**Example:** to reach the Catalog products endpoint through the gateway:
​
```
GET https://<gateway-host>/catalog-service/products
```
​
---
​
## ⚙️ Configuration
​
Key environment variables (set in `docker-compose.override.yml`):
​
- `ConnectionStrings__Database` — per-service database connection.
- `ConnectionStrings__Redis` — Basket distributed cache (`distributedcache:6379`).
- `GrpcSettings__DiscountUrl` — Basket → Discount gRPC endpoint (`https://discount.grpc:8081`).
- `MessageBroker__Host` / `__UserName` / `__Password` — RabbitMQ connection.
- `FeatureManagement__OrderFullfilment` — feature flag toggling order fulfilment.
​
---
​
## 🗺 Roadmap
​
- [ ] Authentication & authorization (e.g. Duende IdentityServer / Keycloak)
- [ ] Centralized logging & distributed tracing (OpenTelemetry, Seq/Jaeger)
- [ ] Resilience policies (Polly retries, circuit breakers)
- [ ] CI/CD pipeline & Kubernetes/Helm deployment
- [ ] Web UI (Blazor / Shopping.Web)
​
---
​
## 📄 License
​
This project is licensed under the **MIT License** — see the `LICENSE` file for details.
​
---
​
## 🙌 Acknowledgements
​
Built while studying modern .NET microservices architecture (DDD, CQRS, Vertical/Clean Architecture, event-driven design).
​
