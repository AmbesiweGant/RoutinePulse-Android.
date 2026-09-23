# RoutinePulse — Android Project Scaffold (Part 2 Starting Point)

This is the initial MVVM project structure for **RoutinePulse**, built to match the
architecture and screens specified in the Part 1 Research, Planning & Design document
for OPSC6312. It is a **starting scaffold**, not a finished prototype — the goal is to
give you a clean, correctly-wired foundation so you can spend your Part 2 time on
features and polish rather than project setup.

## What's implemented end-to-end

- **Login (REQ-01)**: `LoginScreen` → `LoginViewModel` → `AuthRepository` → `AuthApi`
  (Retrofit) + `SessionManager` (DataStore) + `UserDao` (Room). Passwords are hashed
  client-side with SHA-256 before leaving the device.
- **Registration**: same pattern as login, via `RegisterScreen` / `RegisterViewModel`.
- **Dashboard shell (REQ-04)**: `DashboardScreen` reads habits from Room through
  `HabitRepository` (offline-first — habit completion writes locally and instantly).
- **Offline sync skeleton (REQ-08)**: `HabitSyncWorker` + `SyncScheduler` push any
  locally-created/completed habits to the REST API once WorkManager detects a network
  connection. Call `SyncScheduler.schedulePeriodicSync()` once from `MainActivity` or
  `RoutinePulseApp` to activate it.
- **Push notification receiver (REQ-09)**: `RoutinePulseMessagingService` renders
  incoming FCM messages as Android notifications. You still need to (a) add your own
  `google-services.json` from Firebase, and (b) build the server-side sender.
- **Multi-language strings (REQ-10)**: English, isiZulu (`values-zu`), and Afrikaans
  (`values-af`) are wired up. **The isiZulu/Afrikaans strings were machine-translated
  as placeholders — have a native or fluent speaker review them before your final
  submission**, since translation quality is something markers can assess.
- **Unit test + CI**: `PasswordHasherTest.kt` and `.github/workflows/android-ci.yml`
  run `./gradlew test` and `./gradlew assembleDebug` on every push, satisfying the
  GitHub Actions requirement.

## What's stubbed / left for you to build

- **Notes module (REQ-05)** and **Quiz generator (REQ-06)**: DTOs, Room entities, DAOs,
  and the `NoteApi` Retrofit interface already exist — you need a `NoteRepository`,
  `NotesViewModel`/`NotesScreen`, and `QuizViewModel`/`QuizScreen` following the exact
  same pattern as the Habit/Auth modules.
- **Settings screen (REQ-02, REQ-03)**: language switching, theme toggle, and
  Google SSO button click-handling all have `TODO` markers in `NavGraph.kt` and
  `RoutinePulseMessagingService.kt` — wire these to Google Identity Services and
  `AppCompatDelegate.setApplicationLocales()`.
- **The REST API itself**: this repo is the **Android client only**. You still need to
  build and host the Node.js/Express + MongoDB API described in Part 1, Section 5, and
  update `BASE_URL` in `app/build.gradle.kts` to point at it.
- **google-services.json**: required for Firebase (push notifications, Google SSO) to
  compile correctly. Create a Firebase project, register the app with package name
  `com.routinepulse.app`, and drop the downloaded file into `app/`.

## Architecture at a glance

```
UI (Compose Screens)
   ↓ observes
ViewModel (Hilt-injected, exposes StateFlow<UiState>)
   ↓ calls
Repository (single source of truth per feature)
   ↓                              ↓
Room DAO (local cache)      Retrofit API (remote)
```

Room is always the source of truth for what the UI displays — this is what makes
offline mode (REQ-08) work: writes land locally first and sync in the background.

## Project structure

```
app/src/main/java/com/routinepulse/app/
├── RoutinePulseApp.kt          # Hilt Application class
├── MainActivity.kt             # Single Activity, hosts Compose NavHost
├── di/                         # Hilt modules (Network, Database)
├── data/
│   ├── remote/api/             # Retrofit interfaces (AuthApi, HabitApi, NoteApi)
│   ├── remote/dto/             # Request/response DTOs matching Part 1 API table
│   ├── remote/fcm/             # Push notification service
│   ├── local/entity/           # Room entities
│   ├── local/dao/               # Room DAOs
│   ├── local/datastore/        # JWT session storage
│   ├── repository/             # AuthRepository, HabitRepository
│   └── sync/                   # WorkManager offline-sync job
├── domain/model/               # Clean domain models used by the UI layer
├── ui/
│   ├── auth/                   # Login + Register screens & ViewModels
│   ├── dashboard/               # Main Dashboard screen & ViewModel
│   ├── navigation/              # NavGraph + route constants
│   └── theme/                  # Compose Material3 theme (navy/green brand)
└── util/                       # Result wrapper, PasswordHasher
```

## Getting started in Android Studio

1. Open this folder in Android Studio (Koala or newer recommended).
2. Let Gradle sync — it will download the versions pinned in `app/build.gradle.kts`.
3. Add your own `app/google-services.json` (see above) once you've created a Firebase
   project — the build **will fail without it** because the `google-services` plugin
   is applied. If you want to build without Firebase set up yet, temporarily comment
   out the `id("com.google.gms.google-services")` line in `app/build.gradle.kts`.
4. Update `BASE_URL` in `app/build.gradle.kts` once your REST API is hosted.
5. Run on a device/emulator — you'll land on the Login screen. Registration/login will
   fail until the API is live; that's expected at this stage.

## Running tests locally

```bash
./gradlew test          # unit tests (PasswordHasherTest, etc.)
./gradlew assembleDebug # builds the debug APK
```

Both also run automatically via `.github/workflows/android-ci.yml` on every push.
