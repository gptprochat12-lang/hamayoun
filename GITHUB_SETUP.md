# GitHub Actions Setup Instructions

## Files Added to Your Project

| File | Purpose |
|---|---|
| `.github/workflows/android.yml` | Builds Debug + Release APK on every push/PR |
| `gradle/wrapper/gradle-wrapper.properties` | Pins Gradle 8.9 (required by AGP 8.7.0) |
| `gradlew` | Unix build script |
| `gradlew.bat` | Windows build script |
| `.gitignore` | Keeps build artifacts & secrets out of Git |

---

## ⚠️ One Manual Step Required – gradle-wrapper.jar

The `gradle-wrapper.jar` binary cannot be auto-generated here.  
You must add it **once** on your local machine before pushing to GitHub.

### Option A – You already have Android Studio / Gradle installed

Open a terminal in your project root and run:

```bash
gradle wrapper --gradle-version 8.9
```

This creates `gradle/wrapper/gradle-wrapper.jar` automatically.

### Option B – Use Android Studio

1. Open the project in Android Studio.
2. Go to **File → Project Structure → Project**.
3. Set Gradle version to **8.9** and click OK.  
   Android Studio will download the wrapper jar for you.

### Option C – Copy from any other Android project

The `gradle-wrapper.jar` is the same file for all projects using the same
Gradle version. You can copy it from any existing project:

```
any-other-android-project/gradle/wrapper/gradle-wrapper.jar
  →  your-project/gradle/wrapper/gradle-wrapper.jar
```

---

## How the GitHub Actions Workflow Works

```
Push to main/master
        │
        ▼
  ubuntu-latest runner
        │
        ├─ Set up JDK 17
        ├─ Cache Gradle dependencies
        ├─ ./gradlew assembleDebug   → uploads  FlappyBird-Debug-APK
        └─ ./gradlew assembleRelease → uploads  FlappyBird-Release-APK-unsigned
```

After each successful run, go to:  
**GitHub → Your Repo → Actions → (latest run) → Artifacts**  
and download your APK directly.

---

## Signing the Release APK (Optional)

The workflow currently builds an **unsigned** release APK.  
To produce a signed APK ready for distribution:

1. Generate a keystore:
   ```bash
   keytool -genkey -v -keystore my-release-key.jks \
     -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
   ```

2. Add these **GitHub Secrets** (Settings → Secrets → Actions):
   - `KEYSTORE_BASE64` – base64 of your `.jks` file  
     (`base64 -w 0 my-release-key.jks`)
   - `KEY_ALIAS` – your alias
   - `KEY_PASSWORD` – key password
   - `STORE_PASSWORD` – keystore password

3. Add a signing step to the workflow — ask for the updated `android.yml`
   with signing configured if you need it.
