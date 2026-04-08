---
name: ios-ui-migration-v2
description: |
  Migrate iOS screens from UI 1.0 to UI 2.0 design system. Converts XIB-based
  layouts to programmatic SnapKit constraints, replaces old components with V2
  equivalents, and integrates DesignToken (TymeX.*). Uses Figma MCP to extract
  exact spacing/colors. Triggers when user says "migrate to UI 2.0", "convert
  to new design", "update to V2 components", "remove XIB and use SnapKit",
  "apply new design tokens", or mentions "UI 2.0 migration".
---

# Goal

Transform UI 1.0 ViewControllers (XIB-based, old components) to UI 2.0
(programmatic SnapKit, V2 components, DesignToken) in under 10 minutes per
screen, ensuring pixel-perfect match with Figma designs.

# Instructions

## Step 1: Gather Context

1. Ask user: "Which ViewController do you want to migrate? Provide the file path."
2. Ask: "Do you have a Figma design URL? (Optional but recommended for accuracy)"
3. Read the target ViewController file
4. If XIB exists (same name as ViewController), read it to understand current layout
5. If Figma URL provided, use Figma MCP to fetch design specs:
   - Component names (e.g., "Button/Primary/V2")
   - Spacing values (e.g., "16px", "24px")
   - Color names (e.g., "Background/Card/Default")
   - Typography styles

## Step 2: Analyze Current Implementation

Parse the existing code to identify:
- All UI components (buttons, labels, text fields, etc.)
- Layout constraints (if in code) or XIB structure
- Color/font hardcoded values
- Navigation bar setup

Create a mapping table:
```
| Current Component | Figma Name | UI 2.0 Equivalent | Status |
|-------------------|------------|-------------------|--------|
| UIButton          | Button/Primary | TymeXPrimaryButtonV2 | ✅ Available |
| UITextField       | Input/Text | TymeXInputTextFieldV2 | ✅ Available |
```

## Step 3: Check V2 Component Availability

Browse `PodLocals/ios-design-component/Current/TymeXUIComponent/UI2.0/` to verify
all needed V2 components exist. If a component is missing:
- Check if there's a similar V2 component that can be used
- Flag it to user: "Component X doesn't have V2 equivalent yet. Should I keep the old one or find an alternative?"

## Step 3.5: Component API Migration Guide

**CRITICAL**: V2 components don't just have different names — they have **completely different APIs**. You cannot simply rename the component; you must change parameters and method calls.

### Common Component Migrations:

#### 1. Buttons (TymeXPrimaryButton → TymeXPrimaryButtonV2)

```swift
// ❌ UI 1.0 API
let button = TymeXPrimaryButton()
button.setTitle(with: .text("Button Title"))
button.mxSetTitle("Button Title")
button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)

// ✅ UI 2.0 API — Factory method (RECOMMENDED):
let button = makePrimaryButtonV2(
    title: "Button Title",
    disposeBag: disposeBag,
    isShowLoadingAnimation: false,
    action: { [weak self] in
        self?.viewModel.input.onAction.onNext(())
    }
)

// ✅ UI 2.0 API — Direct init:
let button = TymeXPrimaryButtonV2()
button.setTitle(with: .text("Button Title"))
button.setShowLoadingForTouchUpInside(false)
button.translatesAutoresizingMaskIntoConstraints = false
// Use RxSwift binding with throttle:
button.rx.tap.throttle(.seconds(1), scheduler: MainScheduler.asyncInstance)
    .subscribe(onNext: { [weak self] in
        self?.viewModel.input.onAction.onNext(())
    })
    .disposed(by: disposeBag)
// Loading state:
button.showLoading { /* completion */ }
button.hideLoading()
```

**Key Changes:**
- `TymeXPrimaryButton` → `TymeXPrimaryButtonV2` (class name change)
- Factory methods available: `makePrimaryButtonV2()`, `makeSecondaryButtonV2()`
- `setTitle(with:)` works the same in V2
- `setShowLoadingForTouchUpInside()` works the same in V2
- Prefer RxSwift `.rx.tap` with `.throttle(.seconds(1))` over target-action
- Manual loading: `showLoading {}` / `hideLoading()`

