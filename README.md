# Smart Budget Platform

A modern, cloud‑ready financial management system for tracking income, expenses, budgets, assets, and liabilities. Built with a Clean Architecture approach and intended for local development, CI/CD, and deployment to Azure.

Table of Contents
- [Project Overview](#project-overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture & Project Layout](#architecture--project-layout)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Environment variables](#environment-variables)
  - [Local setup (Backend)](#local-setup-backend)
  - [Local setup (Frontend)](#local-setup-frontend)
- [Database: Migrations & Seeding](#database-migrations--seeding)
- [Authentication & Roles](#authentication--roles)
- [Testing](#testing)
- [API Documentation](#api-documentation)
- [Deployment (Azure)](#deployment-azure)
- [CI/CD (GitHub Actions) — sample workflow](#cicd-github-actions---sample-workflow)
- [Contributing](#contributing)
- [License](#license)

Project Overview
A full-stack application where users can:
- Track expenses & income (add/edit/delete transactions)
- Categorise transactions (Food, Rent, Utilities, etc.)
- Set monthly and category budgets + overspending alerts
- View analytics (trends, breakdowns, monthly summaries)
- Track assets & liabilities and calculate net worth
- Export reports (CSV/PDF optional)
- Multi-user support with JWT authentication and role-based access (Admin / User)

Features
- REST API (ASP.NET Core 8)
- EF Core + SQL Server (migrations & seeding)
- JWT Authentication and Role-based Authorisation
- Frontend: React + Vite + TypeScript
- Charts: Recharts (or Chart.js)
- TailwindCSS for styling
- Clean Architecture: Controllers → Services → Repositories → Persistence
- Logging with Serilog
- Validation with FluentValidation
- Unit & integration tests
- CI/CD pipelines and Azure deployment support

Tech Stack
- Backend: .NET 8, ASP.NET Core Web API, EF Core
- Database: SQL Server / Azure SQL
- Frontend: React, Vite, TypeScript, Axios, React Query, TailwindCSS
- Charts: Recharts
- Infrastructure: Azure App Service, Azure SQL, Azure Storage
- CI/CD: GitHub Actions

Architecture & Project Layout (suggested)
- /src
  - /Api (ASP.NET Core Web API project)
  - /Application (business rules, DTOs, interfaces, validators)
  - /Domain (entities, value objects)
  - /Infrastructure.Persistence (EF Core DbContext, migrations, repositories)
  - /Infrastructure.Identity (JWT, authentication services)
  - /Tests (unit & integration tests)
- /frontend
  - /web (React + Vite application)

Getting Started

Prerequisites
- .NET 8 SDK
- Node.js (v18+ recommended) and npm or pnpm
- SQL Server (local or Docker) or Azure SQL
- (Optional) Docker & Docker Compose
- Git

Environment variables

Backend (example in appsettings.Development.json or environment variables)
- ConnectionStrings__DefaultConnection = "Server=...;Database=SmartBudgetDb;User Id=...;Password=...;"
- Jwt__Key = "<very-strong-secret>"
- Jwt__Issuer = "SmartBudget"
- Jwt__Audience = "SmartBudgetClient"
- Jwt__ExpiresInMinutes = 60
- Serilog__MinimumLevel__Default = "Information"

Frontend (.env.local or Vite)
- VITE_API_URL = "http://localhost:5000/api"
- VITE_APP_TITLE = "Smart Budget Platform"

Local setup (Backend)
1. Clone the repo
   - git clone https://github.com/<owner>/Smart-Budget-Platform.git
   - cd Smart-Budget-Platform

2. Restore & build
   - dotnet restore
   - dotnet build

3. Configure connection string & JWT secrets (see above). For local development consider `dotnet user-secrets` or an `appsettings.Development.json` file.

4. Apply EF Migrations & seed data
   - dotnet ef database update --project src/Infrastructure.Persistence --startup-project src/Api
   - (Or run your provided migration/seed script)

5. Run the API
   - dotnet run --project src/Api
   - By default Swagger UI will be available at: https://localhost:<port>/swagger

Local setup (Frontend)
1. cd frontend/web
2. Install dependencies
   - npm install
3. Run dev server
   - npm run dev
4. The frontend will proxy API requests to VITE_API_URL. Open the app at http://localhost:5173 (or as printed by Vite).

Database: Migrations & Seeding
- Add a migration:
  - dotnet ef migrations add InitialCreate --project src/Infrastructure.Persistence --startup-project src/Api
- Apply migrations:
  - dotnet ef database update --project src/Infrastructure.Persistence --startup-project src/Api
- Seeding:
  - Implement a DbInitializer/Seed method that runs on app startup (development only).
  - Seed sample categories, admin user, sample transactions and budgets.

Authentication & Roles
- JWT is used for authentication. Configure a secure Jwt:Key and set issuer/audience.
- Roles: Admin and User. Seed an Admin account or implement a registration flow that allows an Admin to be created.
- Protect API endpoints with role-based policies (e.g., [Authorize(Roles = "Admin")]).

Testing
- Backend unit tests:
  - dotnet test ./src/Tests
- Integration tests:
  - Use an in-memory database or a test SQL Server instance (Docker).
- Frontend tests:
  - npm run test (set up with Jest/React Testing Library as desired)

API Documentation
- Swagger/OpenAPI is included in the API project (development). Visit /swagger to explore endpoints and try requests.
- Consider using Swashbuckle or NSwag for generating clients and documentation.

Deployment (Azure)
1. Azure SQL
   - Create an Azure SQL server & database.
   - Configure firewall rules and obtain the connection string.
   - Set connection string in App Service as a secure setting (ConnectionStrings__DefaultConnection).

2. Azure App Service
   - Create an App Service for the API (Linux or Windows).
   - Configure application settings: JWT secret, connection strings, environment variables.
   - Enable managed identity if you need access to Azure resources.

3. Static frontend
   - Build the frontend: npm run build
   - Deploy to Azure Static Web Apps, Azure Blob Storage + CDN, or serve from a separate App Service.

4. Migrations
   - Either run EF migrations during deployment from a startup task or run a release script as part of your pipeline:
     - dotnet ef database update --project src/Infrastructure.Persistence --startup-project src/Api --environment Production

Security & Secrets
- Never check in secrets. Use Azure Key Vault or App Service application settings to store secrets in production.
- Use HTTPS everywhere and rotate JWT keys periodically.

CI/CD (GitHub Actions) — sample workflow
- A minimal approach:
  - Build & test backend
  - Build & test frontend
  - Publish artifacts
  - Deploy to Azure (uses azure/webapps-deploy action or azure/static-web-apps-deploy)
