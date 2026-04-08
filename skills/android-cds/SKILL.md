# Android: Component Design System Reference

$ARGUMENTS

Reference the design system repo `tc-mx-android-design-system` to map Figma design components to their code implementations and ensure correct usage in feature repos.

## When to use

This skill is invoked during `/android-implement` when:
- A Figma design references UI components (buttons, cards, navigation, inputs, etc.)
- You need to understand how to use a design system component in a feature repo
- You're unsure which composable corresponds to a Figma element

## Design System Repo

- **Repo**: `tc-mx-android-design-system`
- **Location**: Sibling directory to the feature repo (e.g., `~/AndroidRepo/tc-mx-android-design-system`)
- To find it from the current feature repo:
  ```bash
  DS_REPO=$(cd .. && find . -maxdepth 1 -type d -name "tc-mx-android-design-system" | head -1)
  DS_PATH="$(cd .. && pwd)/tc-mx-android-design-system"
  ```

If the repo is not found locally, tell the user:
> "Design system repo `tc-mx-android-design-system` not found. Please clone it as a sibling directory, or provide the path."

## UI 2.0 Components

The design system uses **UI 2.0** components, identified by a `2` suffix in the component name (e.g., `PrimaryButton2`, `TopNavigation2`, `ButtonDock2`, `CardContainer2`). **Always prefer the `2` variant** over legacy components without the suffix.

When searching, prioritize UI 2.0:
- If both `PrimaryButton` and `PrimaryButton2` exist → use `PrimaryButton2`
- If only the non-suffixed version exists → check if a UI 2.0 version is planned or ask the user

## Mappings Cache

Previously researched component mappings are stored in `~/.claude/skills/android-cds/mappings.md`. This avoids re-researching components that have already been mapped.

### Reading the cache

Before researching any component, check if it already exists in `mappings.md`:
1. Read `~/.claude/skills/android-cds/mappings.md` (if it exists)
2. For each Figma component, check if a mapping entry already exists
3. **If cached**: skip to validation (verify the cached composable still exists in the DS repo with the same signature — see "Validating cached mappings" below)
4. **If not cached**: proceed with the full research workflow (Steps 2-4)

### Validating cached mappings

Cached mappings can become outdated (renamed, removed, API changed). For every cached component you plan to use:
1. Verify the composable still exists in the DS repo:
   ```bash
   grep -r "fun ${CachedComposableName}" "$DS_PATH" --include="*.kt" -l
   ```
2. Quick-check the function signature still matches the cached parameters:
   ```bash
   grep -A 10 "fun ${CachedComposableName}" <matched_file> | head -15
   ```
3. **If the composable exists and signature matches** → use the cached mapping as-is
4. **If the composable exists but signature changed** → read the updated API, update the cache entry
5. **If the composable no longer exists** → remove the stale cache entry, run full research

### Writing to the cache

After completing the mapping (Step 4), append any **newly discovered or updated** mappings to `~/.claude/skills/android-cds/mappings.md`. Use this format:

```markdown
## {ComponentName}
- **Figma**: {Figma component name}
- **DS Composable**: {ComposableName} (2 suffix if applicable)
- **File**: {relative path in DS repo}
- **Key Parameters**: {param1, param2, ...}
- **Usage**:
  ```kotlin
  {minimal usage example from playground/preview code}
  ```
- **Last verified**: {YYYY-MM-DD}
```

Only cache components that were **successfully mapped and verified**. Do not cache missing or ambiguous components.

## Workflow

### Step 1: Identify components from Figma

From the Figma design (read via `/figma`), identify all UI components used:
- Buttons (primary, secondary, text, icon)
- Navigation bars (top, bottom)
- Cards, tiles, list items
- Input fields, dropdowns, selectors
- Dialogs, bottom sheets, modals
- Typography styles
- Icons and illustrations
- Spacing, padding, colors

### Step 2: Search the design system repo (UI 2.0 first)

For each identified component, search the design system repo. **Always search for the `2` suffix variant first:**

```bash
# Search for UI 2.0 composable by name (priority)
grep -r "fun ${ComponentName}2" "$DS_PATH" --include="*.kt" -l

# Fallback: search without suffix
grep -r "fun ${ComponentName}" "$DS_PATH" --include="*.kt" -l

# Search by keyword if exact name unknown
grep -r "Button2\|TopNavigation2\|Card2" "$DS_PATH/src" --include="*.kt" -l

# Browse the component package structure
find "$DS_PATH" -type f -name "*.kt" -path "*/composable/*" -o -path "*/component/*" -o -path "*/widget/*"
```

### Step 3: Research playground and example usage

After finding a component, look for **playground/demo/example** code in the design system repo to understand real usage patterns:

