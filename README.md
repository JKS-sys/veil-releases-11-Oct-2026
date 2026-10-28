<p align="center">
  <img src="docs/icon-dark.png" width="112" alt="Veil icon, dark">
  &nbsp;&nbsp;
  <img src="docs/icon-light.png" width="112" alt="Veil icon, light">
</p>

<h1 align="center">Veil</h1>

<p align="center"><b>Separate identities. One private browser.</b><br>
A small private browser for Apple silicon Macs — every profile is its own device,<br>and Chrome and Firefox extensions run inside it the way they do in Orion.</p>

<p align="center">
  <a href="https://github.com/JKS-sys/veil-releases-11-Oct-2026"><img alt="Star Veil on GitHub" src="https://img.shields.io/github/stars/JKS-sys/veil-releases-11-Oct-2026?style=for-the-badge&logo=github&label=Star%20Veil&color=1a5bc6"></a>
</p>

> ⭐ **Like Veil? Please give it a star on GitHub** — press **Star** at the top right of this page. It takes a second, helps other people find Veil, and keeps the creator building it.

<p align="center">
  <a href="https://github.com/JKS-sys/veil-releases-11-Oct-2026/releases/latest"><b>Download the latest release</b></a> ·
  <a href="#plans">Plans</a> ·
  <a href="#demos">Demos</a> ·
  <a href="#extensions-that-work-like-orion">Extensions</a> ·
  <a href="#platforms">Platforms</a> ·
  <a href="#screenshots">Screenshots</a>
</p>

<picture>
  <source media="(prefers-color-scheme: light)" srcset="docs/light-identities.png">
  <img src="docs/dark-identities.png" alt="Veil — identities, each with its own fingerprint, cookies and route">
</picture>

Every profile is a separate world: a different fingerprint, separate cookies and sessions, and your own choice of proxy, VPN, Tor route and extensions. One identity never follows another.

Websites link accounts when they come from the same device, and then one ban takes them all. In Veil each account runs in its own profile, so every site sees a different device.

