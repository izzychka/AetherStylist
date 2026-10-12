# AetherStylist ♡ — C# recommendation-engine starter

A **headless, testable first implementation** of *Build Around This* for a future FFXIV Dalamud plugin.

> **Status:** This archive is not yet an installable Dalamud plugin. It contains a real C# recommendation engine, a console demonstration, example catalogue records, and dependency-free executable checks. The item names and IDs in the demo are **fictional**, not FFXIV data. The engine has not been compiled in the authoring environment (which did not have the .NET SDK); build/test on your machine.

## Quick start

Install the **.NET 10 SDK** (current Dalamud API 15 uses .NET 10). From the archive root:

```bash
dotnet run --project src/AetherStylist.Demo
dotnet run --project tests/AetherStylist.Specs
```

In the demo, the starting item is ID `1001`, *Midnight Oversized Jacket*, with a chosen `alt` style. Change `OutfitRequest` in `src/AetherStylist.Demo/Program.cs`, then rerun it.

### Project map

```
AetherStylist-starter/
├── src/
│   ├── AetherStylist.Core/          # Plugin-agnostic models, matcher and outfit search
│   │   ├── Models.cs
│   │   ├── ColourHarmony.cs
│   │   └── RecommendationEngine.cs
│   └── AetherStylist.Demo/          # Runs without Dalamud / game installation
│       ├── Program.cs
│       └── data/catalogue.json     # Fictional, hand-tagged sample pieces
├── tests/AetherStylist.Specs/      # No-NuGet console checks
├── PluginIntegration/              # Manual integration guide and UI example
└── README.md
```

## What v0.1 can do

- Lock one or two pieces and keep them in the outfit.
- Recommend individual slots or generate full five-slot outfits using **beam search**.
- Score aesthetic tags (40%), colour harmony (25%), top/bottom silhouette balance (20%), materials (15%).
- Use job and owned-only filters and return human-readable recommendation reasons.
- Choose automatic, monochromatic or complementary colour handling.
- Keep all outfit decisions deterministic and testable.

### What it **doesn't** do yet

- No real FFXIV items, character inventory state, job/race/gender validation, item preview or live appearance manipulation.
- No automatic screenshot feature extraction, true perceptual matching, AI model, or Penumbra integration.
- No persistence or finished ImGui interface.
- Doesn't understand multi-slot glamour items, dye channels, item-model gender variants, armour appearance overlap, and some special equipment restrictions. These need to be handled **before** recommending real gear.
- Scores are **heuristics**, never a probability of fashion compatibility. The seed labels are manually curated.

## Design principles

**1. Core engine is completely unaware of the game.** The plugin adapter will turn data from Lumina and local style overrides into `GlamourItem` records. This design allows the same engine to power an ImGui UI, an offline analysis tool, and tests.

**2. Hard filters happen before scoring.** A beautiful piece isn't useful if it cannot be glamoured by the character. `AllowedJobs` and `IsOwned` are simplified demonstration fields; production will need a full availability/character eligibility service that verifies level, job, race/gender, special appearance cases and glamour rules. Don't assume the game sheets already contain style tags or fabric colours.

**3. Visual metadata is an extensible annotation layer.** Later this could contain silhouettes, colour palettes from rendered thumbnails, embeddings, silhouette contrast, and user-provided tags. In v0.1, `RgbColor`, `Silhouette`, `Material`, and `Styles` are authored manually.

**4. User intent dominates recommendations.** Locked pieces never change. We should eventually add reject/like feedback, diversity, and user-defined weights.

## Scoring

Each proposed item gets a breakdown (0–1):

```
score = 0.40 * aesthetic_match
      + 0.25 * colour_harmony
      + 0.20 * silhouette_balance
      + 0.15 * material_match
```

Aesthetic = 80% match to requested style and 20% tag overlap with already selected pieces.

Colour = average colour harmony with selected pieces. Low-saturation neutrals work with most colours; Auto allows similar/analogous/complementary hues; other modes explicitly constrain the hue relationship. This is a deliberately simple HSL heuristic.

Silhouette = checks the body/legs pairing; e.g., loose top with fitted bottoms receives a bonus, loose top with loose bottoms receives a penalty. Other slot combinations are neutral for now.

Material = compatible basic material pairing bonus. When no other item is selected, each visual comparison starts with a neutral baseline.

For each missing slot, beam search expands the best partial outfits with top candidates instead of choosing each item independently. Suggestions are *not* ranked by image ML in this version.

## Development roadmap

**Milestone A — core (this ZIP):** Testable C# engine + mocked catalogue + style and colour rules.

**Milestone B — actual catalogue:** Start from `goatcorp/SamplePlugin`. Use current `IDataManager.GetExcelSheet<Item>()` to read real item IDs, names and equipment categories. Filter glamour-eligible wearable items, skip multi-slot until modelled correctly. Join with a `style-overrides.json` file keyed by item ID. Render a searchable ImGui item list.

**Milestone C — real outfit building:** Correct legality checks, locked slots, pick job/race/gender, browse ranked suggestions and save outfits to Dalamud configuration.

**Milestone D — visual matching:** Build rendered preview images or verified item sprites, derive colours/silhouettes from images, compare appearance embeddings or human tags. This deserves its own data-collection and evaluation pipeline; do not infer aesthetics from item names.

**Milestone E — integrations:** Glamourer preview via a supported IPC API, community lookbooks, Penumbra metadata support, two dye-channel palettes.

### An actual first coding exercise

In `RecommendationEngine.cs`, improve `ShapeScore`: make the result sensitive to the **selected aesthetic**. Streetwear may welcome loose-on-loose silhouettes, while elegant outfits might favour structured/flowing pairings. Add three tests showing why those ratings changed. This is a meaningful improvement you can implement without touching Dalamud.

## Dalamud publication note

Goatcorp's official repository requires genuinely human-led ownership of the code, personal testing and disclosure of substantial AI help. Entirely AI-generated submissions are rejected. Before publishing, review the current policy at https://dalamud.dev/plugin-publishing/ai-policy/ and the restriction guidelines. FFXIV/Dalamud plugin use is not officially supported by Square Enix.

## References

- https://dalamud.dev/plugin-development/getting-started/
- https://dalamud.dev/api/Dalamud.Plugin.Services/Interfaces/IDataManager/
- https://github.com/goatcorp/SamplePlugin
- https://dalamud.dev/versions/v15/
