# RX-msg Android releases

The `RX Android Release Candidate` workflow builds a signed ARM64 FOSS release APK and a SHA-256 checksum. Release builds fail closed when signing credentials are missing or incomplete.

Configure all four repository secrets before running the workflow:

- `RX_ANDROID_KEYSTORE_BASE64`: the complete release keystore encoded as a single Base64 string.
- `RX_ANDROID_STORE_PASSWORD`: the keystore password.
- `RX_ANDROID_KEY_ALIAS`: the signing key alias.
- `RX_ANDROID_KEY_PASSWORD`: the signing key password.

The workflow verifies the package name, app label, version, ARM64-only native libraries, APK signature, and pinned RX-msg signing certificate. The decoded keystore exists only in the GitHub-hosted runner's temporary directory and is removed after the build. Keystores are ignored by Git to reduce the risk of committing private signing material.

Manual runs accept an optional `release_tag`. A value matching `vMAJOR.MINOR.PATCH-rx.NUMBER`, such as `v7.0.1-rx.1`, creates a draft prerelease and attaches the signed APK and checksum. Leave the input empty to build only the Actions artifact. Draft releases must remain unpublished until installation, startup, profile creation, and messaging smoke tests pass on a physical ARM64 Android device.

The application continues to open `simplex:` and compatible HTTPS connection links. It deliberately does not request Android App Link verification for upstream SimpleX domains. Add a future RX-msg domain only after its `/.well-known/assetlinks.json` file includes the RX-msg package name and release signing certificate fingerprint.