```bash
# Search for playground/demo/example screens
find "$DS_PATH" -type f -name "*.kt" -path "*playground*" -o -path "*demo*" -o -path "*example*" -o -path "*sample*" -o -path "*showcase*"

# Search for usage of the specific component in playground/example files
grep -r "${ComponentName}2\|${ComponentName}(" "$DS_PATH" --include="*.kt" -l | grep -i "playground\|demo\|example\|sample\|showcase\|preview"

# Also search for any usage across the entire DS repo (not just definition)
grep -rn "${ComponentName}2(" "$DS_PATH" --include="*.kt" | grep -v "^.*fun ${ComponentName}2"
```

### Step 4: Read and understand the component API

For each matched component:
1. Read the composable function signature (parameters, defaults)
2. **Read playground/example usage** to understand how the component is used in practice (parameter combinations, nesting patterns, state management)
3. Read `@Preview` functions for additional usage examples
4. Identify required vs optional parameters
5. Note any style/theme objects or companion configs (e.g., `TopNavigationDefault.defaultTopNavigationStyle()`)
6. Confirm the `2` suffix variant is being used — flag if only legacy version found

### Step 4: Map Figma → Code

Create a mapping of each Figma component to its design system implementation:

```
=== Figma → Design System Mapping ===

| Figma Component | DS Composable | Key Parameters |
|-----------------|---------------|----------------|
| Primary Button  | PrimaryButton2 | text, onClick, enabled, modifier |
| Top Nav Bar     | TopNavigation2 | title, style, leftAction, rightActions |
| Info Card       | CardContainer2 | modifier, backgroundColor, content |
| Button Dock     | ButtonDock2    | primaryButton, secondaryButton |
```

### Step 5: Handle missing or ambiguous components

**Do NOT improvise or create custom components without user confirmation.** But also do NOT interrupt the workflow for every single unclear component — batch them.

#### 5a. Exhaust all search strategies first

For each unresolved component, try all of these before flagging:

1. **Search with and without the `2` suffix** — try alternative names, partial matches:
   ```bash
   grep -ri "banner\|alert\|notice\|info" "$DS_PATH" --include="*.kt" -l
   grep -ri "banner2\|alert2\|notice2\|info2" "$DS_PATH" --include="*.kt" -l
   ```

2. **Search playground/example code** — the component may be used under a different name:
   ```bash
   grep -ri "banner\|alert\|notice" "$DS_PATH" --include="*.kt" -l | grep -i "playground\|demo\|example\|sample"
   ```

3. **Check if it's a standard Compose component** — some designs use Material components directly (`TextField`, `Checkbox`, `Switch`)

#### 5b. Categorize each unresolved component

After exhausting searches, classify each issue:
- **NOT_FOUND**: no match at all in the DS repo
- **AMBIGUOUS**: multiple possible matches, unclear which is correct
- **STALE_CACHE**: cached mapping points to a composable that no longer exists or has a different API

#### 5c. Batch all issues and ask user ONCE

**Do NOT stop and ask after each component.** Complete the full research for ALL components first. Then present all unresolved components in a single review request:

```
=== CDS Review Required ===

I've mapped {N} of {Total} components successfully. The following {M} need your input:

1. [NOT_FOUND] `BannerAlert` — Figma shows a banner-style alert. No match in DS repo.
   → Options: (a) different name? (b) create in DS first? (c) use standard Material/Compose?

2. [AMBIGUOUS] `InfoCard` — Found both `CardContainer2` and `InfoCard`. Which one?
   → Candidates: CardContainer2 (generic card), InfoCard (legacy, no 2 suffix)

3. [STALE_CACHE] `PrimaryButton2` — cached but signature changed (new `style` param).
   → I'll update the cache. Confirm new usage pattern?

Please resolve these so I can finalize the mapping.
```

#### 5d. Wait for user, then resume

After the user responds:
1. Update mappings based on user answers
2. Update the cache with corrections
3. Resume the workflow from where it paused — do NOT re-run already-completed steps
4. Output the final complete mapping table

## Usage in Feature Repo

When using a design system component in the feature repo, follow these rules:

1. **Import from the design system package** — never copy-paste composables from the DS repo
2. **Use the exact API** as defined — don't wrap DS components in unnecessary abstractions
3. **Follow the preview patterns** — if the DS shows a specific usage pattern in previews, mirror it
4. **Use DS tokens** for colors, typography, dimensions — don't hardcode values:
   - Colors: `Colors.screenColorBackgroundKeyScreen`, etc.
   - Typography: `typography.textDisplayL`, `typography.textTitleM`, etc.
   - Dimensions: `Dimens.sectionPaddingLeftRight`, `Dimens.cardRadiusDefault`, etc.
5. **Check version compatibility** — ensure the feature repo's DS dependency version includes the component you need

## Log output

```
=== [android-cds] ===
Status: PASSED | NEEDS_CLARIFICATION
Components mapped: <count>
Missing components: <list, or "none">
Mapping:
  <Figma name> → <DS composable>
  ...
```

$ARGUMENTS
