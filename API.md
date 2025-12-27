# API Reference Documentation

Complete API reference for the Christmas NFT Marketplace backend.

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Error Handling](#error-handling)
- [User Endpoints](#user-endpoints)
- [NFT Endpoints](#nft-endpoints)
- [Buy/Sale Endpoints](#buysale-endpoints)
- [Auction Endpoints](#auction-endpoints)
- [Bid Endpoints](#bid-endpoints)
- [Search Endpoints](#search-endpoints)
- [History Endpoints](#history-endpoints)
- [Status Codes](#status-codes)

---

## Overview

### Base URL

| Environment | URL |
|-------------|-----|
| Development | `http://localhost:4000/api` |
| Production | `https://your-domain.com/api` |

### Content Type

All requests should include:
```
Content-Type: application/json
```

### Response Format

All responses follow a consistent format:

**Success Response:**
```json
{
    "success": true,
    "data": { },
    "message": "Operation description"
}
```

**Error Response:**
```json
{
    "success": false,
    "error": {
        "code": "ERROR_CODE",
        "message": "Human-readable error description"
    }
}
```

---

## Authentication

The API uses wallet-based authentication. Users authenticate by connecting their MetaMask wallet.

### Authentication Flow

```
1. User connects MetaMask wallet on frontend
2. Frontend calls /api/user/register with wallet address
3. Server creates/retrieves user and returns user data
4. Subsequent requests include user ID in request body
```

### Headers (Future Implementation)

```
Authorization: Bearer <jwt_token>
```

---

## Error Handling

### Error Codes

| Code | Description |
|------|-------------|
| `VALIDATION_ERROR` | Invalid input data |
| `NOT_FOUND` | Resource not found |
| `UNAUTHORIZED` | Authentication required |
| `FORBIDDEN` | Insufficient permissions |
| `DUPLICATE` | Resource already exists |
| `INTERNAL_ERROR` | Server error |

### Error Response Example

```json
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Invalid wallet address format",
        "details": {
            "field": "address",
            "value": "invalid-address"
        }
    }
}
```

---

## User Endpoints

### Register User

Creates a new user or retrieves existing user based on wallet address.

```
POST /api/user/register
```

**Request Body:**
```json
{
    "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8Db31"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "user": {
            "_id": "507f1f77bcf86cd799439011",
            "name": null,
            "bio": null,
            "email": null,
            "avatar": null,
            "banner": null,
            "website": null,
            "twitter": null,
            "instagram": null,
            "discord": null,
            "isVerified": false,
            "isFeatured": false,
            "followers": [],
            "following": [],
            "createdAt": "2024-01-15T10:30:00.000Z",
            "updatedAt": "2024-01-15T10:30:00.000Z"
        },
        "wallet": {
            "_id": "507f1f77bcf86cd799439012",
            "user": "507f1f77bcf86cd799439011",
            "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f8Db31",
            "chain": "ethereum",
            "isActive": true,
            "createdAt": "2024-01-15T10:30:00.000Z"
        },
        "isNew": true
    },
    "message": "User registered successfully"
}
```

---

### Update User Profile

Updates user profile information.

```
POST /api/user/update
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "bio": "NFT collector and crypto enthusiast",
    "email": "john@example.com",
    "avatar": "https://example.com/avatar.png",
    "banner": "https://example.com/banner.png",
    "website": "https://johndoe.com",
    "twitter": "@johndoe",
    "instagram": "johndoe",
    "discord": "johndoe#1234"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "user": {
            "_id": "507f1f77bcf86cd799439011",
            "name": "John Doe",
            "bio": "NFT collector and crypto enthusiast",
            "email": "john@example.com",
            "avatar": "https://example.com/avatar.png",
            "banner": "https://example.com/banner.png",
            "website": "https://johndoe.com",
            "twitter": "@johndoe",
            "instagram": "johndoe",
            "discord": "johndoe#1234",
            "isVerified": false,
            "isFeatured": false,
            "updatedAt": "2024-01-15T11:00:00.000Z"
        }
    },
    "message": "Profile updated successfully"
}
```

---

### Search Users

Search users by name or bio.

```
POST /api/user/search
```

**Request Body:**
```json
{
    "query": "john",
    "limit": 10,
    "skip": 0
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "users": [
            {
                "_id": "507f1f77bcf86cd799439011",
                "name": "John Doe",
                "bio": "NFT collector",
                "avatar": "https://example.com/avatar.png",
                "isVerified": true,
                "followersCount": 150,
                "followingCount": 75
            }
        ],
        "total": 1,
        "limit": 10,
        "skip": 0
    }
}
```

---

### Follow/Unfollow User

Toggle follow status for a user.

```
POST /api/user/follow
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439011",
    "targetUserId": "507f1f77bcf86cd799439022"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "isFollowing": true,
        "followersCount": 151
    },
    "message": "User followed successfully"
}
```

---

### Check Follow Status

Check if user is following another user.

```
POST /api/user/check
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439011",
    "targetUserId": "507f1f77bcf86cd799439022"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "isFollowing": true
    }
}
```

---

### Get Featured Users

Retrieve featured/highlighted users.

```
GET /api/user/featured-users
```

**Response:**
```json
{
    "success": true,
    "data": {
        "users": [
            {
                "_id": "507f1f77bcf86cd799439011",
                "name": "Featured Creator",
                "bio": "Top NFT artist",
                "avatar": "https://example.com/avatar.png",
                "isVerified": true,
                "isFeatured": true,
                "followersCount": 5000
            }
        ]
    }
}
```

---

### Get Total User Count

Get the total number of registered users.

```
GET /api/user/total-count
```

**Response:**
```json
{
    "success": true,
    "data": {
        "count": 1250
    }
}
```

---

## NFT Endpoints

### Create/Mint NFT

Create a new NFT record after minting on blockchain.

```
POST /api/nft/create
```

**Request Body:**
```json
{
    "name": "Christmas Special #1",
    "description": "A festive NFT for the holiday season with animated snowfall",
    "image": "https://ipfs.io/ipfs/QmXxx...",
    "metadata": {
        "attributes": [
            {
                "trait_type": "Season",
                "value": "Winter"
            },
            {
                "trait_type": "Rarity",
                "value": "Rare"
            }
        ],
        "external_url": "https://marketplace.com/nft/1"
    },
    "tokenId": "1",
    "contractAddress": "0x1234567890abcdef1234567890abcdef12345678",
    "creatorId": "507f1f77bcf86cd799439011"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "nft": {
            "_id": "507f1f77bcf86cd799439033",
            "name": "Christmas Special #1",
            "description": "A festive NFT for the holiday season with animated snowfall",
            "image": "https://ipfs.io/ipfs/QmXxx...",
            "metadata": {
                "attributes": [
                    {
                        "trait_type": "Season",
                        "value": "Winter"
                    },
                    {
                        "trait_type": "Rarity",
                        "value": "Rare"
                    }
                ],
                "external_url": "https://marketplace.com/nft/1"
            },
            "tokenId": "1",
            "contractAddress": "0x1234567890abcdef1234567890abcdef12345678",
            "creator": "507f1f77bcf86cd799439011",
            "createdAt": "2024-01-15T12:00:00.000Z"
        }
    },
    "message": "NFT created successfully"
}
```

---

### Get NFTs by Creator

Retrieve all NFTs created by a specific user.

```
POST /api/nft/created
```

**Request Body:**
```json
{
    "creatorId": "507f1f77bcf86cd799439011",
    "limit": 20,
    "skip": 0
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "nfts": [
            {
                "_id": "507f1f77bcf86cd799439033",
                "name": "Christmas Special #1",
                "description": "A festive NFT...",
                "image": "https://ipfs.io/ipfs/QmXxx...",
                "tokenId": "1",
                "contractAddress": "0x1234...",
                "creator": {
                    "_id": "507f1f77bcf86cd799439011",
                    "name": "John Doe",
                    "avatar": "https://example.com/avatar.png"
                },
                "createdAt": "2024-01-15T12:00:00.000Z"
            }
        ],
        "total": 1,
        "limit": 20,
        "skip": 0
    }
}
```

---

### Get Total Mint Count

Get the total number of NFTs minted.

```
GET /api/nft/total-mint-count
```

**Response:**
```json
{
    "success": true,
    "data": {
        "count": 5420
    }
}
```

---

## Buy/Sale Endpoints

### List NFT for Sale

Create a direct sale listing for an NFT.

```
POST /api/buy/buy
```

**Request Body:**
```json
{
    "nftId": "507f1f77bcf86cd799439033",
    "sellerId": "507f1f77bcf86cd799439011",
    "price": 0.5
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "listing": {
            "_id": "507f1f77bcf86cd799439044",
            "nft": "507f1f77bcf86cd799439033",
            "seller": "507f1f77bcf86cd799439011",
            "buyer": null,
            "price": 0.5,
            "status": "active",
            "listedAt": "2024-01-15T13:00:00.000Z",
            "soldAt": null
        }
    },
    "message": "NFT listed for sale successfully"
}
```

---

### Complete Purchase

Complete a direct sale purchase.

```
POST /api/buy/sell
```

**Request Body:**
```json
{
    "buyId": "507f1f77bcf86cd799439044",
    "buyerId": "507f1f77bcf86cd799439022",
    "txHash": "0xabc123..."
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "listing": {
            "_id": "507f1f77bcf86cd799439044",
            "nft": "507f1f77bcf86cd799439033",
            "seller": "507f1f77bcf86cd799439011",
            "buyer": "507f1f77bcf86cd799439022",
            "price": 0.5,
            "status": "sold",
            "listedAt": "2024-01-15T13:00:00.000Z",
            "soldAt": "2024-01-15T14:00:00.000Z"
        },
        "transaction": {
            "_id": "507f1f77bcf86cd799439055",
            "txHash": "0xabc123...",
            "from": "0x742d35...",
            "to": "0x123456...",
            "value": 0.5,
            "nft": "507f1f77bcf86cd799439033"
        }
    },
    "message": "Purchase completed successfully"
}
```

---

### Get Active Listings

Retrieve all active sale listings.

```
POST /api/buy/get
```

**Request Body:**
```json
{
    "limit": 20,
    "skip": 0,
    "sortBy": "listedAt",
    "sortOrder": "desc"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "listings": [
            {
                "_id": "507f1f77bcf86cd799439044",
                "nft": {
                    "_id": "507f1f77bcf86cd799439033",
                    "name": "Christmas Special #1",
                    "image": "https://ipfs.io/ipfs/QmXxx...",
                    "description": "A festive NFT..."
                },
                "seller": {
                    "_id": "507f1f77bcf86cd799439011",
                    "name": "John Doe",
                    "avatar": "https://example.com/avatar.png"
                },
                "price": 0.5,
                "status": "active",
                "listedAt": "2024-01-15T13:00:00.000Z"
            }
        ],
        "total": 1,
        "limit": 20,
        "skip": 0
    }
}
```

---

### Find Specific Listing

Get details of a specific sale listing.

```
POST /api/buy/find
```

**Request Body:**
```json
{
    "buyId": "507f1f77bcf86cd799439044"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "listing": {
            "_id": "507f1f77bcf86cd799439044",
            "nft": {
                "_id": "507f1f77bcf86cd799439033",
                "name": "Christmas Special #1",
                "image": "https://ipfs.io/ipfs/QmXxx...",
                "description": "A festive NFT for the holiday season",
                "metadata": { },
                "tokenId": "1",
                "contractAddress": "0x1234...",
                "creator": {
                    "_id": "507f1f77bcf86cd799439011",
                    "name": "John Doe"
                }
            },
            "seller": {
                "_id": "507f1f77bcf86cd799439011",
                "name": "John Doe",
                "avatar": "https://example.com/avatar.png",
                "isVerified": true
            },
            "price": 0.5,
            "status": "active",
            "listedAt": "2024-01-15T13:00:00.000Z"
        }
    }
}
```

---

## Auction Endpoints

### Create Auction

Create a new auction for an NFT.

```
POST /api/auction/auction
```

**Request Body:**
```json
{
    "nftId": "507f1f77bcf86cd799439033",
    "sellerId": "507f1f77bcf86cd799439011",
    "startPrice": 0.1,
    "minIncrement": 0.01,
    "duration": 86400
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "auction": {
            "_id": "507f1f77bcf86cd799439066",
            "nft": "507f1f77bcf86cd799439033",
            "seller": "507f1f77bcf86cd799439011",
            "highestBidder": null,
            "startPrice": 0.1,
            "currentPrice": 0.1,
            "minIncrement": 0.01,
            "startTime": "2024-01-15T15:00:00.000Z",
            "endTime": "2024-01-16T15:00:00.000Z",
            "status": "active",
            "createdAt": "2024-01-15T15:00:00.000Z"
        }
    },
    "message": "Auction created successfully"
}
```

---

### Place Bid

Place a bid on an active auction.

```
POST /api/auction/bid
```

**Request Body:**
```json
{
    "auctionId": "507f1f77bcf86cd799439066",
    "bidderId": "507f1f77bcf86cd799439022",
    "amount": 0.15
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "bid": {
            "_id": "507f1f77bcf86cd799439077",
            "auction": "507f1f77bcf86cd799439066",
            "bidder": "507f1f77bcf86cd799439022",
            "amount": 0.15,
            "createdAt": "2024-01-15T16:00:00.000Z"
        },
        "auction": {
            "_id": "507f1f77bcf86cd799439066",
            "currentPrice": 0.15,
            "highestBidder": "507f1f77bcf86cd799439022"
        }
    },
    "message": "Bid placed successfully"
}
```

**Error Response (Bid too low):**
```json
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Bid amount must be at least 0.16 ETH (current: 0.15 + increment: 0.01)"
    }
}
```

---

### Get Active Auctions

Retrieve all active auctions.

```
POST /api/auction/get
```

**Request Body:**
```json
{
    "limit": 20,
    "skip": 0,
    "sortBy": "endTime",
    "sortOrder": "asc"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "auctions": [
            {
                "_id": "507f1f77bcf86cd799439066",
                "nft": {
                    "_id": "507f1f77bcf86cd799439033",
                    "name": "Christmas Special #1",
                    "image": "https://ipfs.io/ipfs/QmXxx...",
                    "description": "A festive NFT..."
                },
                "seller": {
                    "_id": "507f1f77bcf86cd799439011",
                    "name": "John Doe",
                    "avatar": "https://example.com/avatar.png"
                },
                "highestBidder": {
                    "_id": "507f1f77bcf86cd799439022",
                    "name": "Jane Smith"
                },
                "startPrice": 0.1,
                "currentPrice": 0.15,
                "minIncrement": 0.01,
                "endTime": "2024-01-16T15:00:00.000Z",
                "status": "active",
                "bidCount": 3
            }
        ],
        "total": 1,
        "limit": 20,
        "skip": 0
    }
}
```

---

### Get Popular Auctions

Retrieve popular auctions based on bid count.

```
POST /api/auction/popular
```

**Request Body:**
```json
{
    "limit": 10
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "auctions": [
            {
                "_id": "507f1f77bcf86cd799439066",
                "nft": {
                    "name": "Christmas Special #1",
                    "image": "https://ipfs.io/ipfs/QmXxx..."
                },
                "currentPrice": 2.5,
                "bidCount": 45,
                "endTime": "2024-01-16T15:00:00.000Z"
            }
        ]
    }
}
```

---

### Find Specific Auction

Get details of a specific auction with bid history.

```
POST /api/auction/find
```

**Request Body:**
```json
{
    "auctionId": "507f1f77bcf86cd799439066"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "auction": {
            "_id": "507f1f77bcf86cd799439066",
            "nft": {
                "_id": "507f1f77bcf86cd799439033",
                "name": "Christmas Special #1",
                "image": "https://ipfs.io/ipfs/QmXxx...",
                "description": "A festive NFT for the holiday season",
                "metadata": { },
                "tokenId": "1",
                "contractAddress": "0x1234...",
                "creator": {
                    "_id": "507f1f77bcf86cd799439011",
                    "name": "John Doe"
                }
            },
            "seller": {
                "_id": "507f1f77bcf86cd799439011",
                "name": "John Doe",
                "avatar": "https://example.com/avatar.png",
                "isVerified": true
            },
            "highestBidder": {
                "_id": "507f1f77bcf86cd799439022",
                "name": "Jane Smith",
                "avatar": "https://example.com/jane.png"
            },
            "startPrice": 0.1,
            "currentPrice": 0.15,
            "minIncrement": 0.01,
            "startTime": "2024-01-15T15:00:00.000Z",
            "endTime": "2024-01-16T15:00:00.000Z",
            "status": "active",
            "bids": [
                {
                    "_id": "507f1f77bcf86cd799439077",
                    "bidder": {
                        "name": "Jane Smith",
                        "avatar": "https://example.com/jane.png"
                    },
                    "amount": 0.15,
                    "createdAt": "2024-01-15T16:00:00.000Z"
                },
                {
                    "_id": "507f1f77bcf86cd799439076",
                    "bidder": {
                        "name": "Bob Wilson",
                        "avatar": "https://example.com/bob.png"
                    },
                    "amount": 0.12,
                    "createdAt": "2024-01-15T15:30:00.000Z"
                }
            ]
        }
    }
}
```

---

## Bid Endpoints

### Get Bids Received

Get all bids received by a user on their auctions.

```
POST /api/bid/received
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439011",
    "limit": 20,
    "skip": 0
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "bids": [
            {
                "_id": "507f1f77bcf86cd799439077",
                "auction": {
                    "_id": "507f1f77bcf86cd799439066",
                    "nft": {
                        "name": "Christmas Special #1",
                        "image": "https://ipfs.io/ipfs/QmXxx..."
                    },
                    "currentPrice": 0.15
                },
                "bidder": {
                    "_id": "507f1f77bcf86cd799439022",
                    "name": "Jane Smith",
                    "avatar": "https://example.com/jane.png"
                },
                "amount": 0.15,
                "createdAt": "2024-01-15T16:00:00.000Z"
            }
        ],
        "total": 1,
        "limit": 20,
        "skip": 0
    }
}
```

---

### Get Bids Made

Get all bids placed by a user.

```
POST /api/bid/made
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439022",
    "limit": 20,
    "skip": 0
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "bids": [
            {
                "_id": "507f1f77bcf86cd799439077",
                "auction": {
                    "_id": "507f1f77bcf86cd799439066",
                    "nft": {
                        "name": "Christmas Special #1",
                        "image": "https://ipfs.io/ipfs/QmXxx..."
                    },
                    "seller": {
                        "name": "John Doe"
                    },
                    "currentPrice": 0.15,
                    "endTime": "2024-01-16T15:00:00.000Z",
                    "status": "active"
                },
                "amount": 0.15,
                "isHighestBid": true,
                "createdAt": "2024-01-15T16:00:00.000Z"
            }
        ],
        "total": 1,
        "limit": 20,
        "skip": 0
    }
}
```

---

## Search Endpoints

### Global Search

Search across NFTs, users, and auctions.

```
POST /api/search/search
```

**Request Body:**
```json
{
    "query": "christmas",
    "type": "all",
    "limit": 20,
    "skip": 0
}
```

**Parameters:**
| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `query` | string | Search term | Yes |
| `type` | string | Filter by type: "all", "nfts", "users", "auctions" | No (default: "all") |
| `limit` | number | Results per page | No (default: 20) |
| `skip` | number | Results to skip | No (default: 0) |

**Response:**
```json
{
    "success": true,
    "data": {
        "nfts": [
            {
                "_id": "507f1f77bcf86cd799439033",
                "name": "Christmas Special #1",
                "image": "https://ipfs.io/ipfs/QmXxx...",
                "description": "A festive NFT...",
                "creator": {
                    "name": "John Doe"
                }
            }
        ],
        "users": [
            {
                "_id": "507f1f77bcf86cd799439088",
                "name": "Christmas Collector",
                "avatar": "https://example.com/avatar.png",
                "bio": "I love Christmas NFTs"
            }
        ],
        "auctions": [
            {
                "_id": "507f1f77bcf86cd799439066",
                "nft": {
                    "name": "Christmas Special #1",
                    "image": "https://ipfs.io/ipfs/QmXxx..."
                },
                "currentPrice": 0.15,
                "endTime": "2024-01-16T15:00:00.000Z"
            }
        ],
        "totals": {
            "nfts": 5,
            "users": 2,
            "auctions": 3
        }
    }
}
```

---

## History Endpoints

### Get Following Activity

Get activity feed from followed users.

```
POST /api/history/following
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439011",
    "limit": 20,
    "skip": 0
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "activities": [
            {
                "_id": "507f1f77bcf86cd799439099",
                "user": {
                    "_id": "507f1f77bcf86cd799439022",
                    "name": "Jane Smith",
                    "avatar": "https://example.com/jane.png"
                },
                "activity": "purchase",
                "nft": {
                    "_id": "507f1f77bcf86cd799439033",
                    "name": "Christmas Special #1",
                    "image": "https://ipfs.io/ipfs/QmXxx..."
                },
                "createdAt": "2024-01-15T16:00:00.000Z"
            },
            {
                "_id": "507f1f77bcf86cd799439098",
                "user": {
                    "_id": "507f1f77bcf86cd799439022",
                    "name": "Jane Smith",
                    "avatar": "https://example.com/jane.png"
                },
                "activity": "bid",
                "auction": {
                    "_id": "507f1f77bcf86cd799439066",
                    "nft": {
                        "name": "Holiday Collection #5"
                    },
                    "currentPrice": 0.25
                },
                "createdAt": "2024-01-15T15:30:00.000Z"
            }
        ],
        "total": 2,
        "limit": 20,
        "skip": 0
    }
}
```

---

### Get Purchase History

Get user's purchase history.

```
POST /api/history/sales
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439011",
    "limit": 20,
    "skip": 0
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "purchases": [
            {
                "_id": "507f1f77bcf86cd799439100",
                "nft": {
                    "_id": "507f1f77bcf86cd799439033",
                    "name": "Christmas Special #1",
                    "image": "https://ipfs.io/ipfs/QmXxx..."
                },
                "seller": {
                    "name": "Original Creator"
                },
                "price": 0.5,
                "purchasedAt": "2024-01-15T14:00:00.000Z",
                "txHash": "0xabc123..."
            }
        ],
        "total": 1,
        "limit": 20,
        "skip": 0
    }
}
```

---

### Get Bidding History

Get user's bidding history.

```
POST /api/history/bidding
```

**Request Body:**
```json
{
    "userId": "507f1f77bcf86cd799439011",
    "limit": 20,
    "skip": 0
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "bids": [
            {
                "_id": "507f1f77bcf86cd799439077",
                "auction": {
                    "_id": "507f1f77bcf86cd799439066",
                    "nft": {
                        "name": "Christmas Special #1",
                        "image": "https://ipfs.io/ipfs/QmXxx..."
                    },
                    "status": "active",
                    "endTime": "2024-01-16T15:00:00.000Z"
                },
                "amount": 0.15,
                "isWinning": true,
                "createdAt": "2024-01-15T16:00:00.000Z"
            }
        ],
        "total": 1,
        "limit": 20,
        "skip": 0
    }
}
```

---

## Status Codes

| Code | Status | Description |
|------|--------|-------------|
| `200` | OK | Successful request |
| `201` | Created | Resource created successfully |
| `400` | Bad Request | Invalid request data |
| `401` | Unauthorized | Authentication required |
| `403` | Forbidden | Insufficient permissions |
| `404` | Not Found | Resource not found |
| `409` | Conflict | Resource already exists |
| `422` | Unprocessable Entity | Validation error |
| `500` | Internal Server Error | Server error |

---

## Rate Limiting

*(Future Implementation)*

| Endpoint Type | Limit |
|---------------|-------|
| General API | 100 requests/minute |
| Search | 30 requests/minute |
| Write Operations | 20 requests/minute |

---

## Pagination

All list endpoints support pagination with the following parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | number | 20 | Number of items per page (max: 100) |
| `skip` | number | 0 | Number of items to skip |

**Response includes:**
```json
{
    "data": {
        "items": [...],
        "total": 150,
        "limit": 20,
        "skip": 0
    }
}
```

---

## Sorting

List endpoints support sorting with the following parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sortBy` | string | varies | Field to sort by |
| `sortOrder` | string | "desc" | "asc" or "desc" |

**Common sortable fields:**
- `createdAt` - Creation date
- `price` / `currentPrice` - Price
- `endTime` - Auction end time
- `name` - Alphabetical

---

## Webhook Events

*(Future Implementation)*

| Event | Description |
|-------|-------------|
| `nft.minted` | New NFT minted |
| `listing.created` | NFT listed for sale |
| `listing.sold` | NFT sold |
| `auction.created` | Auction started |
| `auction.bid` | New bid placed |
| `auction.ended` | Auction ended |
| `user.followed` | User gained follower |

---

This API documentation provides comprehensive information for integrating with the Christmas NFT Marketplace backend.