#### 2. Navigation (TymeXNavigationBarStylist → TymeXNavigationBarStylistV2)

```swift
// ❌ UI 1.0 API
let stylist = TymeXNavigationBarStylist(
    mode: .light(),
    center: .title("Title"),
    left: .icon(TymeX().iconArrowLeft),
    right: .text("Done")
)
mxApplyNavigationBy(stylist: stylist, leftAction: { ... })

// ✅ UI 2.0 API
let stylist = TymeXNavigationBarStylistV2(
    backgroundMode: .defaultMode,  // or .transparent, .onKeyScreen, .onKeyScreenOverflow, .onCard
    center: .byDefault(title: "Title"),
    left: .backButton,  // or .closeButton, .iconButton(icon)
    right: .textButton("Done")  // or .iconButton(icon), .empty
)
mxApplyNavigationV2By(stylist: stylist, leftAction: { ... })
```

**Key Changes:**
- `mode:` → `backgroundMode:` with different enum values
- `.icon()` → `.iconButton()` or predefined `.backButton`/`.closeButton`
- `.title()` → `.byDefault(title:)`
- `.text()` → `.textButton()`
- `mxApplyNavigationBy` → `mxApplyNavigationV2By`

#### 3. ButtonDock (TymeXButtonDock → TymeXButtonDockV2)

```swift
// ❌ UI 1.0 API
let dock = TymeXButtonDock(
    needShowLineView: false,
    backgroundColor: TymeX.colorBackgroundDefault,
    buttons: [button1, button2],
    helperMessage: "Helper text",
    slotView: nil
)

// ✅ UI 2.0 API
let dock = TymeXButtonDockV2(
    allowHandleKeyboard: true,
    termsFeeStatus: .none,  // or .terms("Helper message", false)
    errorMessage: .empty,
    buttons: [button1V2, button2V2],  // Must be V2 buttons
    helperMessage: "Helper text",
    slotView: nil,
    displayMode: .defaultState  // or .overflowState
)

// ButtonDock methods:
dock.showLineView(flag: true)
dock.showTermView(flag: true)
dock.showErrorMessage(message: "Error text", flag: true)

// Layout — always pinned to bottom:
dock.snp.makeConstraints { make in
    make.top.greaterThanOrEqualTo(contentView.snp.bottom).offset(TymeX.sectionGapDefault)
    make.leading.trailing.equalToSuperview()
    make.bottom.equalToSuperview()
}
```

**Key Changes:**
- `needShowLineView` → removed
- `backgroundColor` → handled by token provider
- Added `termsFeeStatus` for terms/conditions handling
- Added `errorMessage` for validation errors
- Added `displayMode` for layout control
- `buttons` must be array of `TymeXBaseButtonV2` (V2 buttons only)

#### 4. List/CardList (TymeXListViewV2 → TymeXCardListV2)

```swift
// ❌ UI 1.0 API
let item = TymeXListItemModelV2(
    leadingStatus: TymeXLeadingStatus.icon(image, false),
    leadingContent: TymeXLeadingContent(title: "Title", subTitle1: "Subtitle"),
    trailingStatus: nil,
    listType: .standard
)
let model = TymeXListModelV2(
    shouldClearBackgroundColor: true,
    items: [item]
)
let listView = TymeXListViewV2()
listView.configuration(with: model)

// ✅ UI 2.0 API
let item = TymeXCardListItemV2(
    leadingStatus: .icon(image, false),  // Enum shorthand
    leadingContent: TymeXLeadingContentV2(title: "Title", subTitle1: "Subtitle"),
    trailingContentStatus: nil,
    trailingStatus: nil,
    listType: .standard
)
let config = TymeXCardListConfiguration(
    cardListMode: .standard,  // or .containerMode
    showDivider: false,
    multipleSectionItems: [[item]]  // Array of arrays for sections
)
let cardListV2 = TymeXCardListV2(tokenProvider: TymeXCardListV2TokenProvider.shared)
cardListV2.configuration(with: config)

// Info list item (key-value pairs for detail/confirmation screens):
let infoItem = TymeXCardListItemV2(
    leadingContent: TymeXLeadingContentV2(title: "Label"),
    trailingContentStatus: TymeXTrailingContentStatusV2.itemContent(
        TymeXTrailingContentItemV2(
            title: "Value",
            subTitle1: nil,
            isHighlightTitle: false
        )
    ),
    listType: .infor  // key-value layout
)
```

