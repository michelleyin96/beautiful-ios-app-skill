# Beautiful iOS Apps — SwiftUI Design System & Architecture

> **Purpose**: This document captures the transferable, reusable patterns for building an iOS app with this stack. Don't follow it prescriptively — adapt it suit your app needs, design system and architecture. Claude code should be able to modify from this spec.
>
> **Stack**:
> - SwiftUI-first for all views. UIKit only for complex/legacy layouts or when SwiftUI has known limitations.
> - `@Observable` + `@Environment` state management. Sheet-based navigation.
> - iOS `{{DeploymentTarget}}` deployment target.
> - Never use raw values — always semantic tokens.
> - Haptics on every tappable action.
> - Never `print()` — use a structured logger (e.g., `os.Logger` or a custom `Log` enum).
>
> **How to use it**: Replace every `{{placeholder}}` with your project's values. Everything else applies as-is. The goal is that Claude never writes a raw hex color, a raw font size, or a raw spacing number — everything goes through your design tokens.

---

## 0. Quick-Start Checklist

Before writing any code, replace these placeholders:

| Placeholder | What It Is | Example |
|---|---|---|
| `{{AppName}}` | Your app's name | Vino |
| `{{BundleID}}` | Bundle identifier | com.yourco.vino |
| `{{DeploymentTarget}}` | Minimum iOS version | 18.0 |
| `{{BodyFontName}}` | Body/UI font PostScript prefix | BananaGrotesk |
| `{{DisplayFontName}}` | Heading/display font PostScript prefix | PlayfairDisplaySC |
| `{{BrandPrimary}}` | Primary brand hex | #6D0000 |
| `{{BrandAccent}}` | Accent brand hex | #810100 |
| `{{ColorScheme}}` | Forced color scheme | .light |

---

## 0b. Build & Run Verification

> After implementing any feature, bug fix, or UI change, run `./scripts/run.sh`. If it fails, fix compile errors before the task is done. Do not skip this step.

**Scaffolding**: If `./scripts/run.sh` does not exist, create it before your first build. The script must:
1. Build the app with `xcodebuild`
2. Launch on the iOS Simulator

**Reference `run.sh` template** (adapt `SCHEME` and `SIMULATOR` to your project):

```bash
#!/bin/bash
set -euo pipefail

SCHEME="{{AppName}}"
SIMULATOR="iPhone 16 Pro"
DESTINATION="platform=iOS Simulator,name=$SIMULATOR"

# Boot simulator if needed
xcrun simctl boot "$SIMULATOR" 2>/dev/null || true

# Build
xcodebuild -scheme "$SCHEME" \
  -destination "$DESTINATION" \
  -derivedDataPath .build \
  build | xcpretty || exit 1

# Install & launch
APP_PATH=$(find .build -name "*.app" -path "*/Debug-iphonesimulator/*" | head -1)
xcrun simctl install booted "$APP_PATH"
BUNDLE_ID=$(defaults read "$APP_PATH/Info.plist" CFBundleIdentifier)
xcrun simctl launch booted "$BUNDLE_ID"

echo "✅ Running $SCHEME on $SIMULATOR"
```

**Note for CLAUDE.md users**: Add this to your project's `CLAUDE.md` so Claude Code picks it up automatically:

> `After implementing any feature or fix, run ./scripts/run.sh to build and launch on the simulator. Do not skip this step.`

---

## 1. Philosophy

- **SwiftUI-first** for all views. No UIKit unless SwiftUI has a known limitation.
- **Never use raw values** — all colors, fonts, and spacing go through semantic tokens.
- **Haptic feedback on every tappable element.** No silent taps.
- **Cache-first loading.** Show cached data instantly, refresh in background.
- **Optimistic local state.** Update UI immediately, persist async.
- **Sheet-based navigation.** No `NavigationLink` push navigation. Sheets + `@Environment(\.dismiss)`.
- **Pick one color scheme and force it** at the app root: `.preferredColorScheme({{ColorScheme}})`. Supporting both light and dark mode is a significant design undertaking — your brand colors, semantic tokens, and component styles all need conditional variants. Ship with one mode first. You can always add the other later once the design system is mature.
- **Never `print()`** — use a structured logger (e.g., `os.Logger` or a custom `Log` enum).
- **Kingfisher** for all remote images. Never raw `AsyncImage`.

