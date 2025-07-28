# Claim Status Notifier

## Project Overview

Claim Status Notifier is a lightweight C#/.NET 8 micro-service plus an optional Angular front-end that automates the “Where’s my claim?” status checks now done by hand.  First-time users can run the CLI importer to seed the database

**Back-end** – ASP.NET Core Minimal API that reads claim data from SQLite or SQL Server, exposes REST endpoints, and—on demand or on a schedule—builds a concise status digest.

**Front-end** – tiny Angular 17 SPA that lets users search a claim, view current status, and trigger an e-mail digest.

**Use case** – Upload a CSV of ClaimID,CustomerEmail,LastUpdated, store it, and let the service poll for updates or send a batch summary.


## Key Features
- Command-line CSV importer: a tiny .NET console tool that ingests your
  `ClaimID,CustomerEmail,LastUpdated` spreadsheet and seeds the database in seconds
- REST API (GET /claim/{id}, POST /notify) with async/await & LINQ  
- CSV → DB import (EF Core) and easy switch to SQL Server  
- E-mail digest stub (logs formatted body; swap in SMTP later)  
- Docker-first: single docker compose up spins API + DB  
- GitHub Actions CI: build, test, and push container image  
- Angular UI: search box, colour-coded status chip, RxJS polling  
- Tests: xUnit for API, Karma/Jasmine for Angular


## Tech Stack
| **Layer**     | **Tech**                                      |
|---------------|-----------------------------------------------|
| API           | .NET 8 • ASP.NET Core Minimal API             |
| Data          | SQLite • EF Core (Dapper optional)            |
| UI (optional) | Angular 17 • RxJS • TypeScript                |
| DevOps        | Docker & docker-compose • GitHub Actions      |
| Cloud         | AWS Elastic Beanstalk or Azure App Service    |


## Quick Start
```bash
# 1 – clone
git clone https://github.com/evoingram/claim-status-notifier.git
cd claim-status-notifier

# 2 – seed the DB with your first CSV
dotnet run --project tools/ClaimCsvParser data/claims.csv

# 3 – build & start everything in detached mode
docker compose up --build -d       # API → :5000 | Angular → :4200

# 4 – open http://localhost:4200  (UI) or http://localhost:5000/swagger  (API docs)



## Available Scripts
| **Task**              | **Command / File**                                |
|-----------------------|---------------------------------------------------|
| Run API only          | `dotnet run --project api/ClaimApi`               |
| Docker up (API + DB)  | `docker compose up -d`                            |
| Unit tests (API)      | `dotnet test`                                     |
| Unit tests (Angular)  | `npm run test --prefix web`                       |
| Lint Angular          | `npm run lint --prefix web`                       |
| Tear down containers  | `docker compose down`                             |
| Console CSV Parser    | `dotnet run --project tools/ClaimCsvParser <csv>` |


## API Documentation

### REST API v1
| **Endpoint**    | **Verb** | **Purpose**                       |
|-----------------|----------|-----------------------------------|
| `/claim/{id}`   | GET      | Return JSON status for a claim    |
| `/claim`        | GET      | List claims (paging, filter)      |
| `/import/csv`   | POST     | Upload CSV → DB                   |
| `/notify`       | POST     | Generate digest (logs e-mail body)|
| `/health`       | GET      | Liveness probe                    |

> Swagger docs are auto-generated at `/swagger`.


## Testing Targets
| **Layer** | **Framework**     | **Coverage Goal**         |
|----------|-------------------|----------------------------|
| API      | xUnit             | ≥ 80 % lines               |
| Angular  | Karma/Jasmine     | ≥ 80 % stmts               |

> (Cypress e2e scaffold included; optional.)


## Cloud Deploy
```bash
# AWS Elastic Beanstalk

eb init ClaimStatusNotifier -p docker

eb create csn-prod
```



```bash
# Azure App Service

az group create -n csn-rg -l eastus

az appservice plan create -g csn-rg -n csn-plan --is-linux

az webapp create -g csn-rg -p csn-plan -n csn-prod --deployment-container-image-name <your-image>
```

> Detailed steps live in `/infra/README-cloud.md`.


## Roadmap After MVP
- Swap log-only digest for real SMTP (SES / SendGrid).
- Listen to Claims event bus (Kafka) instead of CSV polling.
- Add JWT auth & user preferences.
- Promote DB to managed PostgreSQL for HA.
- Integrate OpenTelemetry tracing (API + Angular) and export to OTLP collector.

---
  © 2025 Erica Ingram – MIT License.
