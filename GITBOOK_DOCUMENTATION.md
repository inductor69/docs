# QuantCite Backend - API Documentation

## Table of Contents

1. [Overview](#overview)
2. [WebSocket API Reference](#websocket-api-reference)
3. [Trading Features](#trading-features)
4. [Supported Exchanges](#supported-exchanges)
5. [API Response Formats](#api-response-formats)
6. [Getting Started](#getting-started)

---

## Overview

QuantCite Backend is a high-performance algorithmic trading platform that provides:

- **Multi-Exchange Support**: Trade across major cryptocurrency exchanges
- **Real-time WebSocket APIs**: Live market data and trading functionality
- **Advanced Trading Strategies**: TWAP, Limit, Market, and smart routing algorithms
- **Real-time Order Book**: Aggregated market data across exchanges
- **Account Management**: Portfolio tracking and balance monitoring

### Supported Features

- Real-time market data streaming
- Multi-exchange order placement and management
- Algorithmic trading strategies
- Order book aggregation
- Account balance and position tracking
- Trade history and order management

---

## WebSocket API Reference

### Connection Endpoints

- **User API**: `ws://your-domain:8080/` - Trading and market data
- **Admin API**: `ws://your-domain:8081/` - Administrative functions
- **Health Check**: `http://your-domain:8080/health` (User API health)
- **Admin Health Check**: `http://your-domain:8081/health` (Admin API health)

### Authentication

#### User Login
```json
{
  "id": "1",
  "method": "login",
  "params": {
    "username": "your_username",
    "password": "your_password"
  }
}
```

**Response:**
```json
{
  "id": "1",
  "status": "success",
  "message": "Login successful",
  "data": {
    "user_id": 123
  }
}
```

#### Token Authentication
```json
{
  "id": "1",
  "method": "authenticate_with_token",
  "params": {
    "token": "your_jwt_token"
  }
}
```

**Response:**
```json
{
  "id": "1",
  "status": "success",
  "message": "Authentication successful",
  "data": {
    "user_id": 123,
    "username": "your_username",
    "email": "your_email@example.com",
    "token": "refreshed_jwt_token"
  }
}
```

#### Refresh Token
```json
{
  "id": "1",
  "method": "refresh_token",
  "params": {
    "token": "your_current_token"
  }
}
```

#### Logout
```json
{
  "id": "1",
  "method": "logout",
  "params": {}
}
```

### Market Data APIs

#### Get Available Exchanges
```json
{
  "id": "2",
  "method": "get_exchanges",
  "params": {}
}
```

**Response:**
```json
{
  "id": "2",
  "status": "success",
  "data": [
    {
      "exchange_id": 1,
      "exchange_name": "binance",
      "display_name": "Binance"
    },
    {
      "exchange_id": 2,
      "exchange_name": "coinbase",
      "display_name": "Coinbase"
    }
  ]
}
```

#### Get Markets for Exchanges
Fetches normalized market data from specified exchanges with rich metadata including trading limits, precision information, and permissions.

```json
{
  "id": "3",
  "method": "get_markets",
  "params": {
    "userExchangeIds": [1, 2, 3]
  }
}
```

**Parameters:**
- `userExchangeIds` (array): Array of user exchange IDs to fetch markets from. Must contain positive integers.

**Response:**
```json
{
  "id": "3",
  "status": "success",
  "data": {
    "1": {
      "exchange": "binance",
      "is_testnet": false,
      "timestamp": "2024-01-20T10:00:00Z",
      "markets": [
        {
          "exchange_symbol": "BTCUSDT",
          "normalized_symbol": "BTC/USDT",
          "symbol": "BTC/USDT",
          "base_asset": "BTC",
          "quote_asset": "USDT",
          "status": "TRADING",
          "precision": {
            "price": 2,
            "quantity": 8,
            "base_asset": 8,
            "quote_asset": 8,
            "price_increment": "0.01",
            "quantity_increment": "0.00000001"
          },
          "limits": {
            "min_quantity": "0.00001000",
            "max_quantity": "9000.00000000",
            "min_notional": "5.00000000",
            "max_notional": "9000000.00000000",
            "min_price": "0.01000000",
            "max_price": "1000000.00000000"
          },
          "permissions": {
            "spot": true,
            "margin": true,
            "leveraged_tokens": false,
            "options": false,
            "order_types": ["LIMIT", "MARKET", "STOP_LOSS", "STOP_LOSS_LIMIT", "TAKE_PROFIT", "TAKE_PROFIT_LIMIT"]
          },
          "metadata": {
            "icebergAllowed": true,
            "ocoAllowed": true,
            "allowTrailingStop": true,
            "cancelReplaceAllowed": true,
            "quoteOrderQtyMarketAllowed": true,
            "permissions": ["SPOT"]
          }
        }
      ],
      "count": 1445
    },
    "2": {
      "exchange": "another_exchange",
      "is_testnet": false,
      "timestamp": "2024-01-20T10:00:00Z",
      "markets": [...],
      "count": 892
    }
  }
}
```

**Response Fields:**
- `data` (object): Contains market data keyed by userExchangeId
  - `{userExchangeId}` (object): Market data for specific exchange
    - `exchange` (string): Exchange name (e.g., "binance", "coinbase")
    - `is_testnet` (boolean): Whether this is testnet or mainnet
    - `timestamp` (string): ISO timestamp when data was fetched
    - `count` (integer): Total number of markets available
    - `markets` (array): Array of market objects
      - `exchange_symbol` (string): Exchange-specific symbol format (e.g., "BTCUSDT")
      - `normalized_symbol` (string): Standardized symbol format (e.g., "BTC/USDT")
      - `symbol` (string): Display symbol (same as normalized_symbol)
      - `base_asset` (string): Base currency (e.g., "BTC")
      - `quote_asset` (string): Quote currency (e.g., "USDT")
      - `status` (string): Trading status ("TRADING", "BREAK", "AUCTION")
      - `precision` (object): Precision information for trading
        - `price` (integer): Price decimal places
        - `quantity` (integer): Quantity decimal places
        - `base_asset` (integer): Base asset decimal places
        - `quote_asset` (integer): Quote asset decimal places
        - `price_increment` (string): Minimum price step size
        - `quantity_increment` (string): Minimum quantity step size
      - `limits` (object): Trading limits and constraints
        - `min_quantity` (string): Minimum order quantity
        - `max_quantity` (string): Maximum order quantity
        - `min_notional` (string): Minimum order value
        - `max_notional` (string): Maximum order value
        - `min_price` (string): Minimum price
        - `max_price` (string): Maximum price
      - `permissions` (object): Trading permissions and capabilities
        - `spot` (boolean): Spot trading allowed
        - `margin` (boolean): Margin trading allowed
        - `leveraged_tokens` (boolean): Leveraged tokens allowed
        - `options` (boolean): Options trading allowed
        - `order_types` (array): Supported order types
      - `metadata` (object): Exchange-specific additional information

**Error Responses:**

Invalid Exchange IDs:
```json
{
  "id": "3",
  "status": "error",
  "message": "invalid exchange ID: user exchange data not found for userExchangeId 999. Please check if this exchange ID exists for your account"
}
```

Invalid Parameters:
```json
{
  "id": "3",
  "status": "error",
  "message": "Invalid userExchangeIds detected: [\"abc\", -1, 0]"
}
```

Unsupported Exchange:
```json
{
  "id": "3",
  "status": "error",
  "message": "unsupported exchange 'unsupported_exchange' for userExchangeId 5. Supported exchanges: [\"binance\"]"
}
```

#### Subscribe to Order Book
```json
{
  "id": "4",
  "method": "subscribe_orderbook",
  "params": {
    "userExchangeIds": [1, 2],
    "symbol": "BTCUSDT"
  }
}
```

**Response:**
```json
{
  "id": "4",
  "status": "success",
  "message": "Subscribed to BTCUSDT orderbook"
}
```

**Real-time Updates:**
```json
{
  "type": "orderbook_update",
  "symbol": "BTCUSDT",
  "data": {
    "bids": [[45000.0, 1.5], [44999.0, 2.0]],
    "asks": [[45001.0, 1.2], [45002.0, 0.8]],
    "timestamp": 1640995200000,
    "sources": ["binance", "coinbase"]
  }
}
```

#### Unsubscribe from Updates
```json
{
  "id": "5",
  "method": "unsubscribe",
  "params": {
    "subscriptionId": "BTCUSDT_orderbook"
  }
}
```

**Response:**
```json
{
  "id": "5",
  "status": "success",
  "message": "Successfully unsubscribed"
}
```

**Note:** The `unsubscribe` method works for all subscription types (orderbook, trades, notifications, DEX price, option chain, balance updates) using the `subscriptionId` parameter.

### Trading APIs

#### Place Regular Order
```json
{
  "id": "6",
  "method": "place_order",
  "params": {
    "user_exchange_id": 1,
    "symbol": "BTCUSDT",
    "side": "BUY",
    "type": "LIMIT",
    "quantity": 0.001,
    "price": 45000.0,
    "time_in_force": "GTC"
  }
}
```

**Response:**
```json
{
  "id": "6",
  "status": "success",
  "data": {
    "order_id": "12345",
    "client_order_id": "client_123",
    "symbol": "BTCUSDT",
    "side": "BUY",
    "type": "LIMIT",
    "quantity": 0.001,
    "price": 45000.0,
    "status": "NEW",
    "timestamp": 1640995200000
  }
}
```

#### Place Algorithmic Order
```json
{
  "id": "7",
  "method": "place_algo_order",
  "params": {
    "user_exchange_id": 1,
    "algo_type": "TWAP",
    "symbol": "BTCUSDT",
    "side": "BUY",
    "quantity": 1.0,
    "duration_minutes": 60,
    "start_time": "2024-01-01T10:00:00Z",
    "price_limit": 46000.0
  }
}
```

**Response:**
```json
{
  "id": "7",
  "status": "success",
  "data": {
    "algo_order_id": "algo_123",
    "status": "PENDING",
    "estimated_completion": "2024-01-01T11:00:00Z"
  }
}
```

#### Cancel Algorithmic Order
```json
{
  "id": "8",
  "method": "cancel_algo_order",
  "params": {
    "algo_order_id": "algo_123"
  }
}
```

#### Plan Order (Analyze Order)
```json
{
  "id": "9",
  "method": "plan_order",
  "params": {
    "user_exchange_id": 1,
    "symbol": "BTCUSDT",
    "side": "BUY",
    "type": "LIMIT",
    "quantity": 0.001,
    "price": 45000.0
  }
}
```

**Response:**
```json
{
  "id": "9",
  "status": "success",
  "data": {
    "analysis": {
      "estimated_cost": 45.0,
      "fees": 0.045,
      "market_impact": "LOW",
      "execution_probability": "HIGH"
    },
    "recommendations": [
      "Current price is favorable",
      "Low market volatility detected"
    ]
  }
}
```

#### Confirm Order
```json
{
  "id": "10",
  "method": "confirm_order",
  "params": {
    "user_exchange_id": 1,
    "symbol": "BTCUSDT",
    "side": "BUY",
    "type": "LIMIT",
    "quantity": 0.001,
    "price": 45000.0,
    "confirmation_token": "confirm_123"
  }
}
```

#### Modify Order
```json
{
  "id": "11",
  "method": "modify_order",
  "params": {
    "user_exchange_id": 1,
    "order_id": "12345",
    "quantity": 0.002,
    "price": 44000.0
  }
}
```

#### Place Option Order
```json
{
  "id": "12",
  "method": "place_option_order",
  "params": {
    "user_exchange_id": 1,
    "symbol": "BTC-25DEC24-50000-C",
    "side": "BUY",
    "type": "LIMIT",
    "quantity": 1,
    "price": 1000.0
  }
}
```

#### Cancel All Orders
```json
{
  "id": "13",
  "method": "cancel_all_orders",
  "params": {
    "user_exchange_id": 1,
    "symbol": "BTCUSDT"
  }
}
```

**Response:**
```json
{
  "id": "13",
  "status": "success",
  "data": {
    "cancelled_orders": 5,
    "message": "Successfully cancelled 5 orders for BTCUSDT"
  }
}
```

### Subscription APIs

#### Subscribe to Trades
```json
{
  "id": "20",
  "method": "subscribe_trades",
  "params": {
    "userExchangeIds": [1, 2],
    "symbol": "BTCUSDT"
  }
}
```

**Response:**
```json
{
  "id": "20",
  "status": "success",
  "data": {
    "status": "success",
    "message": "Successfully subscribed to trades for BTCUSDT",
    "symbol": "BTCUSDT",
    "subscriptionId": "20",
    "type": "trades_subscription",
    "initial_data": {
      "trades": [
        {
          "id": 12345,
          "price": "45000.50",
          "qty": "0.001",
          "time": 1640995200000,
          "isBuyerMaker": false
        }
      ]
    }
  }
}
```

**Real-time Updates:**
```json
{
  "type": "trades_update",
  "symbol": "BTCUSDT",
  "subscriptionId": "20",
  "data": {
    "trades": [
      {
        "id": 12346,
        "price": "45001.00",
        "qty": "0.002",
        "time": 1640995210000,
        "isBuyerMaker": true
      }
    ]
  }
}
```

#### Subscribe to Balance Updates
```json
{
  "id": "21",
  "method": "subscribe_balance",
  "params": {
    "userExchangeIds": [1, 2]
  }
}
```

#### Subscribe to Notifications
```json
{
  "id": "22",
  "method": "subscribe_notifications",
  "params": {
    "types": ["order_update", "balance_update", "system_alert"]
  }
}
```

#### Subscribe to Option Chain
```json
{
  "id": "23",
  "method": "subscribe_option_chain",
  "params": {
    "userExchangeId": 1,
    "underlying": "BTC",
    "expiration": "25DEC24"
  }
}
```

### Account Management APIs

#### Get User Exchanges
```json
{
  "id": "9",
  "method": "get_user_exchanges",
  "params": {}
}
```

**Response:**
```json
{
  "id": "9",
  "status": "success",
  "data": [
    {
      "user_exchange_id": 1,
      "exchange_name": "binance",
      "display_name": "My Binance Account",
      "is_testnet": false
    }
  ]
}
```

#### Add User Exchange
```json
{
  "id": "10",
  "method": "add_user_exchange",
  "params": {
    "exchange_id": 1,
    "display_name": "My Binance Account",
    "data": {
      "api_key": "your_api_key",
      "api_secret": "your_api_secret",
      "is_testnet": false
    }
  }
}
```

#### Get Account Information
Retrieves detailed account information for a specific exchange.

```json
{
  "id": "11",
  "method": "get_account_info",
  "params": {
    "userExchangeId": 1
  }
}
```

**Parameters:**
- `userExchangeId` (integer): The user exchange ID to get account information for

**Response:**
```json
{
  "id": "11",
  "status": "success",
  "data": {
    "userExchangeId": 1,
    "exchangeName": "binance",
    "displayName": "My Binance Account",
    "balances": [
      {
        "asset": "BTC",
        "free": 1.5,
        "locked": 0.0,
        "total": 1.5
      }
    ],
    "positions": [],
    "totalValue": {
      "usd": 67500.0,
      "btc": 1.5,
      "eth": 22.5
    },
    "lastUpdated": 1640995200,
    "isConnected": true
  }
}
```

#### Get All Accounts Summary
Retrieves a simplified summary of all user exchanges with basic connection status.

```json
{
  "id": "12",
  "method": "get_all_accounts",
  "params": {}
}
```

**Response:**
```json
{
  "id": "12",
  "status": "success",
  "data": {
    "totalAccounts": 3,
    "connectedCount": 2,
    "accounts": [
      {
        "userExchangeId": 1,
        "exchangeName": "binance",
        "displayName": "My Binance Account",
        "lastUpdated": 1640995200,
        "isConnected": true
      },
      {
        "userExchangeId": 2,
        "exchangeName": "coinbase",
        "displayName": "My Coinbase Account",
        "lastUpdated": 1640995180,
        "isConnected": true
      },
      {
        "userExchangeId": 3,
        "exchangeName": "bybit",
        "displayName": "My Bybit Account",
        "lastUpdated": 1640995150,
        "isConnected": false,
        "error": "Not connected"
      }
    ],
    "lastUpdated": 1640995200
  }
}
```

**Response Fields:**
- `totalAccounts` (integer): Total number of configured exchanges
- `connectedCount` (integer): Number of successfully connected exchanges
- `accounts` (array): Array of simplified account information objects
- `lastUpdated` (integer): Unix timestamp when the summary was generated

**Account Object Fields:**
- `userExchangeId` (integer): Unique identifier for the user's exchange configuration
- `exchangeName` (string): Name of the exchange (e.g., "binance", "coinbase")
- `displayName` (string): User-defined display name for the exchange
- `lastUpdated` (integer): Unix timestamp when the account was last checked
- `isConnected` (boolean): Whether the exchange connection is active
- `error` (string, optional): Error message if connection failed

#### Get Balances
Retrieves account balances for specified exchanges. Supports single exchange, multiple exchanges, or all exchanges.

```json
{
  "id": "13",
  "method": "get_balances",
  "params": {
    "userExchangeIds": [1, 2]
  }
}
```

**Parameters:**
- `userExchangeIds` (array, optional): Array of user exchange IDs. If omitted, returns balances for all exchanges.

**Response for Single Exchange:**
```json
{
  "id": "13",
  "status": "success",
  "data": {
    "balances": [
      {
        "asset": "BTC",
        "free": 1.5,
        "locked": 0.0,
        "total": 1.5
      },
      {
        "asset": "USDT",
        "free": 10000.0,
        "locked": 500.0,
        "total": 10500.0
      }
    ]
  }
}
```

**Response for Multiple Exchanges:**
```json
{
  "id": "13",
  "status": "success",
  "data": {
    "balances": {
      "1": [
        {
          "asset": "BTC",
          "free": 1.5,
          "locked": 0.0,
          "total": 1.5
        },
        {
          "asset": "USDT",
          "free": 10000.0,
          "locked": 500.0,
          "total": 10500.0
        }
      ],
      "2": {
        "error": "Failed to get balances: Exchange not connected"
      }
    }
  }
}
```

**Response for All Exchanges:**
```json
{
  "id": "13",
  "status": "success",
  "data": {
    "balances": {
      "1": [
        {
          "asset": "BTC",
          "free": 1.5,
          "locked": 0.0,
          "total": 1.5
        }
      ],
      "2": [],
      "3": []
    }
  }
}
```

#### Get Open Orders
Retrieves open orders from specified exchanges. Supports filtering by symbol and multiple exchanges.

```json
{
  "id": "14",
  "method": "get_open_orders",
  "params": {
    "userExchangeIds": [1, 2],
    "symbol": "BTCUSDT"
  }
}
```

**Parameters:**
- `userExchangeIds` (array, optional): Array of user exchange IDs. If omitted, queries all exchanges.
- `symbol` (string, optional): Filter orders by specific trading pair

**Response:**
```json
{
  "id": "14",
  "status": "success",
  "data": {
    "userExchangeIds": [1, 2],
    "orders": {
      "1": [
        {
          "orderId": 123456789,
          "clientOrderId": "client_123",
          "symbol": "BTCUSDT",
          "price": "45000.00000000",
          "origQty": "0.00100000",
          "executedQty": "0.00050000",
          "status": "PARTIALLY_FILLED",
          "timeInForce": "GTC",
          "type": "LIMIT",
          "side": "BUY",
          "time": 1640995200000,
          "updateTime": 1640995250000
        }
      ],
      "2": []
    },
    "symbol": "BTCUSDT",
    "partialSuccess": false,
    "errors": []
  }
}
```

**Response Fields:**
- `userExchangeIds` (array): Exchange IDs that were queried
- `orders` (object): Open orders grouped by exchange ID
- `symbol` (string): Symbol filter that was applied (if any)
- `partialSuccess` (boolean): True if some exchanges failed but others succeeded
- `errors` (array): Array of error messages if any exchanges failed

#### Get Order History
Retrieves historical orders for a specific symbol on a user exchange. Currently supported for Binance exchanges.

```json
{
  "id": "15",
  "method": "get_order_history",
  "params": {
    "userExchangeId": 1,
    "symbol": "BTCUSDT",
    "limit": 50
  }
}
```

**Parameters:**
- `userExchangeId` (integer): The user exchange ID to fetch order history from
- `symbol` (string): Trading pair symbol (e.g., "BTCUSDT")
- `limit` (integer, optional): Maximum number of orders to return. Default: 10, Max: 1000

**Response:**
```json
{
  "id": "15",
  "status": "success",
  "data": {
    "userExchangeId": 1,
    "symbol": "BTCUSDT",
    "orders": [
      {
        "orderId": 123456789,
        "clientOrderId": "client_123",
        "symbol": "BTCUSDT",
        "price": "45000.00000000",
        "origQty": "0.00100000",
        "executedQty": "0.00100000",
        "status": "FILLED",
        "timeInForce": "GTC",
        "type": "LIMIT",
        "side": "BUY",
        "time": 1640995200000,
        "updateTime": 1640995250000
      }
    ],
    "count": 1
  }
}
```

**Error Response:**
```json
{
  "id": "15",
  "status": "error",
  "message": "Order history not supported for this exchange yet"
}
```

#### Get Recent Trades
Fetches recent public trades for a symbol from multiple exchanges, sorted by timestamp (newest first).

```json
{
  "id": "16",
  "method": "get_recent_trades",
  "params": {
    "userExchangeIds": [1, 2, 3],
    "symbol": "BTCUSDT",
    "limit": 20
  }
}
```

**Parameters:**
- `userExchangeIds` (array): Array of user exchange IDs to fetch trades from
- `symbol` (string): Trading pair symbol (e.g., "BTCUSDT")
- `limit` (integer, optional): Maximum number of trades per exchange. Default: 10

**Response:**
```json
{
  "id": "16",
  "status": "success",
  "data": {
    "symbol": "BTCUSDT",
    "userExchangeIds": [1, 2],
    "trades": [
      {
        "trade": {
          "id": 987654321,
          "price": "45000.50",
          "qty": "0.00150000",
          "quoteQty": "67.50075000",
          "time": 1640995300000,
          "isBuyerMaker": false,
          "isBestMatch": true
        },
        "userExchangeId": 1,
        "exchangeName": "binance",
        "displayName": "My Binance Account",
        "timestamp": 1640995300000
      }
    ],
    "count": 1,
    "requestedLimit": 20,
    "sortedBy": "timestamp_desc",
    "partialSuccess": false,
    "errors": []
  }
}
```

**Response Fields:**
- `symbol` (string): The requested trading pair
- `userExchangeIds` (array): Exchange IDs that were queried
- `trades` (array): Array of trade objects with exchange metadata
- `count` (integer): Total number of trades returned
- `requestedLimit` (integer): The limit parameter that was requested
- `sortedBy` (string): Sort order applied ("timestamp_desc")
- `partialSuccess` (boolean): True if some exchanges failed but others succeeded
- `errors` (array): Array of error messages if any exchanges failed

#### Get Positions
Retrieves trading positions for specified exchanges. Supports single exchange, multiple exchanges, or all exchanges.

```json
{
  "id": "17",
  "method": "get_positions",
  "params": {
    "userExchangeIds": [1, 2]
  }
}
```

**Parameters:**
- `userExchangeIds` (array, optional): Array of user exchange IDs. If omitted, returns positions for all exchanges.

**Response for Single Exchange:**
```json
{
  "id": "17",
  "status": "success",
  "data": {
    "positions": [
      {
        "symbol": "BTCUSDT",
        "side": "LONG",
        "size": 0.5,
        "value": 22500.0,
        "entryPrice": 45000.0,
        "markPrice": 45100.0,
        "pnl": 50.0,
        "percentage": 0.22
      }
    ]
  }
}
```

**Response for Multiple Exchanges:**
```json
{
  "id": "17",
  "status": "success",
  "data": {
    "positions": {
      "1": [
        {
          "symbol": "BTCUSDT",
          "side": "LONG",
          "size": 0.5,
          "value": 22500.0,
          "entryPrice": 45000.0,
          "markPrice": 45100.0,
          "pnl": 50.0,
          "percentage": 0.22
        }
      ],
      "2": {
        "error": "Failed to get positions: Exchange not connected"
      }
    }
  }
}
```

**Response for All Exchanges:**
```json
{
  "id": "17",
  "status": "success",
  "data": {
    "positions": {
      "1": [],
      "2": [],
      "3": []
    }
  }
}
```

**Note:** For spot trading exchanges like Binance, positions are typically empty as spot trading doesn't have traditional positions. This endpoint is more relevant for futures and margin trading.

### Options Trading APIs

#### Get Expiration Dates
```json
{
  "id": "60",
  "method": "get_expiration",
  "params": {
    "userExchangeId": 1,
    "underlying": "BTC"
  }
}
```

**Response:**
```json
{
  "id": "60",
  "status": "success",
  "data": {
    "underlying": "BTC",
    "expirations": [
      {
        "date": "25DEC24",
        "timestamp": 1735084800000,
        "daysToExpiry": 30
      },
      {
        "date": "31JAN25",
        "timestamp": 1738281600000,
        "daysToExpiry": 67
      }
    ]
  }
}
```

#### Get Option Chain Symbols
```json
{
  "id": "61",
  "method": "get_option_chain_symbols",
  "params": {
    "userExchangeId": 1,
    "underlying": "BTC",
    "expiration": "25DEC24"
  }
}
```

**Response:**
```json
{
  "id": "61",
  "status": "success",
  "data": {
    "underlying": "BTC",
    "expiration": "25DEC24",
    "options": [
      {
        "symbol": "BTC-25DEC24-50000-C",
        "type": "CALL",
        "strike": 50000,
        "expiration": "25DEC24"
      },
      {
        "symbol": "BTC-25DEC24-50000-P",
        "type": "PUT",
        "strike": 50000,
        "expiration": "25DEC24"
      }
    ]
  }
}
```

#### Get Contract Details
```json
{
  "id": "62",
  "method": "get_contract_details",
  "params": {
    "userExchangeId": 1,
    "symbol": "BTC-25DEC24-50000-C"
  }
}
```

**Response:**
```json
{
  "id": "62",
  "status": "success",
  "data": {
    "symbol": "BTC-25DEC24-50000-C",
    "underlying": "BTC",
    "type": "CALL",
    "strike": 50000,
    "expiration": "25DEC24",
    "multiplier": 1,
    "tickSize": 0.0005,
    "minSize": 0.1,
    "quoteCurrency": "USD",
    "settlementType": "CASH"
  }
}
```

#### Refresh Account Data
Forces a refresh of cached account data from exchanges.

```json
{
  "id": "18",
  "method": "refresh_account_data",
  "params": {
    "userExchangeIds": [1, 2]
  }
}
```

**Parameters:**
- `userExchangeIds` (array, optional): Array of user exchange IDs to refresh. If omitted, refreshes all exchanges.

**Response:**
```json
{
  "id": "18",
  "status": "success",
  "message": "Account data refresh initiated"
}
```

### DEX Trading APIs

#### Subscribe to DEX Price
```json
{
  "id": "30",
  "method": "subscribe_dex_price",
  "params": {
    "sellToken": "0xA0b86a33E6441c8C06DD2b7c9a5a7E2e7E8c4B5D",
    "buyToken": "0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599",
    "sellAmount": "1000000000000000000",
    "chainId": 1
  }
}
```

**Response:**
```json
{
  "id": "30",
  "status": "success",
  "data": {
    "subscriptionId": "30",
    "sellToken": "0xA0b86a33E6441c8C06DD2b7c9a5a7E2e7E8c4B5D",
    "buyToken": "0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599",
    "sellAmount": "1000000000000000000",
    "chainId": 1,
    "price": "0.000023456789",
    "buyAmount": "23456789012345"
  }
}
```

#### Get DEX Quote
```json
{
  "id": "31",
  "method": "get_dex_quote",
  "params": {
    "sellToken": "0xA0b86a33E6441c8C06DD2b7c9a5a7E2e7E8c4B5D",
    "buyToken": "0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599",
    "sellAmount": "1000000000000000000",
    "chainId": 1,
    "takerAddress": "0x1234567890123456789012345678901234567890"
  }
}
```

**Response:**
```json
{
  "id": "31",
  "status": "success",
  "data": {
    "sellToken": "0xA0b86a33E6441c8C06DD2b7c9a5a7E2e7E8c4B5D",
    "buyToken": "0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599",
    "sellAmount": "1000000000000000000",
    "buyAmount": "23456789012345",
    "price": "0.000023456789",
    "gasPrice": "20000000000",
    "estimatedGas": "150000",
    "to": "0x...",
    "data": "0x...",
    "value": "0"
  }
}
```

#### Get DEX Supported Chains
```json
{
  "id": "32",
  "method": "get_dex_chains",
  "params": {}
}
```

**Response:**
```json
{
  "id": "32",
  "status": "success",
  "data": {
    "chains": [
      {
        "chainId": 1,
        "name": "Ethereum",
        "supportsGasless": true
      },
      {
        "chainId": 137,
        "name": "Polygon",
        "supportsGasless": true
      }
    ]
  }
}
```

### Gasless Swap APIs

#### Submit Gasless Swap
```json
{
  "id": "33",
  "method": "submit_gasless_swap",
  "params": {
    "sellToken": "0xA0b86a33E6441c8C06DD2b7c9a5a7E2e7E8c4B5D",
    "buyToken": "0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599",
    "sellAmount": "1000000000000000000",
    "chainId": 1,
    "takerAddress": "0x1234567890123456789012345678901234567890",
    "signature": "0x..."
  }
}
```

#### Get Gasless Status
```json
{
  "id": "34",
  "method": "get_gasless_status",
  "params": {
    "tradeHash": "0xabcdef1234567890..."
  }
}
```

#### Get Gasless Approval Tokens
```json
{
  "id": "35",
  "method": "get_gasless_approval_tokens",
  "params": {
    "chainId": 1
  }
}
```

### Token Search APIs

#### Search Tokens
```json
{
  "id": "40",
  "method": "search_tokens",
  "params": {
    "query": "bitcoin",
    "chainIds": [1, 137],
    "limit": 20,
    "page": 0
  }
}
```

**Parameters:**
- `query` (string, optional): Search term for token name or symbol
- `chainIds` (array): Array of chain IDs to search on
- `limit` (integer, optional): Maximum results per page (1-100, default: 50)
- `page` (integer, optional): Page number for pagination (default: 0)

**Response:**
```json
{
  "id": "40",
  "status": "success",
  "data": {
    "tokens": [
      {
        "address": "0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599",
        "symbol": "WBTC",
        "name": "Wrapped Bitcoin",
        "decimals": 8,
        "chainId": 1,
        "logoURI": "https://..."
      }
    ],
    "totalCount": 1,
    "page": 0,
    "limit": 20
  }
}
```

#### Get Token Search Chains
```json
{
  "id": "41",
  "method": "get_token_search_chains",
  "params": {}
}
```

#### Clear Token Cache
```json
{
  "id": "42",
  "method": "clear_token_cache",
  "params": {}
}
```

#### Get Token Cache Stats
```json
{
  "id": "43",
  "method": "get_token_cache_stats",
  "params": {}
}
```

### Smart Limit Configuration APIs

#### Configure Smart Limit
```json
{
  "id": "50",
  "method": "configure_smart_limit",
  "params": {
    "name": "Conservative BTC Strategy",
    "symbol": "BTCUSDT",
    "maxSlippage": 0.5,
    "repriceThreshold": 0.1,
    "maxRepriceAttempts": 3,
    "isDefault": false
  }
}
```

#### Get Smart Limit Configs
```json
{
  "id": "51",
  "method": "get_smart_limit_configs",
  "params": {}
}
```

#### Delete Smart Limit Config
```json
{
  "id": "52",
  "method": "delete_smart_limit_config",
  "params": {
    "configId": "config_123"
  }
}
```

#### Set Default Smart Limit Config
```json
{
  "id": "53",
  "method": "set_default_smart_limit_config",
  "params": {
    "configId": "config_123"
  }
}
```

### Utility APIs

#### Ping
```json
{
  "id": "ping",
  "method": "ping",
  "params": {}
}
```

**Response:**
```json
{
  "id": "ping",
  "status": "success",
  "message": "pong"
}
```

#### Logout
```json
{
  "id": "logout",
  "method": "logout",
  "params": {}
}
```

---

## Trading Features

### Order Types

#### Regular Orders
- **LIMIT**: Buy/sell at specific price or better
- **MARKET**: Immediate execution at current market price
- **STOP_LOSS**: Trigger market order when price reaches stop price
- **STOP_LOSS_LIMIT**: Trigger limit order when price reaches stop price

#### Algorithmic Orders

##### TWAP (Time-Weighted Average Price)
Executes large orders over a specified time period to minimize market impact.

**Parameters:**
- `duration_minutes`: Total execution time
- `price_limit`: Maximum price for buy orders / minimum for sell orders
- `start_time`: When to begin execution (optional)

##### LIMIT Strategy
Enhanced limit orders with intelligent repricing based on market conditions.

**Parameters:**
- `price`: Initial limit price
- `reprice_threshold`: Price movement threshold for repricing
- `max_reprice_attempts`: Maximum number of reprice attempts

##### MARKET Strategy
Immediate execution with slippage protection and optimal sizing.

**Parameters:**
- `max_slippage`: Maximum acceptable slippage percentage
- `chunk_size`: Maximum order size per execution

### Time-in-Force Options
- **GTC (Good Till Cancelled)**: Order remains active until filled or cancelled
- **IOC (Immediate or Cancel)**: Fill immediately or cancel unfilled portion
- **FOK (Fill or Kill)**: Fill entire order immediately or cancel completely

---

## Supported Exchanges

### Binance
- **Spot Trading**: Full support for spot markets
- **Testnet**: Sandbox environment available
- **Features**: Real-time data, order management, account information

### Bitget
- **Spot Trading**: Major cryptocurrency pairs
- **Features**: Market data, order placement

### Bybit
- **Spot Trading**: Comprehensive spot market support
- **Features**: Real-time orderbook, trading functionality

### Coinbase
- **Professional Trading**: Coinbase Pro integration
- **Features**: Advanced order types, market data

### Deribit
- **Options & Futures**: Derivatives trading support
- **Features**: Complex instrument support

### OKX
- **Multi-Asset**: Spot, futures, and options
- **Features**: Comprehensive trading suite

---

## API Response Formats

### Success Response
```json
{
  "id": "request_id",
  "status": "success",
  "data": { /* response data */ },
  "message": "Operation completed successfully"
}
```

### Error Response
```json
{
  "id": "request_id",
  "status": "error",
  "message": "Error description",
  "code": "ERROR_CODE"
}
```

### Common Error Codes
- `INVALID_PARAMS`: Missing or invalid parameters
- `UNAUTHORIZED`: Authentication required
- `INSUFFICIENT_BALANCE`: Not enough funds
- `INVALID_SYMBOL`: Trading pair not supported
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `EXCHANGE_ERROR`: Error from exchange API

### Real-time Update Format
```json
{
  "type": "update_type",
  "symbol": "BTCUSDT",
  "exchange": "binance",
  "data": { /* update data */ },
  "timestamp": 1640995200000
}
```

### Update Types
- `orderbook_update`: Order book changes
- `trade_update`: New trade execution
- `order_update`: Order status change
- `balance_update`: Account balance change

---

## Getting Started

### 1. Connect to WebSocket
Connect to the WebSocket endpoint for your environment:
- Production: `wss://api.quantcite.com/`
- Testnet: `wss://testnet.quantcite.com/`

### 2. Authenticate
Send login credentials to authenticate your session.

### 3. Configure Exchanges
Add your exchange API credentials using the `add_user_exchange` method.

### 4. Start Trading
- Subscribe to market data for your trading pairs
- Place orders using the trading APIs
- Monitor order status and account balances

### Security Features
- **Rate Limiting**: Configurable per-user and per-IP rate limits
- **Connection Limits**: Maximum connections per user and per IP
- **CORS Protection**: Configurable origin validation
- **JWT Authentication**: Secure token-based authentication with refresh tokens
- **Request Validation**: Comprehensive parameter validation and sanitization

### Rate Limits
- **User API**: Configurable requests per minute per user
- **Market Data**: Configurable subscriptions per user
- **Order Placement**: Configurable orders per second per exchange
- **Admin API**: Separate rate limits for administrative operations

### Best Practices
- Always handle WebSocket reconnections
- Implement proper error handling for all API calls
- Use testnet environments for development and testing
- Monitor rate limits to avoid temporary blocks
- Keep API credentials secure and never expose them in client-side code

---

## Admin API Reference

The Admin API runs on port 8081 and provides administrative functions for managing users, exchanges, and system operations.

### Admin Authentication

#### Admin Login
```json
{
  "id": "admin_1",
  "method": "login",
  "params": {
    "username": "admin_username",
    "password": "admin_password"
  }
}
```

**Response:**
```json
{
  "id": "admin_1",
  "status": "success",
  "data": {
    "admin_id": 1,
    "username": "admin_username",
    "email": "admin@example.com",
    "token": "jwt_token_here"
  }
}
```

#### Admin Token Authentication
```json
{
  "id": "admin_2",
  "method": "authenticate_with_token",
  "params": {
    "token": "your_admin_jwt_token"
  }
}
```

### User Management APIs

#### Create Tenant/User
```json
{
  "id": "admin_3",
  "method": "create_tenant",
  "params": {
    "username": "new_user",
    "email": "user@example.com",
    "password": "secure_password"
  }
}
```

#### Create User
```json
{
  "id": "admin_4",
  "method": "create_user",
  "params": {
    "username": "new_user",
    "email": "user@example.com",
    "password": "secure_password"
  }
}
```

#### Get Users
```json
{
  "id": "admin_5",
  "method": "get_users",
  "params": {
    "limit": 50,
    "offset": 0
  }
}
```

#### Get User
```json
{
  "id": "admin_6",
  "method": "get_user",
  "params": {
    "user_id": 123
  }
}
```

#### Update User
```json
{
  "id": "admin_7",
  "method": "update_user",
  "params": {
    "user_id": 123,
    "username": "updated_username",
    "email": "updated@example.com"
  }
}
```

#### Delete User
```json
{
  "id": "admin_8",
  "method": "delete_user",
  "params": {
    "user_id": 123
  }
}
```

### Admin Exchange Management APIs

#### Get Exchanges (Admin)
```json
{
  "id": "admin_9",
  "method": "get_exchanges",
  "params": {}
}
```

#### Add User Exchange (Admin)
```json
{
  "id": "admin_10",
  "method": "add_user_exchange",
  "params": {
    "user_id": 123,
    "exchange_id": 1,
    "display_name": "User's Binance Account",
    "data": {
      "api_key": "user_api_key",
      "api_secret": "user_api_secret",
      "is_testnet": false
    }
  }
}
```

#### Get User Exchanges (Admin)
```json
{
  "id": "admin_11",
  "method": "get_user_exchanges",
  "params": {
    "user_id": 123
  }
}
```

#### Remove User Exchange (Admin)
```json
{
  "id": "admin_12",
  "method": "remove_user_exchange",
  "params": {
    "user_id": 123,
    "user_exchange_id": 456
  }
}
```

---

## Support

For technical support and API questions:
- Documentation: Visit our developer portal
- Support: Contact our technical support team
- Status: Check our system status page for real-time updates

---

*This documentation covers the public API interface for QuantCite Backend. For additional features or custom integrations, please contact our team.* 