By **Jagadeesh Kumar S**, creator · [youtube.com/@JKS-sys](https://www.youtube.com/@JKS-sys) · [Sponsor](https://razorpay.me/@NSBJKS)

---

## Size and speed

| | |
|---|---|
| App binary | under **5 MiB** on Apple silicon (`tests/size.mjs` enforces the budget) |
| Stack | Tauri 2 + Rust + Preact. No HTTP or ZIP crates: Veil has its own small SOCKS/HTTP proxy, ZIP reader, SHA-256 and BLAKE2b; licence and update signatures are checked with a verify-only Ed25519. |
| Fastest engine | The **Built-in** engine is the system WebView: WebKit on macOS, Edge WebView2 on Windows, WebKitGTK on Linux. It adds no download, starts instantly and uses the least memory. |

Heavier engines are optional. They are downloaded once into Veil's own folder and are never bundled, which is how the app stays under 5 MB.

## The four engines

| Engine | What runs | Extensions | Fingerprint |
|---|---|---|---|
| **Built-in** | System WebView, with tabs in a Veil window | **Chrome Web Store and Firefox Add-ons.** macOS and Linux: Veil's extension host runs them like Orion — background pages and service workers, pop-ups, options pages, side panels, messages and ports, storage, tabs, context menus, badges, notifications and block lists. Windows (WebView2): Chrome extensions natively. | Veil Shield script, plus its own data store per profile |
| **Chromium** | Helium (one-click download), or an installed Brave, Thorium, Chromium, Vivaldi or BrowserOS | **Chrome Web Store** | Veil Shield, loaded as an extension in the page's main world, plus `TZ` and language |
| **Firefox** | An installed Firefox, LibreWolf, Floorp, Waterfox or Zen | **Firefox Add-ons** | Firefox's own fingerprinting protection, plus `user.js` privacy settings |
| **Camoufox** | Camoufox (one-click download) | **Firefox Add-ons** | Set at engine level: UA, OS, screen, WebGL, timezone, locale, geolocation and audio. It can claim any OS. |

Branded Google Chrome and Microsoft Edge (version 137 and later) ignore command-line extensions, so the Shield can't load in them. Veil marks them "No shield" and recommends Helium.

## What each profile gets

- **Its own device.** Screen, GPU (WebGL vendor and renderer), CPU cores, memory, canvas noise and audio noise come from the profile's own seed. The same profile looks the same every time; a different profile looks different. This is proven in Chromium by `shield_and_route_in_real_chromium`.
- **Its own storage.** Each profile has a separate data folder (or WebKit data store), so cookies, logins and history never cross. **Burner** profiles are wiped every time they close.
- **Its own route.** The options are Direct, HTTP, SOCKS5 or SOCKS4 with username and password, or **Tor with its own circuit** (Tor's `IsolateSOCKSAuth` option, with one login per profile).
  - The browser connects to a private local port, and Veil logs in upstream. DNS goes through the proxy, and the traffic counter shows up and down.
  - VPNs: use the provider's SOCKS5 or HTTP endpoint (Mullvad, NordVPN, PIA, Proton and others).
- **Location that matches the route.** "Test route" shows the exit IP, city and timezone. "Match proxy location at every launch" keeps timezone, language and geolocation consistent with the exit.
- **No WebRTC leaks.** By default WebRTC only goes through the route; you can also set it to Block.
- **Its own extensions.** Paste a Chrome Web Store or Firefox Add-ons link once, then switch the extension on per profile.
  - Chrome extensions **keep their store id**: Veil writes the CRX's signing key into the manifest, so sync and settings work. This is tested against the real Web Store.
  - Firefox add-ons are side-loaded signed, and switched-off ones are removed again.
  - You can also install from either store directly inside a profile.

## Extensions that work, like Orion

Orion showed that a WebKit browser can run Chrome and Firefox extensions. Veil does the same in its Built-in windows on macOS and Linux, with an **extension host** of its own:

| What extensions need | In Veil |
|---|---|
| Background page / MV3 service worker | runs in a hidden page per extension and identity, with that identity's cookies and route; `onInstalled`, `onStartup`, alarms |
| Storage (`local`, `sync`, `session`) | one store per extension and identity, shared by the background, pop-up, options, side panel and content scripts, saved to disk, `onChanged` everywhere |
| Messages and ports | `runtime.sendMessage`, `tabs.sendMessage`, `runtime.connect`, `tabs.connect`, replies with `sendResponse` or a promise |
| Pop-up | opens under its button and sizes itself to what is on it (as wide as its content wants, up to 800 px); when it is taller or wider than the window, the whole pop-up is shown smaller instead of scrolling or being cut off; knows the current tab; Esc or a click elsewhere closes it |
| Options | `options_ui` pages open in a dialog under the toolbar like Chrome's (or in a tab when the extension asks); right-click an extension's button for pop-up, options, options in a tab, pop-up in a tab, side panel, unpin, store page, switch off |
| Its own origin | every extension lives at `veil://x-<id>/` (like `chrome-extension://<id>/`), so files asked for from the root (`/pages/panel/index.js`) load and one extension can't read another's storage |
| Requests to other sites | `fetch` and `XMLHttpRequest` (axios too) from the extension's pages reach the hosts it was granted without CORS errors, through the identity's route, with its own cookies — like Chrome |
| Proxy and VPN extensions | `chrome.proxy.settings` (fixed servers, PAC, direct, system) switches the identity's route; the proxy's username and password come from the extension's `webRequest.onAuthRequired`; `chrome.privacy.*` settings answer |
| Helper apps (native messaging) | `runtime.connectNative` / `sendNativeMessage` start the helper another browser installed (its `NativeMessagingHosts` manifest), when it allows the extension |
| Options page / side panel | options open in a tab; `sidePanel` / `sidebarAction` open a panel docked to the window |
| Tabs, windows, scripting | `tabs.query/create/update/remove/reload`, `windows.*`, `scripting.executeScript` (functions and files), `insertCSS`, `registerContentScripts`, tab events |
| Toolbar button | badge text and colour, title, icon, `action.onClicked` when there is no pop-up |
| Context menus | `contextMenus` items appear in Veil's right-click menu; `onClicked` goes back to the extension |
| Notifications, downloads | desktop notifications that open Veil when clicked; `downloads.download` |
| Blocking (`declarativeNetRequest`) | static and dynamic **block** rules by domain go into Shields, so uBlock Origin Lite and AdGuard lists block in Veil's route |

Content scripts reach the host through the page title, which a site's Content Security Policy cannot block, and every call carries a secret for that extension and identity. Problems an extension's pages report show under it in the Extensions menu (⚠). Deleting or switching off an extension removes its toolbar button, background page, block lists and proxy in every window at once. Blocking `webRequest` still needs a real Chrome or Firefox, and a Chromium or Firefox identity runs it.

**iCloud Passwords**: since macOS 15.4 Apple's helper only answers browsers on Apple's list (Chrome, Edge, Brave, Arc, Orion…), checked by code signature and Apple Team ID. Veil finds the helper and speaks its protocol; it joins the list once it is signed with an Apple Developer ID — [docs/icloud-passwords.md](docs/icloud-passwords.md) has the entry ready for Apple's repository. Until then the extension's pop-up offers **Open iCloud Passwords**, which opens the same passwords in a Brave, Chrome or Edge identity.

## One account for all the apps

Sign in once in **Settings → Plans → Account** with an email code (or a password when email codes are paused). Wherever Pro was bought — the shop on [ipconfig.co.network](https://ipconfig.co.network/shop#veil), Razorpay inside Veil, a gift or activation code, the Mac App Store or the Microsoft Store builds — it is on your account, and each computer gets an **Ed25519-signed licence** bound to it. Licences work offline for up to 14 days and renew every 12 hours. Monthly and yearly Pro run on 2 computers; when you are over the limit Veil lists your computers so you can deactivate one right there. Redeem gift codes, see and cancel subscriptions, or buy on the website from the same box.

**Owner Panel → Dashboard** is always connected to the ipconfig dashboard — nothing to press: on the owner's Mac Veil says hello and syncs at launch and every 30 minutes, with `X-Owner` (the hash of the owner serial) + `X-App: veil`, or `Authorization: Bearer <APPS_API_KEY or Veil's own ak_veil_… key>` when a key is set (kept in the Keychain; update.sh puts `APPS_API_KEY` from `.env` there; never shown again). From there: say hello and sync, stats, and every table (computers, codes, subscriptions, problems, devices, entitlements, users) with add, edit, delete, tick boxes, **Select all**, bulk actions (revoke, restore, extend, fix, note, label…) through one `POST /owner/bulk`, **Delete all** after typing `ALL <kind>`, search and export — in **16 layouts** (table, cards, board, tiles, timeline, list, compact, sheet, masonry, split, focus, kanban, gallery, mosaic, magazine, stack), remembered per table.

## Right-click menus everywhere

- **In web pages**: open links in a new tab, a background tab, a private or Tor window, or **another identity**; copy the link or a **clean link** without trackers; save or search with an image; picture in picture, loop and speed for video; search, explain, translate or read a selection aloud; Reader, Night mode, translate or summarise the page; and the items your extensions add. Shift + right-click gives the system menu.
- **In the toolbar**: **‹ and ›** list the tab's own history (jump straight to any page, open the previous page in a new tab, duplicate, full history); **⟳** reloads all tabs, stops, zooms or clears site data; the **address bar** pastes and goes, copies the link, a clean link without trackers or a Markdown link, opens the page privately or in the Wayback Machine; tabs (reload, duplicate, pin, mute, sleep, close others / to the right, reopen), bookmarks, the tab strip, the private, Tor, history, downloads, grid, settings and ✦ AI buttons, and extension buttons.
- **In Veil's own pages**: open the browser, private and Tor windows, a link from the clipboard, Veil AI, every section, import, theme, sounds, updates, report a problem, star Veil — in columns when the window is short, never a scroll bar.
- **The Dock (macOS), taskbar (Windows) and launcher (Linux)**: new window, new tab, private and Tor windows, **open a link from the clipboard**, Veil AI, an **Identities** submenu, **Recently Visited** sites, bookmarks, history, downloads, Playlist, extensions, Network & Tor, Plans & Account, Check for Updates, settings.
- **Finder, Files, Dolphin, Nemo and Explorer** (files and folders): *Open in Veil*, *Open in a new Veil tab*, *Open in Veil Private Window*, *… with Tor*, *Open in Veil Identity…*, *Ask Veil AI about this file*, *Explain with Veil AI*, *Translate with Veil AI*, *Search the web for this name in Veil*, *Copy Veil Link (file://)*. New entries install themselves after an update. Switch them off in Settings.

## Downloads

Files you download from a page go to Veil's own download manager (Downloads in the sidebar, or the ⬇ button): up to **16 parallel connections** per file through the identity's own route and with the page's cookies, **pause and resume** (also after quitting Veil), a **queue** with how many run at once, a **speed limit**, **retries**, **scheduled** starts, sorting into Documents / Music / Video / Images / Archives / Programs, a notification when a file is done, its SHA-256, and Finder's *Where from*. Choose the **download folder**, or have Veil ask for every file. Paste several links at once; right-click a link, image or video → *Save* sends it there too.

## Notifications

Sites that ask get Veil's own prompt; allowed sites (and extensions) show **desktop notifications**, and clicking one brings Veil to the front on that very tab. Manage the sites in Settings → Site notifications.

<a id="platforms"></a>
## Platforms

Veil is built and released for **macOS on Apple silicon** (M1 and later), macOS 12 or newer.

## Install

```bash
curl -fsSL https://ipconfig.co.network/pour/install | sh
pour install --cask veil
```

Or download `Veil_…_aarch64.dmg` from the [releases](https://github.com/JKS-sys/veil-releases-11-Oct-2026/releases). Updates arrive inside Veil (Settings → Updates) or with `pour upgrade veil`.

## Opening Veil = opening your browser

Veil opens straight into your **Personal** browser window: cookies, logins, history and bookmarks are kept, and it is its own identity. Everything else — identities, engines, extensions, AI, network and settings — opens as a tab (grid and gear buttons in the toolbar, ⌘,) or in its own window (Window → Identities & Settings). Closing the last Veil window quits Veil completely. The start-up animation plays on the first window each launch (Settings → Appearance). Settings → Browsing switches this to open Identities & Settings instead.

Brave-style features in every Veil window:

- **Shields** (lion button): ads and trackers are blocked in Veil's own route before they leave the computer — a built-in list of about 3,600 hosts, "Update block lists" adds Peter Lowe's list and StevenBlack's hosts (about 75,000). The count ticks up live; click to lower Shields for one site. Shields work in Built-in, Chromium and Firefox identities. Cosmetic filtering hides empty ad boxes.
- **New Tab page**: clock, search, live Shields stats (blocked here, blocked since install, bandwidth and time saved), top sites, bookmarks.
- **Private window** (⇧⌘N) and **Private window with Tor** (⌥⌘N): fresh device, nothing kept, wiped when closed.
- **Bookmarks** (★ / ⌘D, bookmarks bar), **History** (⌘Y), **Downloads**, **Find in page** (⌘F), a choice of search engine (DuckDuckGo, Brave Search, Startpage, Mojeek, Ecosia, Bing, Google).
- **Address-bar suggestions**: a colourful panel (not the system list) that floats over the page on macOS and Windows with Open / Search / Ask Veil AI rows, your bookmarks and history with the match highlighted, and one-click AI and search engines. ↑ ↓ to move, Enter to open, Tab to complete, `@` for an engine, `?` for Veil AI.
- **Extensions button** (puzzle piece): switch your library's extensions on or off for this identity, and open their pop-up or options page in a tab.

### Brave's features, in Veil

| Brave | In Veil |
|---|---|
| Shields: ads & trackers | ✓ built-in list + updatable lists, per-site toggle, live count (all engines) |
| Fingerprinting protection | ✓ a different device per identity (stronger than Brave's randomising) |
| HTTPS upgrade | ✓ http sites open over https when they can, falling back by itself |
| Debouncing, query filtering, De-AMP | ✓ **Clean links**: utm_/fbclid/gclid… removed, Google/Facebook/YouTube/Reddit redirects skipped, AMP → the publisher |
| Cookie consent blocking | ✓ banners hidden, scrolling given back |
| Speedreader | ✓ **Reader view** (⋯ → Reader view): the article alone, font size and theme |
| Night mode | ✓ per page or for every site |
| Picture-in-picture, tab mute | ✓ ⋯ → Picture in picture / Mute this tab |
| Playlist | ✓ ⋯ → Add to Playlist, and a Playlist page that plays direct media |
| Brave News | ✓ **Veil News** on the New Tab: your RSS/Atom feeds, fetched by Veil, no tracking |
| Wayback Machine prompt | ✓ a missing page offers its saved copy |
| Private window with Tor | ✓ (Pro) |
| Leo AI | ✓ Veil AI with local AI, GitHub Copilot and GitHub Models |
| Brave Search & AI answers | ✓ every engine with `@shortcuts`, `?` for Veil AI |
| Install from the Chrome Web Store / Firefox Add-ons | ✓ the stores' own **Add to Chrome / Add to Firefox** buttons install into Veil (the store sees a Chrome or Firefox browser, so no “Switch to Chrome” banner), plus an **Add to Veil** button — added to your library and switched on for that identity |
| Extension buttons & pop-ups | ✓ enabled extensions pinned in the toolbar; the pop-up opens under its button, sized to its content, and knows the current tab |
| Restore tabs after a crash or forced quit | ✓ windows and tabs come back (the front tab live, the others asleep); or Settings → “Continue where you left off” every time |
| Tab menu, pinned tabs, reopen closed tab | ✓ right-click a tab: reload, duplicate, pin, mute, sleep, close others / to the right; ⇧⌘T reopens |
| iCloud Passwords | ✓ on macOS: an “iCloud Passwords” identity in Brave, Chrome or Edge with Apple's helper connected (Settings → Brave features) |
| Memory Saver (tab sleeping) | ✓ background tabs sleep after 10 / 20 / 60 minutes (not while playing sound) and wake on click |
| GPU rendering | ✓ Metal on macOS, GPU rasterisation + zero-copy on Windows, hardware acceleration always on Linux; instant Back/Forward cache |
| Import bookmarks & history | ✓ Chrome, Brave, Edge, Vivaldi, Arc, Opera, Firefox, Safari, or an exported .html file |
| Brave VPN | your own proxy / VPN / Tor route per identity |
| Rewards, Wallet, Talk | left out: crypto ads, a crypto wallet and a calling service are not part of a 4 MB private browser |

Brave itself is not copied into Veil: brave-core is a Chromium fork of tens of gigabytes that builds to about 200 MB. Veil rebuilds the everyday features on the system engine, and can run Brave as an identity's engine when it is installed.

## Veil AI

- **Find every AI on this computer**: Ollama, LM Studio, Jan, GPT4All, Msty, AnythingLLM, llama.cpp / llamafile / LocalAI, KoboldCpp, vLLM, text-generation-webui, plus **GitHub Copilot** (the Copilot CLI), GitHub Models (through `gh`) and any OpenAI-compatible URL with a key.
- **GitHub Copilot with the CLI, signed in through your browser**: Veil installs the Copilot CLI (Pour or npm), starts GitHub's sign-in, shows the one-time code and opens github.com/login/device. The same works for the GitHub CLI (GitHub Models). Copilot answers in an empty folder with no tools, so it can only answer. Models: GPT-5.4, GPT-5.3 Codex, GPT-5 mini, GPT-4.1.
- **Install** (with Pour), **Start**, **download models** with live progress (Llama 3.2, Qwen 2.5, Gemma 3, Phi-4 mini, Mistral, DeepSeek R1, Qwen Coder, LLaVA, or any name), **delete models**, **Use** a model, and **Uninstall** (to the Trash / winget; optionally its models too).
- **The Veil AI window** (✦ in the toolbar, ⌘J) reads the tab in front: Summarise, Key points, Explain simply, Explain selection, Translate (11 languages), Fact-check, Draft a reply, Table, Explain code — or ask anything. Replies stream in, with coloured headings, lists and code (brackets coloured by depth).
- **Rewrite the selection**: Formal, Friendly, Shorter, Fix grammar, Bullet points, Simpler words.
- **All tabs at once** (Pro): Summarise all tabs, Compare tabs (a table and a pick), Research notes.
- **Saved chats and your own prompts** (Pro): keep a chat, reopen it later; save what you typed as a one-click prompt.
- **In the address bar**: `?question` asks Veil AI; `@perplexity`, `@chatgpt`, `@copilot`, `@duckai`, `@braveai`, `@you`, `@phind`, `@mistral`, `@grok` send the question to that AI; `@bing`, `@google`, `@brave`, `@wiki`, `@yt`, `@gh` … search there. Any of them can be the default engine (toolbar picker or Settings).

## Identities in tabs

The ⧉ menu next to + opens a tab in another identity inside the same window: its own cookies, fingerprint, proxy / VPN / Tor route and Shields, marked by a coloured badge on the tab. Links it opens stay in that identity. Chromium and Firefox identities (with their own extensions) still open in their own windows.

## Your everyday browser

- **Set Veil as the default browser** in Settings → *Your everyday browser*. On macOS you confirm the system prompt; on Windows Veil registers itself and opens *Default apps*; on Linux it installs `veil.desktop` and calls `xdg-settings`.
- **Links from other apps** open where you choose: the **chooser** (a small window; keys `1`–`9`, ↑ ↓ Enter, Esc; "Always open this site here" remembers per site), a fixed identity, or a fresh Burner. A link for an identity that is already open becomes a new tab in it (Built-in window, Chromium, or Firefox).
- **Address bar** on the Identities page: type a site or a search and pick the identity.
- **Extensions**: switching a Chrome extension on moves the identity to Chromium; a Firefox add-on moves it to Firefox or Camoufox. Helium or Camoufox is fetched automatically the first time — so Veil works as one browser with both stores. (The system WebView itself cannot run store extensions on macOS or Linux; on Windows WebView2 runs Chrome extensions.)
- **App icon**: Light (the default), Dark, or Match theme (Settings → Appearance).

## Using it

- **New identity**: pick an engine, name it, and set a proxy if you want one. Then press **Open**.
- **Burner window** (⇧⌘N): a throwaway Built-in window with a fresh device. Everything is gone when you close it.
- In a Built-in window: ⌘T new tab, ⌘W close tab, ⌘L address bar, ⌘R reload, ⌘[ / ⌘] back and forward, ⌃Tab next tab. Downloads go to `~/Downloads`.
- Desktop shortcut for one identity: `veil --profile "Shop main" [url]`.
- In each identity's ⋯ menu: Duplicate as new device · New fingerprint · Open the store · Check fingerprint (BrowserLeaks) · Show data folder · Clear cookies and data.

<a id="plans"></a>
## Plans

| | Free · ₹0 | Pro · ₹20 a month | Pro · ₹220 a year |
|---|---|---|---|
| Built-in browser, Shields, private windows, New Tab | ✓ | ✓ | ✓ |
| Identities (own device, cookies, proxy / VPN, extensions) | 3 | unlimited | unlimited |
| Chrome Web Store and Firefox Add-ons | ✓ | ✓ | ✓ |
| Veil AI with local AI, address-bar AI engines | ✓ | ✓ | ✓ |
| Identities in tabs (several accounts in one window) | | ✓ | ✓ |
| Tor routes and private windows with Tor | | ✓ | ✓ |
| Camoufox engine | | ✓ | ✓ |
| GitHub Copilot in Veil AI, AI across all tabs, saved chats and prompts | | ✓ | ✓ |
| Automatic weekly Shields list updates | | ✓ | ✓ |

Pay with UPI, cards or net banking through Razorpay (Settings → Plans) or on the [website](https://ipconfig.co.network/shop#veil), or redeem a code. Pro lives on your account (see [One account](#one-account-for-all-the-apps)): sign in on each computer and it gets its own signed licence, checked offline and renewed every 12 hours.

## Updates

Settings → Updates (or **Veil → Check for Updates…**). Veil asks **GitHub (public)** first, then **GitHub (private)** through Veil's service, then **Cloudflare R2**. The first one that answers decides; the download falls back to the R2 copy of the same file. Every update is checked against Veil's signing key before it replaces the app.

<p align="center"><img src="docs/float-light.png" width="820" alt="Address-bar top sites floating over the page"><br><img src="docs/float-typed-light.png" width="820" alt="Grouped suggestions floating over the page"></p>

## Privacy and problem reports

Once a day Veil sends an anonymous computer hash, its version, system, chip and plan, so the creator knows how many computers use it. Pages, history, identities and proxies never leave your computer. Turn sending off in Settings → Updates.

Problems are collected from every part of Veil — Rust panics, JavaScript errors in every window (toolbar, tabs, Veil AI, the chooser), windows or tabs that could not open, macOS crash reports (`~/Library/Logs/DiagnosticReports/veil-*.ips`) and problems you describe yourself in **Settings → Problems → Report**. They are sent a few seconds after they happen, in batches, with your home folder replaced by `~`, and each one shows *sent* or *queued*. In the Owner Panel they arrive grouped by kind and version, with how many computers saw each one; **Send a test report** checks the whole path.

<a id="demos"></a>
## Demos

Each demo is recorded from Veil at 2× and plays below; click **▶ MP4** for the full-quality video.

**1. Extensions like Orion** — the extension's button shows its badge; its pop-up opens under the button, sized to its content; the extensions menu shows the background running, the side panel and options; right-click the button for more.
<br><img src="docs/demo-extensions.webp" width="900" alt="Extensions demo"> &nbsp; [▶ MP4](docs/demo-extensions.mp4)

**2. The real app on Linux** — an extension's content script talks to its background page (the tab title shows the reply and a port echo), the pop-up reads shared storage and the current tab, then Veil's right-click menu in the page and on a tab.
<br><img src="docs/real-app.webp" width="900" alt="Veil running on Linux"> &nbsp; [▶ MP4](docs/real-app.mp4)

**3. Right-click menus** — ‹ Back lists the tab's history, ⟳ Reload and the address bar have their own menus, then a tab (pin it), a bookmark, the tab strip, and Veil's own pages.
<br><img src="docs/demo-right-click.webp" width="900" alt="Right-click menus demo"> &nbsp; [▶ MP4](docs/demo-right-click.mp4)

**4. Address bar and tab search** — colourful suggestions, `@` for any engine, `?` for Veil AI, and ⌄ / ⇧⌘A to search every open tab in every window.
<br><img src="docs/demo-address-bar.webp" width="900" alt="Address bar demo"> &nbsp; [▶ MP4](docs/demo-address-bar.mp4)

**5. Small windows** — as the window narrows, less-used buttons move into ⋯; nothing overlaps and nothing scrolls sideways.
<br><img src="docs/demo-small-window.webp" width="900" alt="Small window demo"> &nbsp; [▶ MP4](docs/demo-small-window.mp4)

**6. One account** — sign in with an email code; Pro appears with its plan, source, end date and computers; subscriptions with Cancel.
<br><img src="docs/demo-account.webp" width="900" alt="Account demo"> &nbsp; [▶ MP4](docs/demo-account.mp4)

**7. Veil AI** — GitHub Copilot sign-in through the browser, then a streamed answer about the page.
<br><img src="docs/demo-ai.webp" width="900" alt="Veil AI demo"> &nbsp; [▶ MP4](docs/demo-ai.mp4)

**8. Plans and the Owner Panel** — monthly and yearly Pro, then computers, subscriptions and codes.
<br><img src="docs/demo-plans-owner.webp" width="900" alt="Plans and Owner Panel demo"> &nbsp; [▶ MP4](docs/demo-plans-owner.mp4)

**9. Owner Panel → Dashboard** — already connected when it opens; a dashboard key can be added instead (sent as `Authorization: Bearer`), stats arrive, every code is ticked with *Select all* and revoked in one go, then the same table in cards, board, timeline, mosaic, magazine and focus layouts.
<br><img src="docs/demo-dashboard.webp" width="900" alt="Dashboard demo"> &nbsp; [▶ MP4](docs/demo-dashboard.mp4)

**10. Real extensions in the real app** — Ghostery's panel and FastestVPN's sign-in open from their toolbar buttons on Linux, each at its own size: nothing cut off, nothing to scroll.
<br><img src="docs/real-extensions.webp" width="900" alt="Ghostery and FastestVPN pop-ups in Veil"> &nbsp; [▶ MP4](docs/real-extensions.mp4)

<a id="screenshots"></a>
## Screenshots

Captured at 2× so the text stays sharp — click any picture for full size.

| Dark | Light |
|---|---|
| <img src="docs/dark-identities.png" alt="Identities, dark"> | <img src="docs/light-identities.png" alt="Identities, light"> |
| <img src="docs/dark-extension-popup.png" alt="Extension pop-up with its badge, dark"> | <img src="docs/light-extension-popup.png" alt="Extension pop-up with its badge, light"> |
| <img src="docs/dark-tab-menu.png" alt="Tab right-click menu, dark"> | <img src="docs/light-tab-menu.png" alt="Tab right-click menu, light"> |
| <img src="docs/dark-address-bar.png" alt="Address bar suggestions, dark"> | <img src="docs/light-address-bar.png" alt="Address bar suggestions, light"> |
| <img src="docs/dark-extensions.png" alt="Extensions, dark"> | <img src="docs/light-extensions.png" alt="Extensions, light"> |
| <img src="docs/dark-settings.png" alt="Settings, dark"> | <img src="docs/light-settings.png" alt="Settings, light"> |
| <img src="docs/dark-plans.png" alt="Plans and account, dark"> | <img src="docs/light-plans.png" alt="Plans and account, light"> |
| <img src="docs/dark-ai.png" alt="AI, dark"> | <img src="docs/light-ai.png" alt="AI, light"> |
| <img src="docs/dark-owner.png" alt="Owner Panel, dark"> | <img src="docs/light-owner.png" alt="Owner Panel, light"> |
| <img src="docs/dark-network.png" alt="Network and Tor, dark"> | <img src="docs/light-network.png" alt="Network and Tor, light"> |

Real app on Linux:

| | |
|---|---|
| <img src="docs/real-suggestions.png" alt="Suggestions in the real app"> | <img src="docs/real-extension.png" alt="An extension running in a Built-in window"> |
| <img src="docs/real-extensions-menu.png" alt="Extensions menu"> | <img src="docs/real-plans-tab.png" alt="Plans in a tab"> |
| <img src="docs/real-page-tools.png" alt="Page tools"> | <img src="docs/real-reader.png" alt="Reader view"> |
| <img src="docs/real-clean-link.png" alt="Clean link and a hidden cookie banner"> | <img src="docs/real-tab-sleeping.png" alt="A sleeping tab woken up"> |
| <img src="docs/real-ghostery.png" alt="Ghostery's pop-up in Veil"> | <img src="docs/real-fastestvpn.png" alt="FastestVPN's pop-up in Veil"> |

<p align="center">⭐ <b>If Veil is useful to you, a star on GitHub means a lot.</b> ⭐</p>

## macOS DMG

`bash build.sh` builds the `.app` and then makes `Veil_<arch>.dmg` with `hdiutil` (`scripts/make-dmg.sh`). Tauri's own DMG step is skipped because its Finder script fails on folders with spaces and on a leftover mounted "Veil" volume.

## Release (creator)

`bash update.sh` does everything in one go: checks `.env` (names only), makes the licence key once, deploys Veil's Cloudflare Worker (records in the ipconfig dashboard's databases `veil-28-oct-2026` and `veil-kv-28-oct-2026`; R2 bucket `veil-11-oct-2026` for downloads), bumps the version, runs every test, builds and signs the Apple silicon app, writes the notes and `latest.json`, pushes the source to the private repo `veil-11-Oct-2026` and this README to the public repo `veil-releases-11-Oct-2026`, uploads the installers to both GitHub repos and to R2 (`latest.json` last), and only then deletes the old installers from GitHub and R2.

`bash update.sh --doctor` only checks; `--service` only deploys the Worker; `--dry-run` builds without publishing; `--publish` uploads what is already built in `release/` again (each upload is retried, and a release counts as done only when GitHub lists every file).

## Build

```bash
bash build.sh          # tests → release build → Mac signing with one stable identity → size check
bash build.sh --e2e    # also runs real Chromium + real store downloads
npm run dev            # UI with demo data in a normal browser (never live endpoints)
node tests/ui.mjs      # Playwright over every view, both themes, under the app's CSP
```

Releases are built on an Apple silicon Mac only.

## Tests

- **Rust** (72 tests, plus 3 end-to-end): extension host (match patterns, block-list rules, secrets, proxy settings, host permissions), each tab's back/forward history, right-click actions from Finder and the Dock, native-messaging frames, dashboard key masking, account licences (signature, computer, app, expiry),
  - The proxy: HTTP rewrite, SOCKS5 with auth, a wrong password gives a 502 with the reason, HTTP upstream with Basic auth, and an extension's proxy taking over with credentials from the extension.
  - ZIP: traversal refused, CRC checked.
  - CRX3: picks the key whose hash matches the id.
  - SHA-256 test vectors, fingerprint determinism, Camoufox env chunking, `user.js` routing, the store file kept when broken, address-bar parsing.
- **End-to-end** (`--ignored`):
  - Real Chromium with the Shield and the local route: cores, screen, languages, timezone (offset −540 for Tokyo), WebGL renderer, GPC, a native-looking `toString`, and traffic counted through the route. Canvas differs between profiles and stays stable within one.
  - Real Chrome Web Store and AMO downloads.
  - Licences: Ed25519 tokens bound to one computer, and a Worker-signed licence verified by the app.
  - Updates: minisign signatures from the real `tauri signer` verify; the source order is GitHub public → private → R2.
  - Extensions: content scripts, styles, locale messages and pop-up pages for the Built-in engine.
- **Worker** (`node worker/test/worker.test.mjs`): the whole service on SQLite with Razorpay and GitHub faked — ipconfig's pages first with Veil's only as the fallback, installs, problem dedupe, the owner gate, codes (unique, one computer each, revoke, lifetime), subscriptions (paid, cancelled, plan change, MRR/ARR), the webhook signature and update files.
- **Page scripts** (`node tests/pagejs.mjs`): the extension runtime on a real page (calls through the title, answers only with the extension's secret, storage events, i18n), fetch and XMLHttpRequest to other sites, proxy credentials, pop-ups sized to their content, the CSP, Veil's right-click menu and the notification prompt.
- **Small windows** (`node tests/fit.mjs`): every section and the toolbar at 7 sizes from 480×320 to 1280×780 — nothing off-window, cut off or overlapping, no sideways scrolling, no page errors.
- **Store** (`node tests/store.mjs`): on stand-in Chrome Web Store and Firefox Add-ons pages, the store sees Chrome / Firefox, the “Switch to Chrome” banner is gone, and the store's own buttons and install API install into Veil.
- **UI**: Playwright with zero page errors (every view, both themes, suggestions that resize with the window, extensions menu, pop-ups sized to their content under their button, the tab menu, pinned tabs, ⇧⌘T, Copilot sign-in, plans, codes, Owner Panel, updates), plus a check that every colour token is at least 4.5:1 with no pink hues.

## Sources studied

Donut Browser (profile and proxy model), Camoufox (config keys), Helium, Thorium, BrowserOS, Floorp, Otter, Beaker, Browserosaurus, Kiwi, DuckDuckGo's apple-browsers, Brave and Tor. Veil runs these engines; it does not copy their code.

---

<p align="center">⭐ <b>Please star Veil on GitHub</b> — it helps more people find a private browser. Thank you!<br>
<a href="https://github.com/JKS-sys/veil-releases-11-Oct-2026">github.com/JKS-sys/veil-releases-11-Oct-2026</a></p>

<!-- licence:start -->
## Licence

Veil is proprietary software — © 2026 Jagadeesh Kumar S, all rights reserved. You may run the published builds on your own computers under your plan; copying, changing or redistributing it needs written permission. See [LICENSE](LICENSE). Third-party components keep their own licences: [THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt).
<!-- licence:end -->
