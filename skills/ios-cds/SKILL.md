# iOS: Component Design System Reference

$ARGUMENTS

Reference the design system repo `tc-mx-ios-design-component` to map Figma design components to their code implementations and ensure correct usage in feature repos.

## When to use

This skill is invoked during `/ios-implement` when:
- A Figma design references UI components (buttons, cards, navigation, inputs, etc.)
- You need to understand how to use a design system component in a feature repo
- You're unsure which class/view corresponds to a Figma element

## Design System Repo

- **Repo**: `tc-mx-ios-design-component`
- **Location**: Sibling directory to the feature repo (e.g., `~/Developer/iOS/tc-mx-ios-design-component`)
- To find it from the current feature repo:
  ```bash
  DS_PATH=$(find "$(dirname "$(pwd)")" -maxdepth 1 -type d -name "tc-mx-ios-design-component" | head -1)
  ```

If the repo is not found locally, tell the user:
> "Design system repo `tc-mx-ios-design-component` not found. Please clone it as a sibling directory, or provide the path."

## UI 2.0 Components

The design system uses **UI 2.0** components, identified by a `V2` suffix in the component name (e.g., `PrimaryButtonV2`, `NavigationBarV2`, `CardViewV2`). **Always prefer the `V2` variant** over legacy components without the suffix.

When searching, prioritize UI 2.0:
- If both `PrimaryButton` and `PrimaryButtonV2` exist → use `PrimaryButtonV2`
- If only the non-suffixed version exists → check if a V2 version is planned or ask the user

## Mappings Cache

Previously researched component mappings are stored in `~/.claude/skills/ios-cds/mappings.md`. This avoids re-researching components that have already been mapped.

### Reading the cache

Before researching any component, check if it already exists in `mappings.md`:
1. Read `~/.claude/skills/ios-cds/mappings.md` (if it exists)
2. For each Figma component, check if a mapping entry already exists
3. **If cached**: skip to validation (verify the cached class/view still exists in the DS repo with the same signature — see "Validating cached mappings" below)
4. **If not cached**: proceed with the full research workflow (Steps 2-4)

### Validating cached mappings

Cached mappings can become outdated (renamed, removed, API changed). For every cached component you plan to use:
1. Verify the class/struct still exists in the DS repo:
   ```bash
   grep -r "class ${CachedClassName}\|struct ${CachedClassName}" "$DS_PATH" --include="*.swift" -l
   ```
2. Quick-check the init signature still matches the cached parameters:
   ```bash
   grep -A 10 "init(" <matched_file> | head -15
   ```
3. **If the class exists and signature matches** → use the cached mapping as-is
4. **If the class exists but signature changed** → read the updated API, update the cache entry
5. **If the class no longer exists** → remove the stale cache entry, run full research

### Writing to the cache

After completing the mapping (Step 4), append any **newly discovered or updated** mappings to `~/.claude/skills/ios-cds/mappings.md`. Use this format:

```markdown
## {ComponentName}
- **Figma**: {Figma component name}
- **DS Class**: {ClassName} (V2 if applicable)
- **File**: {relative path in DS repo}
- **Key Parameters**: {param1, param2, ...}
- **Usage**:
  ```swift
  {minimal usage example from playground/example code}
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

### Step 2: Search the design system repo (V2 first)

For each identified component, search the design system repo. **Always search for the `V2` suffix variant first:**

```bash
# Search for V2 class/struct by name (priority)
grep -r "class ${ComponentName}V2\|struct ${ComponentName}V2\|protocol ${ComponentName}V2" "$DS_PATH" --include="*.swift" -l

# Fallback: search without suffix
grep -r "class ${ComponentName}\|struct ${ComponentName}\|protocol ${ComponentName}" "$DS_PATH" --include="*.swift" -l

# Search by keyword if exact name unknown
grep -r "ButtonV2\|NavigationBarV2\|CardV2" "$DS_PATH" --include="*.swift" -l

# Browse the component structure
find "$DS_PATH" -type f -name "*.swift" | grep -v "Tests" | grep -v "Pods" | grep -v "PodLocals"
```

### Step 3: Research playground and example usage

After finding a component, look for **playground/demo/example** code in the design system repo to understand real usage patterns:

