---
name: porter-scenes
description: Porter's product logic - arrival and leaving scenes, when to suggest them (distance to home, Android Auto session, dark/light), the leaving check, confirmation and safety rules, the car-screen and phone-screen layouts, free vs Pro. Use whenever changing scenes, triggers, location handling, the car UI flows, or the free/Pro split.
---

# Porter – scenes, triggers and screens

## 1. Product idea
"Your house knows you're coming." HomeLink-style buttons (premium cars) for any car with Android
Auto, plus scenes: 300 m before home, one tap opens the gate and switches on the outside lights.
The phone app is for setup and simple control; the **car screen is the product**.

## 2. Scenes
```
Scene: name, icon, steps[], conditions
 step: device + action (on/off/pulse/temp) + optional delay
 conditions: onlyWhenDark, onlyBetween(hh:mm–hh:mm), requiresSensorState
```
Built-in templates (the user picks devices, Porter fills the rest):
- **Arriving home**: gate pulse → outside lights on (only when dark) → heating to X °C.
- **Leaving home**: gate pulse → lights off → check states after 60 s.
- **I'm home**: garage light off, outside lights off after 5 min.
Steps run **one after another** (cloud rate limits), with a result per step shown in the car:
"Gate ✓ · Lights ✓ · Heating ✗ offline".

## 3. Triggers (suggest, never act alone)
- **Arrival:** while an Android Auto session is active and the car is within **R** of home
  (default 300 m, configurable 100–1000 m) and getting closer → show the **arrival card**:
  "Arriving home – Open gate & lights?" [Run] [Not now].
- **Leaving:** session active, car left the home zone (> 500 m) → check sensor states → if something
  is open/on: "⚠ Gate still open" [Close] [Ignore].
- **Dark/light:** sunset/sunrise calculated on the device (NOAA solar algorithm, no internet), with
  civil twilight as "dark".
- **Location design choice:** read location only **while the car app session is running** (the app
  is in the foreground on the car screen). This avoids background location and Play's background
  location declaration. ⚠ Verify that this holds for Android Auto with the current Play policy.
  Optional later: Geofencing API for phone-only use (needs `ACCESS_BACKGROUND_LOCATION`, a Play
  declaration and a video; avoid in v1).
- Home position: set once on the phone ("I'm at home now" or pick on a map).

## 4. Safety rules (non-negotiable)
- Gates/garages: **always one confirmation tap** in the car, never fully automatic.
- One running scene at a time; a second tap while running does nothing ("Running…").
- A pulse is never retried (see `porter-smart-home`).
- Show real state only when a sensor exists; otherwise "pulse sent hh:mm".
- Every action is logged (phone: History screen).

## 5. Car screens (templates)
1. **Home** (`GridTemplate`, ≤ 6 items): scenes first (Arriving, Leaving), then favourite devices
   with state ("Gate · closed", "Yard lights · off").
2. **Arrival card** (`MessageTemplate`): one sentence, two actions.
3. **Result** (`MessageTemplate` or toast): per-step result, auto-return to Home.
4. **Device** (`ListTemplate` with toggles): only for SWITCH/LIGHT; pulses stay on Home.
No setup in the car. If nothing is configured: "Set up Porter on your phone".

## 6. Phone screens
Connections (Shelly Cloud key, HA URL + token) · Devices (import, rename, kind, favourite) ·
Scenes (templates + editor) · Home position & radius · History · Settings / Pro.

## 7. Free vs Pro (decided by the owner)
| | Free | Pro (one-time) |
|---|---|---|
| Connections | 1 | unlimited |
| Favourite devices in the car | 2 | 6 (template limit) |
| Scenes | 1 (Arriving home) | unlimited + conditions |
| Arrival card / leaving check | ✅ / – | ✅ / ✅ |
| History | 24 h | 90 days |
No own server and no subscription needed (no running costs), so a one-time price fits.

## 8. Wording
Calm and short, like a good porter: "Gate opened", "Lights on", "Welcome home". No exclamation
marks, no jokes in errors. Languages: English and Bulgarian from v1.