---

## 2. Sample Project Structure

```
{{AppName}}/
├── {{AppName}}App.swift             # @main entry, environment injection
├── Auth/
│   ├── AuthManager.swift            # @Observable auth state machine
│   └── LoginView.swift              # Sign-in UI
├── Extensions/
│   ├── ColorExtension.swift         # Color tokens + hex init
│   ├── FontExtension.swift          # Font families + semantic type scale
│   └── SpacingExtension.swift       # Spacing tokens
├── Components/
│   ├── ButtonView.swift             # Primary reusable button
│   ├── ProfileImageView.swift       # Remote avatar with Kingfisher
│   ├── InputFieldView.swift         # Text input with send button
│   ├── NavBarView.swift             # Custom navigation bar
│   ├── ToastView.swift              # Toast notification modifier
│   ├── LoadingOverlay.swift         # Full-screen branded loading
│   └── EmptyStateView.swift         # Empty state pattern
├── Fonts/                           # Custom font files (.otf/.ttf)
├── Networking/
│   ├── APIConfig.swift              # URLs, keys, endpoint builders
│   ├── NetworkClient.swift          # Raw HTTP for storage/edge functions
│   └── SupabaseClient.swift         # Global `supabase` constant
├── Services/
│   ├── Analytics.swift              # Static analytics wrapper
│   └── FileCache.swift              # Generic Codable file cache
├── Views/
│   ├── RootView.swift               # Auth state router
│   ├── MainTabView.swift            # Tab container
│   └── SettingsView.swift           # Settings / account
└── Features/
    └── {{FeatureName}}/
        ├── Models/                  # Domain models + insert types
        ├── Views/                   # Feature-specific views
        └── {{FeatureName}}Service.swift
```

---

## 3. Design Tokens

### 3a. Color System

```swift
import SwiftUI

// MARK: - Grayscale Scale (base5 = near-white → base110 = near-black)

enum BaseColor {
    case base5, base10, base20, base30, base40, base50
    case base60, base70, base80, base90, base100, base110

    var color: Color {
        switch self {
        case .base5:   return Color(red: 0.980, green: 0.980, blue: 0.980)
        case .base10:  return Color(red: 0.988, green: 0.988, blue: 0.988)
        case .base20:  return Color(red: 0.949, green: 0.949, blue: 0.949)
        case .base30:  return Color(red: 0.894, green: 0.894, blue: 0.894)
        case .base40:  return Color(red: 0.733, green: 0.733, blue: 0.733)
        case .base50:  return Color(red: 0.479, green: 0.486, blue: 0.490)
        case .base60:  return Color(red: 0.392, green: 0.400, blue: 0.408)
        case .base70:  return Color(red: 0.306, green: 0.314, blue: 0.322)
        case .base80:  return Color(red: 0.220, green: 0.227, blue: 0.239)
        case .base90:  return Color(red: 0.133, green: 0.141, blue: 0.153)
        case .base100: return Color(red: 0.071, green: 0.071, blue: 0.071)
        case .base110: return Color(red: 0.047, green: 0.047, blue: 0.047)
        }
    }
}

extension Color {
    // Grayscale shortcuts
    static let base5 = BaseColor.base5.color
    static let base10 = BaseColor.base10.color
    static let base20 = BaseColor.base20.color
    static let base30 = BaseColor.base30.color
    static let base40 = BaseColor.base40.color
    static let base50 = BaseColor.base50.color
    static let base60 = BaseColor.base60.color
    static let base70 = BaseColor.base70.color
    static let base80 = BaseColor.base80.color
    static let base90 = BaseColor.base90.color
    static let base100 = BaseColor.base100.color
    static let base110 = BaseColor.base110.color

    // ── Brand Colors — SWAP THESE ──
    static let brandPrimary = Color(hex: "{{BrandPrimary}}")
    static let brandAccent  = Color(hex: "{{BrandAccent}}")

    // ── Semantic Tokens ──
    static let primaryText     = Color.base100
    static let secondaryText   = Color.base50
    static let background      = Color.white
    static let secondaryBg     = Color.base10
    static let overlay1        = Color.base10
    static let overlay2        = Color.base20
    static let overlay3        = Color.base30
    static let positive        = Color(red: 0, green: 0.787, blue: 0.311)
    static let negative        = Color(red: 0.842, green: 0.090, blue: 0.086)
    static let placeholder     = Color.base40

    // MARK: - Hex Initializer

    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 3:
            (a, r, g, b) = (255, (int >> 8) * 17, (int >> 4 & 0xF) * 17, (int & 0xF) * 17)
        case 6:
            (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8:
            (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default:
            (a, r, g, b) = (255, 0, 0, 0)
        }
        self.init(.sRGB, red: Double(r) / 255, green: Double(g) / 255, blue: Double(b) / 255, opacity: Double(a) / 255)
    }
}
```

