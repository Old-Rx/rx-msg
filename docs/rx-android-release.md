# RX-msg Android release candidates

The `RX Android Release Candidate` workflow builds an ARM64 FOSS release APK and a SHA-256 checksum. It can run without signing credentials, in which case the artifact is clearly named `unsigned`.

For a signed APK, configure all four repository secrets before manually running the workflow:

- `RX_ANDROID_KEYSTORE_BASE64`: the complete release keystore encoded as a single Base64 string.
- `RX_ANDROID_STORE_PASSWORD`: the keystore password.
- `RX_ANDROID_KEY_ALIAS`: the signing key alias.
- `RX_ANDROID_KEY_PASSWORD`: the signing key password.

The workflow rejects partially configured credentials. The decoded keystore exists only in the GitHub-hosted runner's temporary directory and is removed after the build. Keystores are ignored by Git to reduce the risk of committing private signing material.

The application continues to open `simplex:` and compatible HTTPS connection links. It deliberately does not request Android App Link verification for upstream SimpleX domains. Add a future RX-msg domain only after its `/.well-known/assetlinks.json` file includes the RX-msg package name and release signing certificate fingerprint.
