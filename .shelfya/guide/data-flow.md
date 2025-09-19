# Data Flow Guide

## Overview
This document explains the data flow and high-level interactions between major backend and frontend modules of the Shelfya application. It highlights how authentication, profile management, wallet management, portfolio data, and history retrieval features are connected, with attention to API exposure, response patterns, and system integration points.

## Key Features

- **Authentication Flow**: Handles user login, registration, email verification, token refresh, and logout via API endpoints. Provides access and refresh tokens to manage session state between frontend and backend.
- **Profile Management**: Allows users to retrieve and update their profile, and reset passwords. Ensures email changes trigger verification.
- **Wallet Operations**: Supports adding, listing, and deleting cryptocurrency wallets tied to a user account.
- **Portfolio Data Retrieval**: Aggregates live price data and wallet balances, returning allocation, price, daily value changes, and computes current portfolio value in real-time.
- **History Retrieval**: Provides transaction and value history for a wallet, filtered by user and date.

## System Errors

- **Authentication Errors**:  
  - *Invalid Credentials*: Returned when login details are incorrect.  
    **Resolution**: Ensure email/password are correct and email is verified.
  - *Token Errors*: Invalid or expired tokens during refresh or logout.  
    **Resolution**: Login again to acquire valid tokens.

- **Wallet Errors**:  
  - *Invalid Wallet ID*: Returned when a non-numeric or non-associated wallet ID is provided.  
    **Resolution**: Verify the wallet belongs to the current user and use correct ID.
  - *Wallet Not Found*: Attempted delete or history retrieval on unknown wallet.  
    **Resolution**: Check wallet list and refer to valid wallet IDs.

- **Profile Errors**:  
  - *Duplicate Email*: Attempt to set an email already in use.  
    **Resolution**: Use a different email address.
  - *Email Not Provided*: When updating profile, email must be present.  
    **Resolution**: Provide a valid email.

- **Portfolio/History Errors**:  
  - *History Not Found*: No value history for wallet and/or date range.  
    **Resolution**: Ensure correct wallet and date filters; add wallets if none exist.

- **General Server Errors**:  
  - *Internal Server Error*: Unexpected backend failure.  
    **Resolution**: Collect error message for debugging, check for invalid input, or report to support.

## Usage Examples

```typescript
// Authentication: Login and store access token
const { data } = await API.post('/auth/login', { email: 'user@mail.com', password: 'securepw' });
TokenService.setToken(data.accessToken);

// Profile: Fetch & update
const profile = await API.get('/profile');
await API.put('/profile', { name: 'New Name', email: 'new@email.com' });

// Wallet: Create, list, and delete
await API.post('/wallets', { address: '0xabc...', title: 'My ETH wallet' });
const wallets = await API.get('/wallets');
await API.delete(`/wallets/${wallets[0].id}`);

// Portfolio: Fetch live portfolio data
const portfolio = await API.get(`/portfolio/${wallets[0].id}`);

// History: Fetch wallet history (optionally with date)
const history = await API.get(`/history/${wallets[0].id}?startDate=2024-01-01`);
```

## System Integration

```mermaid
flowchart LR
  Client["Frontend (client/src/services/api.ts)"]
      -->|REST API requests| AuthController["Auth Module"]
      -->|REST API requests| ProfileController["Profile Module"]
      -->|REST API requests| WalletController["Wallet Module"]
      -->|REST API requests| PortfolioController["Portfolio Module"]
      -->|REST API requests| HistoryController["History Module"]

  AuthController -.-> AuthService["Auth Service"]
  AuthService --> TokenService["Token Service"]
  AuthService --> EmailService["Email Service"]

  ProfileController -.-> ProfileService["Profile Service"]
  ProfileService --> EmailService

  WalletController -.-> WalletService["Wallet Service"]
  WalletService -->|Create/Fetch/Delete| PrismaDB["Database (Prisma)"]

  PortfolioController -.-> PortfolioService["Portfolio Service"]
  PortfolioService -->|Live price fetch| ExternalAPI["Crypto APIs"]
  PortfolioService -->|Value fetch| EtherscanAPI["Etherscan API"]
  PortfolioService -->|History read| PrismaDB

  HistoryController -.-> HistoryService["History Service"]
  HistoryService --> PrismaDB

  PrismaDB -.-> [Details]
  TokenService -.-> [Details]
  ProfileService -.-> [Process]
  PortfolioService -.-> [Process]
  usedBy["Consumers (React app, etc.)"]
  Client --> usedBy
```

**Legend**:  
- Controllers expose REST endpoints, process input/output, and handle errors for the client.  
- Services implement business logic and interact with the data layer or external APIs.  
- Database state and external API data are abstracted behind Services.  
- The client (`api.ts`) automatically manages tokens and refresh logic based on API errors, orchestrating smooth user experience.

---

**When to use**:  
Utilize these flows when building clients or automations that require secure authentication, wallet management, user profile management, live portfolio value, and access to historical wallet performance within the Shelfya ecosystem. This documentation focuses on how modules interact rather than how they're implemented internally.