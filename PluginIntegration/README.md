# Wiring AetherStylist into a real Dalamud plugin

This folder is an **integration recipe**, not a pre-compiled game plugin. The headless code in `src/` is the verified design boundary; the game integration must be tested against the installed Dalamud API.

## 1. Start from the latest template

Use **Use this template** at https://github.com/goatcorp/SamplePlugin and name it AetherStylist. Check its current framework/API version. Reference the core project from the plugin `.csproj`:

```xml
<ItemGroup>
  <ProjectReference Include="../AetherStylist.Core/AetherStylist.Core.csproj" />
</ItemGroup>
```

Move/copy `AetherStylist.Core` alongside your plugin, adjusting the path. Leave the plugin's own dependency/package setup from SamplePlugin intact.

## 2. Build a catalogue adapter

Dalamud exposes `IDataManager.GetExcelSheet<T>()` for Lumina game data. Read `Item` rows in an adapter, *not in the recommendation engine*. API 15 uses `Lumina.Excel.Sheets.Item` (check API version; v16 announces changes to sheet namespaces).

Pseudocode (not ready to compile):

```csharp
// Inject IDataManager using the current SamplePlugin pattern.
// foreach (var row in dataManager.GetExcelSheet<Item>()) {
//     if (!isWearableAndGlamourEligible(row)) continue;
//     var slot = mapEquipSlotCategoryToSupportedSlot(row);
//     if (slot is null || occupiesMultipleSlots(row)) continue; // first milestone
//     // Join a local style-overrides.json by row.RowId to provide
//     // Styles, Color, Silhouette and Material.
//     // Validate job/race/gender restrictions for the current character.
//     // Create new GlamourItem(row.RowId, row.Name.ToString(), slot, ...);
// }
```

**Important:** Until visual metadata is curated, item appearance fields must be treated as unknown. Lumina cannot tell you the aesthetic or dominant fabric colour of an item. The current engine gives neutral/approximate results for missing tags, so avoid advertising it as visually intelligent before adding real annotations.

## 3. Make a minimal ImGui window

Keep a `Window` implementation in the Dalamud project with state:

- `string selectedStyle = "alt";`
- `string selectedJob = "WHM";`
- `List<uint> lockedIds = [];`
- `List<GlamourItem> eligibleCatalogue = [];`
- `RecommendationEngine engine = new(eligibleCatalogue);`

Flow:

1. Search item catalogue; choose one or two pieces and lock them.
2. Select style and palette mode.
3. Create `OutfitRequest(selectedStyle, selectedJob, PaletteMode.Auto, lockedIds)`.
4. Call `engine.BuildOutfits(request)` on **explicit user action**, not every rendered frame.
5. Show the resulting 5-slot lists, their score breakdowns, and the explanations.
6. Later use the SamplePlugin UI image helpers and item icon loading to display thumbnails.

The README in the main folder has the complete v0.1 scope. Avoid adding unverified private API hooks to the first version.

## 4. Production concerns

- Incrementally load/cache the item catalogue; avoid scanning all rows on each Draw.
- Rebuild eligibility when changing jobs or characters.
- Keep item ID + appearance variant (and later dye channels) separate from the human-facing display name.
- Multi-slot items can invalidate an entire outfit; either model occupied slots or exclude them in the MVP.
- Do not apply glamour automatically, require confirmation before any integration changes appearance.
- Consider per-character configuration and separate saved outfits from styles/tag annotations.
- FFXIV's glamour rules can be stricter than equipment slot/job matching. Verify before presenting an outfit as actually wearable.