**Rule**: NEVER use raw hex, `Color.blue`, or `Color(.systemGray)` in views. Always use the semantic tokens or base scale.

### 3b. Typography

```swift
import SwiftUI

// MARK: - Font Families — SWAP THESE

enum FontFamily: String {
    // Display / Serif — headings
    case displayRegular = "{{DisplayFontName}}-Regular"
    case displayBold    = "{{DisplayFontName}}-Bold"

    // Body / Sans-serif — body text and UI
    case bodyRegular  = "{{BodyFontName}}-Regular"
    case bodyMedium   = "{{BodyFontName}}-Medium"
    case bodySemiBold = "{{BodyFontName}}-SemiBold"
    case bodyBold     = "{{BodyFontName}}-Bold"
}

// MARK: - Semantic Type Scale

enum AppFont {
    // Headings (display font)
    case h1          // 32pt bold   — page titles
    case h2          // 28pt bold   — section headers
    case h3          // 22pt regular — subsection headers
    case h4          // 18pt regular — card titles
    case h5          // 16pt semibold — emphasized labels

    // Body (sans-serif font)
    case bodyLarge   // 16pt regular
    case body        // 14pt regular — default body
    case caption     // 12pt regular — timestamps, hints
    case footnote    // 10pt regular — legal, fine print

    // Buttons
    case buttonLarge // 16pt medium  — primary buttons
    case button      // 14pt medium  — standard buttons
    case buttonSmall // 12pt medium  — secondary/tag buttons

    // Special
    case label       // 11pt medium  — tags, badges

    var font: Font {
        switch self {
        case .h1:          return .custom(FontFamily.displayBold.rawValue, size: 32)
        case .h2:          return .custom(FontFamily.displayBold.rawValue, size: 28)
        case .h3:          return .custom(FontFamily.displayRegular.rawValue, size: 22)
        case .h4:          return .custom(FontFamily.displayRegular.rawValue, size: 18)
        case .h5:          return .custom(FontFamily.bodySemiBold.rawValue, size: 16)
        case .bodyLarge:   return .custom(FontFamily.bodyRegular.rawValue, size: 16)
        case .body:        return .custom(FontFamily.bodyRegular.rawValue, size: 14)
        case .caption:     return .custom(FontFamily.bodyRegular.rawValue, size: 12)
        case .footnote:    return .custom(FontFamily.bodyRegular.rawValue, size: 10)
        case .buttonLarge: return .custom(FontFamily.bodyMedium.rawValue, size: 16)
        case .button:      return .custom(FontFamily.bodyMedium.rawValue, size: 14)
        case .buttonSmall: return .custom(FontFamily.bodyMedium.rawValue, size: 12)
        case .label:       return .custom(FontFamily.bodyMedium.rawValue, size: 11)
        }
    }

    func size(_ size: CGFloat) -> Font {
        switch self {
        case .h1, .h2:
            return .custom(FontFamily.displayBold.rawValue, size: size)
        case .h3, .h4:
            return .custom(FontFamily.displayRegular.rawValue, size: size)
        default:
            return .custom(FontFamily.bodyRegular.rawValue, size: size)
        }
    }
}

// MARK: - Static Font Extensions

extension Font {
    static let h1          = AppFont.h1.font
    static let h2          = AppFont.h2.font
    static let h3          = AppFont.h3.font
    static let h4          = AppFont.h4.font
    static let h5          = AppFont.h5.font
    static let bodyLarge   = AppFont.bodyLarge.font
    static let body        = AppFont.body.font
    static let caption     = AppFont.caption.font
    static let footnote    = AppFont.footnote.font
    static let buttonLarge = AppFont.buttonLarge.font
    static let button      = AppFont.button.font
    static let buttonSmall = AppFont.buttonSmall.font
    static let label       = AppFont.label.font
}

// MARK: - View Modifier

extension View {
    func font(_ appFont: AppFont) -> some View {
        self.font(appFont.font)
    }
}
```

