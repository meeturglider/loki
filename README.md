# Loki — V1 Architecture Specification

**Project:** Loki — Physical AI Companion at desk and travel
**Architecture Version:** V1.0  
**Status:** Architecture Locked  
**Platform:** iPhone companion app (iOS)  
**Primary Development Host:** Mac  
**Future Brain Hosts:** Linux / Windows / Raspberry Pi / Cloud VM / Cloud AI

---

## 1. Vision

Loki is a small, portable, expressive physical AI companion designed to feel like a living desk/room buddy rather than a conventional smart speaker.

V1 includes:

- Expressive animated display
- Microphone and speaker
- Wi-Fi and BLE
- AI voice interaction
- Personality and memory
- Rechargeable battery
- GNSS location capability
- Cellular connectivity with eSIM/eUICC support
- iPhone companion app
- Secure OTA firmware updates
- Local-first privacy architecture

Loki is intentionally designed so the physical device does not permanently depend on one computer, LLM provider, STT provider, TTS provider, or cloud platform.

---

## 2. Core Architecture

```text
                    ┌─────────────────────┐
                    │      iPhone App     │
                    │       SwiftUI       │
                    └──────────┬──────────┘
                               │
                         BLE / Internet
                               │
                               ▼
┌──────────────────┐     ┌───────────────┐     ┌──────────────────────┐
│   LOKI DEVICE    │────►│   LOKI API    │────►│      AI BRAIN        │
│    ESP32-S3       │     │ / Gateway     │     │ STT / LLM / TTS      │
│                   │     │               │     │ Personality / Memory │
│ Face / Audio      │     └───────────────┘     └──────────────────────┘
│ Wi-Fi / BLE       │
│ Battery           │
│ GNSS / LTE/eSIM   │
│ Local animation   │
└──────────────────┘
```

**Architectural rule:** The ESP32-S3 is the body/controller, not the primary LLM host. The AI brain is an interchangeable service. The iPhone app is the user control plane.

---

## 3. Hardware Architecture

### Core controller

**ESP32-S3 N16R8**

Target capabilities:

- 16 MB Flash
- 8 MB PSRAM
- Wi-Fi
- BLE
- SPI display
- I2S audio
- OTA firmware
- Secure boot / signed firmware support
- Local animation engine
- Device telemetry
- Battery monitoring interface

### Display

**ST7789 2.8" 240×320 SPI TFT**

Responsibilities:

- Expressive eyes and mouth
- Idle/listening/thinking/emotional animations
- Status/error indications
- Firmware/update/recovery UI when required

AI decisions are represented as semantic animation commands; the ESP32 renders them locally.

### Audio

**Microphone:** INMP441 I2S MEMS microphone  
**Amplifier:** MAX98357A I2S amplifier  
**Speaker:** 4Ω ~3W full-range speaker

Final enclosure requirements:

- Dedicated acoustic chamber
- Speaker grille/opening
- Appropriate damping material
- Physical separation between speaker and microphone
- Minimize acoustic feedback/echo
- Validate volume and distortion before final PCB

---

## 4. Power Architecture

Final Loki is battery powered.

```text
USB-C
  │
  ▼
Charging / Power Management
  │
  ▼
LiPo Battery
  │
  ├──► 3.3V electronics
  └──► Cellular/modem power rail
```

Requirements:

- Rechargeable LiPo battery
- USB-C charging
- Battery protection
- Battery fuel-gauge/measurement
- Stable power rails
- Battery telemetry
- Low-power/sleep modes
- Safe charging design

A 5V USB supply is used during early prototyping.

Battery capacity is deliberately not locked until real device power consumption is measured.

---

## 5. Connectivity

### BLE

Used for:

- First-time setup
- Device discovery
- Wi-Fi provisioning
- Nearby configuration
- Diagnostics
- Recovery/provisioning

The ESP32-S3 provides BLE; no separate Bluetooth module is required.

### Wi-Fi

Primary connection for:

- AI communication
- Local/cloud brain communication
- OTA firmware
- Configuration
- Telemetry

### Cellular

Target:

- 4G LTE
- Cat-1/Cat-1bis class modem
- eSIM/eUICC capability
- Independent connectivity when Wi-Fi is unavailable

