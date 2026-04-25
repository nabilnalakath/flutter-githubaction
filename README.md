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

1. **Generate your keystore** as you normally would for an Android app.
2. **Encode your keystore to Base64** by running the following command in your terminal:
   ```bash
   base64 -i path/to/your/keystore.jks > keystore.txt
   ```
   *(On some operating systems, you may need to use `base64 -w 0 path/to/keystore.jks`)*
3. **Add GitHub Secrets** to your repository (Settings -> Secrets and variables -> Actions):
   - `KEYSTORE_BASE64`: The contents of the `keystore.txt` file you just created.
   - `KEYSTORE_PASSWORD`: The password for your keystore.
   - `KEY_ALIAS`: Your key alias.
   - `KEY_PASSWORD`: The password for your key.

The GitHub action will automatically detect these secrets and use them to securely sign the release APK. If these secrets are not provided, it will gracefully fall back to the default debug signing keys.