**Rule**: NEVER use `.font(.system(size:))` or `.font(.title)`. Always use the semantic scale: `.font(.h1)`, `.font(.body)`, `.font(.button)`.

### 3c. Spacing

```swift
import SwiftUI

enum Spacing {
    // ── Core Scale ──
    static let xxs: CGFloat = 2
    static let xs: CGFloat = 4
    static let sm: CGFloat = 8
    static let md: CGFloat = 16
    static let lg: CGFloat = 24
    static let xl: CGFloat = 32
    static let xxl: CGFloat = 40

    // ── Semantic Aliases ──
    static let screenMargin: CGFloat = 16
    static let sectionGap: CGFloat = 32
    static let elementGap: CGFloat = 8
    static let cardPadding: CGFloat = 16
    static let cornerRadius: CGFloat = 12
    static let cornerRadiusSmall: CGFloat = 8
    static let progressBarHeight: CGFloat = 8
    static let navBarHeight: CGFloat = 44
    static let tabBarHeight: CGFloat = 86
    static let buttonVerticalPadding: CGFloat = 16
    static let buttonVerticalPaddingSmall: CGFloat = 8
}
```

**Rule**: NEVER use raw numbers for padding, spacing, or corner radius. Always `Spacing.screenMargin`, `Spacing.elementGap`, etc.

---

## 4. Architecture

### 4a. App Entry Point

```swift
@main
struct {{AppName}}App: App {
    @State private var authManager = AuthManager()

    init() {
        Analytics.initialize()
    }

    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(authManager)
                .preferredColorScheme({{ColorScheme}})
        }
    }
}
```

**Rules:**
- Top-level managers instantiated as `@State` in the `App` struct
- Injected via `.environment()` — NOT `@EnvironmentObject`
- Force light mode at root
- Initialize analytics in `init()`

### 4b. State Management

Use the **`@Observable` macro** on all manager/service classes. Consume via `@Environment(Type.self)` in views.

```swift
@Observable
class SomeManager {
    var items: [Item] = []
    var isLoading = false

    private let cache = FileCache<[Item]>(filename: "items_cache.json")

    init() {
        // Cache-first: load synchronously on init
        if let cached = cache.load() {
            items = cached
        }
    }

    @MainActor
    func loadItems() async {
        isLoading = true
        defer { isLoading = false }

        let fresh = try? await supabase.from("items").select().execute().value as [Item]
        if let fresh {
            items = fresh
            cache.save(fresh)
        }
    }
}
```

### 4c. Optimistic Local State

Update the UI immediately, then persist to the database in background:

```swift
@MainActor
func toggleFavorite(_ item: Item) async {
    // 1. Update local state immediately
    if let idx = items.firstIndex(where: { $0.id == item.id }) {
        items[idx].isFavorited.toggle()
        cache.save(items)
    }
    // 2. Persist to database
    try? await supabase.from("user_items")
        .update(["is_favorited": items[idx].isFavorited])
        .eq("id", value: item.id.uuidString)
        .execute()
}
```

### 4d. Models

Separate read and write types:

```swift
// Read model — matches database columns, used for SELECT
struct Item: Codable, Identifiable {
    let id: UUID
    let name: String
    let imageURL: String?
    let createdAt: Date
}

// Write model — only the fields you INSERT
struct ItemInsert: Codable {
    let name: String
}
```

