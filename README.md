# 🧠 DimaAI — Local-first LLM Chat App for Android

DimaAI is an offline-capable AI chat app built with Jetpack Compose.  
It supports local and self-hosted models via **Ollama**, **LM Studio**, or **on-device downloads**, and features rich file support, encrypted access token storage, and advanced model management with WorkManager.

<p align="center">
  <img src="https://img.shields.io/badge/Android-Kotlin%20%7C%20Jetpack%20Compose-blueviolet?style=flat-square" />
  <img src="https://img.shields.io/badge/LLM-Ollama%20%7C%20LM%20Studio%20%7C%20Local-green?style=flat-square" />
  <img src="https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square" />
</p>

---

## 📱 Features

- ✅ **Multi-backend LLM support** (Ollama, LM Studio, Local)
- 🔁 Real-time streaming generation with `Flow`-based architecture
- 📁 Upload & preview: `.txt`, `.pdf`, `.jpg`, `.png`
- 🔐 Secure access token encryption with Android Keystore + AES-GCM
- 💾 Chat history via Room + full backup/import system
- ⚙️ Download & manage local models with WorkManager
- 🧠 Inference options: temperature, top-k/p, repetition penalty, context size, etc.
- 🌗 Material 3 + dynamic themes

---

## 🧠 Supported Inference Backends

| Engine       | Streaming | Model List | Offline?  |             Notes                     |
|--------------|-----------|------------|-----------|---------------------------------------|
| Ollama       | ✅        | ✅         | ✅       | Requires local Ollama server         |
| LM Studio    | ✅        | ✅         | ✅       | Custom HTTP endpoints                |
| Local Model  | ✅        | ✅         | ✅       | Fully supported via bundled downloads|

Switch between engines dynamically via Preferences screen.

---

## 🧱 Architecture Overview

```
├── ui/
│   ├── theme/                          # Material 3 theming
│   ├── components/                     # AppShell, layout containers
│   └── screens/                        # Composable screens and ChatViewModel
│
├── data/
│   ├── AppDatabase.kt                  # Room DB: ChatSession, ChatMessage, LocalModelMetadata
│   ├── ChatDao.kt                      # DAO interfaces for Room
│   ├── ChatMessage.kt                  # Message entity (content, files, streaming, etc.)
│   ├── ChatSession.kt                  # Session entity with UUID and metadata
│   ├── ChatRepository.kt               # Core logic for chat, streaming, persistence
│   ├── PreferenceManager.kt            # DataStore with encrypted token logic (AES-GCM + Keystore)
│   ├── WorkManagerChatRepository.kt    # Wrapper around WorkManager use for streaming/download
│   ├── DownloadRepository.kt           # Starts/stops downloads using WorkManager
│   ├── LocalModel.kt                   # Data model for models.json and downloaded models
│   ├── LocalModelMetadata.kt           # DB metadata verification info
│   └── OpenAIModels.kt                 # DTOs for OpenAI-compatible formats
│
├── domain/
│   ├── AttachmentHandler.kt            # Handles PDF, TXT, image extraction
│   ├── LocalModelManager.kt            # Syncs local models with metadata and download state
│   └── UserDataManager.kt              # Handles user-visible data ops
│
├── workers/
│   ├── ChatStreamingWorker.kt          # Background LLM message streaming
│   ├── ChatSyncWorker.kt               # Syncs message history
│   ├── DownloadWorker.kt               # Downloads and extracts models
│   └── SaveFileWorker.kt               # Downloads file attachments
│
├── services/
│   ├── OllamaService.kt                # Ollama backend integration
│   ├── LmStudioService.kt              # LM Studio backend integration
│   └── LocalModelService.kt            # Local inference orchestration
│
├── llm/
│   └── LlmChatModelHelper.kt           # Format abstraction for input messages
│
├── network/
│   ├── NetworkModule.kt                # Provides Retrofit/Moshi/OkHttp
│   └── LLMService.kt                   # Base service for streaming/chat
│
├── ChatApplication.kt                  # Main Application class with WorkManager factory
├── MainActivity.kt                     # Host activity with AppShell & navigation
├── Constants.kt                        # All app constants and WorkManager keys
└── models.json                         # Optional allowlist of downloadable local models
```

✅ MVVM  
✅ DI via `AppContainer`  
✅ `Flow`-based streaming  
✅ WorkManager for reliability

---

## ⚙️ Setup & Installation

### 🧰 Prerequisites

- Android Studio Iguana+
- Minimum SDK: 26 (Android 8.0+)
- Recommended: Android 12+

### 📦 Dependencies

Add in `build.gradle.kts`:
```kotlin
    // Coil (Image Loading)
    implementation(libs.coil.compose)
    implementation(libs.coil.gif)
    // Datastore
    implementation(libs.androidx.datastore.preferences)
    // Glide
    implementation(libs.glide)
    // Google Play - TensorFlow Lite
    implementation(libs.play.services.tflite.gpu)
    implementation(libs.play.services.tflite.java)
    implementation(libs.play.services.tflite.support)
    // Local Models / AI
    implementation(libs.tasks.genai)
    // Moshi (JSON)
    implementation(libs.moshi.kotlin)
    // Retrofit
    implementation(libs.logging.interceptor)
    implementation(libs.okhttp)
    implementation(libs.retrofit)
    implementation(libs.retrofit2.converter.moshi)
    implementation(libs.retrofit2.kotlinx.serialization.converter)
    implementation(libs.kotlinx.serialization.json)
    // Rich Text / Markdown
    implementation(libs.richtext.commonmark)
    implementation(libs.richtext.ui.material3)
    // Room (Database)
    implementation(libs.androidx.room.common.jvm)
    implementation(libs.androidx.room.ktx)
    implementation(libs.androidx.room.runtime)
    ksp(libs.room.compiler)
    // WorkManager
    implementation(libs.androidx.work.runtime.ktx)
    // WindowSizeClass
    implementation(libs.androidx.material3.window.size.class1)
    // MockK for Unit Tests
    testImplementation(libs.mockk)
    // MockK for Android instrumented Tests
    androidTestImplementation(libs.mockk.android)
    // Navigation Testing
    androidTestImplementation(libs.androidx.navigation.testing)
    // Coroutines tests
    testImplementation(libs.kotlinx.coroutines.test)
    // RichContent
    implementation(libs.androidx.appcompat)
```