```bash
# Search for playground/demo/example files
find "$DS_PATH" -type f -name "*.swift" -path "*playground*" -o -path "*demo*" -o -path "*example*" -o -path "*sample*" -o -path "*showcase*"

# Search for usage of the specific component in playground/example files
grep -r "${ComponentName}V2\|${ComponentName}(" "$DS_PATH" --include="*.swift" -l | grep -i "playground\|demo\|example\|sample\|showcase"

# Also search for any usage across the entire DS repo (not just definition)
grep -rn "${ComponentName}V2(" "$DS_PATH" --include="*.swift" | grep -v "class \|struct \|protocol \|func "
```

### Step 4: Read and understand the component API

For each matched component:
1. Read the class/struct definition (public init parameters, defaults)
2. **Read playground/example usage** to understand how the component is used in practice (parameter combinations, nesting patterns, delegate setup)
3. Read any additional demo/sample code for usage patterns
4. Identify required vs optional parameters
5. Note any style/configuration objects or enums
6. Confirm the `V2` suffix variant is being used — flag if only legacy version found
7. Check if it's a UIKit view, SwiftUI view, or both

### Step 4: Map Figma → Code

Create a mapping of each Figma component to its design system implementation:

```
=== Figma → Design System Mapping ===

| Figma Component | DS Class/View | Key Parameters |
|-----------------|---------------|----------------|
| Primary Button  | PrimaryButtonV2 | title, action, isEnabled, style |
| Top Nav Bar     | NavigationBarV2 | title, leftItem, rightItems |
| Info Card       | CardViewV2      | style, content |
```

### Step 5: Handle missing or ambiguous components

**Do NOT improvise or create custom components without user confirmation.** But also do NOT interrupt the workflow for every single unclear component — batch them.

#### 5a. Exhaust all search strategies first

For each unresolved component, try all of these before flagging:

1. **Search with and without the `V2` suffix** — try alternative names, partial matches:
   ```bash
   grep -ri "bannerV2\|alertV2\|noticeV2\|infoV2" "$DS_PATH" --include="*.swift" -l
   grep -ri "banner\|alert\|notice\|info" "$DS_PATH" --include="*.swift" -l
   ```

2. **Search playground/example code** — the component may be used under a different name:
   ```bash
   grep -ri "banner\|alert\|notice" "$DS_PATH" --include="*.swift" -l | grep -i "playground\|demo\|example\|sample"
   ```

3. **Check if it's in a Pod dependency** — some DS components may be exposed via pods:
   ```bash
   grep -r "${ComponentName}" "$DS_PATH"/*.podspec 2>/dev/null
   grep -r "${ComponentName}" "$DS_PATH"/PodLocals/ --include="*.swift" -l 2>/dev/null
   ```

4. **Check if it's a standard UIKit/SwiftUI component** — some designs use platform components directly (`UITextField`, `UISwitch`, `Toggle`, etc.)

#### 5b. Categorize each unresolved component

After exhausting searches, classify each issue:
- **NOT_FOUND**: no match at all in the DS repo
- **AMBIGUOUS**: multiple possible matches, unclear which is correct
- **STALE_CACHE**: cached mapping points to a class that no longer exists or has a different API

#### 5c. Batch all issues and ask user ONCE

**Do NOT stop and ask after each component.** Complete the full research for ALL components first. Then present all unresolved components in a single review request:

```
=== CDS Review Required ===

I've mapped {N} of {Total} components successfully. The following {M} need your input:

1. [NOT_FOUND] `BannerAlert` — Figma shows a banner-style alert. No match in DS repo.
   → Options: (a) different name? (b) create in DS first? (c) use standard UIKit?

2. [AMBIGUOUS] `InfoCard` — Found both `CardViewV2` and `InfoCardView`. Which one?
   → Candidates: CardViewV2 (generic card), InfoCardView (legacy, no V2)

3. [STALE_CACHE] `PrimaryButtonV2` — cached but init signature changed (new `style` param).
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

1. **Import from the design system module** — never copy-paste views from the DS repo
2. **Use the exact API** as defined — don't wrap DS components in unnecessary abstractions
3. **Follow existing usage patterns** — search the feature repo for how other screens use the same DS component
4. **Use DS tokens** for colors, typography, dimensions — don't hardcode values
5. **Check pod version compatibility** — ensure the feature repo's DS pod version includes the component you need. If not, the pod version may need bumping.

## Log output

```
=== [ios-cds] ===
Status: PASSED | NEEDS_CLARIFICATION
Components mapped: <count>
Missing components: <list, or "none">
Mapping:
  <Figma name> → <DS class/view>
  ...
```

$ARGUMENTS
