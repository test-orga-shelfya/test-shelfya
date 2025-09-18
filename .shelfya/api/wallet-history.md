# Wallet History API

## Overview
The Wallet History module provides a public API for retrieving a user's wallet transaction history. It enables client applications to fetch filtered historical records of wallet activities, supporting secure, user-scoped access and optional date-based filtering. This module plays a key role in surfacing wallet events to users and integrating with account overviews or audit tools.

## Key Features
- **Get Wallet History**: Retrieve the transaction history for a specific wallet belonging to the authenticated user, with optional start-date filtering.
- **User-Scoped Query**: Ensures only the wallet history belonging to the requesting user is retrievable.
- **Flexible Filtering**: Supports filtering history by wallet ID and optional minimum (start) date.
- **Error Signaling**: Distinguishes between invalid requests, empty results, and server errors for reliable client interaction.

## System Errors
- **Invalid Wallet ID**:  
  Returned when the wallet ID provided is not a valid number.  
  _Resolution_: Ensure the wallet ID in the request path is a valid integer.
- **Wallet History Not Found**:  
  Returned when there are no history records found for the specified wallet or filters.  
  _Resolution_: Verify the wallet exists, belongs to the authenticated user, and contains transactions matching the filters.
- **Internal Server Error**:  
  Covers unforeseen failures, such as database or parsing errors.  
  _Resolution_: Check server logs for error details; may require infrastructure or code investigation.

## Usage Examples

```typescript
// Usage within an Express route (REST API)

GET /wallets/:id/history?startDate=2024-01-01

Request headers:
Authorization: Bearer <user-token>

Sample response (200):
[
  {
    "id": 12,
    "walletId": 42,
    "type": "credit",
    "amount": 1200,
    "date": "2024-03-15T10:30:00Z",
    ...
  },
  ...
]

Failure response (404):
{ "error": "Wallet history not found" }

Failure response (400):
{ "error": "Invalid wallet id" }
```

## System Integration

```mermaid
flowchart LR
  user_app["User Application (Frontend)"]
    --> api_gateway["API Gateway / Express Router"]
    --> walletHistoryController["Wallet History Controller"]
    --> filtersSchema["Filters Schema\n(validation)"]
    --> walletHistoryService["Wallet History Service"]
    --> database["Database (Prisma)"]
    --> user_app

  user_app -.->|"HTTP Request"\n(GET /wallets/:id/history)" api_gateway
  api_gateway --> walletHistoryController
  walletHistoryController -->|Validates & parses| filtersSchema
  walletHistoryController -->|Fetches data| walletHistoryService
  walletHistoryService -->|Queries via Prisma| database
  database -- history records --> walletHistoryService
  walletHistoryService -->|Returns data| walletHistoryController
  walletHistoryController -->|Formats response| api_gateway
  api_gateway -.->|"HTTP Response"\n(history data or error)| user_app

  %% Architecture Tags
  click walletHistoryController "This Module"
  click filtersSchema "[Process]"
  click api_gateway "[Consumers]"
```