**Key Changes:**
- `TymeXListItemModelV2` → `TymeXCardListItemV2`
- `TymeXLeadingStatus` → enum shorthand (`.icon()` instead of `TymeXLeadingStatus.icon()`)
- `TymeXLeadingContent` → `TymeXLeadingContentV2`
- `TymeXListModelV2` → `TymeXCardListConfiguration`
- `items:` → `multipleSectionItems:` (array of arrays)
- `shouldClearBackgroundColor` → `cardListMode`
- Cell callbacks: `willDisplayItem` now includes `indexPath` parameter
- Two list types: `.standard` (icon + title/subtitle) and `.infor` (key-value pairs)
- Init with token provider: `TymeXCardListV2(tokenProvider: TymeXCardListV2TokenProvider.shared)`

#### 5. IntroScreen (TymeXIntroScreen → TymeXIntroScreenV2)

```swift
// ❌ UI 1.0 API
let introScreen = TymeXIntroScreen(
    isHiddenNav: true,
    titleNavigationBar: .empty,
    subTitleNavigationBar: .empty,
    topContentView: imageView,
    topContentTitle: "Title",
    tymeXListViewV2: listView,
    buttonDock: buttonDock
)

// ✅ UI 2.0 API
let introScreen = TymeXIntroScreenV2(
    isHiddenNav: true,
    rightItemStyle: .empty,
    leftCompletion: nil,
    rightCompletion: nil,
    topContentView: imageView,
    topContentTitle: "Title",
    cardListV2: cardListV2,  // Note: now TymeXCardListV2
    buttonDock: buttonDockV2  // Must be V2
)
```

**Key Changes:**
- `titleNavigationBar`/`subTitleNavigationBar` → removed
- Added `rightItemStyle`, `leftCompletion`, `rightCompletion`
- `tymeXListViewV2` → `cardListV2` (TymeXCardListV2 type)
- `buttonDock` must be `TymeXButtonDockV2`

#### 6. Input Fields (TymeXTextField → TymeXInputTextFieldV2)

```swift
// ❌ UI 1.0 API
let textField = TymeXTextField()
textField.placeholder = "Enter text"
textField.delegate = self

// ✅ UI 2.0 API
let inputField = TymeXInputTextFieldV2()
inputField.mxSetup(
    configuration: TymeXInputTextFieldConfiguration(
        label: "Label",
        placeholder: "Enter text",
        helperMessage: "Helper text"
    )
)
// Use RxSwift for text changes
inputField.rx.text.bind(to: viewModel.input.textValue).disposed(by: disposeBag)
```

**Key Changes:**
- Configuration-based setup via `mxSetup(configuration:)`
- Built-in label and helper message support
- RxSwift reactive bindings recommended

### General V2 API Patterns:

1. **Configuration Objects**: Most V2 components use configuration structs
   ```swift
   component.setup(configuration: ConfigType(...))
   ```

2. **RxSwift First**: V2 components designed for reactive programming
   ```swift
   button.rx.tap.bind(to: ...).disposed(by: disposeBag)
   ```

3. **Token Providers**: V2 components use token providers for styling
   ```swift
   let component = ComponentV2(tokenProvider: TokenProviderV2.shared)
   ```

4. **Enum Shorthand**: Use dot syntax for enum cases
   ```swift
   // Instead of: TymeXLeadingStatus.icon(...)
   // Use: .icon(...)
   ```

5. **Child View Controllers**: Properly manage lifecycle
   ```swift
   addChild(viewController)
   containerView.addSubview(viewController.view)
   viewController.didMove(toParent: self)
   ```

