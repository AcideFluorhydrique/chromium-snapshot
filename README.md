# chromium-snapshot

Every day, this repository republishes Google's official **Chromium trunk snapshot for Android arm64** as a GitHub Release, so it can be tracked by tools such as [Obtainium](https://github.com/ImranR98/Obtainium).

> This repository does **not** build Chromium. The APK is identical to the one in Google's snapshot bucket and is republished unmodified.

## Why this exists

Chromium's official snapshots live in a Google Cloud Storage bucket (`chromium-browser-snapshots`). They are addressed by revision number, come with no version notifications, and can't be tracked directly by app-update tools.

This repository does one simple thing:

1. Fetches the latest `Android_Arm64` revision.
2. Downloads the official `chrome-android.zip` and extracts `ChromePublic.apk`.
3. Reads the version from inside the APK and verifies its signing certificate.
4. Creates a GitHub Release tagged with that version and attaches the APK.

Release tags use the **real version number read from the APK** (e.g. `v1xx.0.xxxx.0`), not the revision or the date. This way Obtainium doesn't mistake the same version for a new one, so you don't get false update notifications.

## Download and install

1. Go to the [Releases](https://github.com/AcideFluorhydrique/chromium-snapshot/releases) page and download the latest `chromium-<version>-r<revision>-android-arm64.apk`.
2. Install it on your Android device (you'll need to allow installing from unknown sources).

File name format:

```
chromium-<version>-r<revision>-android-arm64.apk
```

### Auto-updates with Obtainium (recommended)

1. In Obtainium, tap **Add App**.
2. Paste this repository's URL: `https://github.com/AcideFluorhydrique/chromium-snapshot`
3. Once added, new Releases are tracked automatically.

## APK details

| Item | Value |
| --- | --- |
| Platform | Android arm64 (`Android_Arm64`) |
| Package name | `org.chromium.chrome` |
| Source | `chromium-browser-snapshots/Android_Arm64/<revision>` |
| Signing key | The public debug key bundled in the Chromium source tree |
| Certificate SHA-256 | `32a2fc74d731105859e5a85df16d95f102d85b22099b8064c5d8915c61dad1e0` |
| Update frequency | Once a day (05:23 UTC); can also be triggered manually |
| Retention | The 20 most recent Releases; older ones are deleted along with their tags |

## How it works

The pipeline is defined in [`.github/workflows/publish_release.yaml`](.github/workflows/publish_release.yaml) and runs on GitHub Actions:

- **Download latest snapshot**: reads `LAST_CHANGE` to get the newest revision (it must be purely numeric), downloads `chrome-android.zip`, and extracts `ChromePublic.apk`.
- **Read version and verify signature**: uses `aapt2` from the Android build-tools to read `versionName`, and `apksigner` to get the signing certificate's SHA-256. If it doesn't match the expected public Chromium debug certificate, the run aborts and nothing is published.
- **Publish release**: if a Release for `v<version>` already exists, it skips; otherwise it renames the APK and creates the Release.
- **Remove old releases**: keeps only the most recent `KEEP` Releases (default 20).

## Things to know

- **This is a trunk snapshot, not a stable release.** It changes every day and may be buggy, unstable, or have unfinished features. It isn't recommended as your only daily browser, and shouldn't be used for highly sensitive accounts.
- **The signing key is public.** Because the build is signed with Chromium's public debug key, anyone can sign an APK with the same key. The signature only confirms that the APK matches the signature of Chromium's official snapshots; it doesn't prove who published it. If provenance matters to you, compare against the file in the official bucket yourself.
- **It can't be installed over the Play Store version of Chrome/Chromium, or any build with a different signature.** Android will ask you to uninstall the existing app first.
- This repository is **not affiliated with Google or the Chromium project**. It only republishes publicly available build artifacts.

## Manual trigger

Go to **Actions → Publish Chromium Android snapshot → Run workflow** to check for and publish the latest version immediately (`workflow_dispatch`).

## Forking

After forking, just enable the workflow on the Actions tab. No extra secrets are needed (it uses the built-in `github.token`). To target another platform, change `PLATFORM` and the extracted file path in the workflow.
