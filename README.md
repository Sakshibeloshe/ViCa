# ViCa — Digital Identity, Redefined

> Bring phones together. Cards exchange instantly. No typing. No internet. No paper.

ViCa is a privacy-first, proximity-based digital identity sharing app built for iOS. It replaces traditional business cards with a fast, accessible, offline-first system that works by simply bringing two devices close together.

**Apple WWDC Swift Student Challenge Winner 2025**

---

## What it does

You create digital identity cards for different contexts — personal, business, social, event, or custom — and share them instantly when two devices are brought close together. The receiving device feels the exchange like a physical tap. No accounts required. No cloud. No internet.

Cards you receive land in your inbox, where you can organise them into folders, mark favourites, search, and filter by card type.

---

## Core features

| Feature | Description |
|---|---|
| **Multi-context cards** | Personal, Business, Social, Event, and Custom card types with context-specific fields |
| **Proximity sharing** | Bring devices close → cards transfer instantly over peer-to-peer |
| **Receiver mode** | Intentional sharing — receiver opts in before the transfer begins |
| **Event mode** | Host a group session with a 6-character code; all attendees exchange cards simultaneously |
| **QR fallback** | Generate and scan QR codes when proximity transfer isn't available |
| **Folders + inbox** | Organise received cards into named folders with colour coding |
| **Profile sync** | Cards linked to your profile update automatically when your details change |
| **Fully offline** | All data stored locally on device — no server, no account, no tracking |

---

## Tech stack

```
UI layer          SwiftUI — declarative layouts, custom animations, Canvas rendering
Networking        MultipeerConnectivity — peer-to-peer card transfer
Proximity         NearbyInteraction (UWB) — centimetre-accurate distance on iPhone 11+
Motion            CoreMotion — accelerometer bump detection for tap simulation
Persistence       CoreData — local-only card and folder storage
Profile           @AppStorage — lightweight key-value profile persistence
QR               AVFoundation — camera scanning / CIFilter generation
Haptics           UIImpactFeedbackGenerator — tactile feedback throughout
```

---

## Architecture

ViCa uses a five-layer networking stack:

```
Layer 1 — Discovery      NearbyInteraction (UWB) + CoreBluetooth RSSI fallback
Layer 2 — Intent         CoreMotion accelerometer spike + mutual timestamp confirmation
Layer 3 — Connection     MultipeerConnectivity session (encryption required)
Layer 4 — Transfer       JSON-encoded CardModel payload (~800 bytes typical)
Layer 5 — Fallback       QR code via AVFoundation
```

The tap interaction is not simulated through the OS — both devices independently detect a physical bump via accelerometer, broadcast a timestamped intent packet, and only confirm the tap if both packets arrive within a 300ms window. This eliminates false positives from walking past someone with the app open.

---

## Project structure

```
ViCa/
├── App/
│   ├── CardStackApp.swift          Entry point, environment object injection
│   └── RootTabView.swift           Root navigation, floating tab bar host
├── Models/
│   ├── CardModel.swift             Core data model — Codable, Sendable
│   ├── CardType.swift              5 card types with icons and emoji
│   ├── CardTheme.swift             5 colour themes mapped to app palette
│   └── FolderModel.swift           Folder with name and colour
├── Screens/
│   ├── MyCards/
│   │   ├── MyCardsView.swift       Card stack with parallax scroll
│   │   ├── ShareCardView.swift     Nearby + QR share flow
│   │   └── TouchModeView.swift     Proximity sharing UI state
│   ├── AddCard/
│   │   ├── AddCardView.swift       Template selection
│   │   ├── CardEditorView.swift    Field-by-field card builder
│   │   └── CardPreviewView.swift   Live preview during editing
│   ├── Inbox/
│   │   ├── InboxView.swift         Mixed folder + card list
│   │   └── FolderPickerSheet.swift Assign card to folder
│   ├── Managers/
│   │   ├── PeerManager.swift       MultipeerConnectivity wrapper
│   │   ├── EventModeManager.swift  Event session state
│   │   └── EventPeerManager.swift  Group session peer management
│   ├── CardDetail/
│   │   └── CardDetailView.swift    Full card detail with actions
│   ├── Onboarding/
│   │   └── OnboardingView.swift    8-page first-run flow
│   └── Settings/
│       └── SettingsView.swift      Profile and preferences
├── Components/
│   ├── CardFrontView.swift         Card face — name, photo, links
│   ├── PremiumCardPattern.swift    Dot grid + highlight background
│   ├── FloatingTabBar.swift        Custom capsule tab bar
│   ├── ShimmerModifier.swift       Selected card shimmer effect
│   └── ...                         Filter pills, search bar, buttons
├── Data/
│   ├── AppStore.swift              Central ObservableObject store
│   ├── Color+AppPalette.swift      Named colour palette
│   └── DummyData.swift             Sample cards for preview
├── Persistence/
│   ├── PersistenceController.swift CoreData stack
│   ├── CardEntities.swift          CDCard NSManagedObject
│   ├── CardRepository.swift        Fetch/save abstraction
│   └── ProfileStore.swift          @AppStorage profile wrapper
├── DynamicFields/
│   ├── FieldDefinition.swift       Per-field metadata
│   ├── FieldKind.swift             Field type enum
│   └── FieldCatalog.swift          Field sets per card type
└── Utilities/
    ├── QRCodeGenerator.swift       Encode/decode CardModel ↔ QR
    ├── QRCodeScannerView.swift     AVFoundation camera view
    ├── Haptics.swift               Haptic feedback wrapper
    └── Extensions.swift            View and type extensions
```

