# Project Brief: TrackLore (Media Intelligence & Song Context App)

---

## 1. Executive Summary

**TrackLore** is a lightweight, background-enabled Android application built with React Native. The system listens passively for active media notifications from audio streaming platforms (Spotify, Apple Music, YouTube Music, Audiomack, local players), extracts the currently playing track and artist metadata, and fetches concise contextual analysis—including underlying themes, lyrical meaning, and production background—via an automated AI pipeline.

The primary deliverable is a frictionless user experience: delivering context in real time via companion notifications, floating overlays, or a clean mobile dashboard without requiring manual track search or active app switching.

---

## 2. Core Problem & Objective

* **Problem:** Music listeners often encounter songs with cryptic, poetic, or culturally specific meanings, but manual lookups (opening a browser, searching Genius/SongMeanings, parsing long forum threads or annotation pages) interrupt the listening flow.
* **Objective:** Automate the metadata extraction and semantic interpretation pipeline into a passive, single-touch micro-service that surfaces song meanings instantly on track transition.

---

## 3. Scope of Work

### In Scope (MVP)

* **Android Media Listener:** Custom native Android background module utilizing `NotificationListenerService` or `MediaSessionManager` to capture playback events.
* **Metadata Extraction Engine:** Normalisation of messy track strings (stripping tags like `[Remastered]`, `(feat. ...)`, `Explicit Version`) into clean `artist` and `title` keys.
* **Context Generation Pipeline:** Direct integration with an LLM inference API (structured JSON output) grounded by optional web-retrieval/Genius metadata fallback.
* **Local Persistence & Caching:** On-device caching layer (MMKV or SQLite) storing analyzed tracks by hashed key (`artist:title`) to minimize network egress and latency.
* **Notification Overlay / Companion Card:** Dynamic Android notification displaying a 2–3 line distilled meaning directly under the active media player.
* **Main Dashboard:** Clean list of listening history, complete contextual write-ups, and favorite annotations.

### Out of Scope (MVP)

* **iOS Support:** Excluded due to iOS sandbox restrictions barring cross-app notification interception.
* **Full Lyric Display:** Excluded to avoid licensing/copyright bottlenecks; output focuses on thematic critique, trivia, and semantic summaries.
* **Social Graph / Sharing:** Deprioritized for the initial release in favor of local performance and battery optimization.

---

## 4. Technical Architecture

```
[ Active Media Player ] (Spotify, YouTube Music, etc.)
         │
         ▼  (Posts media notification to Android OS)
[ Android NotificationListenerService ] (Kotlin Native Module)
         │  (Extracts: Package, Artist, Title, Playback State)
         ▼
[ React Native NativeEventEmitter / HeadlessJS ]
         │
         ├──► [ Local Cache (MMKV/SQLite) ] ── (Match Found) ──┐
         │                                                     │
         └──► [ Context Pipeline ]                             │
                     │                                         │
                     ├──► Clean & Sanitize Query               │
                     ├──► LLM Inference API (Structured JSON)  │
                     └──► Write to Local Cache                 │
                                                               │
                                                               ▼
[ Output Layer: Companion Notification / App Dashboard / Floating Overlay ]

```

### Proposed Technology Stack

* **Framework:** React Native (`bare` workflow or Expo with Prebuild / Custom Config Plugins).
* **Native Layer:** Kotlin (`NotificationListenerService`, `MediaSessionManager`, `BroadcastReceiver`).
* **State & Local Storage:** Zustand, React Native MMKV / WatermelonDB (for offline speed and low disk footprint).
* **Intelligence / Context API:** Gemini API / Groq (Llama-3-8B-Instruct) for near-instant inference ($<1\text{s}$ latency) and structured JSON responses.
* **Metadata Normalizer:** Regex pipeline + Genius API search endpoint for disambiguation of obscure tracks.

---

## 5. Functional Requirements

| ID | Feature | Description | Priority |
| --- | --- | --- | --- |
| **FR-01** | Permission Onboarding | Dedicated screen walking users through granting Android `BIND_NOTIFICATION_LISTENER_SERVICE` access. | P0 |
| **FR-02** | Media Stream Interception | Reliably isolate media notifications from system/chat notifications across major streaming packages. | P0 |
| **FR-03** | Song Sanity Normalisation | Filter extraneous string noise (e.g., `"Song - Live At Wembley 2012"` $\rightarrow$ `"Song"`). | P0 |
| **FR-04** | Thematic Breakdown Generation | Query AI backend to return: (1) Core Theme, (2) Lyrical Subtext, (3) Key Trivia/Origin. | P0 |
| **FR-05** | Offline-First Caching | Return cached summaries in $<50\text{ms}$ when a track is replayed; persist across app reboots. | P0 |
| **FR-06** | Companion Notification | Update a persistent low-priority Android notification on track change with the new song summary. | P1 |
| **FR-07** | Floating Overlay (ChatHead) | Optional overlay bubble that expands to show context without leaving the current app. | P2 |

---

## 6. Non-Functional & Operational Requirements

* **Battery Efficiency:** Listener service must remain idle between track change broadcasts. No active polling or continuous wake-locks.
* **Latency:** Total time from track change broadcast to companion notification update must not exceed $2.5\text{ seconds}$ on cold queries ($<100\text{ms}$ on cache hits).
* **OEM Compatibility:** Implement clear user guides to exempt the app from aggressive vendor task killers (Samsung One UI, Transsion HiOS, Xiaomi HyperOS).

---

## 7. Implementation Roadmap

* **Phase 1: Native Spike & Bridge (Week 1–2)**
* Scaffold project via Expo Prebuild.
* Build Kotlin `NotificationListenerService` and bridge emitted events to React Native.
* Validate metadata capture across Spotify, Apple Music, and YouTube Music.


* **Phase 2: Context Pipeline & Caching (Week 3)**
* Implement string cleaning logic.
* Wire up LLM provider with structured system prompts.
* Integrate MMKV local caching and error handling for unrecognised audio tracks.


* **Phase 3: Foreground Presentation (Week 4)**
* Design companion notification service.
* Build main history feed and single-track detailed view in React Native.


* **Phase 4: Optimization & QA (Week 5)**
* Stress-test memory usage and battery consumption during multi-hour playback sessions.
* Validate OEM background survival and refine edge-case track sanitisation.
