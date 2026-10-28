# Porter – Roadmap

Status: ⬜ not started · 🟨 in progress · ✅ done

## Free vs Pro (decided: Pro = one-time purchase)
| | Free | Pro |
|---|---|---|
| Connections (Shelly Cloud, Home Assistant) | 1 | unlimited |
| Favourite devices in the car | 2 | 6 |
| Scenes | 1 (Arriving home) | unlimited + conditions (only when dark, time window) |
| Arrival suggestion (300 m before home) | ✅ | ✅ |
| Leaving check ("gate still open") | – | ✅ |
| History | 24 h | 90 days |

## Phase 0 – Answers before coding ⬜
- ⬜ **Shelly:** may a paid third-party app use the Shelly Cloud key / Control API, or is the
  integrator programme (OAuth) needed? Current cloud API version. (Ask Shelly, Sofia.)
- ⬜ **Android Auto IOT category:** current rules, minimum Car API level, Play review experience.
- ⬜ **Location:** reading location only while the Android Auto session runs, without background
  location permission: allowed and reliable?
- ⬜ What the Shelly app and Home Assistant companion app already do (scenes, location triggers),
  so Porter doesn't copy free features.

## Phase 1 – Phone prototype ⬜
- ⬜ Project setup (Kotlin, package name to decide, GitHub Actions build + unit tests).
- ⬜ Connectors: Shelly local (Gen1 + Gen2+), Shelly Cloud, Home Assistant REST.
- ⬜ Devices list, switch/pulse, action log. Encrypted storage for keys and tokens.
- ⬜ Test with the owner's own devices (gate relay first).

## Phase 2 – Android Auto ⬜
- ⬜ Car home grid (scenes + favourite devices with state), confirmation, result card.
- ⬜ Desktop Head Unit tests (day/night, screen sizes), then in the car.
- ⬜ Mic button with fixed voice commands ("open the gate", "arriving home").

## Phase 3 – Scenes and triggers ⬜
- ⬜ Scene templates (Arriving, Leaving, I'm home) + editor; only-when-dark (local sunset calculation).
- ⬜ Arrival suggestion at 300 m (configurable), leaving check with contact sensors.

## Phase 4 – Google Play ⬜
- ⬜ Privacy policy (EN + BG), store texts and graphics, Android Auto opt-in and review.
- ⬜ Closed test (12 testers, 14 days), then production. Pro purchase via Google Play Billing.

## Later ideas
- More connectors: Tuya, Sonoff eWeLink, MQTT.
- Link with Herald: hail warning → "Open garage".
- Android Automotive OS (cars with Google built in).
