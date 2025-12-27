# Architecture Documentation

This document provides an in-depth technical overview of the Christmas NFT Marketplace architecture, design decisions, and implementation details.

---

## Table of Contents

- [System Overview](#system-overview)
- [High-Level Architecture](#high-level-architecture)
- [Frontend Architecture](#frontend-architecture)
- [Backend Architecture](#backend-architecture)
- [Database Design](#database-design)
- [Web3 Integration](#web3-integration)
- [State Management](#state-management)
- [API Design](#api-design)
- [Security Architecture](#security-architecture)
- [Performance Considerations](#performance-considerations)
- [Deployment Architecture](#deployment-architecture)

---

## System Overview

The Christmas NFT Marketplace is a three-tier web application that follows modern web development patterns:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              THREE-TIER ARCHITECTURE                             │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                               PRESENTATION TIER                                  │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                         React.js Application                             │  │
│   │                                                                          │  │
│   │   • Single Page Application (SPA)                                        │  │
│   │   • Component-based UI architecture                                      │  │
│   │   • Context API for state management                                     │  │
│   │   • ethers.js for blockchain interaction                                 │  │
│   │   • Axios for HTTP communication                                         │  │
│   │                                                                          │  │
│   │   Port: 5173 (development)                                               │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       │ REST API (JSON)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                APPLICATION TIER                                  │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                        Express.js Server                                 │  │
│   │                                                                          │  │
│   │   • RESTful API endpoints                                                │  │
│   │   • MVC architectural pattern                                            │  │
│   │   • Middleware pipeline (auth, validation, logging)                      │  │
│   │   • Passport.js authentication                                           │  │
│   │   • Business logic controllers                                           │  │
│   │                                                                          │  │
│   │   Port: 4000                                                             │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       │ Mongoose ODM
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  DATA TIER                                       │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                           MongoDB                                        │  │
│   │                                                                          │  │
│   │   • Document-based NoSQL database                                        │  │
│   │   • Flexible schema design                                               │  │
│   │   • Text indexing for search                                             │  │
│   │   • Reference-based relationships                                        │  │
│   │                                                                          │  │
│   │   Port: 27017 (default)                                                  │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## High-Level Architecture

### Component Interaction Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         COMPONENT INTERACTION DIAGRAM                            │
└─────────────────────────────────────────────────────────────────────────────────┘

                                    ┌─────────────────┐
                                    │     Browser     │
                                    │                 │
                                    │  ┌───────────┐  │
                                    │  │  MetaMask │  │
                                    │  │ Extension │  │
                                    │  └─────┬─────┘  │
                                    │        │        │
                                    │  ┌─────▼─────┐  │
                                    │  │   React   │  │
                                    │  │    App    │  │
                                    │  └─────┬─────┘  │
                                    └────────┼────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
                    ▼                        ▼                        ▼
         ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
         │   ethers.js     │      │     Axios       │      │    CSS/Assets   │
         │                 │      │                 │      │                 │
         │  • Provider     │      │  • API calls    │      │  • Styling      │
         │  • Signer       │      │  • Interceptors │      │  • Images       │
         │  • Contracts    │      │  • Error handle │      │  • Animations   │
         └────────┬────────┘      └────────┬────────┘      └─────────────────┘
                  │                        │
                  │                        ▼
                  │               ┌─────────────────┐
                  │               │  Express.js     │
                  │               │     Server      │
                  │               │                 │
                  │               │  ┌───────────┐  │
                  │               │  │Middleware │  │
                  │               │  └─────┬─────┘  │
                  │               │        │        │
                  │               │  ┌─────▼─────┐  │
                  │               │  │  Routes   │  │
                  │               │  └─────┬─────┘  │
                  │               │        │        │
                  │               │  ┌─────▼─────┐  │
                  │               │  │Controllers│  │
                  │               │  └─────┬─────┘  │
                  │               │        │        │
                  │               │  ┌─────▼─────┐  │
                  │               │  │  Models   │  │
                  │               │  └─────┬─────┘  │
                  │               └────────┼────────┘
                  │                        │
                  │                        ▼
                  │               ┌─────────────────┐
                  │               │    MongoDB      │
                  │               │                 │
                  │               │  Collections:   │
                  │               │  • users        │
                  │               │  • nfts         │
                  │               │  • auctions     │
                  │               │  • buys         │
                  │               │  • bids         │
                  │               │  • wallets      │
                  │               │  • histories    │
                  │               │  • transactions │
                  │               └─────────────────┘
                  │
                  ▼
         ┌─────────────────┐
         │   Ethereum      │
         │   Blockchain    │
         │                 │
         │  • Transactions │
         │  • Smart Contr. │
         │  • Token Trans. │
         └─────────────────┘
```

---

## Frontend Architecture

### Directory Structure Pattern

```
src/
├── assets/           # Static resources (images, fonts)
├── components/       # Reusable UI components (atoms/molecules)
├── container/        # Page sections (organisms/templates)
├── context/          # React Context providers
├── services/         # External service integrations
├── App.js           # Root component
└── index.js         # Application entry point
```

### Component Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           COMPONENT HIERARCHY                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

                              index.js
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │      WalletProvider    │ ◄─── Context Provider
                    │    (WalletContext.js)  │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │         App.js         │ ◄─── Root Component
                    └───────────┬────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
   │   Navbar     │     │   Header     │     │ Copyright    │
   │  (component) │     │ (container)  │     │ (component)  │
   └──────┬───────┘     └──────────────┘     └──────────────┘
          │
          ▼                     │
   ┌──────────────┐             │
   │WalletButton  │             ▼
   │  (component) │     ┌──────────────┐     ┌──────────────┐
   └──────────────┘     │    Date      │     │    Gifts     │
                        │ (container)  │     │ (container)  │
                        └──────┬───────┘     └──────────────┘
                               │
                               ▼
                        ┌──────────────┐
                        │  Countdown   │
                        │  (component) │
                        └──────────────┘

                                │
                                ▼
                        ┌──────────────┐
                        │ Marketplace  │
                        │ (container)  │
                        └──────┬───────┘
                               │
                               ▼
                        ┌──────────────┐
                        │   NFTCard    │
                        │  (component) │ x N
                        └──────────────┘
```

### State Management Flow

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          STATE MANAGEMENT PATTERN                                │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                              WALLET CONTEXT                                      │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │  State:                                                                  │  │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │  │
│   │  │   account   │  │  provider   │  │   signer    │  │isConnecting │    │  │
│   │  │   string    │  │   object    │  │   object    │  │   boolean   │    │  │
│   │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │  Actions:                                                                │  │
│   │  ┌──────────────────┐  ┌───────────────────┐  ┌──────────────────┐     │  │
│   │  │  connectWallet   │  │ disconnectWallet  │  │  formatAddress   │     │  │
│   │  │    function      │  │     function      │  │    function      │     │  │
│   │  └──────────────────┘  └───────────────────┘  └──────────────────┘     │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       │ useContext(WalletContext)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            CONSUMING COMPONENTS                                  │
│                                                                                  │
│   ┌───────────────┐    ┌───────────────┐    ┌───────────────┐                  │
│   │ WalletButton  │    │  Marketplace  │    │    NFTCard    │                  │
│   │               │    │               │    │               │                  │
│   │ • Connect/    │    │ • Check auth  │    │ • Buy action  │                  │
│   │   Disconnect  │    │ • API calls   │    │ • Bid action  │                  │
│   │ • Show addr   │    │ • Statistics  │    │ • Display     │                  │
│   └───────────────┘    └───────────────┘    └───────────────┘                  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Component Types

| Type | Location | Purpose | Examples |
|------|----------|---------|----------|
| **Components** | `src/components/` | Reusable, stateless UI elements | Navbar, NFTCard, WalletButton |
| **Containers** | `src/container/` | Page sections, business logic | Header, Marketplace, Gifts |
| **Context** | `src/context/` | Global state providers | WalletContext |
| **Services** | `src/services/` | External integrations | api.js (Axios) |

---

## Backend Architecture

### Express.js Application Structure

```
server/
├── index.js          # Server entry point, DB connection
├── app.js            # Express app configuration
├── modules/          # Feature modules (MVC)
│   ├── user/
│   ├── nft/
│   ├── auction/
│   ├── buy/
│   ├── bid/
│   └── ...
├── routes/           # API route definitions
│   ├── index.js
│   └── api/
└── helper/           # Utility functions
```

### MVC Pattern Implementation

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           MVC PATTERN IN SERVER                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

                              HTTP Request
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              ROUTES (View)                                       │
│                                                                                  │
│   router.post('/register', userController.Register);                            │
│   router.post('/update', userController.Update);                                │
│   router.post('/search', userController.Search);                                │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           CONTROLLERS (Controller)                               │
│                                                                                  │
│   const userController = {                                                       │
│       Register: async (req, res) => {                                           │
│           // Business logic                                                      │
│           // Validation                                                          │
│           // Model operations                                                    │
│           // Response formatting                                                 │
│       },                                                                         │
│       Update: async (req, res) => { ... },                                      │
│       Search: async (req, res) => { ... }                                       │
│   };                                                                             │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                             SCHEMAS (Model)                                      │
│                                                                                  │
│   const userSchema = new mongoose.Schema({                                       │
│       name: String,                                                              │
│       bio: String,                                                               │
│       email: String,                                                             │
│       // ... other fields                                                        │
│   });                                                                            │
│                                                                                  │
│   module.exports = mongoose.model('User', userSchema);                          │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
                              HTTP Response
```

### Middleware Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           MIDDLEWARE PIPELINE                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

    Incoming Request
          │
          ▼
    ┌─────────────────┐
    │     Morgan      │  ←── Request logging
    │   (logging)     │      Logs: method, url, status, response time
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │  Express Device │  ←── Device detection
    │   (detection)   │      Detects: mobile, tablet, desktop
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │   Body Parser   │  ←── Request body parsing
    │  (json/urlenc)  │      Parses: JSON, URL-encoded data
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │      HPP        │  ←── Security
    │  (protection)   │      Prevents: HTTP Parameter Pollution
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │ Cookie Session  │  ←── Session management
    │   (session)     │      Handles: user sessions, auth state
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │ Cookie Parser   │  ←── Cookie handling
    │   (cookies)     │      Parses: request cookies
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │    Passport     │  ←── Authentication
    │  (initialize)   │      Initializes: auth strategies
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │     CORS        │  ←── Cross-origin
    │   (headers)     │      Sets: CORS headers
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │     Routes      │  ←── Request routing
    │   (handlers)    │      Routes to: appropriate controller
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │   404 Handler   │  ←── Not found
    │  (not found)    │      Handles: unknown routes
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │ Error Handler   │  ←── Error processing
    │   (errors)      │      Handles: all errors, formatting
    └────────┬────────┘
             │
             ▼
    Outgoing Response
```

### Module Structure

Each feature module follows this structure:

```
modules/
└── [feature]/
    ├── [feature]Schema.js      # Mongoose model definition
    ├── [feature]Controller.js  # Business logic
    └── [feature]Validation.js  # Input validation (optional)
```

**Example: User Module**

```javascript
// userSchema.js
const userSchema = new mongoose.Schema({
    name: { type: String, default: null },
    bio: { type: String, default: null },
    email: { type: String, default: null },
    // ... other fields
}, { timestamps: true });

// userController.js
const userController = {
    Register: async (req, res) => {
        try {
            const { address } = req.body;
            // Check existing user
            // Create new user if needed
            // Return response
        } catch (error) {
            // Handle error
        }
    }
};
```

---

## Database Design

### Collection Relationships

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        DATABASE RELATIONSHIP DIAGRAM                             │
└─────────────────────────────────────────────────────────────────────────────────┘

                                 ┌─────────┐
                                 │  USERS  │
                                 └────┬────┘
                                      │
           ┌──────────────────────────┼──────────────────────────┐
           │                          │                          │
           ▼                          ▼                          ▼
      ┌─────────┐               ┌─────────┐               ┌─────────┐
      │ WALLETS │               │  NFTS   │               │HISTORIES│
      │         │               │         │               │         │
      │ user ◄──┼───────────────┤ creator │               │ user ◄──┼───┐
      │ address │               │ name    │               │ activity│   │
      │ chain   │               │ image   │               │ nft     │   │
      └─────────┘               │ metadata│               │ auction │   │
                                └────┬────┘               │ buy     │   │
                                     │                    └─────────┘   │
              ┌──────────────────────┼──────────────────────┐          │
              │                      │                      │          │
              ▼                      ▼                      ▼          │
         ┌─────────┐            ┌─────────┐            ┌─────────┐    │
         │ OWNERS  │            │  BUYS   │            │AUCTIONS │    │
         │         │            │         │            │         │    │
         │ nft ◄───┼────────────┤ nft ◄───┼────────────┤ nft     │    │
         │ owner   │            │ seller  │            │ seller  │    │
         │ txHash  │            │ buyer   │            │ highest │    │
         └─────────┘            │ price   │            │ Bidder  │    │
                                │ status  │            │ prices  │    │
                                └─────────┘            └────┬────┘    │
                                                            │         │
                                                            ▼         │
                                                       ┌─────────┐    │
                                                       │  BIDS   │    │
                                                       │         │    │
                                                       │ auction │    │
                                                       │ bidder  │◄───┘
                                                       │ amount  │
                                                       └─────────┘

         ┌─────────┐
         │  TRANS  │
         │         │
         │ txHash  │  ←── Stores blockchain transaction records
         │ from/to │
         │ value   │
         │ nft     │
         └─────────┘
```

### Index Strategy

```javascript
// Text indexes for search functionality
nftSchema.index({ name: 'text', description: 'text' });
userSchema.index({ name: 'text', bio: 'text' });

// Regular indexes for query performance
userSchema.index({ createdAt: -1 });
auctionSchema.index({ status: 1, endTime: 1 });
buySchema.index({ status: 1 });
bidSchema.index({ auction: 1, amount: -1 });
```

### Data Integrity

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         DATA INTEGRITY MEASURES                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  REFERENTIAL INTEGRITY                                                          │
│                                                                                  │
│  • ObjectId references between collections                                       │
│  • Mongoose populate() for joins                                                 │
│  • Virtual fields for computed relationships                                     │
│                                                                                  │
│  Example:                                                                        │
│  auctionSchema.virtual('bids', {                                                │
│      ref: 'Bid',                                                                │
│      localField: '_id',                                                         │
│      foreignField: 'auction'                                                    │
│  });                                                                            │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  VALIDATION                                                                      │
│                                                                                  │
│  • Schema-level validation (Mongoose)                                           │
│  • Controller-level validation (validator.js)                                   │
│  • Required fields enforcement                                                   │
│  • Type checking                                                                 │
│                                                                                  │
│  Example:                                                                        │
│  email: {                                                                       │
│      type: String,                                                              │
│      validate: {                                                                │
│          validator: (v) => validator.isEmail(v),                                │
│          message: 'Invalid email format'                                        │
│      }                                                                          │
│  }                                                                              │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Web3 Integration

### Wallet Connection Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        WEB3 INTEGRATION ARCHITECTURE                             │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                              BROWSER ENVIRONMENT                                 │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                         MetaMask Extension                               │  │
│   │                                                                          │  │
│   │   window.ethereum                                                        │  │
│   │   ├── request({ method: 'eth_requestAccounts' })                        │  │
│   │   ├── on('accountsChanged', callback)                                   │  │
│   │   ├── on('chainChanged', callback)                                      │  │
│   │   └── on('disconnect', callback)                                        │  │
│   │                                                                          │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                      │                                          │
│                                      ▼                                          │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                          ethers.js Library                               │  │
│   │                                                                          │  │
│   │   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │  │
│   │   │    Provider     │    │     Signer      │    │    Contract     │    │  │
│   │   │                 │    │                 │    │                 │    │  │
│   │   │ • Read chain    │    │ • Sign tx       │    │ • Call methods  │    │  │
│   │   │ • Get balance   │    │ • Send tx       │    │ • Send tx       │    │  │
│   │   │ • Get block     │    │ • Get address   │    │ • Read state    │    │  │
│   │   │                 │    │                 │    │                 │    │  │
│   │   └─────────────────┘    └─────────────────┘    └─────────────────┘    │  │
│   │                                                                          │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                      │                                          │
│                                      ▼                                          │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                         WalletContext.js                                 │  │
│   │                                                                          │  │
│   │   const connectWallet = async () => {                                   │  │
│   │       const provider = new ethers.providers.Web3Provider(window.ethereum);│
│   │       await provider.send("eth_requestAccounts", []);                   │  │
│   │       const signer = provider.getSigner();                              │  │
│   │       const address = await signer.getAddress();                        │  │
│   │       // Update state                                                    │  │
│   │   };                                                                     │  │
│   │                                                                          │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Transaction Flow

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           TRANSACTION FLOW                                       │
└─────────────────────────────────────────────────────────────────────────────────┘

    User Action                   Frontend                    Blockchain
    (Buy/Bid)                     (React)                     (Ethereum)
        │                            │                            │
        ▼                            │                            │
   ┌─────────┐                       │                            │
   │  Click  │                       │                            │
   │  Button │                       │                            │
   └────┬────┘                       │                            │
        │                            │                            │
        ▼                            │                            │
   ┌─────────────────────────────────┴─────────────────────────────┐
   │                    Build Transaction                          │
   │                                                               │
   │   const tx = {                                                │
   │       to: contractAddress,                                    │
   │       value: ethers.utils.parseEther(price),                 │
   │       data: contract.interface.encodeFunctionData(...)       │
   │   };                                                          │
   └─────────────────────────────────┬─────────────────────────────┘
                                     │
                                     ▼
   ┌─────────────────────────────────────────────────────────────────┐
   │                    MetaMask Popup                               │
   │                                                                 │
   │   ┌─────────────────────────────────────────────────────────┐  │
   │   │  Confirm Transaction                                     │  │
   │   │                                                          │  │
   │   │  To: 0x1234...                                           │  │
   │   │  Amount: 0.5 ETH                                         │  │
   │   │  Gas Fee: ~0.002 ETH                                     │  │
   │   │                                                          │  │
   │   │  [Reject]                          [Confirm]             │  │
   │   └─────────────────────────────────────────────────────────┘  │
   └─────────────────────────────────┬─────────────────────────────┘
                                     │
                        ┌────────────┴────────────┐
                        │                         │
                   [Rejected]                [Confirmed]
                        │                         │
                        ▼                         ▼
              ┌─────────────────┐       ┌─────────────────────────┐
              │  Show Error     │       │   Transaction Pending   │
              │  "Cancelled"    │       │                         │
              └─────────────────┘       │   • Wait for mining     │
                                        │   • Show loading state  │
                                        └───────────┬─────────────┘
                                                    │
                                                    ▼
                                        ┌─────────────────────────┐
                                        │   Transaction Mined     │
                                        │                         │
                                        │   • Get receipt         │
                                        │   • Check status        │
                                        └───────────┬─────────────┘
                                                    │
                         ┌──────────────────────────┴──────────────────────────┐
                         │                                                      │
                    [Failed]                                              [Success]
                         │                                                      │
                         ▼                                                      ▼
              ┌─────────────────┐                                  ┌─────────────────┐
              │  Show Error     │                                  │  Update Backend │
              │  "Tx Failed"    │                                  │  • /api/buy/sell│
              └─────────────────┘                                  │  • Record tx    │
                                                                   │  • Update owner │
                                                                   └────────┬────────┘
                                                                            │
                                                                            ▼
                                                                   ┌─────────────────┐
                                                                   │  Update UI      │
                                                                   │  • Show success │
                                                                   │  • Refresh data │
                                                                   └─────────────────┘
```

---

## State Management

### Context API Pattern

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        CONTEXT API IMPLEMENTATION                                │
└─────────────────────────────────────────────────────────────────────────────────┘

// WalletContext.js

import { createContext, useContext, useState, useEffect } from 'react';
import { ethers } from 'ethers';

// 1. Create Context
const WalletContext = createContext(null);

// 2. Create Provider Component
export const WalletProvider = ({ children }) => {
    // State
    const [account, setAccount] = useState(null);
    const [provider, setProvider] = useState(null);
    const [signer, setSigner] = useState(null);
    const [isConnecting, setIsConnecting] = useState(false);

    // Actions
    const connectWallet = async () => { ... };
    const disconnectWallet = () => { ... };
    const formatAddress = (addr) => { ... };

    // Event Listeners
    useEffect(() => {
        if (window.ethereum) {
            window.ethereum.on('accountsChanged', handleAccountChange);
            window.ethereum.on('chainChanged', handleChainChange);
        }
        return () => { /* cleanup */ };
    }, []);

    // 3. Provide Value
    return (
        <WalletContext.Provider value={{
            account,
            provider,
            signer,
            isConnecting,
            connectWallet,
            disconnectWallet,
            formatAddress
        }}>
            {children}
        </WalletContext.Provider>
    );
};

// 4. Custom Hook
export const useWallet = () => {
    const context = useContext(WalletContext);
    if (!context) {
        throw new Error('useWallet must be used within WalletProvider');
    }
    return context;
};
```

### Local Component State

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          LOCAL STATE PATTERNS                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

// Marketplace.jsx - Example of local state management

const Marketplace = () => {
    // UI State
    const [activeTab, setActiveTab] = useState('buy');
    const [searchQuery, setSearchQuery] = useState('');

    // Data State
    const [nfts, setNfts] = useState([]);
    const [auctions, setAuctions] = useState([]);
    const [stats, setStats] = useState({
        users: 0,
        nfts: 0,
        listings: 0,
        auctions: 0
    });

    // Loading State
    const [isLoading, setIsLoading] = useState(true);
    const [error, setError] = useState(null);

    // Effects
    useEffect(() => {
        fetchData();
    }, [activeTab]);

    // Handlers
    const handleTabChange = (tab) => setActiveTab(tab);
    const handleSearch = (e) => setSearchQuery(e.target.value);

    // ...
};
```

---

## API Design

### RESTful Conventions

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           API DESIGN PATTERNS                                    │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  ENDPOINT STRUCTURE                                                              │
│                                                                                  │
│  Base URL: /api                                                                  │
│                                                                                  │
│  Resource-based:                                                                 │
│  /api/user/*       - User operations                                            │
│  /api/nft/*        - NFT operations                                             │
│  /api/auction/*    - Auction operations                                         │
│  /api/buy/*        - Direct sale operations                                     │
│  /api/bid/*        - Bid operations                                             │
│  /api/search/*     - Search operations                                          │
│  /api/history/*    - History operations                                         │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  HTTP METHODS                                                                    │
│                                                                                  │
│  POST - Create/Action operations                                                 │
│  GET  - Read operations (statistics, counts)                                    │
│                                                                                  │
│  Note: This API uses POST for most operations due to the need to send           │
│  complex data in request bodies                                                  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  RESPONSE FORMAT                                                                 │
│                                                                                  │
│  Success:                                                                        │
│  {                                                                               │
│      "success": true,                                                           │
│      "data": { ... },                                                           │
│      "message": "Operation successful"                                          │
│  }                                                                               │
│                                                                                  │
│  Error:                                                                          │
│  {                                                                               │
│      "success": false,                                                          │
│      "error": {                                                                 │
│          "code": "ERROR_CODE",                                                  │
│          "message": "Error description"                                         │
│      }                                                                          │
│  }                                                                               │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Request/Response Cycle

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         REQUEST/RESPONSE CYCLE                                   │
└─────────────────────────────────────────────────────────────────────────────────┘

    Frontend                        Backend                         Database
        │                              │                               │
        │  POST /api/user/register     │                               │
        │  { "address": "0x123..." }   │                               │
        │─────────────────────────────▶│                               │
        │                              │                               │
        │                              │  1. Validate input            │
        │                              │  2. Check existing user       │
        │                              │─────────────────────────────▶│
        │                              │                               │
        │                              │  User.findOne({ wallet })     │
        │                              │◀─────────────────────────────│
        │                              │                               │
        │                              │  3. Create if not exists      │
        │                              │─────────────────────────────▶│
        │                              │                               │
        │                              │  new User().save()            │
        │                              │  new Wallet().save()          │
        │                              │◀─────────────────────────────│
        │                              │                               │
        │                              │  4. Format response           │
        │  { success: true,            │                               │
        │    data: { user, wallet } }  │                               │
        │◀─────────────────────────────│                               │
        │                              │                               │
        │  5. Update UI state          │                               │
        │                              │                               │
```

---

## Security Architecture

### Security Layers

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SECURITY ARCHITECTURE                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                              NETWORK LAYER                                       │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │  CORS (Cross-Origin Resource Sharing)                                    │  │
│   │                                                                          │  │
│   │  • Configured in server/app.js                                          │  │
│   │  • Allows specified origins                                             │  │
│   │  • Controls allowed methods and headers                                 │  │
│   │                                                                          │  │
│   │  app.use((req, res, next) => {                                         │  │
│   │      res.setHeader('Access-Control-Allow-Origin', '*');                │  │
│   │      res.setHeader('Access-Control-Allow-Methods', 'GET,POST,...');    │  │
│   │      res.setHeader('Access-Control-Allow-Headers', '...');              │  │
│   │      next();                                                             │  │
│   │  });                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            APPLICATION LAYER                                     │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │  HPP (HTTP Parameter Pollution Prevention)                               │  │
│   │                                                                          │  │
│   │  • Prevents array-based parameter pollution attacks                     │  │
│   │  • Selects last parameter value if duplicated                          │  │
│   │                                                                          │  │
│   │  app.use(hpp());                                                        │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │  Input Validation                                                        │  │
│   │                                                                          │  │
│   │  • validator.js for input sanitization                                  │  │
│   │  • Mongoose schema validation                                           │  │
│   │  • Controller-level checks                                              │  │
│   │                                                                          │  │
│   │  if (!validator.isEthereumAddress(address)) {                          │  │
│   │      return res.status(400).json({ error: 'Invalid address' });        │  │
│   │  }                                                                       │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           AUTHENTICATION LAYER                                   │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │  Passport.js with JWT Strategy                                           │  │
│   │                                                                          │  │
│   │  • Session management via cookie-session                                │  │
│   │  • JWT token-based authentication                                       │  │
│   │  • Wallet-based identity verification                                   │  │
│   │                                                                          │  │
│   │  passport.use(new JwtStrategy(opts, (payload, done) => {               │  │
│   │      User.findById(payload.id, (err, user) => {                        │  │
│   │          if (err) return done(err, false);                              │  │
│   │          if (user) return done(null, user);                             │  │
│   │          return done(null, false);                                      │  │
│   │      });                                                                 │  │
│   │  }));                                                                    │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              DATA LAYER                                          │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │  Error Handling                                                          │  │
│   │                                                                          │  │
│   │  • Centralized error handler                                            │  │
│   │  • No sensitive data in error responses                                 │  │
│   │  • Proper HTTP status codes                                             │  │
│   │                                                                          │  │
│   │  app.use((err, req, res, next) => {                                    │  │
│   │      // Log error internally                                             │  │
│   │      console.error(err.stack);                                          │  │
│   │      // Send safe response                                               │  │
│   │      res.status(500).json({                                             │  │
│   │          success: false,                                                 │  │
│   │          error: 'Internal server error'                                 │  │
│   │      });                                                                 │  │
│   │  });                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Wallet Security

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           WALLET SECURITY MODEL                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────────────────┐
    │                         KEY PRINCIPLES                                   │
    │                                                                          │
    │  1. Private keys NEVER leave the user's wallet (MetaMask)              │
    │  2. Server only stores public wallet addresses                          │
    │  3. All transactions require explicit user approval in MetaMask         │
    │  4. Wallet connection is client-side only (ethers.js)                  │
    │  5. No wallet credentials stored in database                            │
    │                                                                          │
    └─────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────────────────┐
    │                         DATA STORED                                      │
    │                                                                          │
    │  Client-Side (Browser):                                                  │
    │  • Provider instance (read-only blockchain access)                      │
    │  • Signer instance (for signing, but keys stay in MetaMask)            │
    │  • Wallet address (public)                                              │
    │                                                                          │
    │  Server-Side (Database):                                                 │
    │  • Wallet address (public)                                              │
    │  • User profile data                                                     │
    │  • Transaction hashes (public, on blockchain anyway)                    │
    │                                                                          │
    └─────────────────────────────────────────────────────────────────────────┘
```

---

## Performance Considerations

### Frontend Optimization

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      FRONTEND PERFORMANCE STRATEGIES                             │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  CODE SPLITTING                                                                  │
│                                                                                  │
│  • React.lazy() for component lazy loading                                      │
│  • Suspense for loading states                                                  │
│  • Route-based code splitting                                                   │
│                                                                                  │
│  const Marketplace = React.lazy(() => import('./container/marketplace'));      │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  ASSET OPTIMIZATION                                                              │
│                                                                                  │
│  • Image compression and proper sizing                                          │
│  • Lazy loading for images below fold                                           │
│  • CSS minification in production build                                         │
│  • JavaScript bundling and minification                                         │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  STATE MANAGEMENT                                                                │
│                                                                                  │
│  • Context API for global state (lightweight)                                   │
│  • Local state for component-specific data                                      │
│  • Memoization with useMemo/useCallback where needed                           │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Backend Optimization

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      BACKEND PERFORMANCE STRATEGIES                              │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  DATABASE                                                                        │
│                                                                                  │
│  • Indexed fields for frequent queries                                          │
│  • Text indexes for search functionality                                        │
│  • Lean queries where full documents aren't needed                             │
│  • Pagination for large result sets                                             │
│                                                                                  │
│  // Example: Indexed query                                                       │
│  Auction.find({ status: 'active' })                                            │
│         .sort({ createdAt: -1 })                                               │
│         .limit(20)                                                              │
│         .lean();                                                                │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  API                                                                             │
│                                                                                  │
│  • Proper error handling to prevent crashes                                     │
│  • Response compression (gzip)                                                  │
│  • Request body size limits                                                     │
│  • Connection pooling for database                                              │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Deployment Architecture

### Production Deployment

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        PRODUCTION DEPLOYMENT                                     │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                              RECOMMENDED STACK                                   │
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                          CDN / Load Balancer                             │  │
│   │                                                                          │  │
│   │   • CloudFlare / AWS CloudFront                                         │  │
│   │   • SSL termination                                                      │  │
│   │   • DDoS protection                                                      │  │
│   │   • Static asset caching                                                 │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                      │                                          │
│                     ┌────────────────┴────────────────┐                        │
│                     │                                 │                        │
│                     ▼                                 ▼                        │
│   ┌─────────────────────────────┐   ┌─────────────────────────────┐          │
│   │    Frontend Hosting         │   │     Backend Hosting          │          │
│   │                             │   │                               │          │
│   │   • Vercel / Netlify        │   │   • AWS EC2 / Heroku         │          │
│   │   • Static file serving     │   │   • Docker container         │          │
│   │   • Edge caching            │   │   • PM2 process manager      │          │
│   │   • SPA routing             │   │   • Auto-scaling             │          │
│   │                             │   │                               │          │
│   │   Build: npm run build      │   │   Start: npm run start-server│          │
│   └─────────────────────────────┘   └───────────────┬───────────────┘          │
│                                                      │                          │
│                                                      ▼                          │
│                                     ┌─────────────────────────────┐            │
│                                     │    Database Hosting          │            │
│                                     │                               │            │
│                                     │   • MongoDB Atlas            │            │
│                                     │   • Replica set              │            │
│                                     │   • Auto backups             │            │
│                                     │   • Monitoring               │            │
│                                     └─────────────────────────────┘            │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Environment Configuration

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        ENVIRONMENT CONFIGURATION                                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  DEVELOPMENT (.env.development)                                                  │
│                                                                                  │
│  PORT=4000                                                                       │
│  ENV=Development                                                                 │
│  MONGODB_URI=mongodb://localhost:27017/nft_marketplace                          │
│  REACT_APP_API_URL=http://localhost:4000/api                                    │
│  JWT_SECRET=dev-secret-key                                                       │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│  PRODUCTION (.env.production)                                                    │
│                                                                                  │
│  PORT=4000                                                                       │
│  ENV=Production                                                                  │
│  MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/db_name    │
│  REACT_APP_API_URL=https://api.yourdomain.com/api                               │
│  JWT_SECRET=<strong-random-secret>                                              │
│                                                                                  │
│  # Additional production settings                                                │
│  NODE_ENV=production                                                             │
│  CORS_ORIGIN=https://yourdomain.com                                             │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Future Architecture Considerations

### Potential Improvements

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      FUTURE ARCHITECTURE IMPROVEMENTS                            │
└─────────────────────────────────────────────────────────────────────────────────┘

1. CACHING LAYER
   • Redis for session storage
   • API response caching
   • Rate limiting with Redis

2. MESSAGE QUEUE
   • RabbitMQ / Bull for async operations
   • Email notifications
   • Transaction processing

3. WEBSOCKETS
   • Real-time auction updates
   • Live bid notifications
   • Price updates

4. MICROSERVICES
   • Separate NFT service
   • Separate auction service
   • API gateway

5. MONITORING
   • APM (Application Performance Monitoring)
   • Error tracking (Sentry)
   • Log aggregation (ELK Stack)

6. SMART CONTRACTS
   • NFT minting contract
   • Marketplace contract
   • Auction contract
```

---

This architecture documentation provides a comprehensive overview of the system design and can serve as a reference for developers working on the project.
