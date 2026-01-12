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
