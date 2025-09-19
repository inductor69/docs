# QuantCite API Documentation

Real-time cryptocurrency market data through WebSocket connections with 50GB monthly data limits.

This documentation site provides comprehensive guides and API reference for integrating with the QuantCite Data Pipeline, which aggregates orderbook data from 52+ active exchanges across 70+ supported platforms.

## Features

- **WebSocket API Documentation** - Complete guide to real-time data streaming
- **HTTP Endpoints** - Health checks and data usage monitoring
- **Client Examples** - Python and JavaScript implementation examples
- **Rate Limiting Guide** - Tier-based limits and best practices
- **Interactive API Reference** - Complete message type documentation

## Development

Install the [Mintlify CLI](https://www.npmjs.com/package/mint) to preview documentation changes locally:

```bash
npm i -g mint
```

Run the development server at the root of your documentation:

```bash
mint dev
```

View your local preview at `http://localhost:3000`.

## API Overview

- **Base URL:** `https://data.quantcite.com`
- **WebSocket:** `wss://data.quantcite.com/api/v1/ws`
- **Data Limit:** 50GB per API key per month
- **Exchanges:** 52+ active exchanges across 70+ supported platforms
- **Trading Pairs:** 50,000+ supported pairs
- **Rate Limits:** 600-10,000 requests/minute (tier-based)
- **Security:** SSL/TLS encryption with secure WebSocket connections

## Quick Start

1. **Get API Key** - Contact support for your API credentials
2. **Connect WebSocket** - Establish connection with authentication
3. **Subscribe to Data** - Choose exchanges and trading pairs
4. **Monitor Usage** - Track your 50GB monthly data allowance

## Need help?

### API Support
- Monitor usage: `/api/v1/data-usage/{api_key}`
- Health check: `/api/v1/health`
- WebSocket endpoint: `/api/v1/ws?api_key=YOUR_API_KEY`

### Documentation
- [Quick Start Guide](/quickstart)
- [WebSocket Messages](/websocket/messages)
- [API Reference](/api-reference/introduction)
- [Client Examples](/examples/python) 
