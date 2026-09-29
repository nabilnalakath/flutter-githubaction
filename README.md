[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Android CI](https://github.com/nabilnalakath/flutter-githubaction/actions/workflows/main.yml/badge.svg)](https://github.com/nabilnalakath/flutter-githubaction/actions/workflows/main.yml)
![Dart SDK](https://img.shields.io/badge/Dart-3.7.2-blue)
![Flutter](https://img.shields.io/badge/Flutter-stable-blue)
![Last Updated](https://img.shields.io/badge/Last%20Updated-September%202026-blue)


# GitHub Action in Flutter Project

A sample Flutter project with CI/CD on GitHub Actions: it runs tests, builds signed APKs for GitHub Releases, and can optionally ship to Google Play and TestFlight with Fastlane.

## Choose your path

| I want to… | Use | Setup |
|---|---|---|
| Test, build signed APKs and attach them to a GitHub Release | `.github/workflows/main.yml` | [Secure Release Signing](#-secure-release-signing) |
| Also ship to Google Play and/or TestFlight with Fastlane (optional) | `.github/workflows/fastlane-android.yml`, `.github/workflows/fastlane-ios.yml` | [docs/fastlane.md](docs/fastlane.md) |

Use `main.yml` on its own, or add the Fastlane workflows next to it. The Fastlane workflows only build and upload once you add their secrets.

This project uses the following GitHub Actions:

* https://github.com/actions/checkout
* https://github.com/actions/setup-java
* https://github.com/marketplace/actions/flutter-action
* https://github.com/marketplace/actions/create-release
* https://github.com/ruby/setup-ruby (Fastlane workflows only)

For a complete guide on implementation, read the tutorial on [Medium](https://medium.com/better-programming/ci-cd-for-flutter-apps-using-github-actions-b833f8f7aac)

## 🔐 Secure Release Signing

To automatically sign your release APK with your own keystore using this GitHub Action, you need to configure your repository secrets.

1. **Generate your keystore**
   If you don't already have a `.jks` keystore file, you can generate one using the `keytool` utility (which comes bundled with Java or Android Studio). Run this in your terminal:
   ```bash
   keytool -genkey -v -keystore upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
   ```
   *(**Note for Mac users:** If you get a "command not found" error because Java isn't installed system-wide, you can use the version bundled with Android Studio by typing `/Applications/Android\ Studio.app/Contents/jbr/Contents/Home/bin/keytool` instead of just `keytool`)*
2. **Encode your keystore to Base64**:
   - **Mac:**
     ```bash
     base64 -b 0 -i "path/to/your/keystore.jks" > keystore.txt
     ```
   - **Linux:**
     ```bash
     base64 -w 0 "path/to/your/keystore.jks" > keystore.txt
     ```
   - **Windows (PowerShell):**
     ```powershell
     [Convert]::ToBase64String([IO.File]::ReadAllBytes("path\to\your\keystore.jks")) | Out-File keystore.txt
     ```
3. **Add GitHub Secrets** to your repository (Settings -> Secrets and variables -> Actions):
   - `KEYSTORE_BASE64`: The contents of the `keystore.txt` file you just created.
   - `KEYSTORE_PASSWORD`: The keystore password you typed when running the `keytool` command.
   - `KEY_ALIAS`: The alias you used (e.g., `upload` if you copied the command exactly).
   - `KEY_PASSWORD`: The key password you typed when running the `keytool` command.

The GitHub action will automatically detect these secrets and use them to securely sign the release APK. If these secrets are not provided, it will gracefully fall back to the default debug signing keys.

> [!WARNING]
> **Never commit your `.jks` keystore file, `keystore.txt`, or any of your passwords directly to your git repository!** Always add them securely via GitHub Secrets and make sure your `.gitignore` is configured to ignore `.jks` and `.txt` key files.

## 🚀 Deploy to the stores with Fastlane (optional)

The Fastlane workflows build, sign and upload the app to **Google Play** (`.github/workflows/fastlane-android.yml`) and **TestFlight** (`.github/workflows/fastlane-ios.yml`) when you push a `vX.Y.Z` tag, following the [Flutter continuous delivery guide](https://docs.flutter.dev/deployment/cd). The Android workflow reuses the signing secrets above, and both only build and upload once you add their store secrets.

**Setup, secrets and usage: [docs/fastlane.md](docs/fastlane.md)**
