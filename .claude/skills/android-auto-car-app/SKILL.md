---
name: android-auto-car-app
description: How to build an Android Auto app with the Android for Cars App Library (androidx.car.app) - app categories, manifest, CarAppService/Session/Screen, templates and their limits, driver-distraction rules, testing with the Desktop Head Unit and Play review. Use whenever writing or changing the car-screen part of the app, the manifest entries for the car, or planning what can be shown in the car.
---

# Android Auto apps (Car App Library)

Porter and Herald are sister apps (same owner, same style). This skill is the same in both repos;
keep the copies in sync when one changes.

## 1. The one rule that decides everything: the category
Android Auto shows only apps of an approved category. Google reviews the car part against it, and
apps outside their category are rejected.

| App | Category (manifest) | Main templates |
|---|---|---|
| **Porter** (home: gates, lights, scenes) | `androidx.car.app.category.IOT` | `GridTemplate`, `ListTemplate` (with toggles), `MessageTemplate` |
| **Herald** (weather and hazards on the route) | `androidx.car.app.category.WEATHER` | `ListTemplate`, `PaneTemplate`, `MessageTemplate`, map template if allowed |

Other categories, for reference: `NAVIGATION`, `POI`, `PARKING`, `CHARGING`, `MESSAGING`, `CALLING`;
media apps use `MediaBrowserService` / Media3 instead of templates.

⚠ **Verify before building** (rules change every year): the current category list and the minimum
Car App API level for IOT and WEATHER; whether WEATHER apps may use `MapWithContentTemplate` (radar)
while driving; whether one app may declare more than one category. **Assume one category per app**;
that's why Porter and Herald are separate apps.

## 2. Structure
```
CarAppService  (entry point, declared in the manifest)
 └─ Session    (one per car connection; lifecycle-aware)
     └─ Screen (stack of screens; each returns a Template from onGetTemplate())
```
- `Screen.invalidate()` asks the host to call `onGetTemplate()` again (use it to refresh state).
- `screenManager.push(screen)` / `pop()`: the stack is limited (task flow limit, see §4).
- `CarContext` gives `CarToast`, `ConstraintManager`, `AppManager`, location (`carContext`'s app context).
- Keep car code in its own package (`…/car/`) and share logic with the phone UI through a common
  layer (repository / use cases). The car screen never talks to devices or APIs directly.

## 3. Manifest (Android Auto)
```xml
<application>
  <service android:name=".car.PorterCarAppService" android:exported="true">
    <intent-filter>
      <action android:name="androidx.car.app.CarAppService" />
      <category android:name="androidx.car.app.category.IOT" />
    </intent-filter>
  </service>
  <meta-data android:name="androidx.car.app.minCarApiLevel" android:value="1" />  <!-- set to what the category needs -->
  <meta-data android:name="com.google.android.gms.car.application"
             android:resource="@xml/automotive_app_desc" />
</application>
```
`res/xml/automotive_app_desc.xml`:
```xml
<automotiveApp><uses name="template" /></automotiveApp>
```
- Dependency: `androidx.car.app:app` (latest **stable**; check the version in the release notes).
- Optional for Android Automotive OS (cars with built-in Google): module with `androidx.car.app:app-automotive`.
- `createHostValidator()`: `HostValidator.ALLOW_ALL_HOSTS_VALIDATOR` only in debug builds; release
  builds use the allow-list of known hosts (Android Auto, Automotive) from the library docs.

## 4. Templates and limits
- **No custom drawing.** Only navigation apps get a free surface (a map). Everything else is templates.
- **Item limits:** ask `ConstraintManager.getContentLimit(CONTENT_LIMIT_TYPE_LIST / _GRID)`; typically
  about 6 list rows while driving. Never assume more.
- **Task flow limit:** about 5 templates deep in one task. Design flows of 1–3 steps.
- **Refresh limits:** refreshing the same template with new values is fine; changing to a different
  template type counts as a step. Don't refresh more often than needed (Porter: on state change;
  Herald: every few minutes, or when a warning changes).
- **Text:** short. Titles of one line, no paragraphs while driving (`LongMessageTemplate` and
  `SignInTemplate` are parked-only).
- **No typing while driving.** Setup (accounts, devices, scenes, destinations) happens on the phone.
- Icons: `CarIcon` from vector drawables; tint for day/night is handled by the host.
- Actions: up to 2 primary actions per `MessageTemplate`; destructive actions need clear labels.

## 5. Driver-distraction rules (design checklist)
- One glance, one tap. The main screen must be usable in under 2 seconds.
- No video, no animations, no ads in the car UI.
- Voice/TTS: short spoken alerts are OK through the host; don't fight the navigation voice.
- Confirm dangerous actions (gate, garage) but keep it to one extra tap.
- Show state, not just buttons ("Gate: closed", "Lights: on").

## 6. Testing without a car
1. SDK Manager → install **Android Auto Desktop Head Unit (DHU)**.
2. On the phone: Android Auto settings → tap the version 10× → developer mode → ⋮ → **Start head unit server**.
   For apps not installed from Play: developer settings → **Unknown sources** on.
3. `adb forward tcp:5277 tcp:5277`, then run `desktop-head-unit` from `SDK/extras/google/auto/`.
4. Test day and night, small and wide screens (DHU config files), touch and rotary input.
5. Then test in a real car (the owner).

## 7. Google Play
- Play Console → app → release → **Advanced settings → Form factors → Android Auto** (opt in, accept
  the Android Auto addendum). The car part gets an extra review against the **Car App Quality
  guidelines**; plan for a rejection round.
- Internal and closed testing tracks work for Android Auto; testers see the app in the car.
- The listing needs car screenshots (DHU screenshots are fine).
- The phone app must be useful on its own (setup + control); the car part is an extension.

## 8. Voice
- **No custom wake word.** The car's microphone and "Hey Google" belong to Google Assistant / Gemini.
  An app cannot listen all the time for "Porter, …" or "Herald, …".
- **What works today:**
  - a **mic button** on the car screen → record with `CarAudioRecord` (Car App Library; ⚠ check the
    minimum Car API level) → on-device `SpeechRecognizer` → match a **small fixed command list**
    (Porter: "open the gate", "arriving home", "lights off"; Herald: "weather ahead", "any warnings");
  - spoken answers with `TextToSpeech`, audio focus `AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK`, usage
    `USAGE_ASSISTANCE_NAVIGATION_GUIDANCE`, one or two short sentences.
  - Porter: a gate command by voice still needs the on-screen confirmation (or a spoken "yes").
- ⚠ Check every year whether Google lets third-party IoT/weather car apps be called through
  Assistant/Gemini ("Hey Google, ask Herald …"). Not available for these apps when this was written.
