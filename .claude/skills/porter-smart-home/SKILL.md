---
name: porter-smart-home
description: How Porter talks to smart-home devices - Shelly (local Gen1 HTTP and Gen2+ RPC, Shelly Cloud), Home Assistant (REST/WebSocket), the device and connector model, gate/garage wiring and status sensors, credentials and security. Use whenever writing or changing device connectors, device discovery, the device model, or anything that switches something in the house.
---

# Porter – smart-home connectors

Porter is "your house knows you're coming": arrival and leaving scenes for gates, garage doors,
lights and heating, controlled from Android Auto and the phone. It is **not** a replacement for the
Shelly app or Home Assistant: those keep device setup, firmware, schedules and charts.
Porter only needs: list devices, read state, switch, pulse.

## 1. Connector model
```
Connector (one per account/system)      ShellyLocal, ShellyCloud, HomeAssistant, (later: Tuya, Sonoff eWeLink, MQTT)
 └─ Device                               id, name, connector, kind, channel
     ├─ kind: SWITCH | PULSE (gate/garage button) | COVER (roller/garage with position) | LIGHT | THERMOSTAT | SENSOR
     ├─ state: ON/OFF/OPEN/CLOSED/OPENING/UNKNOWN, lastSeen
     └─ actions: turnOn, turnOff, toggle, pulse(ms), setTemp(°C), open/close/stop
```
- Every action returns a result (`Ok`, `Offline`, `AuthFailed`, `Timeout`, `Rejected`) and is written
  to the **action log** (time, device, action, source: car/phone/auto, result).
- Route choice per action: **local first when on the home Wi-Fi**, cloud otherwise. On the road
  (the main use case) it is always the cloud path, so the cloud path must be first-class.
- Timeouts: 4 s per call, one retry for idempotent actions (`on`/`off`). **Never retry a pulse**:
  a second pulse can stop or reverse a gate.

## 2. Shelly – local API
Gen1 (Shelly 1, 1PM, 2.5 …), HTTP GET:
- `http://<ip>/relay/<ch>?turn=on|off|toggle` · pulse: `?turn=on&timer=1` (seconds)
- `http://<ip>/status` · `http://<ip>/shelly` (type, auth needed?)
- Auth: HTTP Basic when enabled.

Gen2+ (Plus, Pro, Gen3, Gen4), JSON-RPC over HTTP or WebSocket:
- `http://<ip>/rpc/Switch.Set?id=0&on=true` · pulse: `&toggle_after=1`
- `http://<ip>/rpc/Switch.GetStatus?id=0` · `Input.GetStatus?id=0` (reed switch on the gate)
- `Cover.Open / Close / Stop / GetStatus` for roller and garage profiles
- `Shelly.GetDeviceInfo` (model, gen, auth_en) · discovery by mDNS `_shelly._tcp`
- Auth: HTTP **Digest (SHA-256)** when enabled.

⚠ Verify against the current Shelly API docs (shelly-api-docs.shelly.cloud) before coding; Gen1 and
Gen2+ differ in every detail.

## 3. Shelly – cloud (needed on the road)
- "Cloud Control API": the user copies the **server URI and authorization cloud key** from the
  Shelly app (User settings → Authorization cloud key). Calls like
  `POST <server>/device/relay/control` with `id`, `channel`, `turn`, `auth_key`.
- Rate limit about **1 request per second** per account: queue calls, never fire a scene in parallel.
- ⚠ Open questions to settle before building:
  - Is the cloud key method allowed for a **commercial** third-party app, or is the Shelly
    integrator programme (OAuth) required? Ask Shelly (owner: Bulgarian company, Sofia).
  - Current cloud API version and endpoints (they have changed before).
- The cloud key gives full control of the user's devices: treat it like a password (§6).

## 4. Home Assistant
- REST: `POST <base>/api/services/<domain>/<service>` with JSON `{"entity_id": "switch.gate"}`;
  header `Authorization: Bearer <long-lived access token>`.
- State: `GET <base>/api/states/<entity_id>`; list: `GET <base>/api/states` (filter by domain:
  `switch`, `light`, `cover`, `climate`, `scene`, `script`, `binary_sensor`).
- Prefer calling the user's own HA `script.*` / `scene.*` for complex actions: HA does the logic,
  Porter is the trigger.
- Remote access: Nabu Casa URL or the user's own HTTPS URL. Reject plain `http://` outside the home
  network.
- WebSocket `/api/websocket` for live state (later; REST polling is enough for v1).

## 5. Gate and garage: practical rules
- Most gates and garage doors are opened by a **dry-contact pulse** (like pressing the wall button):
  Shelly relay in parallel with the button, set as PULSE (≈ 0.5–1 s).
- A pulse is a **toggle** for the motor: open → stop → close. Without a position sensor, Porter
  cannot know the result. So:
  - offer an optional **reed switch / contact sensor** (Shelly input or HA `binary_sensor`) and show
    real state "Gate: open/closed";
  - without a sensor, show "Gate: pulse sent 12:03", never claim it is open.
- The leaving check ("gate still open") is only possible with a sensor. Sell this in the setup guide.
- Safety: never pulse automatically without a confirmation in the car; log every pulse.

## 6. Credentials and security
- Store tokens and cloud keys encrypted with an Android Keystore key (no plain SharedPreferences,
  never in logs, never in crash or support reports).
- No own server in v1: the phone talks to Shelly Cloud / HA directly. Nothing about the user's home
  passes through us (good for privacy policy and trust).
- Local calls only on the home network (match the Wi-Fi SSID or the device's IP range).
- Network security config: HTTPS everywhere except local IPs on the LAN.
- Show the user what each connection can do ("This key can switch all your Shelly devices").

## 7. Tests
Unit tests with recorded replies for each connector (Gen1 status JSON, Gen2 RPC replies, HA states,
error codes 401/404/429/timeout). Add a test with every new device kind or API call.
