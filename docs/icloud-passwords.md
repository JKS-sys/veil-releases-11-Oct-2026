# iCloud Passwords in Veil

Apple's **iCloud Passwords** extension talks to a helper that ships with macOS
(`PasswordManagerBrowserExtensionHelper`). Since macOS 15.4 that helper only answers
browsers whose code-signing identifier **and Apple Team ID** are on Apple's list in
[apple/password-manager-resources](https://github.com/apple/password-manager-resources),
file `quirks/web-browser-extension-distribution-information.json` (Orion, Arc, Brave, Chrome,
Edge, Vivaldi… got in that way).

Veil already speaks the extension's native-messaging protocol and finds the helper. What it
still needs is a place on that list, and the list needs a Team ID — which the Developer ID
certificate now gives: `update.sh` signs, notarises and staples Veil with it whenever
`APPLE_SIGN_IDENTITY` (plus a notarisation login) is in `.env` and the certificate is in the
Mac's keychain.

## Once Veil is signed with a Developer ID

1. Release once with the Developer ID (`bash update.sh` — it says "Developer ID release: signed,
   notarised and stapled"). macOS asks for Veil's permissions one more time after that update.
2. Read the Team ID: `codesign -dv /Applications/Veil.app 2>&1 | grep TeamIdentifier`.
3. Open a pull request on apple/password-manager-resources adding this entry to
   `web_browsers` (fill in the Team ID):

```json
{
    "long_name": "Veil",
    "short_name": "Veil",
    "supported_platforms": ["Mac"],
    "platform_specific_information": {
        "Mac": {
            "bundle_identifier": "network.co.ipconfig.veil",
            "code_signing_identifier": "network.co.ipconfig.veil",
            "code_signing_team_identifier": "YOUR_TEAM_ID",
            "extensions_install_path_relative_to_user_library_directory": "Application Support/network.co.ipconfig.veil/extensions"
        }
    },
    "supported_store_identifiers": [1, 3]
}
```

Apple ships the list with macOS updates, so iCloud Passwords starts working in Veil's own
windows after the release that includes it.

## Until then

⋯ → **iCloud Passwords** (or the button in the extension's pop-up) opens your passwords in a
Brave, Chrome or Edge identity — the same iCloud Keychain, one click.

## Passkeys and passwords like Safari (no six-digit code)

The code the iCloud Passwords extension asks for is Apple's pairing step for browser
extensions; Safari never asks because it reads iCloud Keychain itself. A browser that is not
Safari gets the same thing from Apple in two pieces, both tied to an Apple Developer account:

1. **Passkeys** — the *Web Browser Public Key Credential* entitlement
   (`com.apple.developer.web-browser.public-key-credential`). The Account Holder requests it
   with Apple's dedicated form (linked from the entitlement's documentation page). Once Apple
   grants it, make a **Developer ID** provisioning profile for the App ID `network.co.ipconfig.veil`
   with that capability (developer.apple.com → Certificates, IDs & Profiles), put its path in
   `.env` as `VEIL_PROVISION_PROFILE`, and `update.sh` adds the entitlement and embeds the
   profile (`scripts/mac-config.mjs` only uses a profile that names Veil's app id). WebKit then offers iCloud Keychain passkeys on sign-in
   pages without any code. Without the profile the entitlement is left out on purpose — macOS
   refuses to start an app that claims a restricted entitlement it wasn't granted.
2. **Passwords** — Apple's list above (Team ID + signature). On the list, the iCloud Passwords
   extension in Veil pairs once per session like it does in Chrome.

What only Apple can do: grant the passkey entitlement, and merge the list entry. Veil's side is
ready for both. Until the list entry ships in a macOS update, the iCloud Passwords extension
keeps asking for its code (Apple's pairing step) — that is decided by macOS, not by Veil.