### 4e. Auth

```swift
enum AuthState: Equatable {
    case initializing
    case authenticated(Session)
    case unauthenticated

    static func == (lhs: AuthState, rhs: AuthState) -> Bool {
        switch (lhs, rhs) {
        case (.initializing, .initializing),
             (.authenticated, .authenticated),
             (.unauthenticated, .unauthenticated):
            return true
        default: return false
        }
    }
}

@Observable
class AuthManager {
    var state: AuthState = .initializing

    var session: Session? {
        if case .authenticated(let s) = state { return s }
        return nil
    }

    init() {
        Task {
            await restoreSession()
            await observeAuthStateChanges()
        }
    }

    private func restoreSession() async {
        do {
            let session = try await supabase.auth.session
            await MainActor.run { state = .authenticated(session) }
        } catch {
            await MainActor.run { state = .unauthenticated }
        }
    }

    private func observeAuthStateChanges() async {
        for await (_, session) in supabase.auth.authStateChanges {
            await MainActor.run {
                state = session != nil ? .authenticated(session!) : .unauthenticated
            }
        }
    }

    @MainActor
    func signOut() async throws {
        Analytics.reset()
        try await supabase.auth.signOut()
        state = .unauthenticated
    }
}
```

### 4f. Networking

**Global Supabase client** (single file):

```swift
import Supabase

let supabase = SupabaseClient(
    supabaseURL: URL(string: APIConfig.supabaseURL)!,
    supabaseKey: APIConfig.supabaseAnonKey
)
```

**APIConfig**:

```swift
struct APIConfig {
    static let supabaseURL = "{{SupabaseURL}}"
    static let supabaseAnonKey = "{{SupabaseAnonKey}}"
    static let storageBucket = "images"

    static var storageUploadURL: String {
        "\(supabaseURL)/storage/v1/object/\(storageBucket)"
    }

    static func publicImageURL(path: String) -> String {
        "\(supabaseURL)/storage/v1/object/public/\(storageBucket)/\(path)"
    }
}
```

### 4g. FileCache

Generic, reusable for any `Codable` type:

```swift
final class FileCache<T: Codable> {
    private let filename: String
    private let fileManager = FileManager.default

    private var fileURL: URL {
        fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
            .appendingPathComponent(filename)
    }

    init(filename: String) { self.filename = filename }

    func load() -> T? {
        guard fileManager.fileExists(atPath: fileURL.path) else { return nil }
        do {
            let data = try Data(contentsOf: fileURL)
            let decoder = JSONDecoder()
            decoder.dateDecodingStrategy = .iso8601
            return try decoder.decode(T.self, from: data)
        } catch { return nil }
    }

    func save(_ value: T) {
        do {
            let encoder = JSONEncoder()
            encoder.dateEncodingStrategy = .iso8601
            try encoder.encode(value).write(to: fileURL)
        } catch { print("[FileCache] Error saving \(filename): \(error)") }
    }

    func clear() { try? fileManager.removeItem(at: fileURL) }
}
```

### 4h. Analytics

```swift
enum Analytics {
    private static var amplitude: Amplitude?

    static func initialize() {
        amplitude = Amplitude(configuration: Configuration(
            apiKey: "{{AnalyticsKey}}",
            autocapture: [.sessions, .appLifecycles, .screenViews]
        ))
    }

    static func identify(userId: String) { amplitude?.setUserId(userId: userId) }
    static func reset() { amplitude?.reset() }

    static func track(_ event: String, properties: [String: Any]? = nil) {
        amplitude?.track(eventType: event, eventProperties: properties)
    }
}
```

---

## 5. Navigation

### Root View — Auth State Router

```swift
struct RootView: View {
    @Environment(AuthManager.self) private var authManager

    var body: some View {
        Group {
            switch authManager.state {
            case .initializing:
                LoadingOverlay()
            case .authenticated:
                MainTabView()
            case .unauthenticated:
                LoginView()
            }
        }
        .animation(.easeInOut, value: authManager.state)
    }
}
```

### Tab View

