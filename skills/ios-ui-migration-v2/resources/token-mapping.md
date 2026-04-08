# DesignToken Mapping Reference

This guide helps you map Figma design values to DesignToken (TymeX.*) properties.

## CRITICAL COLOR RULE

**ALL colors MUST use `TymeX.patternColor*` prefix**, NOT `TymeX.color*` (old UI 1.0).
Only exceptions: screen/card/section level tokens keep their own prefix:
- `TymeX.screenColorBackgroundDefault` — screen level
- `TymeX.cardColor*` — card level (`cardColorBackgroundDefault`, `cardColorBackgroundOnKeyScreen`)
- `TymeX.sectionColor*` — section level (`sectionColorBackgroundDefault`)

## Spacing Tokens

### Numeric Spacing (Raw Values)

| Figma Value | DesignToken | Usage |
|-------------|-------------|-------|
| 4px | `TymeX.spacing1` | Micro spacing, icon padding |
| 8px | `TymeX.spacing2` | Small gaps, tight spacing |
| 12px | `TymeX.spacing3` | Compact spacing |
| 16px | `TymeX.spacing4` | Standard spacing |
| 20px | `TymeX.spacing5` | Medium spacing |
| 24px | `TymeX.spacing6` | Large spacing |
| 28px | `TymeX.spacing7` | Extra large |
| 32px | `TymeX.spacing8` | Section spacing |
| 40px | `TymeX.spacing9` | Major sections |
| 44px | `TymeX.spacing10` | Touchable area minimum |
| 48px | `TymeX.spacing11` | Large touchable area |
| 54px | `TymeX.spacing12` | Extra large area |

### Semantic Spacing (Component-Specific)

| Figma Name | DesignToken | Context |
|------------|-------------|---------|
| Card/Padding/Default | `TymeX.cardPaddingDefault` | Interior padding of cards |
| Card/Gap/Default | `TymeX.cardGapDefault` | Gap between card sections |
| Section/Gap/Default | `TymeX.sectionGapDefault` | Gap between major sections |
| Section/Padding/LeftRight | `TymeX.sectionPaddingLeftRight` | Horizontal screen margins |
| Screen/Gap/Default | `TymeX.screenGapDefault` | Full screen spacing |
| Group/Card/Gap/Default | `TymeX.groupCardGapDefault` | Gap between cards in list |
| Pattern/Gap/TextToText | `TymeX.patternGapTextToText` | Between text labels |
| Pattern/Gap/TextToElement | `TymeX.patternGapTextToElement` | Text to UI element |
| Pattern/Gap/TextToSmallIcon | `TymeX.patternGapTextToSmallIcon` | Text to small icon |
| Pattern/Gap/ElementToElement | `TymeX.patternGapElementToElement` | Between UI elements |
| Pattern/Padding/List/ListItemTopBottom | `TymeX.patternPaddingListListItemTopBottom` | List cell vertical padding |
| Pattern/Padding/CircleButton/Default | `TymeX.patternPaddingCircleButtonDefault` | Circle button interior padding |
| Non-Card/Gap/Default | `TymeX.nonCardGapDefault` | Gap between non-card elements |

**Matching Strategy:**
1. Look for exact name match (e.g., "Section Gap Default" → `sectionGapDefault`)
2. If Figma token has semantic name, prefer semantic DesignToken
3. If Figma shows raw value (e.g., "16px"), match to closest semantic token first, fallback to numeric

## Color Tokens

### Screen/Card/Section Colors (keep their own prefix)

| Figma Name | DesignToken | Usage |
|------------|-------------|-------|
| Screen/Background/Default | `TymeX.screenColorBackgroundDefault` | Main screen background (white) |
| Section/Background/Default | `TymeX.sectionColorBackgroundDefault` | Section background |
| Card/Color/Background/Default | `TymeX.cardColorBackgroundDefault` | Card background on default screens |
| Card/Color/Background/OnKeyScreen | `TymeX.cardColorBackgroundOnKeyScreen` | Card background on gradient/key screens |

### Background Colors (patternColor prefix)