### ▶️ Running the app

1. Clone the repository:
   ```bash
   git clone https://github.com/dimadusk/DimaAI.git
   cd DimaAI
   ```

2. Open in Android Studio

3. Launch on device/emulator with Android 8+

---

## 💬 Chat System

- `ChatSession`: stores metadata (title, model used, timestamp)
- `ChatMessage`: stores message content, role, timestamp, attachments
- Messages and sessions are stored in Room with cascade delete

---

## 🖼 File & Image Handling

Supports:
- Images: `.jpg`, `.png`
- Documents: `.txt`, `.pdf`
- Files stored in `filesDir` under:
  - `/chat_files/`
  - `/chat_images/`

`AttachmentHandler` handles:
- Preview
- MIME validation
- Secure copy to internal storage

---

## 📥 Model Downloads

Supported via:
- Direct binary (`.bin`)
- Zipped model packages (`.zip` with manifest)

Includes:
- Metadata persistence in Room
- Live download progress
- SHA/size verification

🔧 Bundled models allowlist defined in `res/raw/models.json`

---

## 🧪 Testing

- Room DAO: Unit tests for `ChatDao.kt`
- Repository: `ChatRepositoryTest.kt`
- UI: `composeTestRule` snapshot tests

> 📌 `ChatWorkerFactory` is injected via `Application` class

---

## 🧾 Export & Import

App supports:
- Full ZIP export (database + preferences + files)
- Individual session export (single `.txt`)
- Backup file is versioned, and includes manifest

---

## 🔐 Encrypted Token Storage

Secure access token encryption using:
- Android Keystore
- AES/GCM + IV
- Stored via `DataStore`

Use case:
```kotlin
preferenceManager.saveAccessToken("hf_xxx...")
val token = preferenceManager.readAccessToken()
```

---

## 🧩 Settings

```kotlin
val serverUrlFlow: Flow<String>
val selectedModelFlow: Flow<String>
val inferenceEngineFlow: Flow<EngineMode>
val useReliableScrollingFlow: Flow<Boolean>
```

All settings persist via `DataStore` with safe fallback.

---

## 🌍 Supported Languages
DimaAI supports multilingual UI and LLM chat and text input in the following languages:

English, Ukrainian (uk), Czech (cs), Danish (da), German (de), Greek (el), Spanish (es),
Estonian (et), Finnish (fi), French (fr), Croatian (hr),
Hungarian (hu), Indonesian (in), Italian (it), Japanese (ja), Korean (ko),
Lithuanian (lt), Latvian (lv), Dutch (nl), Norwegian (no), Polish (pl), Portuguese (pt),
Romanian (ro), Slovak (sk), Serbian (sr), Swedish (sv), Turkish (tr),
Traditional Chinese (zh-rTW), Arabic (ar), Hindi (hi), Hebrew (he)

📘 Language support depends on the selected model and backend (e.g., Ollama, LM Studio, or local).

## 🚀 Roadmap

- [ ] **PDF content preview** *(in progress)*
- [x] HuggingFace inference API support
- [x] Offline quantized model UI launcher
- [x] Export as Markdown/HTML
- [x] UI polish for tablet/foldables

---

## 🤝 Contributing

1. Fork the repo
2. Create a new branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -am 'Add feature'`)
4. Push to the branch (`git push origin feature/my-feature`)
5. Open a Pull Request

---

## 🛡 License

```
MIT License

© 2025 dimadusk

Permission is hereby granted, free of charge, to any person obtaining a copy...
```

See full [LICENSE](LICENSE) for more details.

---

## 👤 Author

Built with ❤️ by **@dimadusk**  
If this helped you, consider ⭐ starring the repo!

---

## 📸 Screenshots

### 📱 Phone UI

| Chat | Upload | Settings |
|------|--------|----------|
| ![Chat](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/phone/chat.png) | ![Upload](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/phone/upload.png) | ![Settings](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/phone/settings.png) |

| History | Preferences | Chat (Streaming) |
|---------|-------------|------------------|
| ![History](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/phone/history.png) | ![Preferences](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/phone/preferences.png) | ![Streaming](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/phone/chat_reply.png) |

🎞️ **Phone Demo (.gif)**  
![Phone Demo](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/phone/demo.gif)

---

### 💊 Tablet / Foldable UI

| Chat Split | Preview | Settings |
|------------|---------|----------|
| ![Split](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/tablet/chat_split.png) | ![Preview](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/tablet/preview.png) | ![Settings](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/tablet/settings.png) |

| History | Preferences | Streaming |
|---------|-------------|-----------|
| ![History](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/tablet/history.png) | ![Preferences](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/tablet/preferences.png) | ![Streaming](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/tablet/chat_streaming.png) |

🎞️ **Tablet Demo (.gif)**  
![Tablet Demo](https://raw.githubusercontent.com/dimadusk/DimaAI/main/screenshots/tablet/demo.gif)

---


## ☕ Support This Project

If you find DimaAI helpful, consider supporting its development ❤️

<a href="https://ko-fi.com/dimadusk" target="_blank">
  <img src="https://ko-fi.com/img/githubbutton_sm.svg" alt="Buy Me a Coffee" />
</a>
