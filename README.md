# Smart Home Energy Dashboard

A React + Vite dashboard for monitoring and controlling smart-home loads in real time — built against a mock Express backend that simulates an ESP32-style device (lamp, phone charger, and an auto-cooling fan), reporting voltage, current, power, and cumulative energy per load, with threshold-based alerting and a live power-distribution pie chart.

Live demo: https://smarthome-dashboard-wheat.vercel.app

## How it works

The frontend polls `GET /api/status` once per second and renders the current state of each load. Toggling the Lamp or Phone Charger sends `POST /api/control` with `{ loadId, on }`; the fan is not manually controllable — it's presented as auto-controlled and reacts to total monitored power crossing a configurable threshold (`thresholds.highUsage_W`, currently 50 W).

The mock server (`server/server.js`) holds an in-memory state object for all three loads and perturbs each reading slightly on every poll so the dashboard has something to watch move. Turning a load off zeroes its current/power immediately; turning it on resumes noisy simulated readings. If the fan's power exceeds the threshold, the server appends a `HIGH_USAGE` alert to the state, which the dashboard surfaces in an Alerts panel.

This is designed as a stand-in for a real embedded device: the frontend only depends on the `/api/status` and `/api/control` contract, so swapping the mock server for firmware on an actual ESP32 (or similar) that speaks the same two endpoints would work without frontend changes.

## Project structure

```
.
├── server/
│   ├── server.js         # Express mock device server (in-memory state, fake sensor noise)
│   └── package.json
├── src/
│   ├── App.jsx           # Main dashboard: polling, load cards, pie chart, alerts
│   ├── App.css
│   ├── main.jsx           # React entry point
│   └── index.css
├── public/
├── index.html
├── vite.config.js
└── package.json
```

## Requirements

- Node.js 18+
- npm

## Setup

Install dependencies for both the frontend and the mock server:

```bash
npm install
cd server && npm install && cd ..
```

The frontend reads its API base URL from an environment variable. Create a `.env` file in the project root:

```
VITE_API_BASE_URL=http://localhost:4000
```

## Running locally

Start the mock backend (listens on port 4000):

```bash
cd server
node server.js
```

In a separate terminal, start the Vite dev server:

```bash
npm run dev
```

Vite will print the local dev URL (typically `http://localhost:5173`). The dashboard will begin polling the mock server every second.

## Available scripts

| Command | Description |
|---|---|
| `npm run dev` | Start the Vite dev server with HMR |
| `npm run build` | Production build to `dist/` |
| `npm run preview` | Preview the production build locally |
| `npm run lint` | Run ESLint |

## Notes

- All device state is in-memory on the mock server and resets on restart — there is no persistence layer.
- The simulated fan cannot be toggled manually by design; it exists to demonstrate threshold-driven automation rather than direct control.
- `VITE_API_BASE_URL` must point to wherever `server.js` (or a real device implementing the same API) is reachable — update it for deployed environments accordingly.