### Additional V2 Components (from production usage):

#### 7. Result Screen (TymeXResultScreenViewControllerV2)
```swift
// Success screen:
let result = TymeXResultScreenViewControllerV2(
    title: "Success Title",
    subTitle: "Description text",
    buttonDock: makeButtonDockView(buttons: [primaryBtn, secondaryBtn]),
    lottieView: makeLottieView()  // LottieAnimationView
)
result.setShowCloseButton(isShow: false)

// Error/Processing screen:
let result = TymeXResultScreenViewControllerV2(
    title: "Processing Title",
    subTitle: "Description",
    buttonDock: makeButtonDockView(buttons: [transparentButton]),
    lottieView: makeAnimationError(isProcressing: true),
    onCloseCompletion: { /* handle close */ }
)
```

#### 8. Transaction Confirmation (TymeXTransactionConfirmationViewControllerV2)
```swift
let txVC = TymeXTransactionConfirmationViewControllerV2(
    title: "Confirm Transfer",
    amount: 1000.0,
    amountCurrencySymbol: "$",
    subTitle: "Optional subtitle",
    helperMessage: "Optional helper",
    totalAmount: nil,
    buttonDock: buttonDock,
    slotsView: cardList,
    footerLeftTitle: "",
    onCloseCompletion: { self?.dismiss(animated: true) }
)
addChild(txVC)
containerView.addSubview(txVC.view)
txVC.view.snp.makeConstraints { $0.edges.equalTo(containerView) }
txVC.didMove(toParent: self)
```

#### 9. Modal Alert (TymeXSheetModalAlertViewControllerV2)
```swift
var actions: [TymeXBaseButtonV2] = []
let primaryAction = TymeXPrimaryButtonV2()
primaryAction.setTitle(with: .text("Close"))
primaryAction.rx.tap.subscribe(onNext: { _ in
    alertVC.dismiss(animated: true)
}).disposed(by: disposeBag)
actions.append(primaryAction)

let alertVC = TymeXSheetModalAlertViewControllerV2(
    iconView: nil,
    titleContent: "Alert Title",
    subTitleContent: "Alert message",
    actions: actions
)
viewController.mxPresentActionModalV2(alertVC)
```

#### 10. Bottom Sheet (TymeXActionModalPresentableV2)
```swift
extension MyViewController: TymeXActionModalPresentableV2 {
    var actionScrollable: UIScrollView? { return nil }
    var mxShortFormHeight: TymeXActionModalHeightV2 {
        return .contentHeightIgnoringSafeArea(mainView.bounds.height + 24)
    }
    var mxLongFormHeight: TymeXActionModalHeightV2 { return mxShortFormHeight }
    var mxShowDragIndicator: Bool { return true }
    var mxAllowsTapToDismiss: Bool { return true }
}
```

#### 11. Banner Action (TymeXBannerActionViewV2)
```swift
let bannerView = TymeXBannerActionViewV2()
bannerView.configBanner(
    title: "Term",
    subTitle: "6 months (5.5%)",
    subTitleColor: TymeX.patternColorTextSubtle,
    rightButton: selectTermButton  // TymeXTertiaryContainedButtonV2
)
```

#### 12. Spending Bar (TymeXSpendingBarV2)
```swift
let spendingBar = TymeXSpendingBarV2(navigationBackgroundMode: .defaultMode)
spendingBar.leftTitle = "Available Balance"
spendingBar.rightTitle = "Balance After"
spendingBar.balanceValue = Decimal(10000)
spendingBar.paymentValue = Decimal(5000)
```

#### 13. Amount Input (TymeXAmountInputTextFieldV2)
```swift
let inputAmountTextField = TymeXAmountInputTextFieldV2()
inputAmountTextField.currencySettings = TymeXCurrencySettingsV2(currencySymbol: "$")
inputAmountTextField.setHelperMessage(.text("Available: $1,000"))
inputAmountTextField.showMessage(with: "Error text", isError: true)
// Access value: inputAmountTextField.value (Decimal)
// Access text field: inputAmountTextField.textField.rx.text
```

