# RX-msg attribution and redistribution notice

RX-msg is a modified distribution of the open-source [SimpleX Chat](https://github.com/simplex-chat/simplex-chat) project. The current Android preview is based on the SimpleX Chat 7.0.1 codebase.

The original SimpleX Chat source and contributions remain attributed to their respective copyright holders. RX-msg-specific modifications are maintained by Old-Rx. The Git history preserves upstream authorship for inherited files.

The repository's primary license is the [GNU Affero General Public License version 3](LICENSE). Individual dependencies, assets, and vendored components may use their own compatible licenses; their notices remain in the source tree, including `docs/dependencies/licences` and package-level license files.

The Android CI retrieves `simplex-aarch64.apk` from the official SimpleX Chat `v7.0.1` release and verifies this SHA-256 before extracting the required ARM64 native libraries:

```text
d5b37d2eaf92d560181306973f359518b11048e3f69585d56ab5eed53a97f865
```

The corresponding upstream source is available from the [SimpleX Chat v7.0.1 tag](https://github.com/simplex-chat/simplex-chat/tree/v7.0.1). RX-msg does not claim authorship of the SimpleX protocol, native core, or inherited application code.

`SimpleX`, related project names, and upstream artwork may be trademarks or identifiers of their respective owners. RX-msg uses its own product name, package identifier, and launcher artwork. Remaining SimpleX references are retained for protocol compatibility, technical attribution, or accurate description of inherited code.

RX-msg is an independent fork. It is not an official SimpleX Chat build and is not endorsed by the upstream project.
