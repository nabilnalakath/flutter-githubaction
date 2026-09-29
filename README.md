[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Android CI](https://github.com/nabilnalakath/flutter-githubaction/actions/workflows/main.yml/badge.svg)](https://github.com/nabilnalakath/flutter-githubaction/actions/workflows/main.yml)
![Dart SDK](https://img.shields.io/badge/Dart-3.7.2-blue)
![Flutter](https://img.shields.io/badge/Flutter-stable-blue)
![Last Updated](https://img.shields.io/badge/Last%20Updated-September%202026-blue)


# Github Action in Flutter Project

This is a sample flutter project with CI-CD configuration using Github Actions.

## Choose your path

| I want to… | Use | Setup |
|---|---|---|
| Test, build signed APKs and attach them to a GitHub Release | `.github/workflows/main.yml` | [Secure Release Signing](#-secure-release-signing) |
| Also ship to Google Play and/or TestFlight with Fastlane (optional) | `.github/workflows/fastlane-android.yml`, `.github/workflows/fastlane-ios.yml` | [Deploy to the stores with Fastlane](#-deploy-to-the-stores-with-fastlane-optional) |

The two paths are independent. The Fastlane workflows don't change `main.yml`, and they do nothing costly until you add their secrets.

This project uses the following github actions -

* https://github.com/actions/checkout
* https://github.com/actions/setup-java
* https://github.com/marketplace/actions/flutter-action
* https://github.com/marketplace/actions/create-release
* https://github.com/ruby/setup-ruby (Fastlane workflows only)

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
   - `KEYSTORE_PASSWORD`: The keystore password you typed when running the `keytool` command.
   - `KEY_ALIAS`: The alias you used (e.g., `upload` if you copied the command exactly).
   - `KEY_PASSWORD`: The key password you typed when running the `keytool` command.

The GitHub action will automatically detect these secrets and use them to securely sign the release APK. If these secrets are not provided, it will gracefully fall back to the default debug signing keys.

> [!WARNING]
> **Never commit your `.jks` keystore file, `keystore.txt`, or any of your passwords directly to your git repository!** Always add them securely via GitHub Secrets and make sure your `.gitignore` is configured to ignore `.jks` and `.txt` key files.

## 🚀 Deploy to the stores with Fastlane (optional)

This repo is also the GitHub Actions example for [integrating fastlane with existing workflows](https://docs.flutter.dev/deployment/cd) in the Flutter docs. It follows the setup described there, with one workflow per platform:

| Workflow | Shows in the Actions tab as | Delivers to |
|---|---|---|
| `.github/workflows/fastlane-android.yml` | Fastlane · Android | Google Play (internal track by default) |
| `.github/workflows/fastlane-ios.yml` | Fastlane · iOS | TestFlight |

Each workflow:

- **Pull requests** that change its Fastlane files: builds the app and loads the lanes. No secrets are used, so pull requests from forks work too.
- **Tags** like `v1.2.3`: builds, signs and uploads, if that platform's secrets are set. Otherwise it only runs a few-second check on Ubuntu and stops, so projects that don't use Fastlane pay nothing (and no macOS minutes).
- **Manual run** (Actions → workflow → Run workflow): `check` verifies your credentials without uploading, `deploy` builds and uploads.

You can set up just one platform. The lanes live in `android/fastlane` and `ios/fastlane` and also run on your own machine (see [Run Fastlane locally](#run-fastlane-locally)).

### Android: Google Play

1. **Use your own package name.** Change `applicationId` in `android/app/build.gradle.kts` and `package_name` in `android/fastlane/Appfile`.
2. **Set up release signing** as described in [Secure Release Signing](#-secure-release-signing). The Fastlane workflow uses the same four `KEYSTORE_*` secrets.
3. **Create the app in the [Google Play Console](https://play.google.com/console)** and upload the first release by hand. Google Play doesn't let tools create an app or send its very first upload. Build a signed bundle locally with:
   ```bash
   KEYSTORE_PATH=/absolute/path/to/upload-keystore.jks KEYSTORE_PASSWORD=... KEY_ALIAS=upload KEY_PASSWORD=... \
     flutter build appbundle --release
   ```
   and upload `build/app/outputs/bundle/release/app-release.aab` to a testing track.
4. **Create a service account** and give it access to your app in the Play Console, following [fastlane's supply setup](https://docs.fastlane.tools/getting-started/android/setup/#setting-up-supply). Download its JSON key.
5. **Add the secret** `PLAY_STORE_JSON_KEY`: the full contents of that JSON file.

> [!NOTE]
> Uploads are created as **drafts** by default, because Google Play only accepts drafts until an app has been published once. After that, set the repository variable `PLAY_RELEASE_STATUS` to `completed` (or pick it when running the workflow manually) to release straight to testers.

### iOS: TestFlight

Requires a membership in the [Apple Developer Program](https://developer.apple.com/programs/) and a Mac for the one-time certificate setup.

1. **Use your own bundle identifier.** Change it for the Runner target in Xcode (Signing & Capabilities) and in `ios/fastlane/Appfile`.
2. **Register the bundle identifier** in your [Apple Developer account](https://developer.apple.com/account/resources/identifiers/list) and **create the app** in [App Store Connect](https://appstoreconnect.apple.com/apps).
3. **Create an App Store Connect API key** under *Users and Access → Integrations → App Store Connect API* with the **App Manager** role. Download the `.p8` file (Apple only lets you download it once) and note the **Key ID** and **Issuer ID**.
4. **Create your signing certificate with [match](https://docs.fastlane.tools/actions/match/).** Create an empty **private** repository for certificates, then on your Mac run:
   ```bash
   cd ios
   bundle install
   MATCH_GIT_URL=https://github.com/<you>/<certificates-repo>.git bundle exec fastlane match appstore
   ```
   match asks for a passphrase to encrypt the repository (this becomes `MATCH_PASSWORD`) and for your Apple ID, then creates an App Store distribution certificate and provisioning profile and stores them encrypted in that repository. CI only ever reads them.
5. **Let CI read the certificates repository.** Create a [fine-grained personal access token](https://github.com/settings/personal-access-tokens/new) with read-only *Contents* access to that repository only, then encode it:
   ```bash
   echo -n "<github-username>:<token>" | base64
   ```
6. **Add the secrets** listed below.

The workflow signs in with the API key, installs the certificate with match in read-only mode, archives the app and uploads it to TestFlight without waiting for Apple's processing. `ios/Runner/Info.plist` declares `ITSAppUsesNonExemptEncryption = false` so builds don't wait on the export compliance question. Keep it only if your app uses no encryption beyond what Apple exempts (such as HTTPS).

### Secrets and variables

Add these under **Settings → Secrets and variables → Actions**.

| Name | Type | Platform | Value |
|---|---|---|---|
| `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD` | Secret | Android | See [Secure Release Signing](#-secure-release-signing) |
| `PLAY_STORE_JSON_KEY` | Secret | Android | Contents of the service account JSON key |
| `APP_STORE_CONNECT_KEY_ID` | Secret | iOS | API key ID |
| `APP_STORE_CONNECT_ISSUER_ID` | Secret | iOS | Issuer ID |
| `APP_STORE_CONNECT_KEY_BASE64` | Secret | iOS | The `.p8` file in base64 (`base64 -i AuthKey_XXXX.p8` on macOS) |
| `MATCH_GIT_URL` | Secret | iOS | HTTPS URL of the certificates repository |
| `MATCH_PASSWORD` | Secret | iOS | The match passphrase |
| `MATCH_GIT_BASIC_AUTHORIZATION` | Secret | iOS | The base64 `username:token` from step 5 |
| `PLAY_TRACK` | Variable (optional) | Android | Track for tag releases: `internal` (default), `alpha`, `beta` or `production` |
| `PLAY_RELEASE_STATUS` | Variable (optional) | Android | `draft` (default) or `completed` |
| `BUILD_NUMBER_OFFSET` | Variable (optional) | Both | Added to the build number, for apps whose store already has higher build numbers |

### Verify your setup

Store uploads run against your own Google Play and App Store Connect accounts, so confirm your credentials before your first release:

1. Add the secrets listed above.
2. Go to **Actions → Fastlane · Android** (or **Fastlane · iOS**) → **Run workflow** and choose **`check`**. This signs in to the store and, for iOS, installs your signing certificate, without uploading anything.
3. When `check` passes, push a tag to deploy.

Every pull request that changes the Fastlane setup builds the app and loads the lanes in CI, so the build side is covered before you add any credentials.

### Release

```bash
git tag v1.2.3
git push origin v1.2.3
```

`main.yml` attaches the APKs to a GitHub Release as before, and each configured Fastlane workflow uploads to its store. The version name comes from the tag (`1.2.3`, so tags must look like `vX.Y.Z`) and the build number from the workflow's run number, so every upload gets a new one.

### Run Fastlane locally

Install [Bundler](https://bundler.io), then run the same lanes CI uses:

```bash
# Android
KEYSTORE_PATH=/absolute/path/to/upload-keystore.jks KEYSTORE_PASSWORD=... KEY_ALIAS=upload KEY_PASSWORD=... \
  flutter build appbundle --release
cd android && bundle install
PLAY_STORE_JSON_KEY="$(cat /path/to/service-account.json)" bundle exec fastlane deploy track:internal release_status:draft
```

```bash
# iOS (on a Mac)
flutter build ios --release --no-codesign --config-only
cd ios && bundle install
export APP_STORE_CONNECT_KEY_ID=... APP_STORE_CONNECT_ISSUER_ID=... \
  APP_STORE_CONNECT_KEY_BASE64="$(base64 -i /path/to/AuthKey_XXXX.p8)" \
  MATCH_GIT_URL=... MATCH_PASSWORD=...
bundle exec fastlane beta
```

The `beta` lane switches the Runner target to manual signing in `Runner.xcodeproj`. On CI that change is thrown away; locally, revert it with git if you don't want to keep it. `bundle exec fastlane validate` runs the same credential check as the `check` mode on either platform.

### Using this in your own project

Copy these into your Flutter project:

- `.github/workflows/fastlane-android.yml`, `android/Gemfile`, `android/Gemfile.lock` and `android/fastlane/`
- `.github/workflows/fastlane-ios.yml`, `ios/Gemfile`, `ios/Gemfile.lock` and `ios/fastlane/`
- the `signingConfigs` and `buildTypes` blocks from `android/app/build.gradle.kts`, which read the keystore from the `KEYSTORE_*` environment variables

Then follow the setup steps above for your own package name and bundle identifier.

### Not using Fastlane?

Nothing else depends on it. Delete `.github/workflows/fastlane-android.yml`, `.github/workflows/fastlane-ios.yml`, `android/fastlane/`, `android/Gemfile*`, `ios/fastlane/` and `ios/Gemfile*`, and `main.yml` keeps working exactly as before.