### Gradient Background Pattern (for key screens):
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    let gradientLayer = view.mxAddGradientBackgroundV2()
    view.layer.insertSublayer(gradientLayer, at: 0)
}

override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    if let gradientLayer = view.layer.sublayers?.first as? CAGradientLayer {
        gradientLayer.frame = view.bounds
    }
}
```

### ScrollView Content Layout Pattern:
```swift
let scrollView = UIScrollView()
scrollView.contentInsetAdjustmentBehavior = .never
let contentView = UIView()
scrollView.addSubview(contentView)

contentView.snp.makeConstraints { make in
    make.edges.equalTo(scrollView.contentLayoutGuide)
    make.width.equalTo(scrollView.frameLayoutGuide)
    make.height.greaterThanOrEqualToSuperview()
}
```

## Step 4: Map Design Tokens

From Figma or existing code, map values to DesignToken:

**CRITICAL COLOR RULE**: ALL colors MUST use `TymeX.patternColor*` prefix (NOT
`TymeX.color*` which is UI 1.0). Only exceptions are screen/card/section level
tokens which keep their own prefix:
- `TymeX.screenColorBackgroundDefault` (screen level)
- `TymeX.cardColor*` (card level)
- `TymeX.sectionColor*` (section level)

**Spacing:**
- 4px → `TymeX.spacing1`
- 8px → `TymeX.spacing2`
- 12px → `TymeX.spacing3`
- 16px → `TymeX.spacing4` or `TymeX.cardPaddingDefault`
- 24px → `TymeX.spacing6` or `TymeX.sectionGapDefault`

**Semantic spacing:**
- Card padding → `TymeX.cardPaddingDefault`
- Card gap → `TymeX.cardGapDefault`
- Section gap → `TymeX.sectionGapDefault`
- Section horizontal padding → `TymeX.sectionPaddingLeftRight`
- Screen gap → `TymeX.screenGapDefault`
- Group card gap → `TymeX.groupCardGapDefault`
- Non-card gap → `TymeX.nonCardGapDefault`
- Text to text → `TymeX.patternGapTextToText`
- Text to element → `TymeX.patternGapTextToElement`
- Text to small icon → `TymeX.patternGapTextToSmallIcon`
- Element to element → `TymeX.patternGapElementToElement`
- Text to button → `TymeX.patternGapTextToButton`
- List item top/bottom → `TymeX.patternPaddingListListItemTopBottom`

**Colors (use patternColor prefix!):**
- Screen background → `TymeX.screenColorBackgroundDefault`
- Card background → `TymeX.cardColorBackgroundDefault`
- Card bg on key screen → `TymeX.cardColorBackgroundOnKeyScreen`
- Section background → `TymeX.sectionColorBackgroundDefault`
- Text primary → `TymeX.patternColorTextDefault`
- Text secondary → `TymeX.patternColorTextSubtle`
- Text link → `TymeX.patternColorTextLink`
- Text on primary → `TymeX.patternColorTextOnPrimary`
- Background default → `TymeX.patternColorBackgroundDefault`
- Background info → `TymeX.patternColorBackgroundInfoBase`
- Background select → `TymeX.patternColorBackgroundSelectBase`
- Stroke info → `TymeX.patternColorStrokeInfoBase`
- Stroke select → `TymeX.patternColorStrokeSelect`
- Stroke error → `TymeX.patternColorStrokeErrorBase`
- Divider → `TymeX.patternColorDividerDividerBase`

**Radius:**
- Card corner → `TymeX.cardRadiusDefault`
- Pattern radius → `TymeX.patternRadiusDefault` (8px, skeleton lines, tags)
- Small radius → `TymeX.cornerRadius5` (5pt, body section top corners)

**Typography:**
- Body text → `TymeX.textBodyDefaultM`
- Body large → `TymeX.textBodyDefaultL`
- Body small → `TymeX.textBodyDefaultS`
- Body emphasized → `TymeX.textBodyEmphasizeM`
- Title → `TymeX.textTitleM`
- Title small → `TymeX.textTitleS`
- Display → `TymeX.textDisplayS`
- Display medium → `TymeX.textDisplayM`
- Display XL → `TymeX.textDisplayXl`
- Label emphasized → `TymeX.textLabelEmphasizeS`

**Typography chaining** — use `.color().alignment()`:
```swift
// ✅ CORRECT:
TymeX.textBodyDefaultM.color(TymeX.patternColorTextDefault).alignment(.left)

