# Porter

"Your house knows you're coming." An Android + Android Auto app that opens the gate, switches the
outside lights and runs home scenes from the car (HomeLink-style buttons for any car with Android Auto).
Sister app: **Herald** (`C:\pd\Herald`, weather and hazards on the route). Same owner, same style.

## Skills
In `.claude/skills/`: `android-auto-car-app` (same file in Herald: keep both copies in sync),
`porter-smart-home` (Shelly, Home Assistant, security), `porter-scenes` (scenes, triggers, screens, free/Pro).

## Plan
`ROADMAP.md` is the plan: phases, free/Pro split, open questions. Read it first, and update the
status markers when a phase moves.

## Decisions (owner)
- Name **Porter** (umbrella idea: Porter = home, Herald = weather).
- **Pro = one-time purchase** (no running costs, no own server in v1).
- Gates and garage doors: **always one confirmation tap** in the car, never fully automatic.
- Languages: English and Bulgarian from v1.

## Versioning (owner's rule)
**Every commit gets a new version: +0.01** (versionName +0.01, versionCode +1), including docs-only
commits. Start: 0.01 / versionCode 1. Add an entry at the top of `CHANGELOG.md`, put the version
first in the commit message ("v0.02: …") and tell the owner the new version number.
The owner decides when a version becomes stable. Commit and push only when the owner asks.

## Working notes
- There is no Android SDK on this PC: nothing can be compiled here. The owner builds in Android
  Studio or with GitHub Actions, then tests on the phone and in the car (Desktop Head Unit first).
- Kotlin, minimal dependencies (`androidx.car.app`, an HTTP client). Unit tests with recorded device
  replies; add a test with every new connector call.
- Never put Shelly keys, Home Assistant tokens or other user secrets in code, logs, reports or commits.
- The owner's car: Toyota Yaris Hybrid 2016 (EU) with an Android head unit; Android Auto testing
  needs a phone + a car or head unit with Android Auto.
