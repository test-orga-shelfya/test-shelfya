# System Architecture

## Overview
This document provides a feature-centric overview of the Shelfya system, describing the main modules, their roles, and how the backend and frontend integrate to provide a secure, full-stack wallet and portfolio management application. Shelfya enables users to register, authenticate, manage crypto wallets, visualize historical data, and interact securely through a React frontend and an Express/Prisma/Postgres backend.

## Key Features
- **User Authentication & Authorization**: Secure user registration, login, JWT-based access & refresh token management, and role-based access controls (admin/user).
- **Wallet Management**: Users can add, view, and organize multiple wallets, each containing its transaction history.
- **Portfolio Analytics**: Visualization of wallet and currency history, supporting features like valuation graphs and trends over time.
- **Email Verification**: On registration, users are required to verify their email to activate access.
- **Rate Limiting & Security**: Login and registration endpoints are rate-limited to prevent abuse. The app uses CORS, Helmet, and secure cookie handling for robust security.
- **Prisma ORM Integration**: Reliable, type-safe interaction with a Postgres database, maintaining data integrity for users, wallets, currencies, and histories.
- **Frontend Routing & State Management**: React app with protected routes using context-managed authentication for seamless UX.

## System Errors
- **Authentication Errors**:  
  - **Invalid Token**: Shown when the access or refresh token is expired or invalid.  
    *Resolution*: Re-login or refresh session.
  - **Email Not Verified**: User attempts login without verifying email.  
    *Resolution*: Complete email verification via the link sent to email.
  - **Rate Limit Exceeded**: Too many login/registration attempts in a short period.  
    *Resolution*: Wait for the cooldown window to pass before retrying.

- **Database Connection Errors**:  
  - **DB Unavailable**: Backend cannot connect to Postgres/Prisma.  
    *Resolution*: Check database server status and DATABASE_URL configuration.
  - **Unique Constraint Violation**: Attempt to register/email/wallet that already exists.  
    *Resolution*: Use unique credentials/addresses.

- **Environment Misconfiguration**:  
  - **Missing Env Vars**: Startup validation fails if required variables are not set (e.g., JWT secrets, DB URL).  
    *Resolution*: Set all required environment variables per `REQUIRED_ENV_VARS` in the `.env` file.

## Usage Examples

### 1. User Registration & Login

```javascript
// On client: Register user
fetch(`${process.env.REACT_APP_API_URL}/api/v1/auth/register`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: "User", email: "test@example.com", password: "securePassword123!" })
});

// On client: Login user
fetch(`${process.env.REACT_APP_API_URL}/api/v1/auth/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: "test@example.com", password: "securePassword123!" }),
  credentials: 'include'
});
```

### 2. Adding a Wallet

```javascript
// Authenticated user adds a new wallet
fetch(`${process.env.REACT_APP_API_URL}/api/v1/wallets`, {
  method: 'POST',
  headers: { 
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${accessToken}`,
  },
  body: JSON.stringify({ address: "0x123...", title: "My ETH Wallet" })
});
```

### 3. Fetching Portfolio Graph Data

```javascript
// Fetch historical wallet data for graphs
fetch(`${process.env.REACT_APP_API_URL}/api/v1/wallets/{walletId}/history`, {
  headers: { 
    'Authorization': `Bearer ${accessToken}`,
  }
}).then(res => res.json());
```

## System Integration

```mermaid
flowchart LR
  subgraph Backend
    expressApi["Express API Server"]
    prisma["Prisma ORM"]
    postgres["Postgres Database"]
    constants["Config & Env Constants"]
  end

  subgraph Frontend
    reactApp["React App"]
    reactRouter["React Router"]
    authProvider["Auth Provider (Context)"]
  end

  reactApp --> reactRouter
  reactRouter --> authProvider
  authProvider -->|HTTP (fetch)| expressApi

  expressApi -->|API Calls| prisma
  prisma -->|ORM Queries| postgres
  expressApi --> constants

  expressApi -- "CORS, cookies, JWT" --> reactApp
  expressApi -- "Serve /api/v1/*" --> reactApp

  reactApp -- "Protected routes & Auth" --> authProvider
  reactApp -.->|Email verify links| expressApi

  expressApi -- "Validation, rate-limit errors" --> reactApp
```

**Legend**:  
- **Frontend**: React app routes user actions; handles authentication state and protected navigation.
- **Backend**: Express API handles requests, enforces security, and interacts with Prisma/Postgres for data persistence.
- **Integration**: HTTP (fetch/AJAX) between React and Express API; JWT and cookies for authentication session management. Enforces security policies (CORS, Helmet, rate-limiting).

This feature-centric architecture aims to clarify roles, answer "how do things connect?", and help developers quickly locate system integration points and sources of common errors.