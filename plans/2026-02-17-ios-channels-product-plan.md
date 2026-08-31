# Channels on iOS — Product Plan

## Contents
1. [Overview](#1-overview)
2. [Screens](#2-screens)
   - 2.1 [Chat List](#21-chat-list)
   - 2.2 [Channel Messages & Compose](#22-channel-messages--compose)
   - 2.3 [Channel Creation](#23-channel-creation)
   - 2.4 [Channel Info](#24-channel-info)
   - 2.5 [Chat Relay Management (Network & Servers)](#25-chat-relay-management-network--servers)
   - 2.6 [Joining a Channel](#26-joining-a-channel)
3. [Implementation Order](#3-implementation-order)

---

## 1. Overview

### What
Channels are one-to-many broadcast groups where messages flow **owner → chat relays → subscribers**. Unlike regular groups (N-to-N connections), channels use chat relay infrastructure to scale delivery — an owner sends once, chat relays fan out to all subscribers.

Technically, a channel is a group with `useRelays = true`. All subscribers are observers (read-only). The owner posts as the channel identity.

### Why
Regular SimpleX groups require direct connections between all members. While there is no hard technical limit, in practice large groups of even several hundred members become very inefficient — group state desynchronizes, delivery becomes inefficient and unreliable, and the experience degrades. Channels solve the broadcast use case: organizations, projects, and individuals publishing to large audiences while preserving SimpleX's privacy model (no user identifiers, relay-mediated delivery).

### For Whom

**Channel owners** — creators who want to broadcast to a large audience. They create channels, configure chat relays, post content. Their problem: no way to efficiently reach many people on SimpleX because large groups work badly in practice.

**Channel subscribers** — readers who want to follow public content. They join via link and receive messages through chat relays. Their problem: can't follow public channels/announcements on SimpleX.

---

## 2. Screens

### 2.1 Chat List

New icon (`antenna.radiowaves.left.and.right`) to differentiate channels.

```
┌────────────────────────────────────────┐
│ [👥]  Team Chat            3:42 PM     │
│        alice: Hey everyone...   ●  1   │
├────────────────────────────────────────┤
│ [📡]  SimpleX News         3:38 PM     │
│        Latest update about...   ●  3   │
├────────────────────────────────────────┤
│ [👤]  Bob                  2:15 PM     │
│        See you tomorrow          ✓✓    │
└────────────────────────────────────────┘
```

Chat header uses channel icon when no profile image, same as groups:

```
┌────────────────────────────────────────┐
│  < [📡] SimpleX News             ···   │
└────────────────────────────────────────┘
```

---

### 2.2 Channel Messages & Compose

Messages render with channel avatar + channel name as sender (via existing `showGroupAsSender` path). Consecutive messages group without repeating avatar/name.

**Subscriber view** — compose disabled with "you are subscriber" label (vs. "you are observer" in groups):

```
┌────────────────────────────────────────┐
│  < [📡] SimpleX News             ···   │
├────────────────────────────────────────┤
│                                        │
│  [📡]  SimpleX News                    │
│  ┌──────────────────────────────────┐  │
│  │ We're excited to announce v7.0!  │  │
│  │ New channel feature allows...    │  │
│  │                        3:42 PM   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ Check out the blog post:         │  │
│  │ simplex.chat/blog/v7             │  │
│  │                        3:45 PM   │  │
│  └──────────────────────────────────┘  │
│                                        │
├────────────────────────────────────────┤
│   you are subscriber                   │
└────────────────────────────────────────┘
```

**Owner view** — compose field shows "Broadcast" placeholder. Always sends `asGroup=true` (MVP). Backend also supports sending "as member" (like in regular groups), but this will not be available in MVP UI.

```
├────────────────────────────────────────┤
│     ┌───────────────────────────────┐  │
│  📎 │ Broadcast                   ➤ │  │
│     └───────────────────────────────┘  │
└────────────────────────────────────────┘
```

**Note**: If all chat relays are removed or stop serving the channel, this won't be visible in the UI in MVP.

---

### 2.3 Channel Creation

Entry point: "Create channel" in New Chat menu, after "Create group".

```
┌────────────────────────────────────────┐
│  New message                           │
├────────────────────────────────────────┤
│  🔗  Create 1-time link             >  │
│  📷  Scan / Paste link              >  │
│  👥  Create group                   >  │
│  📡  Create channel                 >  │
├────────────────────────────────────────┤
│  📦  Archived contacts              >  │
└────────────────────────────────────────┘
```

#### Step 1 — Channel profile

```
┌────────────────────────────────────────┐
│  Cancel       Create channel           │
├────────────────────────────────────────┤
│               [  📷  ]                 │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ Enter channel name...            │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Configure relays...                >  │
│                                        │
│  Your profile will be shared with      │
│  chat relays and subscribers.          │
│  Random relays will be selected from   │
│  the list of enabled chat relays.      │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │        Create channel            │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

"Configure relays..." opens Network & Servers view (full settings view) where the user can enable/disable chat relays globally.

There is no explicit relay selection — the app randomly selects from enabled chat relays, same as for SMP/XFTP servers.

> **API note**: Currently `apiNewPublicGroup` takes an explicit list of chat relay IDs. Either the API should be reworked to select relays automatically (consistent with SMP/XFTP server selection), or the UI should randomly select from enabled relays and pass the IDs.

"Create channel" disabled when name is invalid or no relays enabled.

#### Step 2 — Relay connection progress

After tapping "Create channel", chat relays are selected automatically and `apiNewPublicGroup` sends relay invitations. Progress shown as a progress bar with label.

```
┌────────────────────────────────────────┐
│            Creating channel...         │
├────────────────────────────────────────┤
│             [  📷  ]                   │
│          SimpleX News                  │
│                                        │
│  [████████████░░░░░░░░░░░░░░░░░░░░░]   │
│  1/3 relays connected                  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │        Channel link              │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

Tap progress label to expand relay list:

```
│  [████████████░░░░░░░░░░░░░░░░░░░░░]   │
│  ▼ 1/3 relays connected                │
│    relay1.simplex.im        ✓ Active   │
│    relay2.simplex.im      Connecting   │
│    relay3.simplex.im      Connecting   │
```

"Channel link" button enabled when ≥1 relay is active. If tapped while relays are still connecting, warning alert: "Not all relays have connected yet. Channel will start working with N relays. Proceed?" — Proceed / Wait.

#### Step 3 — Channel link

Shown after tapping "Channel link" or auto-transition when all relays active. Standard `GroupLinkView` with QR code + share (same as group creation).

```
┌────────────────────────────────────────┐
│  Back        Channel link     Continue │
├────────────────────────────────────────┤
│                                        │
│  ┌──────────────────────────────────┐  │
│  │                                  │  │
│  │          [ QR CODE ]             │  │
│  │                                  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ https://simplex.chat/...         │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ^ Share link                          │
└────────────────────────────────────────┘
```

#### Failure modes (inline on Step 2)

- **API call fails** (sync — relay invitation send failed): Alert "Error creating channel" + error detail. Retry / Cancel.
- **Partial relay error** (async — some relays don't connect): Progress shows "2/3 relays connected, 1 failed". Expanded view: failed relay with red ● Error. "Channel link" enabled — channel works with fewer relays.
- **All relays error** (async): Progress shows "0/3 relays connected, 3 failed" in red. Alert with Retry / Cancel.

---

### 2.4 Channel Info

Extends `GroupChatInfoView` with conditional sections for `useRelays = true`.

**Design rationale:** Owners/subscribers lists live in a sub-view (not inline) to match patterns familiar from other messengers and reduce main info screen clutter.

#### Owner view

```
┌────────────────────────────────────────┐
│  Done       SimpleX News        Edit   │
├────────────────────────────────────────┤
│             [📡 avatar]                │
│            SimpleX News                │
│                                        │
│  Set chat name...                      │
├────────────────────────────────────────┤
│     🔍 Search     │     🔇 Mute        │
├────────────────────────────────────────┤
│  Channel link                       >  │
│  Owners & subscribers               >  │
├────────────────────────────────────────┤
│  Edit channel profile               >  │
│  Welcome message                    >  │
├────────────────────────────────────────┤
│  Chat theme                         >  │
│  Delete messages after              >  │
├────────────────────────────────────────┤
│  Chat relays                        >  │
│  Clear chat                            │
│  Delete channel                        │
└────────────────────────────────────────┘
```

No "Leave channel" for single (last) owner.

Post-MVP: "Chats with subscribers" navigation link in section 1 for subscriber support.

TBC: share link button in action buttons row.

#### Subscriber view

```
┌────────────────────────────────────────┐
│  Done       SimpleX News               │
├────────────────────────────────────────┤
│             [📡 avatar]                │
│            SimpleX News                │
│                                        │
│  Set chat name...                      │
├────────────────────────────────────────┤
│     🔍 Search     │     🔇 Mute        │
├────────────────────────────────────────┤
│  Channel link                       >  │
│  Owners                             >  │
├────────────────────────────────────────┤
│  Welcome message                    >  │
├────────────────────────────────────────┤
│  Chat theme                         >  │
│  Delete messages after              >  │
├────────────────────────────────────────┤
│  Chat relays                        >  │
│  Clear chat                            │
│  Leave channel                         │
└────────────────────────────────────────┘
```

Differences from owner view:
- **Owners & subscribers**: replaced with **Owners**
- **Edit channel profile**: hidden
- **Delete channel**: replaced with **Leave channel**

#### Owners & subscribers sub-view

Separate sub-view following familiar channel UI patterns from other messengers to increase adoption.

**Owner's view** ("Owners & subscribers"):

```
┌────────────────────────────────────────┐
│  < Back    Owners & subscribers        │
├────────────────────────────────────────┤
│  OWNERS                                │
│  alice (you)                        >  │
├────────────────────────────────────────┤
│  150 SUBSCRIBERS                       │
│  bob                                >  │
│  charlie                            >  │
│  ...                                   │
└────────────────────────────────────────┘
```

**Subscriber's view** ("Owners"):

```
┌────────────────────────────────────────┐
│  < Back           Owners               │
├────────────────────────────────────────┤
│  OWNERS                                │
│  alice                              >  │
└────────────────────────────────────────┘
```

> **Protocol note**: Correct subscriber and owner lists with counts must be implemented for MVP. This requires protocol changes to support relay-reported subscriber counts and subscriber list synchronization. See launch plan §3.3.

#### Chat relays sub-view

```
┌────────────────────────────────────────┐
│  < Back       Chat relays              │
├────────────────────────────────────────┤
│  relay1.simplex.im        ● Active     │
│  relay2.simplex.im        ● Active     │
│  relay3.simplex.im        ● Active     │
│                                        │
│  Chat relays forward messages to       │
│  channel subscribers.                  │
└────────────────────────────────────────┘
```

Read-only for MVP. In future, owner will be able to manage (add, remove) relays from this view.

Relay statuses differ by role:
- **Owner**: based on `RelayStatus` — New, Invited, Accepted, Active
- **Subscriber**: based on connection state — Connecting, Connected, Error (TBC: new type or inferred from connection status)

---

### 2.5 Chat Relay Management (Network & Servers)

Chat relays follow the same placement pattern as SMP/XFTP servers: preset relays appear inside each operator page, custom relays appear in "Your servers" page.

#### Operator page (e.g. SimpleX Chat)

New "Chat relays" section added after "Operator" section, before message and file server sections:

```
┌────────────────────────────────────────┐
│  < Back     SimpleX Chat servers       │
├────────────────────────────────────────┤
│  OPERATOR                              │
│  ...                                   │
├────────────────────────────────────────┤
│  CHAT RELAYS                           │
│  relay1.simplex.im                 ✓   │
│  relay2.simplex.im                 ✓   │
│  relay3.simplex.im                 ✓   │
│                                        │
│  Chat relays forward messages in       │
│  channels you create.                  │
├────────────────────────────────────────┤
│  (message server sections)             │
│  (file server sections)                │
├────────────────────────────────────────┤
│  Test servers                          │
└────────────────────────────────────────┘
```

#### Your servers page

New "Chat relays" section before "Message servers":

```
┌────────────────────────────────────────┐
│  < Back       Your servers             │
├────────────────────────────────────────┤
│  CHAT RELAYS                           │
│  myrelay.example.com               ✗   │
│                                        │
│  Chat relays forward messages in       │
│  channels you create.                  │
├────────────────────────────────────────┤
│  MESSAGE SERVERS                       │
│  ...                                   │
├────────────────────────────────────────┤
│  MEDIA & FILE SERVERS                  │
│  ...                                   │
├────────────────────────────────────────┤
│  Add server...                         │
│  Test servers                          │
│  How to use your servers            >  │
└────────────────────────────────────────┘
```

#### Relay detail view

Follows `ProtocolServerView` pattern. Preset: read-only address + test + enable toggle. Custom: editable address + test + enable + delete. TBC editable name (present in backend).

```
┌────────────────────────────────────────┐
│  < Back      relay1.simplex.im         │
├────────────────────────────────────────┤
│  RELAY ADDRESS                         │
│  ┌──────────────────────────────────┐  │
│  │ https://relay1.simplex.im/...    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Test relay                    ✓       │
│  Use for new channels          [ON]    │
├────────────────────────────────────────┤
│  Delete relay                          │
└────────────────────────────────────────┘
```

If all relays are disabled: footer warning "No chat relays enabled. Channels require at least one relay."

---

### 2.6 Joining a Channel

User taps channel link → pre-join view.

#### Pre-join

```
┌────────────────────────────────────────┐
│  < [📡] SimpleX News            ···    │
├────────────────────────────────────────┤
│                                        │
│             [📡 avatar]                │
│            SimpleX News                │
│                                        │
│  3 relays                           ▶  │
│  ┌──────────────────────────────────┐  │
│  │          Join channel            │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

Relay count visible (from link data). Tapping "3 relays" expands to show relay hostnames.

**Why:** Subscriber can decide whether to join based on which relays are used.

#### Connecting

After "Join channel", relay connections proceed. Progress bar shown above "you are subscriber" — channel already functions with even a single relay connected.

```
┌────────────────────────────────────────┐
│  < [📡] SimpleX News            ···    │
├────────────────────────────────────────┤
│                                        │
│  (chat area — welcome message etc.)    │
│                                        │
├────────────────────────────────────────┤
│  [████████████░░░░░░░░░░░░░░░░░░░░░]   │
│  Connecting... 1/3 relays              │
├────────────────────────────────────────┤
│  you are subscriber                    │
└────────────────────────────────────────┘
```

Tap progress label to expand:

```
├────────────────────────────────────────┤
│  [████████████░░░░░░░░░░░░░░░░░░░░░]   │
│  ▼ Connecting... 1/3 relays            │
│    relay1.simplex.im      ✓ Connected  │
│    relay2.simplex.im      Connecting   │
│    relay3.simplex.im      Connecting   │
├────────────────────────────────────────┤
│  you are subscriber                    │
└────────────────────────────────────────┘
```

All connected → progress bar disappears.

#### Failure modes (inline)
- **Sync failure** (all relays fail on connect call): Alert "Failed to join channel" + Retry / Cancel.
- **Partial failure**: "2/3 relays connected, 1 failed". Channel works. Expanded view shows failed relay with red indicator.
- **All relays fail async**: Red error bar "Channel not connected". TBC: programmatic retry, or only failure indication.

---

## 3. Implementation Order

| # | Screen | Backend Dependency | Complexity |
|---|--------|--------------------|------------|
| 1 | Chat List — channel icon | None | Low |
| 2 | Channel Messages — `CIChannelRcv` rendering | None | Low |
| 3 | Owner Compose — "Broadcast" placeholder + `asGroup` | None | Low |
| 4 | Channel Info — extended `GroupChatInfoView` | Subscriber/owner lists: protocol changes (§3.3) | Medium |
| 5 | Chat Relay Management — Network & Servers | `APITestChatRelay` (launch plan §2.5) | Medium |
| 6 | Channel Creation — 3-step flow | Relay state events (launch plan §3.2) | High |
| 7 | Join Channel — progress bar + relay states | Relay state events (launch plan §3.2) | Medium |

Items 1–3 have no backend blockers and can start immediately. Item 4 requires protocol changes for subscriber/owner lists and counts. Items 5–7 depend on backend work.
