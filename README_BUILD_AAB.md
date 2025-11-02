# SPC Timesheet — One-Click AAB Builder
_Last updated: 2025-11-02_

This kit lets you produce the **single file Google Play needs** — an **Android App Bundle (.aab)** — with one GitHub Actions run.

## What you'll do (once)
1. Push these files to your Android project root (where `settings.gradle` lives).
2. On GitHub → **Settings → Secrets and variables → Actions → New repository secret**, add:
   - `ANDROID_KEYSTORE_BASE64` – your signing keystore **base64** (see below).
   - `ANDROID_KEY_ALIAS` – the key alias (e.g., `spc_release`).
   - `ANDROID_KEY_PASSWORD` – the key password.
   - `ANDROID_KEYSTORE_PASSWORD` – the keystore password.
3. Ensure your Gradle script uses these props OR Android Gradle Plugin default signing config:
   - Example in `app/build.gradle(.kts)`:
     ```groovy
     android { 
       signingConfigs {
         release {
           storeFile file(System.getProperty("user.home") + "/signing_keystore.jks")
           storePassword System.getenv("SIGNING_STORE_PASSWORD") ?: project.findProperty("MY_STORE_PASSWORD")
           keyAlias System.getenv("SIGNING_KEY_ALIAS") ?: project.findProperty("MY_KEY_ALIAS")
           keyPassword System.getenv("SIGNING_KEY_PASSWORD") ?: project.findProperty("MY_KEY_PASSWORD")
         }
       }
       buildTypes {
         release {
           signingConfig signingConfigs.release
           minifyEnabled false // or true if you use R8/proguard
         }
       }
     }
     ```

## Generate a keystore (if you don’t have one)
On your machine, run:
```bash
bash scripts/generate-keystore.sh
```
It creates `spc_release.jks` and prints a base64 value you can paste into `ANDROID_KEYSTORE_BASE64`.

## Build the AAB
1. Go to **Actions** → **Build Android App Bundle** → **Run workflow**.
2. Enter `versionName` (e.g., `1.0.0`) and `versionCode` (integer, e.g., `1`).
3. When it finishes, download the artifact. That `.aab` is your **single upload file** for Google Play.

## Upload to Google Play
In Play Console → **Production** (or Test track) → **Create new release** → upload the AAB you downloaded.

### Notes
- Requires Java 17 on CI (handled by the workflow).
- Make sure you target **API 35** and have a Release build variant.
- If your outputs path differs, the workflow searches the repo for any `.aab`.