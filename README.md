# Tunnel Agent CLI

<div align="center">

![Tunnel Agent CLI](https://img.shields.io/badge/tunnel-agent--cli-blue?style=for-the-badge)
![Version](https://img.shields.io/badge/version-1.0.0-green?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-yellow?style=for-the-badge)
![Node](https://img.shields.io/badge/node-%3E%3D22.16.0-brightgreen?style=for-the-badge&logo=node.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue?style=for-the-badge&logo=typescript)

**A powerful CLI tool to expose your local services to the internet via secure tunnels.**

*Similar to ngrok and localtunnel — built with TypeScript and Socket.IO.*

</div>

---

## Table of Contents

- [Tunnel Agent CLI](#tunnel-agent-cli)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Features](#features)
  - [Architecture](#architecture)
  - [Prerequisites](#prerequisites)
  - [Getting Started](#getting-started)
    - [Clone the Repository](#clone-the-repository)
    - [Install Dependencies](#install-dependencies)
    - [Environment Setup](#environment-setup)
    - [Build the Project](#build-the-project)
  - [Usage](#usage)
    - [Development Mode](#development-mode)
    - [Production Mode](#production-mode)
  - [Commands](#commands)
    - [Start an HTTP Tunnel](#start-an-http-tunnel)
    - [Check Tunnel Status](#check-tunnel-status)
    - [Manage Authentication Tokens](#manage-authentication-tokens)
      - [Create a Token (requires an account)](#create-a-token-requires-an-account)
  - [Project Structure](#project-structure)
  - [How It Works](#how-it-works)
  - [Configuration](#configuration)
    - [Environment Variables (Development Only)](#environment-variables-development-only)
  - [Scripts](#scripts)
  - [Dependencies](#dependencies)
    - [Runtime Dependencies](#runtime-dependencies)
    - [Dev Dependencies](#dev-dependencies)
  - [Publishing a New Version](#publishing-a-new-version)
    - [How Publishing Works](#how-publishing-works)
    - [Before Pushing a New Version](#before-pushing-a-new-version)
    - [Publishing to npm (Optional)](#publishing-to-npm-optional)
  - [Contributing](#contributing)
    - [Reporting Issues](#reporting-issues)
  - [License](#license)

---

## Overview

**Tunnel Agent CLI** is the client-side agent for the Tunnel tunneling service. It creates a secure WebSocket connection (via Socket.IO) to a Tunnel server and forwards incoming HTTP requests from the public internet to your locally running service.

Think of it as your personal ngrok — run a command, get a public URL, and share your local app with the world instantly. The agent connects to whatever domain your Tunnel server is deployed on, configured via the `--server` flag or the `TUNNEL_SERVER_URL` environment variable.

---

## Features

- 🚀 **Instant public URLs** — Expose any local HTTP service in seconds
- 🔒 **Token-based authentication** — Secure your tunnels with auth tokens
- 🌐 **Custom subdomains** — Request a specific subdomain for your tunnel
- 🔄 **Auto-reconnect** — Automatically reconnects with exponential backoff (up to 10 retries)
- 💓 **Heartbeat monitoring** — Keeps the connection alive with periodic heartbeats
- 📡 **Local service health ping** — Continuously monitors your local service availability
- 🧪 **Quick no-auth tokens** — Generate test tokens instantly without signing up
- 📊 **Request logging** — Logs all forwarded requests back to the server
- 🎨 **Colorful CLI output** — Clear, color-coded terminal feedback via Chalk

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Internet / Browser                       │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTP Request
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Tunnel Server                                │
│              Receives public HTTP requests and routes           │
│              them to the correct agent via Socket.IO            │
└───────────────────────────┬─────────────────────────────────────┘
                            │ WebSocket (Socket.IO)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Tunnel Agent CLI (this repo)                   │
│   Receives request messages → forwards to localhost → responds   │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTP (localhost)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Your Local Service (e.g. :8080)                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

Before you begin, ensure you have the following installed:

| Tool                                          | Version      | Notes                         |
| --------------------------------------------- | ------------ | ----------------------------- |
| [Node.js](https://nodejs.org/)                | `>= 22.16.0` | Required                      |
| [npm](https://www.npmjs.com/)                 | `>= 10.x`    | Comes with Node.js            |
| [Git](https://git-scm.com/)                   | Any          | For cloning the repo          |
| [TypeScript](https://www.typescriptlang.org/) | `>= 5.x`     | Installed via devDependencies |

---

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/uzochukwueddie/tunnel.git
cd tunnel
```

### Install Dependencies

```bash
npm install
```

### Environment Setup

Create a `.env` file in the root of the project for local development:

```bash
cp .env.example .env
```

> **Note:** There is no `.env.example` committed to the repo (it's gitignored). Create a `.env` file manually with the following variables:

```env
# Required for development mode
NODE_ENV=development

# The URL of your local or self-hosted tunnel server
TUNNEL_SERVER_URL=http://localhost:8080

# Agent version (used in development)
AGENT_VERSION=1.0.0
```

> In **production**, the CLI connects to the domain hardcoded in `src/config.ts`. Before building and publishing, update the `SERVER_URL` value in `src/config.ts` to point to your deployed Tunnel server domain. The `.env` file is only needed when running in development mode.

### Build the Project

```bash
npm run build
```

This runs the following steps:
1. Compiles TypeScript to ESM (`dist/esm/`)
2. Compiles TypeScript to CJS (`dist/cjs/`)
3. Copies a stripped `package.json` to `dist/` (without build scripts)

---

## Usage

### Development Mode

Run the CLI directly from TypeScript source using `nodemon`:

```bash
npm run dev -- <command> [options]
```

**Examples:**

```bash
# Create an authentication token
# Note: Use --email and --password (long flags) in development mode.
# Short flags -e and -p are intercepted by nodemon and will not work.
npm run dev -- token create --email user@example.com --password password123 -n my-dev-token

# Start an HTTP tunnel on port 3000
npm run dev -- http 3000 -t <your-token>

# Start a tunnel with a custom subdomain
npm run dev -- http 3000 -t <your-token> -s myapp

# Check tunnel server status
npm run dev -- status
```

### Production Mode

After building, run the CLI using Node.js:

```bash
npm run start -- <command> [options]
```

Or, if installed globally:

```bash
tunnel <command> [options]
```

---

## Commands

### Start an HTTP Tunnel

Expose a local HTTP service to the internet.

```bash
tunnel http <port> -t <token> [options]
```

| Argument / Option             | Required | Description                                                      |
| ----------------------------- | -------- | ---------------------------------------------------------------- |
| `<port>`                      | ✅ Yes    | Local port number to forward traffic to (1–65535)                |
| `-t, --token <token>`         | ✅ Yes    | Authentication token                                             |
| `-s, --subdomain <subdomain>` | ❌ No     | Request a specific subdomain                                     |
| `--server <url>`              | ✅ Yes    | Your deployed Tunnel server URL (e.g. `https://your-domain.com`) |
| `--no-reconnect`              | ❌ No     | Disable automatic reconnection                                   |

**Examples:**

```bash
# Basic tunnel on port 3000
tunnel http 3000 -t abc123...

# Tunnel with a custom subdomain
tunnel http 3000 -t abc123... -s myapp

# Tunnel against a self-hosted server
tunnel http 3000 -t abc123... --server http://localhost:8080

# Tunnel without auto-reconnect
tunnel http 3000 -t abc123... --no-reconnect
```

**Successful connection output:**

```
╔═══════════════════════════════════════════════════════════════╗
║                  Tunnel Established Successfully              ║
╠═══════════════════════════════════════════════════════════════╣
║  Public URL:     https://<subdomain>.<server-domain>          ║
║  Subdomain:      <subdomain>                                  ║
║  Tunnel ID:      abc123-def456-...                            ║
║  Forwarding to:  http://localhost:3000                        ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### Check Tunnel Status

View all currently active tunnels on the server.

```bash
tunnel status [options]
```

| Option           | Required | Description                                                      |
| ---------------- | -------- | ---------------------------------------------------------------- |
| `--server <url>` | ✅ Yes    | Your deployed Tunnel server URL (e.g. `https://your-domain.com`) |

**Example:**

```bash
tunnel status --server https://your-domain.com
tunnel status --server http://localhost:8080
```

---

### Manage Authentication Tokens

The `token` command group lets you create and manage authentication tokens.

#### Create a Token (requires an account)

> ⚠️ **Short flag note:** The `-e` and `-p` short flags **only work in production** (i.e. when running the installed `tunnel` binary). When running locally via `npm run dev`, nodemon intercepts `-e` (as `--ext`) and `-p` (as `--polling-interval`), so those flags never reach your CLI. **Always use `--email` and `--password` in development mode.**

**Production (installed binary):**

```bash
tunnel token create -e <email> -p <password> -n <name> [options]
```

**Development (`npm run dev`):**

```bash
npm run dev -- token create --email <email> --password <password> -n <name> [options]
```

| Option                      | Required | Description                                                      |
| --------------------------- | -------- | ---------------------------------------------------------------- |
| `-e, --email <email>`       | ✅ Yes    | Your account email                                               |
| `-p, --password <password>` | ✅ Yes    | Your account password                                            |
| `-n, --name <name>`         | ✅ Yes    | A label for the token (e.g. `my-dev-token`)                      |
| `--server <url>`            | ✅ Yes    | Your deployed Tunnel server URL (e.g. `https://your-domain.com`) |

**Examples:**

```bash
# Production — short flags work fine
tunnel token create -e user@example.com -p mypassword -n my-dev-token --server https://your-domain.com

# Development — must use long flags (--email / --password)
npm run dev -- token create --email user@example.com --password mypassword -n my-dev-token
```

> ⚠️ **Important:** Save the token immediately after creation — it will not be shown again.

---

## Project Structure

```
tunnel-agent-cli/
├── src/
│   ├── index.ts                          # CLI entry point (Commander.js commands)
│   ├── config.ts                         # Server URL and version configuration
│   ├── interfaces/
│   │   ├── http-proxy-interface.ts       # ProxyRequest / ProxyResponse types
│   │   ├── protocol.interface.ts         # WebSocket message types & interfaces
│   │   └── tunnel-client.interface.ts    # TunnelClientOptions interface
│   ├── services/
│   │   ├── tunnel-client.service.ts      # Main tunnel client (connect/disconnect)
│   │   ├── message-handler.service.ts    # Handles all incoming socket messages
│   │   └── http-proxy.service.ts         # Forwards HTTP requests to localhost
│   ├── socketIO/
│   │   └── socket-handler.ts             # Socket.IO event listeners & routing
│   └── utils/
│       └── utils.ts                      # Message helpers, serialization, URL fixing
├── scripts/
│   └── copy-package.js                   # Strips build scripts from dist/package.json
├── .editorconfig                         # Editor formatting rules
├── .gitignore                            # Git ignored files
├── .npmrc                                # npm registry configuration
├── package.json                          # Project metadata and scripts
├── tsconfig.json                         # TypeScript config (ESM output)
└── tsconfig.cjs.json                     # TypeScript config (CJS output)
```

---

## How It Works

1. **CLI Invocation** — You run `tunnel http <port> -t <token>`. Commander.js parses the arguments.
2. **TunnelClient connects** — A Socket.IO WebSocket connection is established to the tunnel server at `/agent`.
3. **CONNECT message sent** — The agent sends a `CONNECT` message with your token, requested subdomain, local port, and agent version.
4. **CONNECT_ACK received** — The server responds with a `tunnelId`, `subdomain`, and `publicUrl`. The public URL is displayed in the terminal.
5. **Heartbeat starts** — The agent sends a `HEARTBEAT` message every 30 seconds to keep the connection alive.
6. **Local service ping starts** — The agent pings your local service every 5 seconds and reports its availability to the server.
7. **REQUEST received** — When someone visits your public URL, the server sends a `REQUEST` message to the agent.
8. **HTTP forwarded** — The `HttpProxy` service forwards the request to `http://localhost:<port>` using Axios.
9. **RESPONSE sent back** — The response (status, headers, body as base64) is sent back to the server via a `RESPONSE` message.
10. **Request log sent** — A `REQUEST_LOG` message is sent to the server with method, path, status code, and response time.
11. **Auto-reconnect** — If the connection drops, the agent retries up to 10 times with exponential backoff (5s → 60s max).

---

## Configuration

The production server URL and agent version are set directly in `src/config.ts`:

```ts
// src/config.ts
const SERVER_URL =
  process.env.NODE_ENV === 'development'
    ? process.env.TUNNEL_SERVER_URL
    : 'https://your-deployed-tunnel-server.com'; // ← Update this before building

const APP_AGENT_VERSION =
  process.env.NODE_ENV === 'development' ? process.env.AGENT_VERSION : '1.0.2';
```

> ⚠️ **Before building for production**, open `src/config.ts` and replace the production `SERVER_URL` value with the domain of your deployed Tunnel server. This is the URL the CLI will connect to by default when `NODE_ENV` is not `development`.

### Environment Variables (Development Only)

| Variable            | Description                                                   |
| ------------------- | ------------------------------------------------------------- |
| `NODE_ENV`          | Set to `development` to use `.env` values instead of defaults |
| `TUNNEL_SERVER_URL` | URL of your local or self-hosted tunnel server (dev only)     |
| `AGENT_VERSION`     | Agent version reported to the server (dev only)               |

You can also override the server URL at runtime using the `--server` flag on any command, regardless of what is set in `config.ts`.

---

## Scripts

| Script      | Command             | Description                                       |
| ----------- | ------------------- | ------------------------------------------------- |
| `dev`       | `npm run dev`       | Run in development mode with nodemon (hot reload) |
| `build`     | `npm run build`     | Build ESM + CJS outputs and copy package.json     |
| `build:esm` | `npm run build:esm` | Compile TypeScript to ESM only                    |
| `build:cjs` | `npm run build:cjs` | Compile TypeScript to CJS only                    |
| `start`     | `npm run start`     | Run the built CLI from `dist/cjs/index.js`        |
| `lint`      | `npm run lint`      | Lint TypeScript source files with ESLint          |

---

## Dependencies

### Runtime Dependencies

| Package                                         | Version   | Purpose                                              |
| ----------------------------------------------- | --------- | ---------------------------------------------------- |
| [axios](https://axios-http.com/)                | `^1.13.5` | HTTP client for forwarding requests to local service |
| [chalk](https://github.com/chalk/chalk)         | `^5.6.2`  | Terminal string styling                              |
| [commander](https://github.com/tj/commander.js) | `^14.0.3` | CLI argument parsing                                 |
| [dotenv](https://github.com/motdotla/dotenv)    | `^17.2.4` | Environment variable loading                         |
| [socket.io-client](https://socket.io/)          | `^4.8.3`  | WebSocket client for tunnel server communication     |

### Dev Dependencies

| Package                                                     | Version   | Purpose                                           |
| ----------------------------------------------------------- | --------- | ------------------------------------------------- |
| [typescript](https://www.typescriptlang.org/)               | `^5.9.3`  | TypeScript compiler                               |
| [ts-node](https://typestrong.org/ts-node/)                  | `^10.9.2` | Run TypeScript directly in Node.js                |
| [tsc-alias](https://github.com/justkey007/tsc-alias)        | `^1.8.16` | Resolve TypeScript path aliases after compilation |
| [tsconfig-paths](https://github.com/dividab/tsconfig-paths) | `^4.2.0`  | Runtime path alias resolution (for `nodemon`)     |

---

## Publishing a New Version

### How Publishing Works

Publishing is triggered automatically via a **GitHub Actions CI/CD pipeline** whenever new changes are pushed to the repository. You do not need to run `npm publish` manually — pushing to the main branch (or your configured release branch) will trigger the pipeline and publish the package to the **GitHub Package Registry**.

> Make sure your repository has a GitHub Actions workflow configured for publishing. You will also need a `NODE_AUTH_TOKEN` secret set in your repository settings (under **Settings → Secrets and variables → Actions**) with a GitHub Personal Access Token that has `write:packages` permission.

### Before Pushing a New Version

Before pushing, make sure to update the package name in `package.json` to use **your own GitHub username as the scope**:

```json
{
  "name": "@<your-github-username>/tunnel-cli"
}
```

Also update `.npmrc` to point to your own scope:

```
@<your-github-username>:registry=https://npm.pkg.github.com
```

Then update the version in **both** of the following places:

1. **`package.json`** — Update the `"version"` field:
   ```json
   {
     "version": "1.0.3"
   }
   ```

2. **`src/config.ts`** — Update the `APP_AGENT_VERSION` production value to match:
   ```ts
   const APP_AGENT_VERSION =
     process.env.NODE_ENV === 'development' ? process.env.AGENT_VERSION : '1.0.3';
   ```

> ⚠️ Both values must be kept in sync. The version in `config.ts` is what the agent reports to the tunnel server on connection.

Once the version is updated, commit and push your changes:

```bash
git add .
git commit -m "chore: bump version to 1.0.3"
git push origin main
```

The CI/CD pipeline will automatically build and publish the new version to the GitHub Package Registry.

### Publishing to npm (Optional)

If you prefer to publish to the **public npm registry** instead of (or in addition to) the GitHub Package Registry, update `package.json` to remove the scoped name or use a public scope, and update `publishConfig`:

```json
{
  "name": "tunnel-cli",
  "publishConfig": {
    "registry": "https://registry.npmjs.org/"
  }
}
```

Then log in to npm and publish manually:

```bash
npm login
npm run build
npm publish
```

Or configure your CI/CD pipeline to publish to npm using an `NPM_TOKEN` secret.

---

## Contributing

Contributions are welcome! Here's how to get started:

1. **Fork** the repository on GitHub
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/<your-username>/tunnel.git
   cd tunnel
   ```
3. **Install** dependencies:
   ```bash
   npm install
   ```
4. **Create** a new branch for your feature or fix:
   ```bash
   git checkout -b feature/your-feature-name
   ```
5. **Make** your changes and ensure the project builds:
   ```bash
   npm run build
   ```
6. **Commit** your changes with a descriptive message:
   ```bash
   git commit -m "feat: add your feature description"
   ```
7. **Push** to your fork and open a **Pull Request**:
   ```bash
   git push origin feature/your-feature-name
   ```

### Reporting Issues

Found a bug or have a feature request? Please [open an issue](https://github.com/uzochukwueddie/tunnel/issues) on GitHub.

---

## License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

<div align="center">

Made with ❤️ by [Uzochukwu Eddie Odozi](https://github.com/uzochukwueddie)

⭐ If you find this project useful, please consider giving it a star on GitHub!

</div>
