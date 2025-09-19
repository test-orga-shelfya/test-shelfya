# Getting Started with shelfya

## Overview
The **Getting Started** guide provides a step-by-step introduction to launching and developing with the shelfya project. It covers both the backend (Bun/Express + PostgreSQL) and frontend (React) applications, illustrating how these components work together as a cohesive system for local development and prototyping.

## Key Features

- **Backend Quickstart**: Run a Bun-powered server, set up a PostgreSQL database, and generate a Prisma client for database operations.
- **Frontend Quickstart**: Start the React-based web application and connect it with the backend via HTTP APIs.
- **Local Development Workflow**: Integrated Docker Compose for required infrastructure (PostgreSQL, Mailhog) and environment variable management.
- **System Interoperability**: Backend exposes public HTTP APIs, consumed by the React frontend.
- **Extensibility Guidance**: Foundation for customizing data models, API endpoints, and UI components.

## System Errors

- **Missing Environment Variables**:  
  If required environment variables (database, API keys) are unset, the backend will fail to start or connect to PostgreSQL.  
  **Resolution**: Copy `.env.example` to `.env`, fill in the required keys (see backend configuration).

- **Database Connection Error**:  
  Backend or services may fail with authentication or connection errors if the PostgreSQL instance is not running or credentials are incorrect.  
  **Resolution**: Ensure Docker Compose is running the `postgres` service, and credentials match `.env` values.

- **Port Conflicts**:  
  The default backend (`8080`) or frontend (`3000`) ports may already be in use.  
  **Resolution**: Edit the port in `.env` or the associated run scripts as needed.

- **Client-Backend Network CORS Error**:  
  If React frontend attempts to call backend APIs on a different origin and CORS is not allowed, API requests will fail.  
  **Resolution**: Ensure the backend Express app has appropriate CORS settings, and both apps run on compatible hostnames for local development.

## Usage Examples

**1. Backend Setup and Launch**
```shell
# Move into backend directory, install and run backend server
cd backend
bun i               # Install dependencies (Bun)
bun dev             # Start backend server (default: http://localhost:8080)

# (Optional) Set up the database with Prisma
bunx prisma generate    # Generate Prisma client
bunx prisma migrate dev # Run migrations
```

**2. Infrastructure Services (Database, Mailhog)**
```shell
# In the backend directory, start necessary services
docker-compose up      # Launches PostgreSQL & Mailhog for local development
```

**3. Frontend Setup and Launch**
```shell
cd client
npm install            # Install dependencies
npm start              # Runs React frontend (http://localhost:3000)
```

**4. Example Integration: Open Frontend, Connect to Backend**
- Open http://localhost:3000 (React frontend) in your browser.
- The application will make API requests to http://localhost:8080 (backend server).

## System Integration

```mermaid
flowchart LR
  subgraph Infra
    db[(PostgreSQL)]
    mail[Mailhog]
  end

  client[React App (Frontend)] -- HTTP/REST --> backend[Bun/Express API (Backend)]
  backend -- DB Connection --> db
  backend -- SMTP (Development) --> mail

  Infra --> backend
  backend --> client
```

- **Dependencies**: PostgreSQL (db), Mailhog (mail) services via Docker Compose
- **This Module**: The Getting Started flow (backend, frontend, infrastructure)
- **Used By**: Developers aiming to run or extend shelfya locally

**Details**:
- Backend: Handles all API logic, authentication, and database access.
- Frontend: React SPA consuming backend APIs.
- Process: Install dependencies, start services, and develop end-to-end.
- Consumers: Developers customizing or building applications using shelfya.

---

This guide helps you understand the high-level workflow and integration points to successfully develop or prototype with shelfya. For additional details on API endpoints or customizing logic, consult the respective backend/client documentation.