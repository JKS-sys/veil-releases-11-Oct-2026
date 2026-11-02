# Building and releasing Veil

Two ways out: **ipconfig.co.network** (your site, GitHub releases, R2, Pour — Veil updates itself)
and the **Mac App Store**. Both are built on your Apple-silicon Mac.

## 1. Website release (ipconfig, GitHub, Pour) — the normal one

1. Put the certificates on the Mac that builds:
   - In Xcode → Settings → Accounts (or developer.apple.com → Certificates) make a
     **Developer ID Application** certificate on this Mac. Check it is there:
     `security find-identity -v -p codesigning` shows `Developer ID Application: <name> (<TEAM>)`.
   - Make an **App Store Connect API key** (App Store Connect → Users and Access → Integrations →
     Keys, role Developer) and download the `.p8` once.
2. `.env` (or ship.env — `update.sh` fills empty `APPLE_*` keys from ship.env):
   `APPLE_SIGN_IDENTITY` (the exact name above), `APPLE_TEAM_ID`, and either
   `APPLE_API_KEY_ID` + `APPLE_API_ISSUER` + `APPLE_API_KEY_PATH` (the .p8) or
   `APPLE_ID` + `APPLE_APP_PASSWORD` (an app-specific password from account.apple.com).
   Optional: `VEIL_PROVISION_PROFILE` once Apple grants the passkey entitlement
   (docs/icloud-passwords.md).
3. In the Veil folder: `git pull`, then `bash update.sh`.
   It checks the keys, deploys the Worker, runs every test, builds Veil and its torrent engine,
   **signs with the Developer ID, notarises and staples** the app and the DMG, checks them with
   Gatekeeper, publishes to GitHub + R2 (`latest.json` last), updates Pour, and pushes.
4. Users: `pour install --cask veil`, or the DMG from ipconfig.co.network/veil; existing installs
   update themselves. With the Developer ID no `xattr` step is needed any more.
   The first Developer ID release changes Veil's signature once, so macOS asks for its
   permissions one more time.

Other modes: `bash update.sh --doctor` (only the checks), `--service` (Worker only),
`--dry-run` (build, publish nothing), `--publish` (publish what is built).

## 2. Mac App Store — what it takes

The App Store build is a **separate, sandboxed variant**, and Apple reviews it. Things Veil does
today that the App Store does not allow or will likely reject:

- Installing tools with Pour and running outside programs (Tor, the torrent engine helper is
  fine if bundled and sandboxed, but yt-dlp/ffmpeg/whisper downloads are not allowed),
- downloading videos from YouTube and similar sites (guideline 5.2.3),
- a self-updater and payments other than Apple's (Razorpay, activation codes → StoreKit),
- reading other apps' data (Chromium/Firefox engines, cookie import).

So the store variant would be: the built-in WebKit browser with identities, Shields, Reader,
Reading List, Safari-like features, Veil AI (online providers), Downloads; no Media page,
no Pour/Tor installs, no updater; Pro via StoreKit 2. If you want it, say "build the App Store
variant" and it gets its own bundle id (`network.co.ipconfig.veil.store`), a cargo feature
`appstore`, sandbox entitlements and `bash update.sh --appstore`, like Dial's.

Your steps for the store, once that variant exists:
1. App Store Connect → Apps → New App (macOS, bundle id `network.co.ipconfig.veil.store`, SKU).
2. Certificates on this Mac: **Apple Distribution** and **Mac Installer Distribution**; a
   **Mac App Store** provisioning profile for the store bundle id → `.env`
   `APPLE_STORE_SIGN_IDENTITY`, `APPLE_STORE_INSTALLER_IDENTITY`, `APPLE_STORE_PROVISION_PROFILE`.
3. Install **Transporter** from the Mac App Store (it uploads the build).
4. `bash update.sh --appstore` → builds, signs, packs the .pkg and uploads.
5. In App Store Connect: subscriptions (₹20 monthly, ₹220 yearly), screenshots, privacy
   answers (Veil collects nothing), then Submit for Review.
