# RX-msg

[![Android baseline](https://github.com/Old-Rx/rx-msg/actions/workflows/rx-android-baseline.yml/badge.svg)](https://github.com/Old-Rx/rx-msg/actions/workflows/rx-android-baseline.yml)
[![Android release candidate](https://github.com/Old-Rx/rx-msg/actions/workflows/rx-android-release.yml/badge.svg)](https://github.com/Old-Rx/rx-msg/actions/workflows/rx-android-release.yml)

RX-msg is an independent, Android-focused fork of [SimpleX Chat](https://github.com/simplex-chat/simplex-chat). It retains the SimpleX protocol and native messaging core while using an independent Android application identity.

## Current status

RX-msg is an early preview and is not yet a general-availability release.

- Android package: `com.oldrx.rxmsg`
- Application name: `RX-msg`
- Current codebase version: SimpleX Chat 7.0.1 with RX-msg changes
- Current APK target: FOSS, ARM64 (`arm64-v8a`)
- Distribution state: signed draft prerelease pending physical-device smoke tests

The fork has not received an independent security audit. Upstream SimpleX security reviews and documentation are useful technical references, but they must not be interpreted as an audit or endorsement of RX-msg.

## RX-msg changes

- Independent Android package, provider authority, application name, and launcher artwork
- RX-msg product-facing copy and removal of upstream store, fundraising, and support destinations from the app
- Reproducible CI preparation of pinned native ARM64 libraries from the official SimpleX 7.0.1 Android artifact
- Independent Android release-signing identity
- Release gates for package metadata, version, native ABI, required native libraries, signing certificate, and checksum

See [NOTICE.md](NOTICE.md) for upstream attribution and redistribution information.

## Android builds

The baseline workflow builds an ARM64 FOSS debug APK. The release workflow builds only signed ARM64 FOSS release candidates and fails closed if signing credentials are absent or incomplete.

- [Android baseline workflow](https://github.com/Old-Rx/rx-msg/actions/workflows/rx-android-baseline.yml)
- [Android release workflow](https://github.com/Old-Rx/rx-msg/actions/workflows/rx-android-release.yml)
- [Release and signing documentation](docs/rx-android-release.md)

Release candidates must pass installation, startup, profile creation, and messaging tests on a physical ARM64 Android device before a draft is published.

## Source and license

RX-msg is distributed under the [GNU Affero General Public License version 3](LICENSE), subject to the separate licenses of bundled dependencies. The complete modified source is maintained in this repository.

Protocol names, internal namespaces, link schemes, and compatibility references may continue to use `SimpleX` where changing them would break interoperability or misrepresent the underlying protocol.

RX-msg is maintained independently by Old-Rx. It is not an official SimpleX Chat release and does not imply endorsement by the upstream project.

## Development

The repository retains the upstream multiplatform and server source tree. Current RX-msg work is focused on the Android application under `apps/multiplatform` and the dedicated workflows under `.github/workflows`.

When reporting a problem, include the commit SHA, Android version, device architecture, and the workflow or APK variant used. Do not publish private message content, invitation links, signing material, or database files in an issue.
