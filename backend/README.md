# MEV Exorcist Backend — The Seer

Backend component of **MEV Exorcist** that monitors the **Ethereum Sepolia mempool** to detect potential MEV attack targets in real time.

It connects to **Alchemy WebSocket**, filters **Uniswap V3 swap transactions**, classifies them by **risk level**, and broadcasts them to frontend clients via **Socket.io**.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Scripts](#scripts)
- [API](#api)
- [Socket.io Events](#socketio-events)
- [Deployment](#deployment)
- [Architecture](#architecture)
- [Testing](#testing)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)
- [License](#license)

---

## Features

- Real-time mempool monitoring via **Alchemy WebSocket**
- Automatic reconnection (exponential backoff)
- Uniswap V3 Router transaction filtering
- ABI decoding for swap function calls
- Risk classification based on ETH threshold
- Broadcast to multiple clients via **Socket.io**
- Health check endpoint (`/health`)
- Graceful shutdown handling

---

## Tech Stack

- Node.js + TypeScript
- Alchemy WebSocket API
- Socket.io
- Jest + fast-check (property-based testing)

---

## Prerequisites

- Node.js **18+**
- npm / yarn
- Alchemy API key (with Sepolia WebSocket access)

---

## Getting Started

```bash
# Install dependencies
npm install

# Copy env template
cp .env.example .env

# Edit your .env
```

Run dev server:

```bash
npm run dev
```

---

## Environment Variables

Create a `.env` file:

```bash
# Required: Alchemy WebSocket URL (Sepolia)
ALCHEMY_WSS_URL=wss://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY

# Required: Uniswap V3 Router address (Sepolia)
UNISWAP_V3_ROUTER=0xE592427A0AEce92De3Edee1F18E0157C05861564

# Optional: Server port (default: 3001)
PORT=3001

# Optional: Risk threshold in ETH (default: 0.1)
RISK_THRESHOLD_ETH=0.1

# Optional: Environment (default: development)
NODE_ENV=development
```

---

## Scripts

```bash
# Dev server (auto reload)
npm run dev

# Build TypeScript
npm run build

# Start production server
npm start

# Tests
npm test
npm run test:watch
npm run test:coverage
```

---

## API

### Health Check

**GET** `/health`

Example response:

```json
{
  "status": "ok",
  "connections": 2,
  "uptime": 123.456
}
```

**Fields**
- `status`: `"ok"` when running
- `connections`: number of connected Socket.io clients
- `uptime`: uptime in seconds

---

## Socket.io Events

### `transaction`

Emitted when a classified transaction is detected.

Payload:

```ts
{
  hash: string;
  from: string;
  ethValue: string; // formatted to 4 decimals
  riskLevel: "HIGH" | "LOW";
  timestamp: number; // unix timestamp
  functionName: string;
}
```

---

## Deployment

Full guide: **[DEPLOYMENT.md](./DEPLOYMENT.md)**

### Railway

```bash
npm install -g @railway/cli
railway login
railway init
railway up
```

### Render

1. Connect repo in Render  
2. Create a **Web Service**
3. Render auto-detects `render.yaml`
4. Set env variables
5. Deploy

### Docker

```bash
docker build -t mev-exorcist-backend .

docker run -p 3001:3001   -e ALCHEMY_WSS_URL="wss://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY"   -e UNISWAP_V3_ROUTER="0xE592427A0AEce92De3Edee1F18E0157C05861564"   mev-exorcist-backend
```

### Verify Deployment

```bash
curl https://your-backend-url.com/health

# Or use verification script
node scripts/verify-deployment.js https://your-backend-url.com
```

---

## Architecture

```txt
┌──────────────────────────────────────────────────────┐
│                 MEV Exorcist Backend                 │
├──────────────────────────────────────────────────────┤
│  Alchemy WS → Tx Processor → ABI Decoder → Risk CLS  │
│                          ↓                           │
│                    Socket.io Broadcaster             │
└──────────────────────────────────────────────────────┘
                          ↓
                    Frontend Clients
```

### Components

- **AlchemyWebSocketClient**: manages Alchemy WS connection + reconnection
- **TransactionProcessor**: fetches and filters transactions
- **ABIDecoder**: decodes Uniswap V3 swap calls
- **RiskClassifier**: labels tx based on ETH value threshold
- **TransactionBroadcaster**: broadcasts to all clients

---

## Testing

Included tests:
- Unit tests
- Property-based tests (fast-check)
- Integration tests

Examples:

```bash
npm test
npm test -- ABIDecoder.test.ts
npm run test:coverage
```

---

## Monitoring

### Logs

Example output:

```txt
=== MEV EXORCIST BACKEND ===
Environment: production
WebSocket: wss://eth-sepolia.g.alchemy.com/v2/...
Uniswap V3 Router: 0xE592427...
Server Port: 3001
Risk Threshold: 0.1 ETH

✓ Connected to Alchemy WebSocket
✓ Subscribed to pending transactions

🔍 Monitoring mempool for MEV targets...

✓ LOW  | 0.0500 ETH | exactInputSingle | 0x1234...
🚨 HIGH | 0.5000 ETH | exactInput       | 0xabcd...
```

### Recommended Metrics

- `/health` response time
- connected clients count
- tx processing rate
- WebSocket reconnect frequency
- memory + CPU usage

---

## Troubleshooting

### WebSocket cannot connect

**Symptoms**: `Failed to connect to Alchemy`

**Fix**
- confirm `ALCHEMY_WSS_URL` is correct
- ensure API key has WebSocket enabled
- check firewall/network restrictions

### No transactions detected

**Fix**
- ensure you're on **Sepolia** not mainnet
- confirm Uniswap V3 activity exists
- verify `UNISWAP_V3_ROUTER`
- check Alchemy dashboard (errors / quota)

### High memory usage

**Fix**
- run prod build: `npm run build && npm start`
- check potential leaks
- monitor tx rate
- scale resources

---

## Security Notes

- Never commit `.env`
- Keep API keys secret
- Use HTTPS in production
- Restrict CORS in production
- Monitor API usage to prevent abuse

---

## License

MIT
