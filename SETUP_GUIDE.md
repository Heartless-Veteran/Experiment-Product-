# Setup Guide

This guide will help you set up the development environment and build the Experiment Product Android app.

## Prerequisites

### Required Software

1. **Android Studio** (Recommended: Hedgehog 2023.1.1 or later)
   - Download from: https://developer.android.com/studio
   - Includes Android SDK, Gradle, and necessary build tools

2. **Java Development Kit (JDK) 17**
   - Included with Android Studio
   - Or download from: https://adoptium.net/

3. **Git**
   - For cloning the repository
   - Download from: https://git-scm.com/

### API Keys and Tokens

1. **Mapbox Access Token**
   - Sign up at: https://account.mapbox.com/
   - Create a new secret token with Downloads:Read scope
   - You'll need this for dependency resolution

## Step-by-Step Setup

### 1. Clone the Repository

```bash
git clone https://github.com/Heartless-Veteran/Experiment-Product-.git
cd Experiment-Product-
```

### 2. Configure Mapbox Token

Create or edit `gradle.properties` in the project root:

```properties
MAPBOX_DOWNLOADS_TOKEN=sk.your_actual_secret_token_here
```

**Important**: 
- This token must be a **SECRET** token with Downloads:Read scope
- Never commit this token to version control
- The file is already in `.gitignore`

### 3. Configure Android SDK (Optional)

If Android Studio doesn't automatically detect your SDK, create `local.properties`:

```properties
sdk.dir=/path/to/your/android/sdk
```

Common SDK locations:
- **macOS**: `~/Library/Android/sdk`
- **Windows**: `C:\Users\YourUsername\AppData\Local\Android\Sdk`
- **Linux**: `~/Android/Sdk`

### 4. Open in Android Studio

1. Launch Android Studio
2. Select "Open an Existing Project"
3. Navigate to the cloned repository folder
4. Click "OK"

Android Studio will:
- Import the Gradle project
- Download dependencies
- Index the project
- This may take 5-10 minutes on first run

### 5. Sync Gradle

If Gradle doesn't sync automatically:
1. Click "Sync Project with Gradle Files" button (elephant icon) in the toolbar
2. Or go to: File → Sync Project with Gradle Files

### 6. Build the Project

#### Using Android Studio:
- **Menu**: Build → Make Project (Ctrl+F9 / Cmd+F9)
- **Or**: Build → Rebuild Project

#### Using Command Line:
```bash
# macOS/Linux
./gradlew build

# Windows
gradlew.bat build
```

### 7. Run the App

#### Prerequisites for Running:
- Connected Android device with USB debugging enabled
- **OR** Android Emulator configured in Android Studio

#### Using Android Studio:
1. Select a device/emulator from the device dropdown
2. Click the "Run" button (green play icon) or press Shift+F10
3. Select the "app" configuration if prompted

#### Using Command Line:
```bash
# Install on connected device/emulator
./gradlew installDebug

# Run with ADB
adb shell am start -n com.heartless.experimentproduct/.presentation.MainActivity
```

## Troubleshooting

### Common Issues

#### 1. "Plugin [id: 'com.android.application'] was not found"

**Solution**: Ensure you have internet connectivity to download the Android Gradle Plugin from Google Maven repository.

```bash
# Test connectivity
curl -I https://dl.google.com/dl/android/maven2/
```

If blocked, work from a network with full internet access or configure a proxy.

#### 2. "Mapbox SDK not found" or "401 Unauthorized"

**Solution**: Check your Mapbox token configuration:
- Ensure token is in `gradle.properties`
- Verify it's a **secret** token (starts with `sk.`)
- Verify it has Downloads:Read scope
- Try regenerating the token if needed

#### 3. "SDK location not found"

**Solution**: Create `local.properties` with correct SDK path (see Step 3).

#### 4. "Gradle sync failed"

**Solutions**:
- File → Invalidate Caches / Restart
- Delete `.gradle` folder and resync
- Update Gradle: ./gradlew wrapper --gradle-version 8.2
- Check internet connectivity

#### 5. Build fails with "Compilation error"

**Solutions**:
- Ensure JDK 17 is being used
- Check Android Studio settings: File → Settings → Build, Execution, Deployment → Build Tools → Gradle → Gradle JDK
- Clean and rebuild: ./gradlew clean build

#### 6. App crashes on launch

**Possible causes**:
- Missing Mapbox token at runtime
- Permissions not granted
- Emulator/device API level < 26 (minSdk)

**Solutions**:
- Check Logcat for error messages
- Verify app permissions in device settings
- Use device/emulator with API 26 or higher

### Getting Help

If you encounter issues:
1. Check the error message in Android Studio's Build window
2. Review Gradle Console for detailed logs
3. Run with `--stacktrace` for more details:
   ```bash
   ./gradlew build --stacktrace
   ```
4. Check [Android Studio Troubleshooting Guide](https://developer.android.com/studio/troubleshoot)

## Verification

### Successful Setup Checklist

After setup, verify:
- [ ] Project opens in Android Studio without errors
- [ ] Gradle sync completes successfully
- [ ] Build succeeds (Build → Make Project)
- [ ] App installs on device/emulator
- [ ] App launches and shows a map
- [ ] No runtime crashes

### Expected Build Output

A successful build should show:
```
BUILD SUCCESSFUL in XXs
XX actionable tasks: XX executed
```

### Runtime Verification

When the app runs, you should see:
- App launches successfully
- Mapbox map loads and displays
- "Mapbox Map Loaded" text appears at top
- No crashes or errors in Logcat

## Development Workflow

### Daily Development

```bash
# Pull latest changes
git pull

# Clean build (if needed)
./gradlew clean

# Build and run
./gradlew installDebug
```

### Before Committing

```bash
# Run lint checks (when configured)
./gradlew lint

# Run tests (when tests are added)
./gradlew test

# Verify build
./gradlew build
```

## Project Structure Overview

```
Experiment-Product-/
├── app/
│   ├── src/
│   │   └── main/
│   │       ├── java/          # Kotlin source files
│   │       ├── res/           # Resources (layouts, strings, etc.)
│   │       └── AndroidManifest.xml
│   └── build.gradle.kts       # App-level build config
├── gradle/
│   └── wrapper/               # Gradle wrapper files
├── build.gradle.kts           # Project-level build config
├── settings.gradle.kts        # Project settings
├── gradle.properties          # Gradle properties (includes Mapbox token)
├── local.properties           # Local SDK path (not in git)
└── README.md                  # Project documentation
```

## Next Steps

After successful setup:
1. Explore the codebase architecture (see ARCHITECTURE.md)
2. Review the Clean Architecture implementation
3. Understand the Hilt dependency injection setup
4. Examine the Jetpack Compose UI components
5. Start developing new features!

## Additional Resources

- [Android Developer Documentation](https://developer.android.com/)
- [Jetpack Compose Documentation](https://developer.android.com/jetpack/compose)
- [Mapbox Android SDK](https://docs.mapbox.com/android/maps/guides/)
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Hilt Documentation](https://dagger.dev/hilt/)
- [Room Database Guide](https://developer.android.com/training/data-storage/room)

## Support

For project-specific issues, please open an issue on the GitHub repository.
