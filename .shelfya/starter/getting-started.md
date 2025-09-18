# Getting Started

## Overview
This starter module provides the essential setup and orchestration for the Shelfya wallet tracker system. Its purpose is to unify backend and client bootstrapping, database initialization, and offer guidance for new contributors. It ensures that both the backend API and client interface are correctly connected, configured, and ready for development or demo usage.

## Key Features
- **Unified Project Orchestration**: Coordinates setup for both backend (API) and frontend (React client) components, allowing contributors to easily spin up the entire stack.
- **Database & Email Service Initialization**: Leverages `docker-compose` to provision a local PostgreSQL database and mock email server (Mailhog) required for core features (such as authentication, email verification, and wallet management).
- **Development Workflow Guidance**: Provides documented commands and flows for launching development servers and running key development scripts.
- **API/Client Route Overview**: Clearly lists and summarizes all public API endpoints and client routes so developers know integration paths and system touchpoints.

## System Errors
- **Service Connection Errors**: If Docker services (Postgres, Mailhog) are not running, backend authentication and email verification features will fail.  
  _Resolution_: Ensure `docker-compose up` has been executed in the backend directory before starting the backend server.
- **Port Conflicts**: If ports 5432 (DB), 1025/8025 (Mailhog), or 3000 (client) are in use, startup will fail.  
  _Resolution_: Stop conflicting services or customize port mappings as needed in `docker-compose.yml` or relevant configs.
- **Environment Misconfiguration**: Missing or incorrect environment variables (e.g., database credentials) will cause backend launch or migration issues.  
  _Resolution_: Populate the required `.env` files as referenced in the backend and ensure variable values align with those in `docker-compose.yml`.

## Usage Examples

```bash
# 1. Start backend dependencies (PostgreSQL and Mailhog)
cd backend
docker-compose up -d

# 2. Install backend dependencies and launch development API server
bun i          # Install all backend dependencies
bun dev        # Start backend development server (by default on http://localhost:3001)

# 3. (Optional) Generate Prisma client after DB changes
bunx prisma generate

# 4. In a new terminal, start the frontend React client
cd ../client
npm install    # Install frontend dependencies
npm start      # Runs client on http://localhost:3000

# Backend API root:    http://localhost:3001/api/v1
# Client app root:     http://localhost:3000/
# Mailhog interface:   http://localhost:8025/
```

## System Integration

```mermaid
flowchart LR
  devEnv["Developer Machine"]
  
  subgraph Backend
    depsPg["PostgreSQL DB (docker)"]
    depsMH["Mailhog (docker)"]
    backendSrv["Backend Server (bun/express)"]
  end

  subgraph Frontend
    clientApp["React Client"]
  end

  devEnv-->|"docker-compose up"|depsPg
  devEnv-->|"docker-compose up"|depsMH
  devEnv-->|"bun dev / bun i"|backendSrv
  devEnv-->|"npm start"|clientApp

  clientApp-->|"HTTP|API"/"REST calls"|backendSrv
  backendSrv-->|"DB queries"|depsPg
  backendSrv-->|"SMTP|email flow"|depsMH

  backendSrv-->|"public APIs"|clientApp

  depsPg["[Postgres DB]"]:::details
  depsMH["[Mailhog Email]"]:::details
  backendSrv["[API Process]"]:::process
  clientApp["[User App]"]:::consumers
```
