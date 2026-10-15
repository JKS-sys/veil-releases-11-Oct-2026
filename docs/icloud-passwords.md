# iCloud Passwords in Veil

Apple's **iCloud Passwords** extension talks to a helper that ships with macOS
(`PasswordManagerBrowserExtensionHelper`). Since macOS 15.4 that helper only answers
browsers whose code-signing identifier **and Apple Team ID** are on Apple's list in
[apple/password-manager-resources](https://github.com/apple/password-manager-resources),
file `quirks/web-browser-extension-distribution-information.json` (Orion, Arc, Brave, Chrome,
Edge, Vivaldi… got in that way).

Veil already speaks the extension's native-messaging protocol and finds the helper. What it
still needs is a place on that list, and the list needs a Team ID, which only an
**Apple Developer ID** certificate gives (the `APPLE_*` keys in `.env` are empty today).

## Once Veil is signed with a Developer ID

1. Sign and notarise the macOS build (`npm run apple-sign`, run by `update.sh`).
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