```swift
struct MainTabView: View {
    @State private var selectedTab = 0

    var body: some View {
        TabView(selection: $selectedTab) {
            Tab("Home", systemImage: "house", value: 0) {
                HomeView()
            }
            Tab("Profile", systemImage: "person", value: 1) {
                ProfileView()
            }
        }
        .tint(.brandAccent)
    }
}
```

### Navigation Conventions

- **Sheet-based navigation exclusively** — no `NavigationLink` for pushing views
- **`@Environment(\.dismiss)`** for sheet dismissal
- **State-driven animated transitions**: `.animation(.easeInOut, value: state)`
- **Close button pattern** for sheets:
  ```swift
  Button { dismiss() } label: {
      Image(systemName: "xmark")
          .font(.system(size: 14, weight: .bold))
          .foregroundColor(.base80)
          .frame(width: 32, height: 32)
          .background(Color.overlay2)
          .clipShape(Circle())
  }
  ```

---

## 6. Component Patterns

### 6a. ButtonView

```swift
enum ButtonType {
    case primary        // brandPrimary bg, white text
    case primaryDark    // base90 bg, white text
    case accent         // brandAccent bg, white text
    case secondary      // overlay2 bg, base80 text
    case tertiary       // clear bg, primaryText
    case destructive    // overlay2 bg, negative text
}

enum ButtonSize {
    case small   // 8px vertical padding, .button font
    case medium  // 12px vertical padding, .button font
    case large   // 16px vertical padding, .buttonLarge font
}

struct ButtonView: View {
    var type: ButtonType = .primary
    var size: ButtonSize = .large
    let title: String
    var isEnabled: Bool = true
    var fillWidth: Bool = true
    var action: () -> Void

    var body: some View {
        Button(action: {
            UIImpactFeedbackGenerator(style: .medium).impactOccurred()
            action()
        }) {
            Text(title)
                .font(size == .large ? .buttonLarge : .button)
                .foregroundColor(foregroundColor)
                .frame(maxWidth: fillWidth ? .infinity : nil)
                .padding(.vertical, verticalPadding)
                .padding(.horizontal, Spacing.md)
                .background(backgroundColor)
                .cornerRadius(Spacing.cornerRadius)
        }
        .disabled(!isEnabled)
        .opacity(isEnabled ? 1 : 0.5)
    }

    // ... computed properties for foregroundColor, backgroundColor, verticalPadding based on type/size
}
```

### 6b. View Decomposition

Use computed properties for sub-views within the same file. Only extract to separate files when truly reusable:

```swift
struct FeatureView: View {
    var body: some View {
        ScrollView {
            headerSection
            contentSection
            footerSection
        }
    }

    private var headerSection: some View {
        VStack(alignment: .leading, spacing: Spacing.sm) {
            Text("Title").font(.h2)
            Text("Subtitle").font(.body).foregroundColor(.secondaryText)
        }
        .padding(.horizontal, Spacing.screenMargin)
    }
}
```

### 6c. Cards

```swift
HStack(spacing: Spacing.sm) {
    // card content
}
.padding(Spacing.cardPadding)
.background(Color.white)
.cornerRadius(Spacing.cornerRadius)
```

### 6d. Lists

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
            .listRowSeparator(.hidden)
            .listRowInsets(EdgeInsets(
                top: Spacing.xs,
                leading: Spacing.screenMargin,
                bottom: Spacing.xs,
                trailing: Spacing.screenMargin
            ))
    }
}
.listStyle(.plain)
.scrollContentBackground(.hidden)
```

### 6e. Section Dividers

```swift
Rectangle()
    .fill(Color.base30)
    .frame(height: 1)
    .padding(.vertical, Spacing.lg)
```

### 6f. Remote Images (Kingfisher)

Always use Kingfisher for remote images:

```swift
KFImage(URL(string: imageURL))
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(width: 60, height: 60)
    .clipped()
    .cornerRadius(Spacing.cornerRadiusSmall)