// ✅ Highlighted text with multiple styles:
label.highlightText(
    fullText: "Total: $1,000",
    boldedTexts: ["$1,000"],
    textColor: TymeX.patternColorTextDefault,
    boldFontAttribute: TymeX.textBodyEmphasizeM
)

// ❌ WRONG — do NOT use .paragraphStyle(lineSpacing:alignment:)
```

Match Figma token names with DesignToken properties by name similarity.

## Step 5: Generate UI 2.0 Code

Rewrite the ViewController following this structure:

```swift
import UIKit
import TymeXCore
import TymeXUIComponent
import DesignToken
import SnapKit

final class YourViewController: BaseViewController<YourViewModel> {
    // MARK: - UI Components
    private let componentName: TymeXComponentV2 = {
        let component = TymeXComponentV2()
        component.translatesAutoresizingMaskIntoConstraints = false
        return component
    }()

    // MARK: - Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
        setupViews()
        setupConstraints()
        setupStyling()
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        setupNavigation()
    }

    // MARK: - Setup
    private func setupViews() {
        view.addSubview(componentName)
        // Add all subviews to hierarchy
    }

    private func setupConstraints() {
        componentName.snp.makeConstraints { make in
            make.top.equalToSuperview().inset(TymeX.sectionGapDefault)
            make.leading.trailing.equalToSuperview().inset(TymeX.cardPaddingDefault)
        }
    }

    private func setupStyling() {
        view.backgroundColor = TymeX.screenColorBackgroundDefault
        // Apply colors, fonts via DesignToken
    }

    private func setupNavigation() {
        mxApplyNavigationV2By(
            stylist: TymeXNavigationBarStylistV2(
                backgroundMode: .onKeyScreen,
                center: .byDefault(title: "Your Title"),
                left: .backButton,
                right: .empty
            ),
            leftAction: { [weak self] in
                self?.navigationController?.popViewController(animated: true)
            }
        )
    }
}
```

**Key migration rules:**
- **All properties**: `translatesAutoresizingMaskIntoConstraints = false`
- **Remove**: Any XIB loading code (`loadNibNamed`, `fromNib()`)
- **Replace**: Old navigation setup with `mxApplyNavigationV2By`
- **Use**: SnapKit `.snp.makeConstraints` for ALL constraints
- **Typography**: Use `NSAttributedString` with `TymeX.text*` instead of `UIFont`
  ```swift
  label.attributedText = NSAttributedString(
      string: "Text",
      attributes: TymeX.textBodyDefaultM.color(TymeX.patternColorTextDefault)
  )
  ```

## Step 6: Update Component Bindings

If the ViewController uses RxSwift bindings:
- Keep all `.rx` bindings intact
- Update only the UI component references to V2 versions
- Ensure V2 components support the same reactive properties

If binding fails, check V2 component documentation in UI2.0 directory.

## Step 7: Clean Up

1. **Delete the XIB file** (ask user for confirmation first)
   ```bash
   rm path/to/YourViewController.xib
   ```

2. **Remove XIB reference from project.pbxproj**
   - Open `GoalSave.xcodeproj/project.pbxproj` in a text editor
   - Search for the XIB filename (e.g., `GSIntroductionViewController.xib`)
   - Remove all lines containing this XIB reference
   - Or use git to check changes:
   ```bash
   git status GoalSave.xcodeproj/project.pbxproj
   git add GoalSave.xcodeproj/project.pbxproj
   ```
   - **IMPORTANT**: Xcode may auto-remove the reference when you open the project, but verify manually to be safe

3. **Run `pod install`** after deleting XIB files — this ensures CocoaPods properly updates project references and removes stale XIB entries from the build system
   ```bash
   pod install
   ```

4. **Clean Xcode build folder** to remove any cached XIB references
   ```bash
   # In Xcode: Product → Clean Build Folder (⌘ + Shift + K)
   # Or via command line:
   rm -rf ~/Library/Developer/Xcode/DerivedData
   ```

5. Clean up unused imports (e.g., remove `import Reusable` if only used for XIB)

6. Run SwiftLint and fix any warnings

## Step 8: Verification Checklist

Present this checklist to user:
- [ ] All UI components replaced with V2 equivalents
- [ ] All constraints use SnapKit
- [ ] All spacing uses DesignToken (TymeX.*)
- [ ] All colors use DesignToken
- [ ] Navigation uses V2 API
- [ ] XIB file deleted from filesystem
- [ ] XIB reference removed from `project.pbxproj`
- [ ] `pod install` executed after XIB deletion
- [ ] Xcode build folder cleaned (⌘ + Shift + K)
- [ ] Code compiles without errors
- [ ] Layout matches Figma design (if provided)

Ask: "Ready to test? Please run `pod install` if not done yet, clean build folder (⌘ + Shift + K), then build and run to verify the layout."

# Examples

## Example 1: Simple Screen with Button and Label

**Input:**
```
User: "Migrate FDClosedListViewController to UI 2.0"
Figma: https://www.figma.com/design/abc123?node-id=2058-26280
```

**Current Code (UI 1.0):**
```swift
// FDClosedListViewController.swift
class FDClosedListViewController: BaseViewController<FDClosedListViewModel> {
    @IBOutlet weak var titleLabel: UILabel!
    @IBOutlet weak var actionButton: UIButton!

