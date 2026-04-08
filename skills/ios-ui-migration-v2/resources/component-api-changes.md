# Component API Changes: UI 1.0 → UI 2.0

Complete reference guide for migrating component APIs from V1 to V2.

## Table of Contents

1. [Buttons](#buttons)
2. [Navigation](#navigation)
3. [Button Dock](#button-dock)
4. [Lists & Card Lists](#lists--card-lists)
5. [Input Fields](#input-fields)
6. [Intro Screen](#intro-screen)
7. [Search Bar](#search-bar)
8. [General Patterns](#general-patterns)

---

## Buttons

### Primary Button

#### UI 1.0
```swift
let button = TymeXPrimaryButton()
button.setTitle(with: .text("Click Me"))
button.mxSetTitle("Click Me")
button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
button.setShowLoadingForTouchUpInside(true)
```

#### UI 2.0
```swift
let button = TymeXPrimaryButtonV2()

// Set title (same API name as V1 but uses V2 class):
button.setTitle(with: .text("Click Me"))

// Control loading animation on tap:
button.setShowLoadingForTouchUpInside(false)  // or true for auto-loading

// RxSwift binding (preferred over target-action):
button.rx.tap
    .throttle(.seconds(1), scheduler: MainScheduler.asyncInstance)
    .subscribe(onNext: { [weak self] in
        self?.viewModel.input.onAction.onNext(())
    })
    .disposed(by: disposeBag)

// Manual loading state:
button.showLoading { /* completion */ }
button.hideLoading()

// IMPORTANT: Always set translatesAutoresizingMaskIntoConstraints
button.translatesAutoresizingMaskIntoConstraints = false
```

**API Changes:**
- `TymeXPrimaryButton` -> `TymeXPrimaryButtonV2` (class rename)
- `setTitle(with:)` still works in V2
- `setShowLoadingForTouchUpInside()` still works in V2
- `showLoading {}` / `hideLoading()` for manual loading control
- Prefer RxSwift `.rx.tap` with `.throttle(.seconds(1))` over target-action

### Secondary Button

Same API as Primary Button, just use `TymeXSecondaryButtonV2`.

### Tertiary Buttons

```swift
// UI 1.0
let button = TymeXTertiaryButton()

// UI 2.0 - Two variants:
// Contained (filled background):
let containedButton = TymeXTertiaryContainedButtonV2()
containedButton.setTitle(with: .text("Select"))
containedButton.setShowLoadingForTouchUpInside(false)

// Outlined (border only):
let outlinedButton = TymeXTertiaryOutlinedButtonV2()
outlinedButton.setTitle(with: .text("Edit"))
outlinedButton.setShowLoadingForTouchUpInside(false)

// Note: TymeXTertiaryContainedButton (V1) uses different API:
// button.setup(configuration: TymeXButtonConfiguration(contentMode: .text("Title")))
```

### Circle Button

#### UI 1.0
```swift
let button = TymeXCircleButton()
button.setIcon(TymeX().iconClose)
```

#### UI 2.0
```swift
let button = TymeXCircleButtonV2(
    type: .primary,  // or .secondary, .tertiary
    icon: TymeX().iconClose
)
button.mxSetIcon(TymeX().iconClose)
```

**API Changes:**
- Constructor now requires `type` and `icon`
- `setIcon()` → `mxSetIcon()`

---

## Navigation

### Navigation Bar Stylist

#### UI 1.0
```swift
let stylist = TymeXNavigationBarStylist(
    mode: .light(),           // or .dark(), .transparent()
    center: .title("My Title"),
    left: .icon(TymeX().iconArrowLeft),
    right: .text("Done")
)

mxApplyNavigationBy(
    stylist: stylist,
    leftAction: { [weak self] in
        self?.navigationController?.popViewController(animated: true)
    },
    rightAction: { [weak self] in
        self?.saveAction()
    }
)
```

#### UI 2.0
```swift
let stylist = TymeXNavigationBarStylistV2(
    backgroundMode: .defaultMode,  // or .transparent, .onKeyScreen, .onCard
    center: .byDefault(title: "My Title"),
    left: .backButton,             // or .closeButton, .iconButton(icon), .empty
    right: .textButton("Done")     // or .iconButton(icon), .empty
)

mxApplyNavigationV2By(
    stylist: stylist,
    leftAction: { [weak self] in
        self?.navigationController?.popViewController(animated: true)
    },
    rightAction: { [weak self] in
        self?.saveAction()
    }
)
```

**API Changes:**

| UI 1.0 | UI 2.0 | Notes |
|--------|--------|-------|
| `mode: .light()` | `backgroundMode: .defaultMode` | Different enum |
| `mode: .dark()` | `backgroundMode: .defaultMode` | V2 handles dark mode automatically |
| `mode: .transparent()` | `backgroundMode: .transparent` | Similar |
| `.title("Text")` | `.byDefault(title: "Text")` | More explicit |
| `.icon(image)` | `.iconButton(image)` | Clearer naming |
| n/a | `.backButton` | New predefined style |
| n/a | `.closeButton` | New predefined style |
| `.text("Text")` | `.textButton("Text")` | Clearer naming |
| `mxApplyNavigationBy` | `mxApplyNavigationV2By` | V2 suffix |

### Background Modes

| UI 1.0 | UI 2.0 Equivalent | Use Case |
|--------|-------------------|----------|
| `.light()` | `.defaultMode` | Standard white/light background |
| `.transparent()` | `.transparent` | Transparent navigation bar |
| n/a | `.onKeyScreen` | Transparent for gradient backgrounds |
| n/a | `.onKeyScreenOverflow` | Becomes solid when scrolling (gradient bg) |
| n/a | `.onCard` | On card-style background |

### Present Modal Pattern
```swift
// Close button on right side:
func setupPresentNavigationV2() {
    mxApplyNavigationV2By(
        stylist: TymeXNavigationBarStylistV2(
            backgroundMode: .onKeyScreen,
            center: .empty,
            left: .empty,
            right: .icon(TymeX().iconExit)
        ),
        rightAction: { [weak self] in
            self?.dismiss(animated: true)
        }
    )
}
```

---

## Button Dock

### Basic Button Dock

#### UI 1.0
```swift
let dock = TymeXButtonDock(
    needShowLineView: false,
    backgroundColor: TymeX.colorBackgroundDefault,
    buttons: [primaryButton, secondaryButton],
    helperMessage: "This is helper text",
    slotView: customView
)
```

#### UI 2.0
```swift
let dock = TymeXButtonDockV2(
    allowHandleKeyboard: true,
    termsFeeStatus: .none,
    errorMessage: .empty,
    buttons: [primaryButtonV2, secondaryButtonV2],
    helperMessage: "This is helper text",
    slotView: customView,
    displayMode: .defaultState
)
```

**API Changes:**

| UI 1.0 Parameter | UI 2.0 Parameter | Notes |
|-----------------|------------------|-------|
| `needShowLineView` | Removed | Handled by token provider |
| `backgroundColor` | Removed | Handled by token provider |
| n/a | `allowHandleKeyboard` | New - keyboard handling |
| n/a | `termsFeeStatus` | New - terms/conditions UI |
| n/a | `errorMessage` | New - validation error display |
| `buttons` | `buttons` | Must use V2 buttons |
| `helperMessage` | `helperMessage` | Same |
| `slotView` | `slotView` | Same |
| n/a | `displayMode` | New - layout control |

### Terms/Fee Status Options

```swift
// UI 2.0 only
termsFeeStatus: .none                    // No terms/fees
termsFeeStatus: .withCheckbox            // Show checkbox for terms
termsFeeStatus: .withoutCheckbox         // Show terms without checkbox
```

### Display Modes

```swift
// UI 2.0 only
displayMode: .defaultState               // Standard bottom dock
displayMode: .overflowState              // Overflow scrolling mode
```

---

## Lists & Card Lists

### List Item

#### UI 1.0
```swift
let item = TymeXListItemModelV2(
    leadingStatus: TymeXLeadingStatus.icon(iconImage, false),
    leadingContent: TymeXLeadingContent(
        title: "Main Title",
        subTitle1: "Subtitle 1",
        subTitle2: "Subtitle 2",
        isHighlightTitle: true
    ),
    trailingStatus: TymeXTrailingStatus.text("Value"),
    listType: .standard
)
```

#### UI 2.0
```swift
let item = TymeXCardListItemV2(
    leadingStatus: .icon(iconImage, false),  // Enum shorthand
    leadingContent: TymeXLeadingContentV2(
        title: "Main Title",
        subTitle1: "Subtitle 1",
        subTitle2: "Subtitle 2",
        isHighlightTitle: true
    ),
    trailingContentStatus: .text("Value"),   // New parameter
    trailingStatus: nil,                     // Separate from content
    listType: .standard                      // or .infor
)
```

**API Changes:**
- `TymeXListItemModelV2` → `TymeXCardListItemV2`
- `TymeXLeadingStatus` → Use enum shorthand (`.icon()`)
- `TymeXLeadingContent` → `TymeXLeadingContentV2`
- `trailingStatus` split into `trailingContentStatus` and `trailingStatus`

### List Configuration

#### UI 1.0
```swift
let model = TymeXListModelV2(
    shouldClearBackgroundColor: true,
    items: [item1, item2, item3]
)

let listView = TymeXListViewV2()
listView.configuration(with: model)
```

#### UI 2.0
```swift
let config = TymeXCardListConfiguration(
    cardListMode: .standard,          // or .containerMode
    showDivider: false,
    isForcePaddingLeftForLeading: false,
    valueForcePaddingLeftForLeading: -16.0,
    multipleSectionItems: [[item1, item2, item3]]
)

let cardList = TymeXCardListV2()
cardList.configuration(with: config)
```

**API Changes:**

| UI 1.0 | UI 2.0 | Notes |
|--------|--------|-------|
| `TymeXListModelV2` | `TymeXCardListConfiguration` | Different type |
| `shouldClearBackgroundColor` | `cardListMode` | Mode-based instead of boolean |
| `items: [Item]` | `multipleSectionItems: [[Item]]` | Supports multiple sections |
| n/a | `showDivider` | New - control divider visibility |
| n/a | `isForcePaddingLeftForLeading` | New - override padding |

### List Callbacks

#### UI 1.0
```swift
listView.willDisplayItem = { item, cell in
    guard let item = item else { return }
    // Configure cell
}

listView.didSelectItem = { index, item in
    // Handle selection
}
```

#### UI 2.0
```swift
cardList.willDisplayItem = { item, cell, indexPath in
    guard let item = item else { return }
    // Configure cell - now includes indexPath
}

cardList.didSelectIndexPath = { indexPath in
    // Handle selection by indexPath
}

cardList.didSelectItem = { index, item in
    // Still available for single-section lists
}
```

**API Changes:**
- `willDisplayItem` now includes `indexPath` parameter
- Added `didSelectIndexPath` for multi-section support

---

## Input Fields

### Text Input

#### UI 1.0
```swift
let textField = TymeXTextField()
textField.placeholder = "Enter your name"
textField.delegate = self
textField.font = UIFont.systemFont(ofSize: 16)
```

#### UI 2.0
```swift
let inputField = TymeXInputTextFieldV2()
inputField.mxSetup(
    configuration: TymeXInputTextFieldConfiguration(
        label: "Name",
        placeholder: "Enter your name",
        helperMessage: "This field is required",
        maxLength: 50
    )
)

// RxSwift binding
inputField.rx.text
    .bind(to: viewModel.input.nameText)
    .disposed(by: disposeBag)

// State management
inputField.mxUpdateState(.error("Invalid name"))
inputField.mxUpdateState(.success)
inputField.mxUpdateState(.normal)
```

**API Changes:**
- Configuration-based setup
- Built-in label, placeholder, helper message
- Built-in state management (error, success, normal)
- RxSwift-first design

### Amount Input

#### UI 1.0
```swift
let amountField = TymeXAmountInputField()
amountField.currencySymbol = "₱"
amountField.maxAmount = 1_000_000
```

#### UI 2.0
```swift
let amountField = TymeXAmountInputTextFieldV2()
amountField.mxSetup(
    configuration: TymeXAmountInputConfiguration(
        label: "Amount",
        currencySymbol: "₱",
        maxAmount: 1_000_000,
        helperMessage: "Min: ₱100"
    )
)
```

---

## Intro Screen

#### UI 1.0
```swift
let introScreen = TymeXIntroScreen(
    isHiddenNav: true,
    titleNavigationBar: "Welcome",
    subTitleNavigationBar: "Get Started",
    topContentView: UIImageView(image: logoImage),
    topContentTitle: "Welcome to App",
    tymeXListViewV2: listView,
    buttonDock: buttonDock
)
```

#### UI 2.0
```swift
let introScreen = TymeXIntroScreenV2(
    isHiddenNav: true,
    rightItemStyle: .empty,
    leftCompletion: nil,
    rightCompletion: nil,
    topContentView: UIImageView(image: logoImage),
    topContentTitle: "Welcome to App",
    cardListV2: cardListV2,        // Must be V2
    buttonDock: buttonDockV2       // Must be V2
)
```

**API Changes:**

| UI 1.0 | UI 2.0 | Notes |
|--------|--------|-------|
| `titleNavigationBar` | Removed | Use navigation API instead |
| `subTitleNavigationBar` | Removed | Use navigation API instead |
| n/a | `rightItemStyle` | New - navigation item |
| n/a | `leftCompletion` | New - left button action |
| n/a | `rightCompletion` | New - right button action |
| `tymeXListViewV2` | `cardListV2` | Must use `TymeXCardListV2` |
| `buttonDock` | `buttonDock` | Must use `TymeXButtonDockV2` |

---

## Search Bar

#### UI 1.0
```swift
let searchBar = TymeXSearchBar()
searchBar.placeholder = "Search..."
searchBar.delegate = self
```

#### UI 2.0
```swift
let searchBar = TymeXSearchBarV2()
searchBar.mxSetup(
    configuration: TymeXSearchBarConfiguration(
        placeholder: "Search...",
        cancelButtonTitle: "Cancel"
    )
)

// RxSwift binding
searchBar.rx.text
    .bind(to: viewModel.input.searchQuery)
    .disposed(by: disposeBag)
```

---

## General Patterns

### 1. Configuration Objects

Most V2 components use configuration structs for setup:

```swift
component.setup(configuration: ConfigurationType(...))
// or
component.mxSetup(configuration: ConfigurationType(...))
```

### 2. RxSwift-First Design

V2 components are designed for reactive programming:

```swift
// Events
button.rx.tap
textField.rx.text
searchBar.rx.searchButtonClicked

// Binding
observable.bind(to: component.rx.property).disposed(by: disposeBag)
```

### 3. Token Providers

V2 components use token providers for consistent styling:

```swift
let component = ComponentV2(
    tokenProvider: ComponentV2TokenProvider.shared
)
```

You rarely need to pass custom token providers.

### 4. State Management

V2 input components have built-in state management:

```swift
inputField.mxUpdateState(.normal)
inputField.mxUpdateState(.error("Error message"))
inputField.mxUpdateState(.success)
inputField.mxUpdateState(.disabled)
```

### 5. Child View Controller Lifecycle

When using container components like `TymeXIntroScreenV2`:

```swift
let childVC = TymeXIntroScreenV2(...)

// Proper lifecycle management
addChild(childVC)
containerView.addSubview(childVC.view)
childVC.didMove(toParent: self)

// Setup constraints
childVC.view.snp.makeConstraints { make in
    make.edges.equalToSuperview()
}
```

### 6. Enum Shorthand

V2 uses Swift enum shorthand extensively:

```swift
// Instead of:
leadingStatus: TymeXLeadingStatusV2.icon(image, false)

// Use:
leadingStatus: .icon(image, false)
```

### 7. Method Naming Conventions

V2 components use consistent prefixes:

- `mxSetup()` - Initial configuration
- `mxSetTitle()` - Set title/text
- `mxUpdateState()` - Change component state
- `mxShowLoading()` - Toggle loading state
- `mxSetIcon()` - Set icon/image

---

## Migration Checklist

When migrating a component:

- [ ] Change class name (add V2 suffix)
- [ ] Update initialization parameters
- [ ] Replace method calls with V2 API
- [ ] Convert target-action to RxSwift if applicable
- [ ] Update configuration to use configuration objects
- [ ] Verify token provider is passed if needed
- [ ] Update any delegate protocols to V2 versions
- [ ] Test all user interactions work correctly

---

## Common Mistakes

### ❌ Mixing V1 Class with V2 Dock

```swift
// WRONG - ButtonDock V2 requires V2 button classes
let button = TymeXPrimaryButton()  // V1 class!
let dock = TymeXButtonDockV2(buttons: [button])  // Won't work
```

### ❌ Mixing V1 and V2

```swift
// WRONG - ButtonDock V2 requires V2 buttons
let dock = TymeXButtonDockV2(
    buttons: [TymeXPrimaryButton()]  // V1 button in V2 dock
)
```

### ❌ Forgetting translatesAutoresizingMaskIntoConstraints

```swift
// WRONG - Will have Auto Layout issues
let button = TymeXPrimaryButtonV2()
button.setTitle(with: .text("Click"))
// Missing: button.translatesAutoresizingMaskIntoConstraints = false

// CORRECT
let button = TymeXPrimaryButtonV2()
button.setTitle(with: .text("Click"))
button.translatesAutoresizingMaskIntoConstraints = false
```

### ❌ Not Using RxSwift

```swift
// WORKS but not recommended for V2
button.addTarget(self, action: #selector(tapped), for: .touchUpInside)

// PREFERRED - RxSwift binding
button.rx.tap.bind(to: viewModel.input.action).disposed(by: disposeBag)
```

---

## Quick Reference

| Component Type | V1 → V2 Class Name |
|----------------|-------------------|
| Primary Button | `TymeXPrimaryButton` → `TymeXPrimaryButtonV2` |
| Secondary Button | `TymeXSecondaryButton` → `TymeXSecondaryButtonV2` |
| Tertiary Button | `TymeXTertiaryButton` → `TymeXTertiaryContainedButtonV2` / `TymeXTertiaryOutlinedButtonV2` |
| Circle Button | `TymeXCircleButton` → `TymeXCircleButtonV2` |
| Button Dock | `TymeXButtonDock` → `TymeXButtonDockV2` |
| Text Input | `TymeXTextField` → `TymeXInputTextFieldV2` |
| Amount Input | `TymeXAmountInputField` → `TymeXAmountInputTextFieldV2` |
| Phone Input | `TymeXPhoneInputView` → `TymeXPhoneInputViewV2` |
| OTP Input | `TymeXOTPInputView` → `TymeXOTPInputViewV2` |
| Passcode Input | `TymeXPasscodeInputView` → `TymeXPasscodeInputViewV2` |
| Search Bar | `TymeXSearchBar` → `TymeXSearchBarV2` |
| List View | `TymeXListViewV2` → `TymeXCardListV2` |
| Intro Screen | `TymeXIntroScreen` → `TymeXIntroScreenV2` |
| Navigation Stylist | `TymeXNavigationBarStylist` → `TymeXNavigationBarStylistV2` |
| Result Screen | n/a → `TymeXResultScreenViewControllerV2` |
| Transaction Confirm | n/a → `TymeXTransactionConfirmationViewControllerV2` |
| Modal Alert | `TymeXSheetAlertViewController` → `TymeXSheetModalAlertViewControllerV2` |
| Action Modal | `TymeXActionModalPresentable` → `TymeXActionModalPresentableV2` |
| Present Modal | `mxPresentActionModal()` → `mxPresentActionModalV2()` |
| Modal Height | `TymeXActionModalHeight` → `TymeXActionModalHeightV2` |
| Banner Action | n/a → `TymeXBannerActionViewV2` |
| Spending Bar | n/a → `TymeXSpendingBarV2` |
| Transaction Detail List | n/a → `TymeXTransactionDetailCardListV2` |

---

**Remember**: V2 is not just a rename — it's a complete API redesign. Always check the V2 component's public interface before migrating!
