# Veil for the Mac App Store

A separate edition of Veil, built on the owner's Mac with `bash update.sh --appstore`. It installs beside the
website build (its own bundle id) and never touches GitHub, R2, latest.json or the normal release.

## What is in, what is out

| In the store edition | Left out (compile time, `cargo --features appstore` + `VEIL_APPSTORE=1`) |
| --- | --- |
| Identities (each its own WebKit data store), private windows | Chromium / Firefox / Camoufox engines (Pour installs, other processes) |
| Shields, Reader, Safari-like features, address bar, New Tab | Tor (routes and private windows with Tor) |
| Downloads, bookmarks / history import from a file you pick | Import straight from other browsers' profile folders |
| Extensions in the built-in browser | Native messaging hosts, the iCloud Passwords helper |
| Veil AI with online providers (and a local AI already running) | Installing / starting / pulling local AI models |
| Default-browser link chooser | Media page (yt-dlp, guideline 5.2.3), Torrents |
| Pro through StoreKit 2 (`veil.pro.monthly`, `veil.pro.yearly`) | Razorpay, activation codes, ipconfig account, Owner Panel, Sponsor, GitHub links |
| Problems list kept on this Mac | Crash reports and pings sent to the service, the self-updater, Finder menus |

Where a left-out feature would be, the page shows one line: "Not in the Mac App Store edition of Veil — the App
Sandbox does not allow it." `src-tauri/src/appstore.rs` lists the commands the store build must never register; its
test checks the real command list (`cargo test --features appstore`).

Pro is decided on the Mac from StoreKit 2's verified current entitlements (`mac-store/Bridge.swift` →
`libveilstore.dylib`, loaded by `src-tauri/src/storekit.rs`). Nothing is sent to Veil's service:
`storekit::REPORT_TO_WORKER` is `false`, and there is no `/veil/apple` Worker route yet. If you ever turn it on, add a
Worker route that verifies the JWS x5c chain up to Apple Root CA G3 and stores a row in `apple_subs`, and change the
privacy answer to "Purchases → Purchase History, not linked to you, not used for tracking".

## 1. Apple Developer (developer.apple.com)

1. Certificates (all made on this Mac, Xcode → Settings → Accounts → Manage Certificates): **Apple Distribution** and
   **Mac Installer Distribution**. Their names go into `.env` as `APPLE_STORE_SIGN_IDENTITY` and
   `APPLE_STORE_INSTALLER_IDENTITY`.
2. Identifiers → the App ID of your profile (explicit, not a wildcard), capability **In-App Purchase** on.
3. Profiles → **Mac App Store Connect** profile for that App ID, ticking the Apple Distribution certificate above →
   download → its path in `.env` as `APPLE_STORE_PROVISION_PROFILE`. The build reads the bundle id from this profile
   (`Entitlements → application-identifier`, team prefix removed); without a profile it uses
   `network.co.ipconfig.veil.store` from `appstore.json`.

## 2. App Store Connect — the app record

1. Apps → **+** → New App → platform **macOS**.
2. Name: `Veil — Private Browser` (or any free name). Primary language: English (U.S.).
3. Bundle ID: the one from your profile (the doctor prints it: `bash update.sh --appstore --doctor`).
4. SKU: `veil-mac-2026`. User access: Full access.
5. App Information → Category: **Productivity** (secondary: Utilities). Content rights: no third-party content.
   Age rating questionnaire: unrestricted web access → **Yes** (that gives 17+, normal for browsers).
6. Pricing and Availability: **Free**, all countries.

## 3. Subscriptions

1. Monetization → Subscriptions → **+** Subscription group: `Veil Pro`.
2. Subscription `veil.pro.monthly` — reference name "Veil Pro monthly", duration **1 month**, price: the **₹20** tier
   for India (let Apple fill the other countries), display name "Veil Pro", description "Unlimited identities and
   AI across tabs".
3. Subscription `veil.pro.yearly` — "Veil Pro yearly", duration **1 year**, the **₹220** tier, same group, ranked
   above monthly (higher level of service order: yearly first).
4. Each needs a review screenshot (the Plans page of the store build) and localization; status must reach
   "Ready to Submit". Attach both to the first app version (version page → In-App Purchases and Subscriptions).
5. Agreements, Tax and Banking → the **Paid Apps** agreement must be active, or StoreKit returns no products.

## 4. Privacy

- App Privacy → Data collection: **No, we do not collect data from this app.** (The store edition sends no crash
  reports, pings or purchase records; browsing data stays in its sandbox container.)
- Privacy Policy URL: `https://ipconfig.co.network/veil/privacy`.

## 5. Version page

- Screenshots: **2880 × 1800**, only of features the store edition has. Take them from the store page build
  (`VEIL_APPSTORE=1 npm run build`, then the UI tests' screenshots), then `python3 scripts/appstore-shots.py`
  → `appstore/screenshots/`. Never show Torrents, Media, Tor, engines, Owner Panel, updates or a Sponsor link.
- Promotional text, description, keywords (`browser, privacy, identities, multiple accounts, shields, reader`),
  support URL `https://ipconfig.co.network/veil`, copyright `2026 NewsCraft Studio`.
- Build: after the upload is processed (`node scripts/asc-builds.mjs` says VALID), **+** next to Build.
- Version release: **Manually release this version**.

### Review notes (paste)

> Veil is a web browser built on the system WebKit. Each "identity" is a separate WKWebsiteDataStore (its own
> cookies, storage and cache), so a person can keep work and personal accounts apart in one app. No account or
> sign-in is needed to review it; Pro (unlimited identities, identities in tabs, AI across tabs) is an
> auto-renewing subscription through StoreKit (Settings > Plans), with Restore Purchases there.
>
> Entitlements: network.client — loading web pages and the optional online AI provider the person configures;
> network.server — Veil runs a local route on 127.0.0.1 per identity (proxy login, ad/tracker blocking), never
> reachable from other machines; files.user-selected.read-write and downloads.read-write — saving downloads and
> importing a bookmarks file the person picks; device.camera, device.audio-input and personal-information.location
> — only when a web page asks (e.g. a video call or a map) and the person allows it in Veil's per-site prompt;
> print — printing pages.
>
> Veil does not download videos, does not install other software, and collects no data.

## 6. Build and upload (on the Mac)

```bash
bash update.sh --appstore --doctor     # names anything missing: identities, profile, login, Transporter
bash update.sh --appstore --dry-run    # build + sign, upload nothing
bash update.sh --appstore              # build, sign, .pkg, upload with Transporter
node scripts/asc-builds.mjs            # what App Store Connect did with the uploads
```

Needs: Command Line Tools (`xcode-select --install`), Rust, Node 22, and **Transporter** from the Mac App Store
(opened once). Every upload gets a new build number (UTC `yymmddHHMM`); the visible version is package.json's.
`.pkg` and `transporter.log` land in `release/appstore/`. Apple's verdict arrives by email (ITMS-…).

## 7. Check on the Mac before submitting

- The app starts sandboxed (Activity Monitor → Sandbox column: Yes) and opens pages, private windows and identities.
- Settings > Plans shows App Store prices; a sandbox tester account can buy, cancel and Restore Purchases.
- Camera / microphone / location prompts appear only after a page asks.
- Downloads land in ~/Downloads; Save As and bookmark-file import work through the open/save panel.
- The local route works under the sandbox (an identity with an HTTP / SOCKS proxy, and Shields counting blocks).
