<!-- .github/copilot-instructions.md -->
# Repo-specific Copilot Instructions

This file teaches AI coding agents the minimal, high-value context to be productive in this Flutter + native-Whisper demo repository.

**Overview**
- **Purpose:** demo pipeline: Speech -> Whisper (on-device) -> app UI. See `README.md`.
- **Stack:** Flutter (Dart) UI, native Whisper integrations on Android (Kotlin) and iOS (Swift), model assets under `assets/models/whisper`.

**Big-picture architecture & data flow**
- Flutter UI (`lib/main.dart`, `ui/`) drives user flows and calls the platform bridge in `lib/whisper/whisper_flutter_bridge.dart`.
- Platform channel names: Method channel `whisper_method_channel` and Event channel `whisper_events` (must match on all sides).
- Native code (Android: `android/app/src/.../WhisperFlutterBridge.kt`, iOS: `ios/Runner/Whisper/WhisperFlutterBridge.swift`) initializes models, records audio, transcribes, and emits events back to Dart.
- Model binary should be copied from assets into a writable path (Dart helper `getModelFilePath()` / `copyAssetToDocuments`) before calling `initializeModel(path)`.

**Key files & roles (quick references)**
- `lib/whisper/whisper_flutter_bridge.dart` — Dart method/event channel wrapper; examples: `initializeModel(String path)`, `toggleRecording()`, `transcribeSample(path)` and `events` stream.
- `lib/whisper/whisper.dart` — higher-level Dart helpers that use the bridge and example flows (copy assets, listen to events).
- `android/app/src/main/kotlin/.../WhisperFlutterBridge.kt` — Kotlin native implementation, exposes the same method names and publishes events via `EventChannel`.
- `android/app/src/main/kotlin/.../MainActivity.kt` — wires method & event channels into the Flutter engine and forwards permission result to the bridge.
- `ios/Runner/Whisper/WhisperFlutterBridge.swift` — Swift native implementation using `WhisperCore` and `FlutterEventChannel`/`FlutterMethodChannel`.
- `ios/Runner/AppDelegate.swift` — registers method/event handlers and forwards calls to `WhisperFlutterBridge.shared`.
- `pubspec.yaml` — declares assets (note: large model binaries may be excluded from VCS and must be added manually).

**Platform channel contract (exact strings & payloads)**
- Method channel name: `whisper_method_channel`.
- Event channel name: `whisper_events`.
- Common methods and expected args:
  - `initializeModel` : { "path": "<local file path>" } -> returns `bool` (true on success) or error codes `INIT_FAIL`/`FILE_NOT_FOUND`.
  - `transcribeSample` : { "path": "<sample wav path>" }
  - `toggleRecording`, `startRecording`, `stopRecording`, `reset`, `enablePlayback` : boolean/void as appropriate.
  - `isModelLoaded`, `canTranscribe`, `isRecording` : return boolean.
- Events: published as maps with `event` key plus event-specific payloads, e.g. `{"event":"didTranscribe","text":"..."}` or `{"event":"recordingFailed","error":"..."}`.

**Conventions & project-specific patterns**
- Permission handling differs by platform: Dart uses `permission_handler` for Android paths and calls method channel on non-Android. See `callRequestRecordPermission()` in `lib/whisper/whisper_flutter_bridge.dart`.
- Large model binaries are expected under `assets/models/whisper` but may not be checked in; the code copies assets to app documents via `copyAssetToDocuments()` before initializing the native model.
- Native implementations are asynchronous: Swift uses `Task`/`@MainActor`, Kotlin uses `CoroutineScope(Dispatchers.Main)` — follow those async patterns when modifying native code.

**Build, run & debug (platform notes)**
- Standard Flutter steps (Windows PowerShell):
  - `flutter pub get`
  - `flutter run -d <device id>` (Android devices or emulator). iOS builds require macOS/Xcode.
  - `flutter test`
  - `flutter build apk` for release Android
- iOS-specific: run `pod install` inside `ios/` and open Xcode for device builds: `cd ios; pod install` (must run on macOS).

**Common gotchas & developer notes**
- If `initializeModel` fails with `FILE_NOT_FOUND`, ensure the model binary was copied to the app documents folder and pass that exact path.
- The sample `jfk.wav` is present under `assets/models/whisper/jfk.wav` — useful for smoke-testing `transcribeSample`.
- Event maps are untyped; prefer defensive access (check for keys) when consuming events in Dart.
- When changing method/event names, update all 3 places: Dart, Android, and iOS.

**If you need to add a model binary**
1. Place model file at `assets/models/whisper/ggml-base.en.bin` (or update `pubspec.yaml` if using a different name).
2. Run `flutter pub get` to ensure assets are packaged (note: very large binaries sometimes exceed package limits — consider downloading at runtime and copying into the documents folder instead).

**Extending the bridge**
- Add the new method name (string) to:
  1. `lib/whisper/whisper_flutter_bridge.dart` (Dart method wrapper)
  2. `android/.../WhisperFlutterBridge.kt` (handle MethodCall)
  3. `ios/Runner/Whisper/WhisperFlutterBridge.swift` (handle the method, return typed result)
- Add unit/integration tests in Dart that exercise `whisper.dart` flows where possible.

---
If anything above is unclear or you want a different level of detail (examples, tests, or CI steps), tell me which section to expand and I'll iterate.
