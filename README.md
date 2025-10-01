# HUB Automation Web & ESP32 Controller Documentation

## Overview

This documentation covers:
- The main web application repository: **TPULIGILLA-YOTZERON/HUB-AUTOMATION-WEB**
- Two ESP32 Arduino sketches:
  - `ESP32_HTTP_Relay_Controller.ino` &mdash; For relay control using HTTP polling
  - `ESP32_HTTP_RFID_Reader.ino` &mdash; For RFID-based access control using HTTP

---

## 1. ESP32_HTTP_Relay_Controller.ino

### Purpose

Controls an 8-channel relay module using an ESP32 board. 
- Periodically polls a server for relay commands.
- Updates relays according to server instructions.
- Optionally reads manual switches for local control and can report changes.
- Sends periodic heartbeats to the server to indicate it is online.

### Hardware Connections

**Relays:**  
| Relay (Device) | ESP32 GPIO |
|----------------|------------|
| Fan 1          | 13         |
| Light 1        | 12         |
| Fan 2          | 14         |
| Light 2        | 27         |
| Fan 3          | 26         |
| Light 3        | 25         |
| Fan 4          | 33         |
| Light 4        | 32         |

**Manual Switches (Optional):**  
Connect each switch between its GPIO and GND.

| Switch (Function) | ESP32 GPIO |
|-------------------|------------|
| Switch 1 (Fan 1)  | 34         |
| Switch 2 (Light 1)| 35         |
| Switch 3 (Fan 2)  | 36         |
| Switch 4 (Light 2)| 39         |
| Switch 5 (Fan 3)  | 4          |
| Switch 6 (Light 3)| 2          |
| Switch 7 (Fan 4)  | 15         |
| Switch 8 (Light 4)| 0          |

### Features

- Connects to a WiFi network
- Polls a web server (Replit-based) for relay state commands every 5 seconds
- Updates relay states (active LOW logic: LOW = ON, HIGH = OFF)
- Reads manual switches and updates relays accordingly (with TODO for reporting to server)
- Sends heartbeat messages to the server every 30 seconds with current relay states

### Configuration

In the source, update these lines:

```cpp
const char* WIFI_SSID = "YOUR_WIFI_NAME";           // WiFi SSID
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";   // WiFi password
const char* SERVER_URL = "https://your-replit-app.repl.co";  // Server endpoint
const char* API_KEY = "AfTCwDOhwWqa7XegseVqS8Idaq5xyoIM";    // Provided API key
const char* DEVICE_ID = "esp32_relay"; // Device identifier
```

### Dependencies

- ArduinoJson
- WiFi.h
- HTTPClient.h

### Main Functions

- **setup()**: Initializes serial, relays, switches, and connects to WiFi.
- **loop()**: Handles polling, heartbeat, WiFi reconnection, and switch detection.
- **pollServerForCommands()**: GETs `/api/esp32/commands?deviceId=...` and updates relays.
- **setRelay(index, state)**: Sets relay state with correct logic.
- **checkManualSwitches()**: Reads switches, updates relays, (TODO: report to server).
- **sendHeartbeat()**: POSTs to `/api/esp32/commands` with current relay status.

---

## 2. ESP32_HTTP_RFID_Reader.ino

### Purpose

Reads RFID cards using an RC522 module and an ESP32 board.
- When a card is scanned, sends the UID to the server over HTTP.
- Server determines access rights and can trigger relays.

### Hardware Connections (RC522 to ESP32)

| RC522 Pin | ESP32 GPIO |
|-----------|------------|
| SDA (SS)  | 5          |
| SCK       | 18         |
| MOSI      | 23         |
| MISO      | 19         |
| GND       | GND        |
| RST       | 22         |
| 3.3V      | 3.3V       |

**LED Feedback:**  
- GPIO 2 (onboard LED on most ESP32 boards) flashes for status.

### Features

- Connects to WiFi
- Waits for RFID cards
- Sends UID to server via HTTP POST (`/api/rfid-scan`)
- Displays access granted/denied based on server reply (via LED and serial)
- Implements cooldown to avoid duplicate readings from the same card

### Configuration

In the source, update these lines:

```cpp
const char* WIFI_SSID = "YOUR_WIFI_NAME";           // WiFi SSID
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";   // WiFi password
const char* SERVER_URL = "https://your-replit-app.repl.co";  // Server endpoint
const char* API_KEY = "AfTCwDOhwWqa7XegseVqS8Idaq5xyoIM";    // Provided API key
```

### Dependencies

- MFRC522
- ArduinoJson
- WiFi.h
- HTTPClient.h
- SPI.h

### Main Functions

- **setup()**: Initializes serial, SPI, RC522, LED, and connects to WiFi.
- **loop()**: Waits for new RFID card, debounces, and sends UID to server.
- **sendRFIDToServer(uid)**: POSTs UID and deviceId to `/api/rfid-scan`. Handles response.
- **blinkLED(times, delayMs)**: Flashes LED for user feedback.

---

## 3. HUB-AUTOMATION-WEB Repository

### General Description

This is the main web backend (and possibly frontend) for the home automation system interacting with the ESP32 devices.  
It provides HTTP API endpoints for:
- Relay command polling and reporting (for the relay controller)
- Receiving RFID scans and making access decisions (for the RFID reader)
- Storing and managing device states and user access

### API Endpoints (as referenced by ESP32 code)

#### `/api/esp32/commands?deviceId=esp32_relay` (GET)
- Returns a JSON array of relay device commands.

#### `/api/esp32/commands` (POST)
- Receives device heartbeat and relay states.

#### `/api/rfid-scan` (POST)
- Receives RFID UID, responds with access grant/deny.

### Technologies

- Likely Node.js, Express, or Replit server
- Uses an API key for authentication (`x-api-key` header)
- Stores user/device states (details in repo source)

### How It Works

- ESP32 devices communicate via HTTP (not MQTT)
- Centralized server controls relay state and access
- API key protects endpoints
- Easy to integrate with other services or UIs

---

## General Setup Guide

1. **Configure and flash** the ESP32 sketches with your WiFi and server details.
2. **Wire hardware** according to the pin tables above.
3. **Run the web server** (see HUB-AUTOMATION-WEB repo for instructions).
4. **Test**:
   - For relays: Observe relays toggling in response to web dashboard/API changes.
   - For RFID: Scan cards and check server response/LED feedback.

---

## Security Reminders

- Change all default credentials and API keys before deploying.
- Use HTTPS for server connections to secure data.
- Restrict API access using API keys and/or IP whitelisting.

---

## Resources

- [ArduinoJson library](https://arduinojson.org/)
- [MFRC522 library](https://github.com/miguelbalboa/rfid)
- [ESP32 Documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/)

---

## Troubleshooting

- **WiFi not connecting**: Double-check SSID/password, ensure 2.4GHz network.
- **Relays not responding**: Check wiring, power supply, relay logic levels.
- **RFID not reading**: Check wiring, use 3.3V for RC522, ensure connections are firm.
- **Server errors**: Check API key, server logs, endpoint URLs.

---

## Contributors

- [TPULIGILLA-YOTZERON](https://github.com/TPULIGILLA-YOTZERON)
- [Your Name Here]

---