---

## Card model

Every card is a `CardModel` — a lightweight, `Codable`, `Sendable` struct that serialises to ~800 bytes of JSON for transfer.

```swift
struct CardModel: Identifiable, Hashable, Codable, Sendable {
    let id: UUID
    var type: CardType          // .personal .business .social .event .blank
    var theme: CardTheme        // .pink .lime .sky .lavender .peach
    var fullName: String
    var title: String
    var company: String
    var email: String?
    var phone: String?
    var linkedin: String?
    var instagram: String?
    var github: String?
    // ... 15+ optional social/contact fields
    var isReceived: Bool        // false = my card, true = inbox card
    var isFavorite: Bool
    var folderId: UUID?
    var eventName: String?
    var createdAt: Date
}
```

The same JSON schema is used for transfer, QR encoding, and local persistence — no transformation layer needed.

---

## Sharing flow

```
Both devices open ViCa
        ↓
MultipeerConnectivity advertises + browses simultaneously
        ↓
Devices come within range (~30cm RSSI gate)
        ↓
User brings phones together — both accelerometers spike > 0.8g
        ↓
Each device broadcasts timestamped BUMP packet
        ↓
Delta < 300ms on both sides → tap confirmed
        ↓
Sender serialises CardModel → JSON → sends via MCSession
        ↓
Receiver decodes, saves to CoreData inbox
        ↓
Toast notification + haptic confirmation
```

---

## Event mode

Event mode hosts a shared session where any number of attendees can exchange cards simultaneously.

1. Host creates a session — sets event name, folder name, selects a card, gets a 6-character code
2. Attendees enter the code to join
3. All connected devices exchange cards automatically
4. Cards are saved to the inbox and auto-assigned to the event folder
5. The event banner shows live peer count and received card count

The session code uses an unambiguous character set (`ABCDEFGHJKLMNPQRSTUVWXYZ23456789`) — no `0/O/1/I` to avoid confusion when reading aloud.

---

## Colour palette

| Name | Use | Hex |
|---|---|---|
| `obsidianBlack` | App background | `#0D0D0D` |
| `charcoalGrey` | Card text, tab bar | `#2C2C2C` |
| `softRose` | Pink card theme, Add tab | `#FFB3BA` |
| `freshLime` | Lime card theme, My Cards tab | `#C8F59A` |
| `skyBlue` | Sky card theme, Inbox tab | `#99D6F5` |
| `lavenderPurple` | Lavender card theme | `#CFB8F5` |
| `softTerracotta` | Peach card theme | `#E8A898` |

---

## Privacy

- No user accounts
- No network requests — ever
- No analytics or tracking
- All card data stored on-device via CoreData
- Peer-to-peer transfer only — data never touches a server
- Sharing is always intentional — receiver must opt in

---

## Requirements

| | |
|---|---|
| Platform | iOS 17.0+ |
| Xcode | 15.0+ |
| Swift | 5.9+ |
| Devices | iPhone (iPad layout not optimised) |
| UWB precision | iPhone 11 and later |

---

## Roadmap

- [ ] Android version — Jetpack Compose + Nearby Connections API
- [ ] Cross-platform sharing — iOS ↔ Android via BLE + NFC
- [ ] NFC tap trigger — hardware tap on Android-to-Android
- [ ] Larger event mode — BLE mesh, no device cap
- [ ] B2B team cards — company-branded templates, admin dashboard
- [ ] Optional cloud sync — user-controlled, opt-in only

---

## License

MIT License — see `LICENSE` for details.

---

*Built by Sakshi Beloshe · Swift Student Challenge 2025*
