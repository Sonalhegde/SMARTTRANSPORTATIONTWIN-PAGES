# Smart Transportation Digital Twin

<p align="left">
  <a href="https://sonalhegde.github.io/SMARTTRANSPORTATIONTWIN-PAGES/"><img alt="Live Demo" src="https://img.shields.io/badge/demo-live-brightgreen"></a>
  <a href="https://github.com/Sonalhegde/SMARTTRANSPORTATIONTWIN/tree/main"><img alt="Repository" src="https://img.shields.io/badge/repo-GitHub-blue"></a>
</p>

**Live demo:** https://sonalhegde.github.io/SMARTTRANSPORTATIONTWIN-PAGES/
**Repository:** https://github.com/Sonalhegde/SMARTTRANSPORTATIONTWIN/tree/main

## Table of Contents

- [Abstract](#abstract)
- [Applied Cyber Physical Systems Initiative](#applied-cyber-physical-systems-initiative)
- [Core Capabilities](#core-capabilities)
- [System Architecture](#system-architecture)
- [Source Layout](#source-layout)
- [Vehicle and Infrastructure Flow](#vehicle-and-infrastructure-flow)
- [Cargo Configuration](#cargo-configuration)
- [HiveMQ Device Mode](#hivemq-device-mode)
- [Installation](#installation)
- [GitHub Pages Hosting](#github-pages-hosting)
- [Verification Checklist](#verification-checklist)
- [Repository](#repository)

## Abstract

The Smart Transportation Digital Twin is a browser-based cyber-physical simulation for studying coordinated mobility, automated infrastructure, factory logistics, safety events, and connected sensing.

The application combines a live Three.js environment with route-driven vehicle agents, collision avoidance, railway and toll control, RFID-authorized factory access, automated goods loading, accident response, activity logging, and HiveMQ telemetry. Manual Mode provides deterministic simulation scenarios. Device Mode consumes acknowledged hardware state from a shared MQTT connection and applies it to the same infrastructure animation controllers.

## Applied Cyber Physical Systems Initiative

This project is positioned under the **Applied Cyber Physical Systems** initiative at the **Center for System Design (CSD), National Institute of Technology Karnataka, Surathkal**.

CSD provides a multidisciplinary setting for integrating physical infrastructure, sensing, embedded systems, communication networks, computation, automation, and system-level design. The digital twin demonstrates this integration through synchronized transport, factory, safety, and IoT subsystems.

## Core Capabilities

- Real-time low-poly city rendered with Three.js.
- Train, bus, car, and truck agents following waypoint routes.
- Shared traffic manager for safe spacing, convoy slowdown, spawn clearance, swept-path checks, and junction occupancy.
- Railway crossing with safe stopping points and protected-zone clearance.
- Toll barrier animation and simulated toll deductions.
- RFID-controlled factory access with a single common gate.
- Animated factory shutter, automated guided vehicle, multi-crate goods transfer, and cargo telemetry.
- Configurable truck capacity with aggregate visualization for larger loads.
- Staggered dispatch from the toll-side parking area.
- Road-vehicle collision scenario with smoke, fire, alarm, GSM narrative, and reset flow.
- Railway surveillance camera with one-shot vehicle detection events.
- Manual Mode simulation Activity Log.
- Device Mode HiveMQ-only Activity Log and acknowledged gate controls.
- Responsive operations dashboard, landing screen, documentation view, keyboard focus styles, and reduced-motion/transparency fallbacks.

## System Architecture

```text
┌──────────────────────────────── Presentation ────────────────────────────────┐
│ Three.js scene · Operations dashboard · Activity Log · Documentation view   │
└─────────────────────────────────────┬────────────────────────────────────────┘
                                      │
┌──────────────────────────────── Runtime coordination ───────────────────────┐
│ SimulationManager                                                           │
│ dispatch · shared state · UI commands · infrastructure holds · log events    │
└──────────────┬──────────────────────┬──────────────────────┬─────────────────┘
               │                      │                      │
        RouteController         TrafficManager      AccidentScenario
        waypoint actions        spacing/safety      scripted collision
               │                      │
┌──────────────┴──────────────────────┴────────────────────────────────────────┐
│ Vehicle agents: train · bus · car · truck                                   │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────── Infrastructure controllers ──────────────────────┐
│ TollController · CrossingController · FactoryController · FactoryOperations │
└─────────────────────────────────────┬────────────────────────────────────────┘
                                      │
┌──────────────────────────────── Live integration ────────────────────────────┐
│ One MQTT.js/HiveMQ WebSocket connection                                     │
│ normalize once → animation consumer + Device Mode Activity Log consumer      │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Runtime data flow

1. `src/main.js` composes the renderer, scene, world, vehicles, controllers, simulation manager, and dashboard.
2. `SimulationManager` owns global dispatch and infrastructure safety decisions.
3. Each road agent delegates waypoint movement and actions to `RouteController`.
4. `TrafficManager` evaluates proximity, movement probes, spawn clearance, and conflict occupancy.
5. Toll, railway, and factory controllers animate their own pivots through damped per-frame updates.
6. Factory loading is coordinated by `FactoryOperations`, which synchronizes truck doors, factory shutter, AGV trips, crate transfer, and cargo telemetry.
7. Simulation and MQTT events feed the appropriate Activity Log without creating duplicate MQTT clients.

## Source Layout

```text
src/
├── main.js                         Application composition and frame loop
├── effects/
│   └── steam.js                    Train steam effect
├── MQTT/
│   ├── mqttclient.js               Browser WebSocket MQTT client
│   ├── connmanager.js              Singleton connection, reconnect, fan-out
│   ├── mqtthandler.js              Validation and topic-to-animation mapping
│   └── mqtttopic.js                Telemetry and command topics
├── simulation/
│   ├── routeData.js                Vehicle waypoint definitions
│   ├── routes.js                   RouteController state machine
│   ├── trafficManager.js           Collision avoidance and spacing
│   ├── simulationManager.js        Shared runtime coordinator
│   └── AccTrain.js                 Road-vehicle accident scenario
├── ui/
│   ├── dashboard.js                Operations/Documentation navigation
│   ├── panels/                     Control Center and Activity Log
│   └── styles/                     Responsive glass UI and accessibility
├── utils/
│   └── constants.js                Shared rendering, route, MQTT, cargo config
├── vehicles/
│   ├── train.js
│   ├── bus.js
│   ├── car.js
│   └── truck.js                    Cargo bounds, doors, capacity indicator
└── world/
    ├── roads.js
    ├── railway.js
    ├── tunnel.js
    ├── parkingLot.js
    ├── skyAtmosphere.js
    ├── tollController.js
    ├── crossing/
    └── factory/
        ├── factoryStatic.js         Premises, shutter, AGV geometry
        ├── factoryDynamic.js        RFID, common gate, portal
        ├── factoryController.js     RFID and gate state machine
        └── factoryOperations.js     State-driven goods loading
```

The physical-device firmware is maintained separately under `arduino/`. Each sketch keeps its original source folder and filename so its Arduino IDE build contract remains intact. Credentials must be supplied through ignored local headers rather than committed to the repository.

## Vehicle and Infrastructure Flow

### Bus

Toll-side dispatch → toll → passenger stops → railway crossing → factory-side stops → loop → return crossing → toll → dispatch.

### Car

Toll-side dispatch → toll → railway crossing → immediate turn away from factory → destination parking → return crossing → toll → dispatch.

### Truck

Toll-side dispatch → toll → railway crossing → RFID checkpoint → common factory gate → loading dock → AGV goods transfer → factory exit → return crossing → toll → dispatch.

### Railway crossing

Road vehicles stop before the barrier while a train approaches. Vehicles already inside the protected rail zone are allowed to clear instead of being frozen on the tracks.

### Factory logistics

1. RFID verification resolves to approved or rejected.
2. Approved access opens the common factory barrier.
3. The truck parks at a collision-safe loading anchor.
4. The factory shutter and truck cargo doors open.
5. The AGV transports crates from the factory to the truck.
6. Crates move into computed positions inside the declared cargo bounds, fade as they become enclosed cargo, and are unmounted.
7. The truck and dashboard aggregate indicators update from the same cargo state.
8. Cargo doors and shutter close before route movement resumes.

## Cargo Configuration

Cargo capacity is controlled through environment variables:

```env
VITE_TRUCK_CARGO_CAPACITY=36
VITE_DEFAULT_LOAD_UNITS=3
```

The default capacity is 36 units. Up to 20 transfers are rendered as distinguishable crate movements; higher quantities are represented through aggregate truck and dashboard telemetry to avoid overflowing the cargo volume or increasing draw calls unnecessarily.

## HiveMQ Device Mode

Device Mode uses one MQTT.js connection over browser-compatible WebSockets.

```text
HiveMQ digitaltwin/#
        │
        ▼
connmanager.js
  normalize once
        │
        ├──► mqtthandler.js ──► simulationManager.applyDeviceState()
        │
        └──► activityPanel.js ──► Device Mode Activity Log
```

Create a local `.env.local` file:

```env
VITE_HIVEMQ_HOST=YOUR_CLUSTER.s1.eu.hivemq.cloud
VITE_HIVEMQ_PORT=8884
VITE_HIVEMQ_PATH=/mqtt
VITE_HIVEMQ_USERNAME=YOUR_USERNAME
VITE_HIVEMQ_PASSWORD=YOUR_PASSWORD
VITE_HIVEMQ_TLS=true
VITE_MQTT_TOPIC_ROOT=digitaltwin
```

Never commit `.env.local`. Use `.env.example` as the safe configuration template.

### Animation topics

| Topic | Example value | Result |
| --- | --- | --- |
| `digitaltwin/toll/status` | `open`, `closed` | Toll barrier animation |
| `digitaltwin/railway/status` | `open`, `closed` | Railway barrier animation |
| `digitaltwin/factory/gate/status` | `open`, `closed`, `restricted` | Common factory gate |
| `digitaltwin/camera/detection` | vehicle detection | Camera telemetry/log |
| `digitaltwin/delivery/status` | `loading`, `docked` | Loading sequence state |
| `digitaltwin/delivery/truck/position` | `docked` | Truck delivery trigger |
| `digitaltwin/delivery/trolley/status` | `start`, `stop` | AGV delivery control |
| `digitaltwin/delivery/goods/count` | numeric count | Cargo target/count |
| `digitaltwin/delivery/rfid` | `verified` | Delivery authorization |

All messages under `digitaltwin/#` appear in the Device Mode Activity Log while connected. A disconnect clears that feed rather than substituting simulated data.

## Installation

Requirements:

- Node.js 20 or newer
- npm
- Browser with WebGL and WebSocket support

```bash
git clone https://github.com/Sonalhegde/SMARTTRANSPORTATIONTWIN.git
cd SMARTTRANSPORTATIONTWIN
npm install
npm run dev
```

Open:

```text
http://127.0.0.1:5173/
```

Production build:

```bash
npm run build
```

## GitHub Pages Hosting

The application is deployed as a Vite project site at:

```text
https://sonalhegde.github.io/SMARTTRANSPORTATIONTWIN-PAGES/
```

`vite.config.js` sets the production base path to `/SMARTTRANSPORTATIONTWIN-PAGES/`. The credential-free contents of `dist/` are published to the separate public [`SMARTTRANSPORTATIONTWIN-PAGES`](https://github.com/Sonalhegde/SMARTTRANSPORTATIONTWIN-PAGES) repository. Its Pages source is `main` at `/` (repository root).

Manual Mode requires no secrets. Device Mode is intentionally disconnected in the public bundle because Vite browser variables are embedded at build time and must never contain private broker credentials in a publicly downloadable JavaScript file.

Windows local-shim fallback:

```powershell
.\node_modules\.bin\vite.cmd build
```

## Verification Checklist

- Landing screen waits for the first rendered frame before enabling entry.
- Operations and Documentation views remain responsive on desktop and mobile.
- Vehicle dispatch is staggered and spawn points remain clear.
- Car, bus, and truck routes complete without skipped actions.
- Traffic spacing prevents overlap at junctions and railway approaches.
- Road vehicles stop before closed railway gates and never remain trapped on the rails.
- Toll, railway, and factory gate states animate and log correctly.
- RFID approved and restricted flows behave correctly.
- Truck cargo remains within its declared bounds and no crate trails behind a moving truck.
- Cargo count and capacity indicators remain synchronized.
- Accident simulation excludes the train and resets cleanly.
- Device Mode uses one MQTT connection and clears stale messages on disconnect.
- Keyboard focus, reduced-motion, and reduced-transparency behavior remain available.

## Repository

Project source and current `main` branch: https://github.com/Sonalhegde/SMARTTRANSPORTATIONTWIN/tree/main

Live deployment: https://sonalhegde.github.io/SMARTTRANSPORTATIONTWIN-PAGES/
