# EVSEPro Prototype

Interactive HTML prototype for EVSEPro — an EV charging companion app for EU/US markets.

## What's inside

Single self-contained HTML file (`index.html`) rendering **45 mobile screens** including:

- **Auth flow** — sign in / sign up with email verification / forgot password
- **Home** — device list with groups, empty state
- **Device (DLB Pro 22)** — 3 segments (Now / Detail / Energy) + 3 modules (Status / Settings / History)
  - **Now** segment with 3D wallbox illustration + 7 state-aware variants (IDLE / READY / WAITING / CHARGING / STOPPED·charger / STOPPED·EV / COMPLETE), each with state-matched LCD content, LED color, and ambient glow
  - **Detail** segment with merged Cost · Range · Carbon triple-output card
  - **Energy** segment with PV/Grid/EV/Home flow diagram
- **Scenes** — automation triggers + actions
- **Charging schedule** — one-time + recurring time windows
- **Profile** — preferences, support, delete account flow
- **About this charger** — device identity, firmware, specs, network

## Tech notes

- Pure HTML + inline CSS + SVG. No build, no framework, no external deps.
- Tabler Icons via CDN.
- Renders at 360×800 mobile viewport.
- Dark theme with green/blue/amber/purple state palette.

## Design exploration

A left sidebar lists all 45 screens for quick navigation during design review.

## Status

Design prototype — not connected to a real backend.
