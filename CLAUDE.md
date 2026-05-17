# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ioBroker adapter for VW Group connected car services: VW We Connect, We Connect ID, We Charge, myAudi, Skoda Connect, Seat Connect, Cupra, and We Connect Go. It authenticates with VAG (Volkswagen AG) cloud APIs, polls vehicle data, and exposes remote control commands (climate, lock, charging, etc.) as ioBroker states.

## Commands

- **Install**: `npm install`
- **Lint**: `npm run lint` (ESLint with flat config)
- **Type check**: `npm run check` (TypeScript `--noEmit` against JS via `tsconfig.check.json`)
- **All tests**: `npm test` (runs `test:js` then `test:package`)
- **Unit tests only**: `npm run test:unit`
- **Package tests only**: `npm run test:package`
- **Integration tests**: `npm run test:integration`
- **Release**: `npm run release` (uses `@alcalzone/release-script`)

Requires Node.js >= 22.12.0.

## Code Style

- 2-space indentation, double quotes, semicolons required
- `prefer-const`, `no-var`
- Unused vars error (except rest siblings and `_`-prefixed args)
- CommonJS modules (`require`/`module.exports`), not ESM

## Architecture

This is a single-class adapter. Nearly all logic lives in **`main.js`** (~7k lines), a single `VwWeconnect` class extending `utils.Adapter`:

- **Authentication**: Multiple OAuth2 flows for different brands (VW, Audi, Skoda, Seat/Cupra, VW ID). Each brand has its own client ID, scope, and login endpoint. Token refresh is handled via intervals.
- **Data polling**: `statesArray` defines API endpoints with URL templates (`$homeregion`, `$type`, `$vin`, etc.) that are expanded per-vehicle. Polling interval is configurable; responses are parsed and written to ioBroker object tree via `json2iob` and `extractKeys`.
- **Remote commands**: `onStateChange` handles user-initiated commands (lock, climate, charging, etc.) by sending requests to brand-specific API endpoints.
- **MQTT**: Skoda uses MQTT (via `mqtt` package) for real-time vehicle status updates alongside REST polling.

Supporting files:
- **`lib/extractKeys.js`**: Recursively walks JSON responses and creates/updates ioBroker state objects. Handles arrays, nested objects, and settings (writable states).
- **`lib/tools.js`**: Utility helpers (`isObject`, `isArray`, `translateText`).
- **`admin/index_m.html`**: Adapter configuration UI (brand selection, credentials, polling intervals).
- **`io-package.json`**: ioBroker adapter metadata, version history, and instance configuration schema.

## Brand-Specific Logic

The adapter handles each car brand differently. Key config field is `this.config.type` which determines login flow, API base URLs, and available features. The brands share some common VW Group APIs but diverge significantly in auth endpoints and data formats.
