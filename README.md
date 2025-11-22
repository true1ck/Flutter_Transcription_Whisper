# Flutter Transcription Whisper

This project demonstrates a **Speech-to-Text (STT)** pipeline integrated with **Whisper** and a **Large Language Model (LLM)**. It is built using Flutter and supports both Android and iOS platforms.

## Features
- Speech-to-Text transcription using Whisper.
- Integration with a Large Language Model for advanced text processing.
- Cross-platform support (Android, iOS, and Web).

## Getting Started

### Prerequisites
- Flutter SDK (Install from [flutter.dev](https://flutter.dev))
- Android Studio or Xcode for platform-specific builds.
- A device or emulator for testing.

### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/true1ck/Flutter_Transcription_Whisper.git
   cd Flutter_Transcription_Whisper
   ```
2. Fetch dependencies:
   ```bash
   flutter pub get
   ```

### Running the App
- For Android:
  ```bash
  flutter run -d android
  ```
- For iOS:
  ```bash
  flutter run -d ios
  ```
- For Web:
  ```bash
  flutter run -d chrome
  ```

## Project Structure
- `lib/`: Contains the Flutter application code.
- `assets/models/`: Stores the Whisper model files.
- `android/` and `ios/`: Platform-specific configurations.

## Contributing
Contributions are welcome! Feel free to open issues or submit pull requests.

## License
This project is licensed under the MIT License. See the LICENSE file for details.