The exact modem will be selected based on India network compatibility, LTE bands, eSIM/eUICC support, GNSS capability, power consumption, availability, antenna requirements, and ESP32 integration.

### GNSS

Used for:

- Device location
- Last-known location
- Lost/recovery mode
- Optional periodic location reporting

Use “GNSS” rather than “GPS” in technical documentation because the final module may support multiple satellite constellations.

---

## 6. Lost Loki Mode

Loki should avoid continuously consuming maximum power for tracking.

```text
Wi-Fi unavailable
       │
       ▼
Check tracking policy
       │
       ▼
Wake GNSS
       │
       ▼
Obtain location
       │
       ▼
Connect cellular
       │
       ▼
Send location
       │
       ▼
Return to low-power mode
```

The iPhone app can display:

- Last known location
- Timestamp
- Battery level
- Cellular status
- GNSS accuracy
- Tracking status

Location tracking must be explicitly controllable by the owner.

---

## 7. AI Brain

The Mac is the initial development AI host only.

The architecture must support, where practical:

- macOS
- Linux
- Windows
- Raspberry Pi-class systems
- Home servers
- Cloud VMs
- Cloud AI APIs

Loki communicates through a documented API rather than directly depending on a specific AI vendor.

### Brain pipeline

```text
Audio Input
    │
    ▼
Wake Word
    │
    ▼
STT
    │
    ▼
Conversation / Context
    │
    ▼
LLM
    │
    ├── Personality
    ├── Memory
    ├── Context
    └── Emotion / Action selection
    │
    ▼
TTS
    │
    ▼
Audio Output
```

---

## 8. LLM / STT / TTS Agnosticism

Each provider must be replaceable behind an interface.

Examples:

```text
LLM
 ├── Local Ollama
 ├── OpenAI-compatible API
 ├── Gemini-compatible API
 ├── Other local/cloud model
 └── Future providers

STT
 ├── Local model
 ├── Cloud STT
 └── Future provider

TTS
 ├── Local TTS
 ├── Cloud TTS
 └── Future provider
```

Loki exchanges normalized data instead of provider-specific response formats.

---

## 9. Semantic Response Protocol

The AI brain returns semantic instructions.

```json
{
  "text": "Haha, that's actually funny!",
  "emotion": "happy",
  "intensity": 0.82,
  "animation": "laugh",
  "voice": "playful"
}
```

The ESP32 maps these semantic instructions to face animation, audio playback, and device behavior.

This keeps the animation engine independent of the AI backend.

---

## 10. Animation Engine

The animation system is a local ESP32 subsystem.

Initial states:

- IDLE
- LISTENING
- THINKING
- HAPPY
- SAD
- ANGRY
- EXCITED
- SLEEPY
- SURPRISED
- SUSPICIOUS
- LAUGH
- ERROR

The engine supports transitions rather than only switching between static images.

Example:

```text
IDLE
  ↓
LISTENING
  ↓
THINKING
  ↓
SURPRISED
  ↓
HAPPY
  ↓
IDLE
```

Expression parameters may include:

- Eye position
- Eye size
- Eyelid position
- Pupil position
- Pupil size
- Blink timing
- Mouth shape
- Animation speed
- Transition duration
- Emotion intensity

---

## 11. Voice Pipeline

```text
"Hey Loki"
     │
     ▼
Wake-word detection
     │
     ▼
Capture speech
     │
     ▼
STT
     │
     ▼
LLM
     │
     ▼
TTS
     │
     ▼
Speaker + Face
```

Wake-word detection is modular. A lightweight local engine such as Porcupine can be evaluated, but no vendor is mandatory.

### Voice performance requirement

Measure:

**end of user speech → beginning of Loki response**

If network-dependent STT creates unacceptable latency, local STT should be evaluated.

---

## 12. iPhone App

The official companion application is **iOS/iPhone**.

Recommended stack:

- Swift
- SwiftUI
- CoreBluetooth
- Network.framework / standard Apple networking APIs
- Keychain for secrets/tokens
- SwiftData or another Apple-native persistence layer where appropriate

The iPhone app is Loki's primary user control plane.

### App sections

**Home**
- Online/offline
- Battery
- Current emotion
- Network
- Firmware version

**Setup**
- Pair Loki
- BLE discovery
- Wi-Fi provisioning
- Device authentication
- Initial configuration

