# Beautiful iOS Apps — SwiftUI Skill File for Claude Code

A prescriptive skill file that teaches Claude Code how to build beautiful, production-quality SwiftUI iOS apps with a consistent design system.

## What's Included

`SKILL.md` contains a complete design system and architecture guide covering:

- **Design Tokens** — Color system (grayscale scale + brand colors + semantic tokens), typography (semantic type scale with display and body fonts), and spacing tokens
- **Architecture** — App entry point, `@Observable` state management, optimistic local state, cache-first loading, auth state machine
- **Navigation** — Sheet-based navigation (no `NavigationLink`), auth state router, tab view patterns
- **Components** — Buttons, cards, lists, empty states, progress bars, remote images (Kingfisher), cover photos with gradient fade
- **Networking** — Supabase client setup, API config, generic file cache
- **Animation** — State transitions, looping animations, staggered cascades
- **Haptics** — Feedback on every tappable element

## How to Use

1. **Copy `SKILL.md` into your project root and rename it to `CLAUDE.md`:**

   ```bash
   cp SKILL.md /path/to/your/ios-project/CLAUDE.md
   ```

2. **Fill in the placeholders.** Open `CLAUDE.md` and replace every `{{placeholder}}` with your project's values:

   | Placeholder | What It Is | Example |
   |---|---|---|
   | `{{AppName}}` | Your app's name | Vino |
   | `{{BundleID}}` | Bundle identifier | com.yourco.vino |
   | `{{DeploymentTarget}}` | Minimum iOS version | 18.0 |
   | `{{BodyFontName}}` | Body/UI font PostScript prefix | BananaGrotesk |
   | `{{DisplayFontName}}` | Heading/display font PostScript prefix | PlayfairDisplaySC |
   | `{{BrandPrimary}}` | Primary brand hex | #6D0000 |
   | `{{BrandAccent}}` | Accent brand hex | #810100 |
   | `{{SupabaseURL}}` | Supabase project URL | https://xxx.supabase.co |
   | `{{SupabaseAnonKey}}` | Supabase anon/public key | eyJhbGc... |
   | `{{AnalyticsKey}}` | Amplitude (or other) API key | abc123 |
   | `{{ColorScheme}}` | Forced color scheme | .light |

3. **Start building.** Claude Code will now follow your design system automatically — no raw hex colors, no raw font sizes, no raw spacing numbers.

## Key Principles

- **Never use raw values** — all colors, fonts, and spacing go through semantic tokens
- **Cache-first loading** — show cached data instantly, refresh in background
- **Optimistic local state** — update UI immediately, persist async
- **Sheet-based navigation** — no `NavigationLink` push navigation
- **Single color scheme** — ship with one mode first, add the other later
- **Kingfisher** for all remote images, never raw `AsyncImage`
- **Haptic feedback** on every tappable element

## License

See [LICENSE](LICENSE) for details.
