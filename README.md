# ViCa

**Apple WWDC Swift Student Challenge Winner 2025**

---

You're at a hackathon. You meet someone interesting. You want to share your contact.

So you both fumble for your phones, open your contacts app, squint at each other's screens, and manually type in a number — hoping you got the digits right. Or worse, you hand over a paper card that ends up at the bottom of a bag and never gets opened.

**ViCa makes that moment instant.** You just bring two phones close together, and the cards exchange. That's it. No typing, no scanning, no paper. The whole thing takes less than a second and works completely without internet.

---

## The idea

Think of it like AirDrop, but for your identity — and designed so it actually feels like handing someone a card. You create a digital card for yourself (or several, for different situations), hold your phone near someone else's, and the card transfers the moment the phones are close enough to touch. The other person gets your card in their inbox instantly, with all your details exactly as you set them up.

It works offline, stores everything on your device, and never asks you to make an account.

---

## Screenshots

<img width="585" height="1266" alt="IMG_0006" src="https://github.com/user-attachments/assets/39951617-4f6a-4b86-add3-5933020faa54" />


---

## Cards for every context

ViCa lets you build different cards for different situations, so you're always sharing the right version of yourself.

| Card type | Best for |
|---|---|
|  **Personal** | Friends, casual meetups — social handles, WhatsApp, location |
|  **Business** | Professional networking — LinkedIn, email, company, title |
|  **Social** | Creative spaces — Instagram, Snapchat, Spotify, vibe |
|  **Event** | Conferences, hackathons — GitHub, skills, event badge |
|  **Custom** | Build from scratch — any combination of fields |

Every card has five colour themes — rose, lime, sky, lavender, and peach — with a distinctive dot-grid texture that makes each one feel premium in the hand.

---

## How sharing works

There are two ways to share, depending on what's available:

**Nearby (the main way)**
Both people open ViCa, bring their phones close together, and give them a small tap. The app detects the physical bump on both devices at the same time, confirms it was intentional, and transfers the card over a direct peer-to-peer connection — no Wi-Fi, no mobile data, no Bluetooth pairing screens.

**QR code (the fallback)**
If proximity isn't working, the sender shows a QR code on screen and the receiver scans it with their camera. Slower, but always works.

---

## Event mode

At a conference or networking event, one person hosts a session and shares a 6-character code. Everyone who joins exchanges cards with the whole group at once — no one-to-one tapping needed. Received cards get automatically sorted into a folder named after the event, so you always know where you met someone.

---

## Privacy

ViCa was built around one rule: **your data stays on your phone.**

There are no servers, no accounts, no analytics, no tracking of any kind. Cards transfer directly between devices over an encrypted peer-to-peer connection. Nothing is ever uploaded anywhere. If you delete the app, everything is gone — because it was only ever on your device.

---

## Tech stack

```
UI                SwiftUI — custom animations, Canvas dot-grid, parallax scroll
Networking        MultipeerConnectivity — encrypted peer-to-peer transfer
Proximity         NearbyInteraction (UWB) — centimetre-accurate distance, iPhone 11+
Motion            CoreMotion — accelerometer bump detection for tap simulation  
Persistence       CoreData — local-only card and folder storage
QR                AVFoundation — camera scanning / CIFilter generation
Haptics           UIImpactFeedbackGenerator — tactile feedback throughout
```

---

## Architecture

ViCa uses a five-layer networking stack so sharing works reliably across different device capabilities:

```
Layer 1 — Discovery      NearbyInteraction (UWB) + CoreBluetooth RSSI fallback
Layer 2 — Intent         CoreMotion accelerometer spike + mutual timestamp confirmation
Layer 3 — Connection     MultipeerConnectivity session (encryption required)
Layer 4 — Transfer       JSON-encoded CardModel payload (~800 bytes typical)
Layer 5 — Fallback       QR code via AVFoundation
```

The tap interaction works by having both devices independently detect a physical bump via accelerometer, broadcast a timestamped intent packet, and only confirm the exchange if both packets arrive within a 300ms window. This means walking past someone with the app open never accidentally triggers a transfer — both phones have to be tapped together at the same moment.

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

## Colour palette

| Name | Hex | Used for |
|---|---|---|
| `obsidianBlack` | `#0D0D0D` | App background |
| `charcoalGrey` | `#2C2C2C` | Card text, tab bar |
| `softRose` | `#FFB3BA` | Pink card theme, Add tab active |
| `freshLime` | `#C8F59A` | Lime card theme, My Cards tab active |
| `skyBlue` | `#99D6F5` | Sky card theme, Inbox tab active |
| `lavenderPurple` | `#CFB8F5` | Lavender card theme |
| `softTerracotta` | `#E8A898` | Peach card theme |

---

## Requirements

| | |
|---|---|
| Platform | iOS 17.0+ |
| Xcode | 15.0+ |
| Swift | 5.9+ |
| Devices | iPhone |
| UWB precision | iPhone 11 and later |

---

## Roadmap

- [ ] Android — Jetpack Compose + Nearby Connections API
- [ ] Cross-platform sharing — iOS ↔ Android via BLE + NFC
- [ ] NFC tap trigger — true hardware tap on Android
- [ ] Larger event mode — BLE mesh, no device cap
- [ ] B2B team cards — company-branded templates, admin dashboard
- [ ] Optional cloud sync — user-controlled, opt-in only

---


*Built by Sakshi Beloshe · Swift Student Challenge 2025 winner*