**Talk**
- Voice interaction
- Optional text interaction
- Conversation status

**Personality**
- Personality configuration
- Voice
- Behaviour
- Talkativeness
- Playfulness

**Memory**
- View memories
- Delete memories
- Export data
- Memory/privacy controls

**Location**
- Last known location
- Map
- GNSS status
- Battery
- Cellular status
- Tracking mode

**Device**
- Volume
- Brightness
- Sleep settings
- Network
- Cellular/eSIM status
- Diagnostics

**Firmware**
- Current version
- Available update
- Release notes
- OTA update
- Update progress
- Recovery guidance

**Developer Mode**
- Device logs
- Firmware information
- ESP32 telemetry
- Wi-Fi RSSI
- Cellular signal
- GNSS satellites/accuracy
- Memory/CPU information
- Audio/display diagnostics
- Restart device
- Export diagnostics

---

## 13. OTA Firmware

OTA is a core requirement.

```text
iPhone / Loki Backend
        │
        ▼
Firmware Package
        │
        ▼
Secure Download
        │
        ▼
Inactive Firmware Partition
        │
        ▼
Signature / Integrity Verification
        │
        ▼
Reboot
        │
        ▼
Health Check
        │
   ┌────┴────┐
   │         │
 Success   Failure
   │         │
 Confirm   Rollback
```

Requirements:

- Versioned firmware
- Signed firmware
- Secure transport
- Integrity verification
- A/B or equivalent safe-update strategy
- Automatic rollback
- Recovery mode
- Known-good firmware restore

USB recovery remains available for development/emergency recovery.

---

## 14. Security

Requirements:

- TLS/HTTPS for application communication
- Secure device authentication
- Secure Wi-Fi provisioning
- Signed firmware
- Secure boot where practical
- OTA integrity verification
- Encrypted sensitive local storage
- Protected credentials
- iPhone Keychain for app secrets/tokens
- No plaintext Wi-Fi/cellular credentials
- Debug interfaces disabled or protected in production

WPA2/WPA3 protects the network connection, but Loki application traffic must still use encrypted application-layer transport.

---

## 15. Privacy

Loki follows a **local-first privacy model**.

Data categories:

- Device telemetry
- Conversation/audio/transcripts where enabled
- User-approved memories
- GNSS location
- Cloud data required by selected AI services

The iPhone app should provide granular controls for:

- Cloud AI processing
- Conversation history
- Memory storage
- Location tracking
- Telemetry
- Data export
- Data deletion

The owner should understand where data is stored and what leaves the device.

### Audio privacy

Because Loki contains a microphone:

- Prefer local wake-word processing where practical
- Do not continuously upload audio by default
- Make recording/active-listening status clear
- Make audio retention configurable
- Disable debug audio capture in normal production operation

---

## 16. Physical Design

Target form: **small rounded cube / desk companion**.

Final dimensions are not locked yet. They will be derived from:

- Display
- Speaker/acoustic volume
- Battery
- LTE/GNSS modem
- Antennas
- PCB
- USB-C placement
- Thermal requirements

Conceptual internal layout:

```text
FRONT
┌─────────────────────────┐
│      ST7789 DISPLAY     │
├─────────────────────────┤
│ ESP32-S3 / PCB          │
│ Audio electronics       │
│                         │
│ Speaker chamber         │
│                         │
│ Battery                 │
│                         │
│ LTE/GNSS + antennas     │
└──────────┬──────────────┘
           │
         USB-C
          BACK
```

RF antenna placement must be considered early because the enclosure and PCB can affect performance.

---

## 17. Development Strategy

### Phase 1 — Electronics + software prototype

Build:

- ESP32-S3
- ST7789
- INMP441
- MAX98357A
- Speaker
- Wi-Fi
- Basic animation engine
- Mac-based AI brain
- Basic device API

Use breadboard and USB power.

### Phase 2 — Voice AI

Add:

- Wake word
- STT
- LLM
- TTS
- Personality
- Semantic emotion protocol
- Conversation latency measurements

### Phase 3 — iPhone app

Build:

- BLE discovery
- Pairing
- Wi-Fi provisioning
- Device status
- Basic settings
- Diagnostics

### Phase 4 — Battery + connectivity

Engineer:

- LiPo
- Charging
- Battery telemetry
- LTE/eSIM
- GNSS
- Antennas

### Phase 5 — OTA + security

Implement:

- Secure OTA
- Signed firmware
- Rollback
- Recovery
- Device authentication
- Secure storage

### Phase 6 — Custom PCB

Only after measuring:

- Power consumption
- Audio performance
- Heat
- RF performance
- Component dimensions
- Connector placement

### Phase 7 — Enclosure

Design and 3D print the enclosure around the validated PCB and components.

---

## 18. Prototype Shopping List

Buy now:

- ESP32-S3 N16R8
- ST7789 2.8" 240×320 SPI TFT
- INMP441
- MAX98357A
- 4Ω ~3W speaker
- Solderless breadboard
- Dupont jumper wires
- USB-C data cable
- 5V 2A/3A USB power supply

Do not buy yet unless needed for engineering:

- LTE modem
- eSIM/eUICC hardware/service
- GNSS antenna/module
- LiPo battery
- Final charger/power-management hardware
- Custom PCB
- Camera
- Motors
- IMU
- ToF sensor

The final architecture reserves interfaces and physical space for future/engineering components without complicating the first prototype.

---

## 19. Repository Structure

```text
loki/
├── firmware/
│   └── esp32/
│       ├── src/
│       │   ├── display/
│       │   ├── animation/
│       │   ├── audio/
│       │   ├── connectivity/
│       │   ├── battery/
│       │   ├── gnss/
│       │   ├── cellular/
│       │   ├── ota/
│       │   ├── security/
│       │   └── main/
│       ├── include/
│       └── README.md
│
├── brain/
│   ├── api/
│   ├── stt/
│   ├── llm/
│   ├── tts/
│   ├── memory/
│   ├── personality/
│   └── README.md
│
├── ios/
│   └── Loki/
│       ├── App/
│       ├── Bluetooth/
│       ├── Networking/
│       ├── Device/
│       ├── Firmware/
│       ├── Location/
│       ├── Memory/
│       ├── Settings/
│       ├── Diagnostics/
│       └── README.md
│
├── protocol/
│   ├── device-api.md
│   ├── semantic-response.md
│   ├── ble-protocol.md
│   ├── ota.md
│   └── telemetry.md
│
├── hardware/
│   ├── prototype/
│   ├── pcb/
│   ├── enclosure/
│   └── README.md
│
├── simulation/
│
├── docs/
│   ├── architecture.md
│   ├── hardware.md
│   ├── software.md
│   ├── security.md
│   └── privacy.md
│
└── README.md
```

---

## 20. Design Principles

1. Local-first
2. AI-provider agnostic
3. Hardware modularity
4. Secure by design
5. Privacy by design
6. OTA upgradeability
7. Recoverability
8. Low latency
9. Expressive physical interaction
10. No unnecessary cloud dependency
11. Reuse prototype components where practical
12. Separate device, AI brain, and mobile app responsibilities

---

## 21. Current Status

**Architecture:** LOCKED — V1.0  
**Prototype hardware:** Ready to begin  
**AI brain:** Mac-first implementation  
**Mobile:** iPhone / iOS companion app  
**Final PCB:** Not yet designed  
**Final enclosure:** Not yet dimension-locked  
**Battery:** Architecture defined; capacity TBD  
**LTE/eSIM:** Architecture defined; modem TBD  
**GNSS:** Architecture defined; hardware TBD  
**OTA:** Required; implementation pending  
**Security/privacy:** Architecture defined; implementation pending

---

## 22. Future Possibilities

Potential future versions may add:

- Camera / computer vision
- Advanced local AI
- Better microphone array
- Environmental sensors
- Advanced emotion modelling
- Multiple Loki devices
- Home automation
- Optional movement/robotics
- Richer location/recovery features
- Cloud synchronization
- Family/user profiles

These remain outside locked V1 unless promoted through a future architecture revision.

---

# Final V1 Definition

**Loki V1 is a portable, battery-powered, expressive physical AI companion built around an ESP32-S3, with an iPhone companion app, modular AI brain, local animation engine, Wi-Fi/BLE, GNSS, LTE/eSIM capability, secure OTA firmware, privacy controls, and a local-first architecture.**

The first prototype remains deliberately simple.

**The architecture does not.**
