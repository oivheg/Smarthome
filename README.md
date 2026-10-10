# SmartHome Automation System

A comprehensive Node-RED based smart home automation system with advanced temperature control, presence detection, visitor mode management, and intelligent lighting automation.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Integration](#hardware-integration)
- [Key Automation Flows](#key-automation-flows)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Dashboard](#dashboard)
- [Contributing](#contributing)

## 🏠 Overview

This Node-RED project provides a complete smart home automation solution designed for a multi-floor residence. The system intelligently manages lighting, climate control, presence detection, and various modes (home/away/visitor) while optimizing for comfort and energy efficiency.

**Key Highlights:**
- **Multi-zone temperature control** with adaptive morning heating and zone balancing
- **Advanced presence detection** using Bluetooth MAC tracking and mmWave sensors
- **Visitor mode management** for day visits, overnight stays, and vacation house sitters
- **Intelligent bathroom automation** with sensor fusion and door-closed protection
- **Seasonal adaptation** with automatic summer/winter mode switching
- **Energy monitoring** via Tibber API integration
- **Voice control** through Google Home integration

## ✨ Features

### 🌡️ Temperature Control
- **Dual-zone climate management** (upstairs/downstairs)
- **Adaptive morning heating** based on outdoor temperature
- **Zone balancing** to maintain comfortable temperature differences
- **Learning algorithms** that adjust based on performance history
- **Seasonal modes** with automatic detection (May-September = Summer)
- **Smart swing control**: 
  - Upstairs heating → `fixedTop` (warm air rises)
  - Downstairs heating → `fixedBottom` (direct heat)
  - Cooling → `fixedTop` (cool air descends)
- **Safety validators** preventing extreme temperatures

### 👥 Presence & Mode Management
- **Home/Away detection** via Bluetooth MAC address tracking
- **Phone presence monitoring** for multiple occupants
- **Visitor Mode** with three types:
  - `day_visit`: Temporary visitors while owners are at work
  - `overnight`: Guests staying for the night
  - `house_sitter`: Vacation mode with extended access
- **Area-based restrictions** to limit automation in specific rooms
- **Automatic away mode blocking** when visitors are present

### 💡 Intelligent Lighting
- **Motion-triggered lighting** with timeout management
- **Time-based dimming** (day vs. night brightness)
- **Area-based control** respecting visitor mode restrictions
- **Light scheduler** integration for programmable scenes
- **FP2 mmWave presence** detection for continuous occupancy tracking

### 🚿 Bathroom Automation
- **Sensor fusion** combining FP2 mmWave, IR motion, and door sensors
- **Activity tracking** with timestamp logging
- **Door-closed protection** with three-layer safety:
  1. Recent activity within 5 minutes → Block lights OFF
  2. FP2 detecting presence → Block lights OFF
  3. IR motion detected → Block lights OFF
- **Visual status feedback** on all sensor nodes
- **Dashboard widgets** showing real-time sensor states

### 🗑️ Utilities
- **Garbage collection reminders** via Tømmekalender integration
- **Robot vacuum scheduling** (Roborock integration)
- **Energy monitoring** with Tibber API
- **Google Home TTS** for voice announcements
- **Pushbullet notifications** for important events

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Node-RED Core Engine                      │
├─────────────────────────────────────────────────────────────┤
│  Flows (20+ tabs)                                           │
│  ├─ Temperature Control      ├─ Bathroom Automation         │
│  ├─ Presence Detection       ├─ Visitor Mode Scheduler      │
│  ├─ Lighting (Lys)           ├─ Bedroom Control             │
│  ├─ Basement                 ├─ Stairs                      │
│  ├─ Google Home Integration  ├─ Dashboard GUI               │
│  ├─ Smarthome_MODE          ├─ Robot Vacuum                │
│  ├─ Strøm og Tibber         ├─ Garbage Collector           │
│  ├─ Remote Content          ├─ FH MQTT                     │
│  └─ Node-Red Control        └─ Deconz Connections          │
└─────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Integration Layer                         │
├─────────────────────────────────────────────────────────────┤
│  Protocols & APIs                                           │
│  ├─ Home Assistant WebSocket  ├─ deCONZ (ZigBee)           │
│  ├─ MQTT (FutureHomeHub)      ├─ Sensibo (Heat Pump)       │
│  ├─ Castv2 (Google Home)      ├─ Tibber API                │
│  ├─ Pushbullet                ├─ Miio (Roborock)           │
│  └─ ARP Scanner               └─ Traffic Monitor            │
└─────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Physical Devices                          │
├─────────────────────────────────────────────────────────────┤
│  Sensors                      Actuators                     │
│  ├─ Aqara FP2 mmWave         ├─ Sensibo Heat Pump          │
│  ├─ Aqara IR Motion          ├─ ZigBee Lights              │
│  ├─ Door Sensors (ZHA)       ├─ Google Home Speakers       │
│  ├─ Temperature Sensors      ├─ Robot Vacuum               │
│  ├─ Bluetooth Tracking       └─ Smart Switches             │
│  └─ Energy Monitor (Tibber)                                │
└─────────────────────────────────────────────────────────────┘
```

## 🔌 Hardware Integration

### Required Components
- **Node-RED Server** (Raspberry Pi, NUC, or similar)
- **ZigBee Coordinator** (deCONZ compatible)
- **Sensibo Heat Pump Controller**
- **Aqara FP2 mmWave Presence Sensor**
- **Aqara Motion Sensors** (ZigBee)
- **Door/Window Sensors** (ZigBee)
- **Google Home Devices** (for TTS announcements)
- **Bluetooth-capable server** (for phone tracking)

### Supported Integrations
| Integration | Purpose | Configuration |
|-------------|---------|---------------|
| **Home Assistant** | Central hub & sensor data | WebSocket API |
| **deCONZ** | ZigBee device management | REST API + WebSocket |
| **Sensibo** | Heat pump control | Cloud API |
| **MQTT** | FutureHomeHub integration | Local broker |
| **Tibber** | Energy monitoring & pricing | REST API |
| **Castv2** | Google Home casting | Local network |
| **Miio** | Robot vacuum control | Local protocol |

## 🔄 Key Automation Flows

### Temperature Control Flow
**Location:** `Temperaturecontroll` tab

**Logic:**
1. **Season Detection**: Auto-detects Summer (May-Sep) vs Winter
2. **Priority Zone Calculation**: 
   - Day (9am-9pm) + Home → Upstairs priority
   - Night/Away → Downstairs priority
3. **Adaptive Morning Heating**:
   ```
   Morning Target = 20.5°C + 0.2°C × max(0, 5°C - OutsideTemp)
   Capped at 22°C
   ```
4. **Zone Balancing**: Maintains max 2.5°C difference between floors
5. **Safety Validation**: Prevents extreme temperatures and cross-floor imbalances

**Key Variables:**
- `desiredTemp`: User-set target temperature
- `upstairsCurrentTemp`: Upper floor temperature
- `downstairsCurrentTemp`: Lower floor temperature
- `OutsideTemp`: External temperature
- `seasonName`: 'Summer' or 'Winter'
- `controlReason`: Current automation decision context

### TemperatureControllV2 (current)
**Location:** `TemperatureControllV2` tab

`Context & State Ingestion` → `Core Rules Engine` (3 outputs):
1. **Sensibo command** (only when changed) → `Sensibo queue (1 / 15s)` → Sensibo. Commands are spaced ≥15 s so IR commands are never sent back-to-back (that made the unit lose swing and stay on `fixedTop`).
2. **State** (every cycle) → UI + `Mill Controller`.
3. **Re-assert**: the latest ON command is re-sent once 60 s later if it is still current, keeping swing/mode in sync with what Sensibo reports.

**Mill Controller** (downstairs Mill heater) runs every cycle and is closed-loop on the downstairs sensor:
- Idles (`millIdleTemp`, default 16 °C) when downstairs ≥ target + 0.3 °C, when downstairs is rising > 0.5 °C/h near target, or while the heat pump cools.
- Holds the target, or boosts +1/+2 °C when downstairs is 0.25/0.75 °C below it; no boost while the heat pump heats downstairs unless ≥1 °C below.
- `millWinterTemp` / `millSummerTemp` (dashboard "Mill max °C") are the **maximum** setpoint.
- Zone target: day = `desiredTemp`, 1 h before pre-bedtime = min(desired, bedtime + 1) (coast-down), night/pre-bedtime = bedtime − 1, away/vacation use their own targets.
- Re-sent every 30 min in case it was changed from the Mill app.

**Pre-bedtime (20:30–22:00)** cools the downstairs bedrooms towards `bedtimeTemp`: starts at +0.5 °C, stops at −0.3 °C, setpoint/fan scale with how warm it is (fan max medium while TV is on). Upstairs-only heat is cooled gently with `fixedTop` without overcooling bedrooms.

### Visitor Mode System
**Location:** `Visitor Mode Scheduler` tab

**Features:**
- **Type Selection**: `day_visit`, `overnight`, `house_sitter`
- **Scheduled Activation**: Vacation start/end times
- **Area Restrictions**: Limit automation in specific rooms
- **Away Mode Override**: Prevents system from entering away state
- **FP2 Fallback Detection**: Uses mmWave sensors when phones absent

**Data Structure:**
```javascript
VisitorMode = {
  enabled: boolean,
  type: 'day_visit' | 'overnight' | 'house_sitter',
  startTime: timestamp,
  expectedEndTime: timestamp,
  allowedAreas: ['bathroom', 'kitchen', 'living_room'],
  restrictedAreas: ['bedroom', 'office'],
  autoLightsEnabled: boolean,
  climateAdjusted: boolean
}
```

### Bathroom Automation
**Location:** `Bathroom` tab

**Sensor Inputs:**
- **FP2 mmWave**: Continuous presence detection
- **IR Motion Sensor**: Movement detection
- **Door Sensor**: Entry/exit tracking

**Protection Logic:**
```javascript
// Rule 1: Door closed + recent activity (<5min)
if (!doorOpen && timeSinceActivity < 300000) → BLOCK OFF

// Rule 2: Door closed + FP2 active
if (!doorOpen && fp2Presence) → BLOCK OFF

// Rule 3: Movement detected
if (movement) → BLOCK OFF

// Otherwise: Safe to turn off
```

**Dashboard Widgets:**
- FP2 Status (🟢 Active / ⚫ Inactive)
- IR Motion Status (🔵 Detected / ⚫ Clear)
- Last Activity Time (X minutes ago)
- Protection Status (🟡 Protected / 🟢 Safe)

### Presence Detection
**Location:** `Presence Detection` tab

**Methods:**
1. **Bluetooth MAC Tracking**: Primary method for owner phones
2. **ARP Scanner**: Network-based device detection
3. **FP2 Fallback**: mmWave sensors when phones not detected
4. **5-Minute Activity Window**: Grace period for temporary absence

**State Machine:**
```
AWAY ──(phone detected)──> HOME
HOME ──(no phones + 5min)──> AWAY
AWAY ──(visitor mode + FP2)──> VISITOR_PRESENT
```

## 📦 Installation

### Prerequisites
```bash
# Node.js 18+ required
node --version

# Node-RED installation
npm install -g --unsafe-perm node-red
```

### Clone Repository
```bash
git clone https://github.com/oivheg/Smarthome.git
cd Smarthome
```

### Install Dependencies
```bash
npm install
```

### Deploy to Node-RED
```bash
# Copy flows to Node-RED directory
cp flows_smarthome.json ~/.node-red/
cp flows_smarthome_cred.json ~/.node-red/

# Restart Node-RED
node-red-restart
```

## ⚙️ Configuration

### 1. Temperature Control Settings
Edit the `Initialize Configuration` node in the `Temperaturecontroll` tab:

```javascript
flow.set('minTemp', 20);               // Minimum heat pump temp
flow.set('maxTemp', 30);               // Maximum heat pump temp
flow.set('comfortTolerance', 0.5);     // Comfort range (±°C)
flow.set('upstairsComfortMin', 19.5);  // Winter upstairs minimum
flow.set('maxZoneDifference', 2.5);    // Max temp diff between zones
```

### 2. Presence Detection
Update MAC addresses in the `Presence Detection` tab:

```javascript
const ownerPhones = [
  'XX:XX:XX:XX:XX:XX',  // Phone 1
  'YY:YY:YY:YY:YY:YY'   // Phone 2
];
```

### 3. Integration Credentials
Configure in `flows_smarthome_cred.json`:

```json
{
  "sensibo_api_key": "YOUR_SENSIBO_KEY",
  "tibber_token": "YOUR_TIBBER_TOKEN",
  "pushbullet_key": "YOUR_PUSHBULLET_KEY",
  "mqtt_username": "YOUR_MQTT_USER",
  "mqtt_password": "YOUR_MQTT_PASS"
}
```

### 4. Dashboard Access
Access the dashboard at: `http://<node-red-ip>:1880/dashboard`

## 🎯 Usage

### Setting Desired Temperature
1. Navigate to Dashboard GUI tab
2. Adjust temperature slider
3. System automatically calculates optimal heat pump settings

### Activating Visitor Mode
**Manual:**
1. Open Dashboard
2. Select visitor type from dropdown
3. Toggle "Visitor Mode" switch

**Scheduled (Vacation):**
1. Open `Visitor Mode Scheduler` tab
2. Configure start/end inject nodes
3. Set vacation dates and times

### Monitoring Bathroom Sensors
View real-time status in Dashboard → Bathroom group:
- **Door Status**: Red (open) / Green (closed)
- **FP2 Presence**: Active/Inactive with last activity time
- **Motion Detection**: Detected/Clear
- **Protection Status**: Shows active protection rule

### Checking Temperature Control
Dashboard displays:
- Current upstairs/downstairs temperatures
- Active control reason
- Heat pump mode and target temperature
- Zone temperature difference
- Season and priority zone

## 📊 Dashboard

The `@flowfuse/node-red-dashboard` provides a responsive web interface:

### Main Dashboard Page
- **Temperature Control**: Current temps, desired temp slider, mode display
- **Presence Status**: Home/Away indicator, detected devices
- **Visitor Mode**: Type selector, status display
- **Energy Monitoring**: Current price, consumption, daily cost
- **Bathroom**: Sensor states and protection status

### Dashboard Groups
| Group | Widgets | Purpose |
|-------|---------|---------|
| **Temperature** | Text displays, sliders, charts | Climate control |
| **Presence** | Status indicators, switches | Occupancy tracking |
| **Visitor Mode** | Dropdown, text, notifications | Guest management |
| **Bathroom** | Status texts, switch | Bathroom automation |
| **Energy** | Gauges, charts | Power monitoring |

## 🔧 Development

### Branch Structure
- `main`: Stable production branch
- `feature/VisistorMode`: Visitor mode enhancements
- `feature/bathroom-dashboard`: Bathroom dashboard widgets
- `TemperatureControll`: Temperature logic improvements
- `V2`: Major version upgrades

### Making Changes
```bash
# Create feature branch
git checkout -b feature/your-feature-name

# Make changes to flows_smarthome.json

# Test in Node-RED

# Commit with descriptive message
git add flows_smarthome.json
git commit -m "feat: description of your changes"

# Push to remote
git push -u origin feature/your-feature-name
```

### Testing
1. Deploy changes in Node-RED editor
2. Monitor debug panel for errors
3. Test automation triggers manually
4. Verify dashboard widgets update correctly
5. Check flow variables in context sidebar

## 📝 Node-RED Modules

### Core Dependencies
```json
{
  "@flowfuse/node-red-dashboard": "1.29.0",
  "node-red-contrib-deconz": "2.3.16",
  "node-red-contrib-home-assistant-websocket": "0.78.1",
  "node-red-contrib-sensibo": "0.6.1",
  "node-red-contrib-tibber-api": "6.4.1",
  "node-red-contrib-castv2": "4.3.0",
  "node-red-contrib-miio-roborock": "2.5.0",
  "node-red-contrib-light-scheduler": "0.0.19",
  "node-red-contrib-bigtimer": "2.8.6",
  "node-red-contrib-aedes": "0.15.0",
  "node-red-node-pushbullet": "0.0.19"
}
```

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork** the repository
2. **Create** a feature branch
3. **Commit** changes with clear messages
4. **Test** thoroughly in Node-RED
5. **Submit** a pull request with description

### Commit Message Format
```
feat: Add new feature
fix: Fix bug in temperature control
docs: Update README
refactor: Improve bathroom logic
style: Format code
```

## 📄 License

This project is shared as-is for educational and personal use. Please respect third-party integration terms of service.

## 🙏 Acknowledgments

- **Node-RED Community**: For the excellent platform
- **deCONZ**: ZigBee integration
- **Sensibo**: Heat pump API
- **Tibber**: Energy monitoring
- **Home Assistant**: Comprehensive smart home platform
- **FlowFuse**: Modern dashboard framework

---

**Project maintained by:** [oivheg](https://github.com/oivheg)

**Current Version:** 0.0.1

**Last Updated:** December 2025 