    override func viewDidLoad() {
        super.viewDidLoad()
        titleLabel.text = "Closed Accounts"
        titleLabel.font = UIFont.systemFont(ofSize: 16)
        actionButton.backgroundColor = .blue
    }
}
```

**Migrated Code (UI 2.0):**
```swift
import SnapKit
import DesignToken
import TymeXUIComponent

final class FDClosedListViewController: BaseViewController<FDClosedListViewModel> {
    // MARK: - UI Components
    private let titleLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()

    private let actionButton: TymeXPrimaryButtonV2 = {
        let button = TymeXPrimaryButtonV2()
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        setupViews()
        setupConstraints()
        setupStyling()
    }

    private func setupViews() {
        view.addSubview(titleLabel)
        view.addSubview(actionButton)
    }

    private func setupConstraints() {
        titleLabel.snp.makeConstraints { make in
            make.top.equalToSuperview().inset(TymeX.sectionGapDefault)
            make.leading.trailing.equalToSuperview().inset(TymeX.spacing4)
        }

        actionButton.snp.makeConstraints { make in
            make.top.equalTo(titleLabel.snp.bottom).offset(TymeX.spacing6)
            make.leading.trailing.equalToSuperview().inset(TymeX.spacing4)
            make.height.equalTo(48)
        }
    }

    private func setupStyling() {
        view.backgroundColor = TymeX.screenColorBackgroundDefault

        titleLabel.attributedText = NSAttributedString(
            string: "Closed Accounts",
            attributes: TymeX.textTitleM.color(TymeX.patternColorTextDefault)
        )
    }
}
```

**Changes:**
- ❌ Removed: `@IBOutlet`, XIB file
- ✅ Added: SnapKit constraints, DesignToken, V2 button
- ✅ Applied: Proper structure (setupViews/Constraints/Styling)

## Example 2: TableView Cell Migration

**Current (UI 1.0):**
```swift
class FDClosedCell: UITableViewCell {
    @IBOutlet weak var containerView: UIView!
    @IBOutlet weak var amountLabel: UILabel!