| Figma Name | DesignToken | Usage |
|------------|-------------|-------|
| Pattern/Color/Background/Default | `TymeX.patternColorBackgroundDefault` | Default pattern background (white) |
| Pattern/Color/Background/Info/Base | `TymeX.patternColorBackgroundInfoBase` | Info background (#E2ECEE) — icons, skeleton |
| Pattern/Color/Background/Info/Heavy | `TymeX.patternColorBackgroundInfoHeavy` | Info pattern (heavy) |
| Pattern/Color/Background/Select/Base | `TymeX.patternColorBackgroundSelectBase` | Selection background (purple) |
| Pattern/Color/Background/Select/Light | `TymeX.patternColorBackgroundSelectLight` | Light selection background |

### Text Colors (patternColor prefix)

| Figma Name | DesignToken | Usage |
|------------|-------------|-------|
| Pattern/Color/Text/Default | `TymeX.patternColorTextDefault` | Primary text color (#2D2D3A) |
| Pattern/Color/Text/Subtle | `TymeX.patternColorTextSubtle` | Secondary/muted text (#595969) |
| Pattern/Color/Text/Link | `TymeX.patternColorTextLink` | Link/interactive text (blue) |
| Pattern/Color/Text/OnPrimary | `TymeX.patternColorTextOnPrimary` | Text on primary-colored backgrounds |
| Pattern/Color/Text/OnStaticDark | `TymeX.patternColorTextOnStaticDark` | Text on dark overlays (white) |
| Pattern/Color/Text/Disabled | `TymeX.patternColorTextDisabled` | Disabled state text |

### Stroke Colors (patternColor prefix)

| Figma Name | DesignToken | Usage |
|------------|-------------|-------|
| Pattern/Color/Stroke/Info/Base | `TymeX.patternColorStrokeInfoBase` | Info stroke (#E2ECEE) — borders |
| Pattern/Color/Stroke/Select | `TymeX.patternColorStrokeSelect` | Selection stroke (purple) |
| Pattern/Color/Stroke/Error/Base | `TymeX.patternColorStrokeErrorBase` | Error stroke |
| Pattern/Color/Stroke/Secondary/Base | `TymeX.patternColorStrokeSecondaryBase` | Secondary stroke |
| Pattern/Color/Stroke/Secondary/Extralight | `TymeX.patternColorStrokeSecondaryExtralight` | Light separator |

### Divider Colors

| Figma Name | DesignToken | Usage |
|------------|-------------|-------|
| Pattern/Color/Divider/Base | `TymeX.patternColorDividerDividerBase` | Divider line (preferred V2) |
| Pattern/Color/Divider/Strong | `TymeX.patternColorDividerDividerStrong` | Strong separator lines |

## Corner Radius Tokens

| Figma Name | DesignToken | Usage |
|------------|-------------|-------|
| Card/Radius/Default | `TymeX.cardRadiusDefault` | Card corner radius |
| Pattern/Radius/Default | `TymeX.patternRadiusDefault` | Standard pattern corner radius (8px) |
| Corner/Radius/5 | `TymeX.cornerRadius5` | 5pt radius (body section top corners) |
| Pattern/Radius/Circle | `9999px` or `.cornerRadius(view.bounds.width / 2)` | Full circle |

**Corner radius application:**
```swift
view.mxCornerRadius = TymeX.cardRadiusDefault
// or
view.layer.cornerRadius = TymeX.cardRadiusDefault
view.clipsToBounds = true

// Custom corners:
bodyView.makeCustomRoundCorners([.topLeft, .topRight], radius: TymeX.cornerRadius5)
```

## Typography Tokens

Typography in UI 2.0 uses `NSAttributedString` with `TymeX.text*` styles.

### Text Styles

| Figma Style | DesignToken | Usage |
|-------------|-------------|-------|
| Display/XL | `TymeX.textDisplayXl` | Extra large display text |
| Display/M | `TymeX.textDisplayM` | Medium display text |
| Display/S | `TymeX.textDisplayS` | Large display text (amounts, headers) |
| Display/XS | `TymeX.textDisplayXs` | Extra small display text |
| Title/M | `TymeX.textTitleM` | Section titles |
| Title/S | `TymeX.textTitleS` | Small titles |
| Body/Default/L | `TymeX.textBodyDefaultL` | Large body text (regular weight) |
| Body/Default/M | `TymeX.textBodyDefaultM` | Standard body text (regular weight) |
| Body/Default/S | `TymeX.textBodyDefaultS` | Small body text |
| Body/Emphasize/M | `TymeX.textBodyEmphasizeM` | Medium body text (bold/emphasized) |
| Label/Emphasize/S | `TymeX.textLabelEmphasizeS` | Small emphasized label |

### Usage Pattern

```swift
// Simple attributed string:
label.attributedText = NSAttributedString(
    string: "Your Text",
    attributes: TymeX.textBodyDefaultM.color(TymeX.patternColorTextDefault)
)

// Chaining modifiers:
let attrs = TymeX.textBodyDefaultM
    .color(TymeX.patternColorTextSubtle)
    .alignment(.center)

// Highlighted text with multiple styles:
label.highlightText(
    fullText: "Total: $1,000",
    boldedTexts: ["$1,000"],
    textColor: TymeX.patternColorTextDefault,
    boldFontAttribute: TymeX.textBodyEmphasizeM
)
```

**Key Methods:**
- `.color(UIColor)` — set text color
- `.alignment(.left/.center/.right)` — text alignment
- `.lineHeightMultiple(CGFloat)` — line height

**IMPORTANT:** Use `.color().alignment()` chaining. Do NOT use `.paragraphStyle(lineSpacing:alignment:)`

## Component Mapping (UI 1.0 → UI 2.0)

### Buttons

| UI 1.0 | Figma Component | UI 2.0 |
|--------|-----------------|--------|
| `TymeXButton` (primary) | Button/Primary/V2 | `TymeXPrimaryButtonV2` |
| `TymeXButton` (secondary) | Button/Secondary/V2 | `TymeXSecondaryButtonV2` |
| `TymeXButton` (tertiary) | Button/Tertiary/Contained/V2 | `TymeXTertiaryContainedButtonV2` |
| `TymeXButton` (tertiary outlined) | Button/Tertiary/Outlined/V2 | `TymeXTertiaryOutlinedButtonV2` |
| `TymeXCircleButton` | Button/Circle/V2 | `TymeXCircleButtonV2` |

### Input Fields

| UI 1.0 | Figma Component | UI 2.0 |
|--------|-----------------|--------|
| `TymeXTextField` | Input/Text/V2 | `TymeXInputTextFieldV2` |
| `TymeXAmountInputField` | Input/Amount/V2 | `TymeXAmountInputTextFieldV2` |
| `TymeXPhoneInputView` | Input/Phone/V2 | `TymeXPhoneInputViewV2` |
| `TymeXOTPInputView` | Input/OTP/V2 | `TymeXOTPInputViewV2` |
| `TymeXPasscodeInputView` | Input/Passcode/V2 | `TymeXPasscodeInputViewV2` |
| `TymeXPickerInputView` | Input/Picker/V2 | `TymeXInputPickerViewV2` |
| `TymeXDropdownInputField` | Input/Dropdown/V2 | `TymeXInputDropdownFieldV2` |

### Other Components

| UI 1.0 | Figma Component | UI 2.0 |
|--------|-----------------|--------|
| `TymeXSearchBar` | SearchBar/V2 | `TymeXSearchBarV2` |
| `TymeXButtonDock` | ButtonDock/V2 | `TymeXButtonDockV2` |

## Navigation V2

Old navigation API → New V2 API:

**Before (UI 1.0):**
```swift
navigationItem.title = "Title"
navigationItem.leftBarButtonItem = UIBarButtonItem(...)
```

**After (UI 2.0):**
```swift
mxApplyNavigationV2By(
    stylist: TymeXNavigationBarStylistV2(
        backgroundMode: .onKeyScreen,  // or .onCard, .transparent
        center: .byDefault(title: "Title"),
        left: .backButton,  // or .closeButton, .empty
        right: .empty  // or .textButton("Save"), .iconButton(icon)
    ),
    leftAction: { [weak self] in
        self?.navigationController?.popViewController(animated: true)
    },
    rightAction: {
        // Right button action if needed
    }
)
```

**Background Modes:**
- `.onKeyScreen` — Standard screen background
- `.onCard` — Card-style background
- `.transparent` — Transparent navigation bar

## Common Figma → DesignToken Patterns

### Pattern Recognition

When you see Figma token names, extract keywords and match:

**Examples:**
1. "Padding/Section/LeftRight/16px"
   - Keywords: `padding`, `section`, `leftRight`
   - Match: `TymeX.sectionPaddingLeftRight`

2. "Gap/Text/ToText/8px"
   - Keywords: `gap`, `text`, `toText`
   - Match: `TymeX.patternGapTextToText`

3. "Color/Background/Card/Default/#FFFFFF"
   - Keywords: `color`, `background`, `card`, `default`
   - Match: `TymeX.cardColorBackgroundDefault`

4. "Text/Body/M/Default"
   - Keywords: `text`, `body`, `m`
   - Match: `TymeX.textBodyDefaultM`

### Fuzzy Matching Algorithm

```
1. Normalize names: lowercase, remove spaces/slashes
2. Extract key terms: ["card", "padding", "default"]
3. Search DesignToken for properties containing ALL key terms
4. If multiple matches, pick shortest name (more specific)
5. If no match, fallback to raw value
```

## Quick Reference Cheat Sheet

```swift
// SPACING
TymeX.spacing1...12                      // 4px to 54px
TymeX.cardPaddingDefault                 // Card interior
TymeX.sectionGapDefault                  // Between sections
TymeX.sectionPaddingLeftRight            // Screen horizontal margins
TymeX.patternGapTextToText               // Text spacing
TymeX.patternGapTextToElement            // Text to element
TymeX.patternGapTextToSmallIcon          // Text to small icon
TymeX.patternGapElementToElement         // Between elements
TymeX.patternPaddingCircleButtonDefault  // Circle button padding
TymeX.nonCardGapDefault                  // Non-card gap

// COLORS
TymeX.colorBackgroundDefault     // Screen background
TymeX.cardColorBackgroundDefault // Card background
TymeX.patternColorTextDefault    // Primary text
TymeX.patternColorTextSubtle     // Secondary text

// RADIUS
TymeX.cardRadiusDefault          // Card corners
TymeX.patternRadiusDefault       // Pattern corners

// TYPOGRAPHY
TymeX.textDisplayS               // Display text
TymeX.textTitleM                 // Titles
TymeX.textBodyDefaultM           // Body text
```

## Best Practices

### Do NOT Include Pixel Values in Comments

**❌ BAD:**
```swift
// Wrong - redundant comment
$0.top.equalToSuperview().offset(TymeX.sectionGapDefault) // 24px
$0.leading.equalTo(label.snp.trailing).offset(TymeX.patternGapTextToElement) // 4px
```

**✅ GOOD:**
```swift
// Correct - token name is self-documenting
$0.top.equalToSuperview().offset(TymeX.sectionGapDefault)
$0.leading.equalTo(label.snp.trailing).offset(TymeX.patternGapTextToElement)
```

**Why?**
- Design token names are semantic and self-explanatory
- Pixel values in comments become outdated when tokens change
- Cleaner, more maintainable code

### When to Add Comments

Only add comments for:
- Complex layout logic that needs explanation
- Non-obvious constraints or calculations
- Temporary workarounds with context

**Example of good comment:**
```swift
// Adjust for safe area on devices with notch
$0.top.equalTo(safeAreaLayoutGuide).offset(TymeX.spacing4)
```

## Troubleshooting

**Q: Figma shows "24px" but I'm not sure which token to use?**
A: Check the context:
- Between sections? → `TymeX.sectionGapDefault`
- Card spacing? → `TymeX.spacing6`
- If unsure, use `TymeX.spacing6` (raw 24px value)

**Q: Figma color hex doesn't match any DesignToken?**
A: Use closest semantic color. If it's truly custom, use `UIColor(hex: "...")` and add TODO comment to check with design team.

**Q: V2 component doesn't exist?**
A: Keep UI 1.0 component, add comment:
```swift
// TODO: Migrate to UI 2.0 when TymeXComponentV2 is available
let component = TymeXComponentOld()
```