```

### 6g. Cover Photo with Gradient Fade

```swift
KFImage(URL(string: imageURL))
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(width: UIScreen.main.bounds.width, height: UIScreen.main.bounds.height / 4)
    .clipped()
    .overlay(alignment: .bottom) {
        LinearGradient(colors: [.clear, .white], startPoint: .top, endPoint: .bottom)
            .frame(height: 80)
    }
```

### 6h. Empty States

```swift
VStack(spacing: Spacing.sm) {
    Text("Nothing here yet")
        .font(.h4)
        .foregroundColor(.base80)
    Text("Context-aware subtitle explaining what to do")
        .font(.body)
        .foregroundColor(.secondaryText)
        .multilineTextAlignment(.center)
}
.frame(maxWidth: .infinity, maxHeight: .infinity)
.padding(.horizontal, Spacing.screenMargin)
```

### 6i. Haptic Feedback

```swift
// On every interactive tap
UIImpactFeedbackGenerator(style: .light).impactOccurred()

// On confirmations
UIImpactFeedbackGenerator(style: .medium).impactOccurred()

// On destructive actions
UINotificationFeedbackGenerator().notificationOccurred(.warning)
```

### 6j. Text in ScrollViews

Prevent truncation:

```swift
Text(longString)
    .font(.body)
    .fixedSize(horizontal: false, vertical: true)
```

### 6k. Progress Bars

```swift
GeometryReader { geometry in
    ZStack(alignment: .leading) {
        Rectangle()
            .fill(Color.base20)
            .frame(height: Spacing.progressBarHeight)
            .cornerRadius(4)
        Rectangle()
            .fill(Color.brandAccent)
            .frame(width: geometry.size.width * progress, height: Spacing.progressBarHeight)
            .cornerRadius(4)
    }
}
.frame(height: Spacing.progressBarHeight)
```

---

## 7. Animation Principles

**The goal**: Every state change should feel intentional. No jarring pops.

### State Transitions

Animate every state change that affects layout or visibility:

```swift
.animation(.easeInOut(duration: 0.2), value: someState)
```

### Looping Animations

Use `@State private var isAnimating = false` + `.onAppear` + `.repeatForever` for any ambient motion (loading indicators, attention pulses, etc.). Always use brand colors and keep durations between 0.8–2.0 seconds.

```swift
SomeShape()
    .scaleEffect(isAnimating ? 1.1 : 0.9)
    .onAppear {
        withAnimation(.easeInOut(duration: 1.2).repeatForever(autoreverses: true)) {
            isAnimating = true
        }
    }
```

### Staggered Animations

For lists or repeated elements appearing sequentially, use `.delay(Double(index) * 0.1)` to create a cascade effect rather than everything appearing at once.

### Conventions

- Always pass `value:` to `.animation()` — never use the implicit (deprecated) form
- Prefer `.easeInOut` for UI transitions, `.spring()` for interactive gestures
- Keep transition durations under 0.3s for UI state changes, 1–2s for ambient/loading animations
- Loading states should feel branded — use your brand colors, not a generic `ProgressView()`

---

## 8. Rules Summary (Quick Reference)

1. **Colors**: Never raw hex or system colors. Always `Color.brandPrimary`, `Color.secondaryText`, `Color.base50`, etc.
2. **Fonts**: Never `.font(.system(...))` or `.font(.title)`. Always `.font(.h1)`, `.font(.body)`, `.font(.button)`.
3. **Spacing**: Never raw numbers. Always `Spacing.screenMargin`, `Spacing.elementGap`, `Spacing.cornerRadius`.
4. **Images**: Always Kingfisher (`KFImage`) for remote images. Never `AsyncImage`.
5. **Navigation**: Always sheets. Never `NavigationLink`. Use `@Environment(\.dismiss)`.
6. **State**: Always `@Observable` + `@Environment`. Never `ObservableObject` + `@EnvironmentObject`.
7. **Haptics**: Every tappable element gets haptic feedback.
8. **Loading**: Cache-first. Show cached data instantly, refresh in background.
9. **Updates**: Optimistic local state. Update UI immediately, persist async.
10. **Single color scheme**: Force `.preferredColorScheme({{ColorScheme}})` at app root. Don't support both until the design system is mature.
