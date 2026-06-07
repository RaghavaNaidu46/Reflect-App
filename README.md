# Reflect

Private on-device AI journaling for iOS 26. Every entry gets a structured reflection from Apple's Foundation Models — no backend, no API keys, no network.

## Requirements

- Xcode 26+ with iOS 26 SDK
- iOS 26 device or simulator

## Before building: add the fonts

The hand-drawn wireframe aesthetic requires two fonts (SIL OFL — free to bundle):

1. Download **Cabin Sketch Bold** from [fonts.google.com/specimen/Cabin+Sketch](https://fonts.google.com/specimen/Cabin+Sketch)
2. Download **Patrick Hand Regular** from [fonts.google.com/specimen/Patrick+Hand](https://fonts.google.com/specimen/Patrick+Hand)
3. Copy `CabinSketch-Bold.ttf` and `PatrickHand-Regular.ttf` into `Reflect/Resources/Fonts/`
4. In Xcode, ensure both files are added to the **Reflect** target (Build Phases → Copy Bundle Resources)

See `Reflect/Resources/Fonts/README.md` for font name verification steps.

## Architecture

```
Reflect/
├── ReflectApp.swift          @main, .modelContainer
├── Model/
│   ├── JournalEntry.swift    @Model — flat schema for SwiftData/CloudKit
│   └── Reflection.swift      @Generable structured output
├── Services/
│   └── ReflectionService.swift  @MainActor @Observable; availability state
├── DesignSystem/
│   ├── Theme.swift           Color tokens (paper/ink/accent, light+dark)
│   ├── Fonts.swift           Font extension (Cabin Sketch + Patrick Hand)
│   ├── RoughShapes.swift     Seeded-RNG rough shapes (no per-frame jitter)
│   ├── SketchComponents.swift SketchCard, SketchButton, SketchTag, SketchDivider
│   └── PaperBackground.swift  Ruled paper + grain overlay
└── Views/
    ├── EntryListView.swift   List + empty state + availability banner
    ├── ComposerView.swift    Ruled-paper composer + reflection flow
    └── EntryDetailView.swift Entry text + full reflection
```

## Design notes

The UI uses a **hand-drawn wireframe / analog-sketch** aesthetic throughout:
- `RoughRoundedRectangle` / `RoughCapsule` / `RoughLine` — seeded jitter, never recomputed per frame
- Cards have ±1° fixed rotation derived from their UUID bytes (stable across launches)
- Monochrome ink-on-paper with a single muted yellow accent for mood tags
- Both light ("day paper") and dark ("night paper") themes

## On-device AI

On launch, `ReflectionService.prepare()` checks `SystemLanguageModel.default.availability`. If unavailable (device not eligible, Apple Intelligence disabled, model still downloading), the app still saves entries — it just shows a friendly inline message where reflections would appear.
