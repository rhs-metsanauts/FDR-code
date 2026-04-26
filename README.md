# METSAnauts FDR — Rover Control System

Ground control and onboard software for the HERA mission rover. Runs across three nodes: a **laptop** (mission control UI), a **Raspberry Pi** (rover compute), and an **NVIDIA Jetson** (3D mapping).

## Architecture

```
┌─────────────────────┐        Wi-Fi / LoRa        ┌──────────────────────┐
│   Laptop            │ ─────────────────────────▶  │  Raspberry Pi        │
│  FlaskServer.py     │                             │  main.py (FastAPI)   │
│  Mission Control UI │  ◀─────────── video ──────  │  robot/Robot.py      │
│  AI assistant       │                             │  comms/              │
└─────────────────────┘                             └──────────────────────┘
                                                             │
                                                    USB / Serial
                                                             │
                                                    ┌────────▼─────────────┐
                                                    │  NVIDIA Jetson       │
                                                    │  mapping/            │
                                                    │  ZED 2i camera       │
                                                    └──────────────────────┘
```

## Repository Structure

```
├── FlaskServer.py          # Mission control web UI (run on laptop)
├── main.py                 # FastAPI command server (run on Pi or Jetson)
├── run_server.bat          # Windows launcher for FlaskServer
│
├── robot/                  # Rover hardware abstraction
│   ├── Robot.py            # Rover, Drivebase, RockerBogie, Camera classes
│   └── command_executor.py # Remote command dispatcher (bash, file I/O, Python exec)
│
├── comms/                  # Communication drivers
│   ├── receiver_lora.py    # LoRa RFM9x receiver (Pi)
│   ├── transmitter_lora.py # LoRa RFM9x transmitter (laptop)
│   └── serial_reader.py    # Serial-to-command bridge
│
├── mapping/                # 3D spatial mapping (Jetson)
│   ├── jetson_mapper.py    # ZED voxel map builder + WebSocket server
│   └── zed_stream.py       # ZED camera RTSP stream
│
├── server/
│   └── pi_server.py        # Lightweight HTTP server (no FastAPI, pure stdlib)
│
├── templates/
│   └── index.html          # Mission control frontend (served by FlaskServer)
│
├── tests/                  # Test suite
│   ├── test_map_endpoints.py
│   ├── test_voxel.py
│   ├── test_depth.py
│   ├── test_mapping.py
│   ├── test_ws.py
│   └── legacy/             # Early hardware validation scripts
│
└── docs/                   # Design specs and architecture docs
```

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

> **Note:** `pyzed` is installed via the [ZED SDK installer](https://www.stereolabs.com/developers/release), not pip.

### 2. Run Mission Control (laptop)

```bash
python FlaskServer.py
# Open http://localhost:5000
```

### 3. Run Command Server (Raspberry Pi)

```bash
# FastAPI (recommended)
uvicorn main:app --host 0.0.0.0 --port 8000

# Or lightweight stdlib server (no dependencies)
python server/pi_server.py
```

### 4. Run 3D Mapper (Jetson)

```bash
python mapping/jetson_mapper.py
```

## Hardware

| Component | Purpose |
|-----------|---------|
| Raspberry Pi 4 | Onboard compute, motor control, camera |
| Pi Servo HAT | Rocker-bogie suspension servos |
| Adafruit RFM9x LoRa | Long-range radio backup comms |
| NVIDIA Jetson | ZED 2i camera, 3D spatial mapping |
| ZED 2i Stereo Camera | Depth sensing, voxel map generation |

## Communication Modes

| Mode | When Used |
|------|-----------|
| Wi-Fi (WebSocket) | Primary — low latency, high bandwidth |
| LoRa radio | Fallback — long range, low bandwidth |
| USB Serial | Direct tether / debugging |

## Team

**RHS METSAnauts** — Competing in NASA HERA (Human Exploration Research Analog)