    override func awakeFromNib() {
        super.awakeFromNib()
        containerView.layer.cornerRadius = 8
    }
}
```

**Migrated (UI 2.0):**
```swift
final class FDClosedCell: UITableViewCell {
    private let containerView: UIView = {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()

    private let amountLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupViews()
        setupConstraints()
        setupStyling()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupViews() {
        contentView.addSubview(containerView)
        containerView.addSubview(amountLabel)
    }

    private func setupConstraints() {
        containerView.snp.makeConstraints { make in
            make.edges.equalToSuperview().inset(TymeX.spacing4)
        }

        amountLabel.snp.makeConstraints { make in
            make.edges.equalToSuperview().inset(TymeX.cardPaddingDefault)
        }
    }

    private func setupStyling() {
        backgroundColor = .clear
        containerView.backgroundColor = TymeX.cardColorBackgroundDefault
        containerView.layer.cornerRadius = TymeX.cardRadiusDefault
    }
}
```

# Constraints

## Safety Rules (Hard requirements)

- 🚫 **NEVER delete XIB without user confirmation** — they may have uncommitted work
- 🚫 **NEVER modify ViewModel or business logic** — only touch View layer
- 🚫 **NEVER break existing bindings** — RxSwift subscriptions must stay intact
- ✅ **ALWAYS check V2 component exists** before replacing — fallback to old if missing
- ✅ **ALWAYS preserve accessibility identifiers** — used in UI tests

## Quality Rules

- **Color prefix**: ALL colors MUST use `TymeX.patternColor*` (NOT `TymeX.color*`).
  Only exceptions: `TymeX.screenColor*`, `TymeX.cardColor*`, `TymeX.sectionColor*`
- **Figma first**: If Figma URL provided, extract exact values — don't guess
- **Token matching**: Match Figma token names (e.g., "padding/card/default") to
  DesignToken properties (e.g., `TymeX.cardPaddingDefault`) by semantic similarity
- **Component parity**: If old component had delegate/datasource, ensure V2 supports it
- **Navigation V2**: Use `mxApplyNavigationV2By` with `TymeXNavigationBarStylistV2` —
  don't use old `navigationItem` APIs
- **Typography**: Always use `NSAttributedString` with `TymeX.text*` — never `UIFont` directly.
  Chain with `.color().alignment()`, NOT `.paragraphStyle(lineSpacing:alignment:)`
- **Lottie**: Always use `LottieAnimationView(tymeXNamedWithFallback:) ?? .init()`,
  NEVER `LottieAnimationView(name:bundle:)`
- **Images**: Always use `UIImage(tymeXNamedWithFallback:)`, NEVER load from specific bundles

## When to Skip Migration

Ask user before proceeding if:
- Screen is scheduled for deletion/redesign soon
- More than 50% of components lack V2 equivalents
- Screen has complex custom animations tied to XIB

## Error Handling

| Error | Action |
|-------|--------|
| XIB not found | Proceed anyway — may already be migrated |
| Figma fetch fails | Ask user for manual token values or proceed with best-guess mapping |
| V2 component missing | Keep old component, add TODO comment, notify user |
| Constraint conflict | Use `.priority(.high)` or ask user which takes precedence |

# Resources

- `resources/token-mapping.md` - Complete DesignToken reference (spacing, colors, typography)
- `resources/component-api-changes.md` - **CRITICAL**: Detailed V1 → V2 API changes for each component type
- `resources/skeleton-loading-guide.md` - Skeleton/shimmer loading patterns with SkeletonView pod
- `resources/image-resource-guide.md` - Image loading (`tymeXNamedWithFallback`), Lottie animations, localization
- `examples/` - Real migration examples (complex cells, custom views, etc.)

## Reference Project (Fully Migrated)
The **FixedDeposit** module (`tc-mx-ios-fixed-deposit`) is fully migrated to UI 2.0.
Use it as the canonical reference for real-world patterns:
- 18 ViewControllers fully using V2 components
- Complete skeleton loading implementations
- ButtonDock, CardList, TransactionConfirmation, ResultScreen patterns
- Navigation V2 with all background modes
- Modal alerts and bottom sheets

---

<!-- Generated by Skill Creator Ultra v1.0 -->
