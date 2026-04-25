[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![](https://github.com/nabilnalakath/flutter-action/workflows/main.yml/badge.svg)
![Dart SDK](https://img.shields.io/badge/Dart-3.7.2-blue)
![Flutter](https://img.shields.io/badge/Flutter-stable-blue)
![Last Updated](https://img.shields.io/badge/Last%20Updated-May%202025-blue)


# Github Action in Flutter Project

This is a sample flutter project with CI-CD configuration using Github Actions.

This project uses the following github actions -

* https://github.com/actions/checkout
* https://github.com/actions/setup-java
* https://github.com/marketplace/actions/flutter-action
* https://github.com/marketplace/actions/create-release

For a complete guide on implemenatation read the tutorial on [Medium](https://medium.com/better-programming/ci-cd-for-flutter-apps-using-github-actions-b833f8f7aac)

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
   - `KEYSTORE_PASSWORD`: The password for your keystore.
   - `KEY_ALIAS`: Your key alias.
   - `KEY_PASSWORD`: The password for your key.

The GitHub action will automatically detect these secrets and use them to securely sign the release APK. If these secrets are not provided, it will gracefully fall back to the default debug signing keys.

> [!WARNING]
> **Never commit your `.jks` keystore file, `keystore.txt`, or any of your passwords directly to your git repository!** Always add them securely via GitHub Secrets and make sure your `.gitignore` is configured to ignore `.jks` and `.txt` key files.
