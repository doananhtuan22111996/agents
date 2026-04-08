---
name: ios-event-tracking
description: |
  Generate event tracking implementation for iOS screens following the MVVM-C
  middleware pattern. Reads a CSV tracking spec and Android/KMM code to produce
  the tracking enum, ViewModel wiring, Middleware, View triggers, and unit tests.
  Triggers when user says "add tracking", "implement tracking events",
  "wire up tracking", "add analytics events", "generate tracking for this screen",
  or mentions tracking/analytics in the context of an iOS screen.
---

# Goal

Generate complete event tracking implementation for an iOS screen by reading
a CSV tracking spec and KMM/Android payload code, then producing all necessary
Swift files following the project's BaseMiddleware + RxSwift pattern.

# Instructions

## Step 1: Gather Inputs

Ask the user for these inputs **one at a time, in order**:

1. **CSV tracking spec file path** — contains columns: `event_name`,
   `event_description`, `event_type_cd`, `button_name`, `button_id`, etc.
2. **Which page/screen rows from the CSV** — the CSV may contain events for
   multiple screens. Ask user which `page_purpose` or `pageId` prefix to filter.
   Example: "transitionalPage", "introScreen", "dashboardScreen".
3. **Android/KMM code** — the Kotlin code showing available payload builder
   functions from the `shared` module (e.g., `SVTrackingPayload`). This tells
   us what functions exist and their parameter signatures.
4. **Screen name prefix** — the prefix used for this module's classes.
   Example: "SD" for SavingDashboard, "FD" for FixedDeposit.
5. **Screen name** — the screen identifier used in class names.
   Example: "TransitionalPage", "IntroScreen", "Dashboard".

## Step 2: Parse the CSV

Read the CSV file. Filter rows matching the user's chosen page/screen.
Extract for each event:
- `event_name` — the tracking event string (e.g., `exploreFixedDeposit_introScreen_pageEnter`)
- `event_description` — human-readable description (e.g., "Client enters Fixed Deposit intro screen")
- `event_type_cd` — event type: `pageEnter`, `buttonTap`, `pageScroll`
- `button_name` / `button_id` — for button tap events
- Whether the event needs `CGPoint` (buttonTap events) or scroll data (pageScroll events)

## Step 3: Match KMM Payload Functions

From the Android/KMM code provided by user, identify which function corresponds
to each CSV event. Map them by matching `event_name` or `event_description`.

If a KMM function can't be found for a CSV event, flag it to the user:
> "I couldn't find a KMM payload function for event: `{event_description}`.
> Is the function named differently, or should I skip this event?"

## Step 4: Generate the Tracking Enum

Add to the existing ViewModel protocol file (e.g., `{Prefix}{Screen}ViewModel.swift`):

```swift
enum {Prefix}{Screen}TrackingEvent {
    // For pageEnter events — no associated value
    case screenShowed
    case error

    // For buttonTap events — CGPoint for tap coordinates
    case tapOnCtaButton(CGPoint)
    case back(CGPoint)

    // For pageScroll events — ScrollMetrics from BusinessAnalyticsUtilities
    case scrolled(ScrollMetrics)

    // For pageSwipe events — SwipeMetric from BusinessAnalyticsUtilities
    case swiped(SwipeMetric)
}
```

**Naming rules for enum cases:**
- `pageEnter` → descriptive name like `screenShowed`, `errorScreenShowed`
- `buttonTap` → `tapOn{ButtonName}(CGPoint)` or `back(CGPoint)`
- `pageScroll` → `scrolled(ScrollMetrics)` — uses struct from BusinessAnalyticsUtilities
- `pageSwipe` → `swiped(SwipeMetric)` — uses struct from BusinessAnalyticsUtilities
- Error events → `error`

Also add to the ViewModel Input protocol:
```swift
var onTracking: AnyObserver<{Prefix}{Screen}TrackingEvent> { get }
```

And to the ViewModel Output protocol:
```swift
var onStartTracking: Driver<{Prefix}{Screen}TrackingEvent> { get }
```

## Step 5: Add ViewModel Subject & StreamFlow Wiring

In `{Prefix}{Screen}ViewModelImpl.swift`, add:
```swift
let on{Prefix}TrackingSubject = PublishSubject<{Prefix}{Screen}TrackingEvent>()
```

In `{Prefix}{Screen}ViewModelImpl+StreamFlow.swift`, add Input conformance:
```swift
var onTracking: AnyObserver<{Prefix}{Screen}TrackingEvent> {
    on{Prefix}TrackingSubject.asObserver()
}
```

And Output conformance:
```swift
var onStartTracking: Driver<{Prefix}{Screen}TrackingEvent> {
    on{Prefix}TrackingSubject.asDriver(onErrorJustReturn: .screenShowed)
}
```

## Step 6: Generate Middleware

Create `{Prefix}{Screen}Middleware.swift`:

```swift
import BusinessAnalytics
import BusinessAnalyticsUtilities
import Foundation
import shared
import TymeXCore

final class {Prefix}{Screen}Middleware: BaseMiddleware<{Prefix}{Screen}ViewModel> {
    convenience init(
        _ viewModel: {Prefix}{Screen}ViewModel,
        trackingManager: BusinessManager = BusinessManager.shared
    ) {
        self.init(viewModel: viewModel, trackingManager: trackingManager)
    }

    override func viewModelBinding() {
        super.viewModelBinding()
        viewModel.output.onStartTracking
            .drive(onNext: { [weak self] event in
                guard let self = self else { return }
                let payload = getPayload(with: event)
                track(payload: payload)
            }).disposed(by: disposeBag)
    }

    private func getPayload(with event: {Prefix}{Screen}TrackingEvent) -> [String: Any] {
        switch event {
        // Map each enum case to its KMM payload function
        case .screenShowed:
            return {KMMPayloadClass}.shared.{kmmFunctionName}()
        case .tapOnCtaButton(let point):
            return {KMMPayloadClass}.shared.{kmmFunctionName}(
                tapX: .init(point.x), tapY: .init(point.y)
            )
        // ... etc for each event
        }
    }
}
```

**CGPoint handling patterns (V1 KMM functions — Int32 params):**
- Most button taps: `tapX: .init(point.x), tapY: .init(point.y)` (Int32 conversion)
- Back button: `tapX: .init(int: Int32(point.x)), tapY: .init(int: Int32(point.y))`
- Follow the KMM function signature — check if parameter is `Int32` or `KotlinInt`

**V2 KMM functions — ButtonTapReportV2/ViewScrollReportV2 params:**
Some newer modules use V2 payload classes with report structs instead of raw coordinates:
```swift
// V2 Button Tap — uses ButtonTapReportV2:
case .tapOnCtaButton(let point):
    return {KMMPayloadClass}.shared.{kmmFunctionName}(
        productId: Configuration.productId,
        buttonTapReport: ButtonTapReportV2(
            tapHorizontalCoord: Int32(point.x),
            tapVerticalCoord: Int32(point.y)
        )
    )

// V2 Page Scroll — uses ViewScrollReportV2:
case .scrolled(let metrics):
    return {KMMPayloadClass}.shared.{kmmFunctionName}(
        productId: Configuration.productId,
        viewScrollReport: ViewScrollReportV2(/* scroll metrics */)
    )
```
Check the KMM payload class to determine if it uses V1 (raw Int32 params) or
V2 (report struct params). V2 functions also typically require a `productId` parameter.

## Step 7: Wire Coordinator

Show the user what to add to the Coordinator's `init`:
```swift
middleware = {Prefix}{Screen}Middleware(viewModel: viewModel, trackingManager: trackingManager)
```

And ensure the Coordinator has:
```swift
var trackingManager: BusinessManagerProtocol = BusinessManager.shared
```

## Step 8: Generate View Trigger Points

Requires `import BusinessAnalyticsUtilities` in the ViewController.

The `BusinessAnalyticsUtilities` module provides 5 gesture tracking helpers
across 3 extensions. Choose the right one based on `event_type_cd` and the
view type:

### Available Gesture Helpers

**1. `onTrackButtonTap` — for standalone buttons/views** (on `UIView`)
```swift
func onTrackButtonTap(in inView: UIView? = nil, _ callback: @escaping (CGPoint) -> Void)
```
- Uses `UILongPressGestureRecognizer` with `minimumPressDuration = 0`
- On `.ended`: checks if touch is still within the view's bounds, then fires
  callback with the touch-down point
- **Returns pixel coordinates** — the utility calls `.pixelCoordinate` internally
- Best for: standalone CTA buttons, any non-scrollable view

**2. `onTrackScrollItemTap` — for items inside scroll views/table views** (on `UIView`)
```swift
func onTrackScrollItemTap(in inView: UIView? = nil, _ callback: @escaping (CGPoint) -> Void)
```
- Also uses `UILongPressGestureRecognizer` with `minimumPressDuration = 0`
- On `.ended`: validates tap distance < 10px threshold (to distinguish taps from
  scrolls), then fires callback with the touch-down point
- **Returns pixel coordinates** — the utility calls `.pixelCoordinate` internally
- Best for: table view cells, collection view cells, items in a scroll container
- The 10px threshold prevents scroll gestures from being mistaken as taps

**3. Navigation bar button tracking — via `.customView?.onTrackButtonTap`**

⚠️ **CRITICAL**: In this project, navigation bar buttons set via
`TymeXNavigationBarStylistV2` always use `customView`-based bar items.
You MUST use `.customView?.onTrackButtonTap` — do NOT use
`.onTrackBarButtonTap` on `UIBarButtonItem` directly, as it does not work
reliably with the `TymeXNavigationBarStylistV2` pattern.

```swift
// CORRECT — always use this pattern:
navigationItem.leftBarButtonItem?.customView?.onTrackButtonTap { [weak self] point in
    self?.viewModel.input.onTracking.onNext(.tapBackButton(point))
}

// WRONG — do NOT use this:
// navigationItem.leftBarButtonItem?.onTrackBarButtonTap { ... }
```

**Determining left vs right bar button:**
You MUST check the `TymeXNavigationBarStylistV2` configuration to determine
which side the button is on:
- `left: .backButton` → use `navigationItem.leftBarButtonItem`
- `right: .icon(...)` (e.g., close/exit icon) → use `navigationItem.rightBarButtonItem`
- `setupPresentNavigationV2()` places close icon on `right:` → use `rightBarButtonItem`

```swift
// Back button (left side) — from setupNavigation with left: .backButton
navigationItem.leftBarButtonItem?.customView?.onTrackButtonTap { [weak self] point in
    self?.viewModel.input.onTracking.onNext(.tapBackButton(point))
}

// Close/exit button (right side) — from setupPresentNavigationV2 with right: .icon
navigationItem.rightBarButtonItem?.customView?.onTrackButtonTap { [weak self] point in
    self?.viewModel.input.onTracking.onNext(.tapCloseButton(point))
}
```

- **Returns pixel coordinates** — the `onTrackButtonTap` utility calls
  `.pixelCoordinate` internally
- Best for: back button, close/exit button, any navigation bar item

**4. `onTrackViewSwipe` — for swipe/pan gesture tracking** (on `UIView`)
```swift
func onTrackViewSwipe(
    delay: RxTimeInterval = .seconds(2),
    axis: SwipeAxis,           // .horizontal or .vertical
    disposeBag: DisposeBag,
    handler: @escaping (SwipeMetric) -> Void
) -> Self
```
- Uses `UIPanGestureRecognizer` to track swipe start/end points
- Returns a `SwipeMetric` struct (not a dictionary) with properties:
  - `direction: String` (LEFT/RIGHT/UP/DOWN)
  - `startHorizontal: Int`, `startVertical: Int` (pixel coords)
  - `endHorizontal: Int`, `endVertical: Int` (pixel coords)
  - `velocity: Int` (pixels/second)
- Call `.asDictionary` on the result to get `[String: Any]` with keys:
  `swipe_direction`, `swipe_start_horizontal_coordinate`,
  `swipe_start_vertical_coordinate`, `swipe_end_horizontal_coordinate`,
  `swipe_end_vertical_coordinate`, `swipe_velocity`
- All coordinates are already in pixel coordinates (multiplied by screen scale)
- Has a configurable delay (default 2 seconds) before firing
- Best for: swipe tracking on carousels, banners, or generic views without
  a UIScrollView delegate

**5. `onTrackViewScroll` — for full scroll metrics tracking** (on `UIScrollView`)
```swift
func onTrackViewScroll(
    delay: RxTimeInterval = .seconds(2),
    disposeBag: DisposeBag,
    onScrollEnd: @escaping (UIScrollView) -> Void
) -> Self
```
- Uses RxSwift delegation proxies (`willBeginDragging`, `didEndDragging`,
  `didEndDecelerating`) — works with `UITableView`, `UICollectionView`, and
  plain `UIScrollView`
- Tracks `contentSize` changes via KVO for dynamic content (load more)
- On scroll end, access `scrollView.scrollMetrics` which returns a
  `ScrollMetrics` struct with `Int` properties:
  - `scrollStartHorizontalCoordinate` / `scrollStartVerticalCoordinate`
  - `scrollEndHorizontalCoordinate` / `scrollEndVerticalCoordinate`
  - `scrollDepthPercentage` (0–100)
  - `scrollPageContentSize` (total content length in pixels)
  - `scrollVelocity` (pixels/second)
- Call `.asDictionary` to get `[String: Int]` with snake_case keys
- Best for: `PAGE_SCROLL` events on table/collection views where you need
  depth percentage and content size metrics
- Prefer this over `onTrackViewSwipe` for UIScrollView subclasses

**6. `AppLifecycleTimeTracker` — for app time spent metrics** (singleton)
```swift
AppLifecycleTimeTracker.shared.getCurrentMetrics() -> AppLifecycleTimeMetrics
```
- Tracks `appTotalTimeSpent` (seconds since app opened) and
  `lastInBackgroundTime` (seconds of last background period)
- Must call `onAppOpened()`, `onAppEnterForeground()`, `onAppEnterBackground()`
  at appropriate lifecycle events
- Call `.asDictionary` on the result for `[String: Any]` with keys:
  `app_total_time_spent`, `last_in_background_time`
- Best for: events that need app session duration context

### When to use `onTrackViewSwipe` vs `onTrackViewScroll`

| Criteria | `onTrackViewSwipe` (UIView) | `onTrackViewScroll` (UIScrollView) |
|---|---|---|
| View type | Any UIView | UIScrollView subclass only |
| Returns | `SwipeMetric` struct | `ScrollMetrics` struct (via `.scrollMetrics`) |
| Metrics | direction, start/end coords, velocity | start/end coords, depth%, content size, velocity |
| Mechanism | UIPanGestureRecognizer | RxSwift delegate proxies |
| Use when | Tracking swipes on carousels/banners | Tracking scroll on table/collection views |

### Pixel Coordinate Handling (IMPORTANT)

All gesture helpers (`onTrackButtonTap`, `onTrackScrollItemTap`,
`onTrackBarButtonTap`) now return `CGPoint` already in **pixel coordinates**
(scaled by `UIScreen.main.scale` and rounded). The conversion happens inside
the utility — you do NOT need to convert manually.

The `CGPoint` extension provides:
```swift
point.pixelCoordinate      // CGPoint(x: rounded(x * scale), y: rounded(y * scale))
point.pixelCoordinateInt   // (x: Int(x.rounded()), y: Int(y.rounded()))
                           // NOTE: does NOT scale — use on already-scaled points
```

Since callbacks already return pixel-scaled `CGPoint`, pass them directly to
the middleware. The KMM payload function parameters that accept `Int32` or
`KotlinInt` should use the CGPoint values as-is:
```swift
// CGPoint from callback is already in pixel coords
tapX: .init(point.x), tapY: .init(point.y)
```

### Trigger Point Patterns

For each `event_type_cd`, wire the tracking in the ViewController as follows:

- **pageEnter (screenShowed)** → in `viewWillAppear`:
  ```swift
  viewModel.input.onTracking.onNext(.screenShowed)
  ```

- **buttonTap (standalone button)** → using `onTrackButtonTap`:
  ```swift
  someButton.onTrackButtonTap { [weak self] point in
      self?.viewModel.input.onTracking.onNext(.tapOnCtaButton(point))
  }
  ```

- **buttonTap (back button / nav bar item)** → using `.customView?.onTrackButtonTap`:
  ```swift
  // Back button (left side, set via left: .backButton in TymeXNavigationBarStylistV2)
  navigationItem.leftBarButtonItem?.customView?.onTrackButtonTap { [weak self] point in
      self?.viewModel.input.onTracking.onNext(.tapBackButton(point))
  }

  // Close/exit button (right side, set via right: .icon(...) or setupPresentNavigationV2)
  navigationItem.rightBarButtonItem?.customView?.onTrackButtonTap { [weak self] point in
      self?.viewModel.input.onTracking.onNext(.tapCloseButton(point))
  }
  ```
  ⚠️ IMPORTANT: Use `.customView?.onTrackButtonTap` (NOT `.onTrackBarButtonTap`).
  Check the `TymeXNavigationBarStylistV2` config to determine if the button
  is on the left or right side. Never assume — always verify.

- **buttonTap (cell in table/collection view)** → using `onTrackScrollItemTap`:
  ```swift
  cell.onTrackScrollItemTap { [weak self] point in
      self?.viewModel.input.onTracking.onNext(.tapOnProductCard(point))
  }
  ```
  Note: Call this inside `cellForRowAt` / `cellForItemAt` after cell configuration.

- **pageScroll (on UIScrollView/UITableView/UICollectionView)** → using
  `onTrackViewScroll`:
  ```swift
  tableView.onTrackViewScroll(disposeBag: disposeBag) { [weak self] scrollView in
      let metrics = scrollView.scrollMetrics
      self?.viewModel.input.onTracking.onNext(.scrolled(metrics))
  }
  ```
  The scroll enum case should accept `ScrollMetrics`. In the middleware,
  use `metrics.asDictionary` or access individual properties when passing
  to KMM payload functions.

- **pageSwipe (on generic UIView)** → using `onTrackViewSwipe`:
  ```swift
  bannerView.onTrackViewSwipe(axis: .horizontal, disposeBag: disposeBag) { [weak self] metric in
      self?.viewModel.input.onTracking.onNext(.swiped(metric))
  }
  ```
  The swipe enum case should accept `SwipeMetric`. In the middleware,
  use `metric.asDictionary` or access individual properties when passing
  to KMM payload functions.

- **error** → in error handling blocks of ViewModel (not ViewController):
  ```swift
  self?.on{Prefix}TrackingSubject.onNext(.error)
  ```
  Error tracking fires from `ViewModelImpl` directly since errors come from
  API responses, not UI gestures.

## Step 9: Generate Middleware Unit Tests

Create `{Prefix}{Screen}MiddlewareTests.swift` using Cuckoo mocks:

```swift
import Cuckoo
@testable import {ModuleName}
import RxCocoa
import RxSwift
import shared
import XCTest

final class {Prefix}{Screen}MiddlewareTests: XCTestCase {
    var sut: {Prefix}{Screen}Middleware!
    var viewModel: Mock{Prefix}{Screen}ViewModel!
    var trackingManager: MockBusinessManagerProtocol!

    override func setUpWithError() throws {
        try super.setUpWithError()
        viewModel = Mock{Prefix}{Screen}ViewModel()
        trackingManager = MockBusinessManagerProtocol()
        stubTrackingManager()
    }

    override func tearDownWithError() throws {
        reset(viewModel)
        viewModel = nil
        try super.tearDownWithError()
    }

    // Generate one test per event:
    func testTrack{EventName}() {
        // Given:
        let onStartTrackingSubject = PublishSubject<{Prefix}{Screen}TrackingEvent>()
        stubViewModel(onStartTracking: onStartTrackingSubject.asDriver(onErrorDriveWith: .empty()))
        stubTrackingManager()
        sut = {Prefix}{Screen}Middleware(viewModel: viewModel, trackingManager: trackingManager)

        // When:
        onStartTrackingSubject.onNext(.{enumCase})

        // Then:
        verify(trackingManager).track(event: "{event_name}", properties: any())
    }

    // Helper stubs:
    private func stubViewModel(
        onStartTracking: Driver<{Prefix}{Screen}TrackingEvent> = .empty()
    ) {
        let outputMock = Mock{Prefix}{Screen}Output()
        let inputMock = Mock{Prefix}{Screen}Input()
        Cuckoo.stub(outputMock) {
            when($0.onStartTracking.get).thenReturn(onStartTracking)
        }
        Cuckoo.stub(viewModel) {
            when($0.baseOutput.get).thenReturn(stubBaseOutput())
            when($0.baseInput.get).thenReturn(stubBaseInput())
            when($0.output.get).thenReturn(outputMock)
            when($0.input.get).thenReturn(inputMock)
        }
    }

    private func stubTrackingManager() {
        Cuckoo.stub(trackingManager) {
            when($0.track(event: any(), properties: any())).thenDoNothing()
        }
    }
}
```

**Test verification strategy:** Use `properties: any()` matcher by default to
reduce maintenance effort. The key assertion is that the correct `event_name`
string is passed. If user wants stricter verification, they can add
`ParameterMatcher<[String: Any]>` for specific properties like
`event_description` or `event_type_cd`.

## Step 10: Update Mock Generation Script

Remind the user to add the ViewModel protocol to `script/iosCuckooGenerateMocks.sh`
if not already present:
```bash
mainFiles=( \
    ...
    "{Prefix}{Screen}ViewModel.swift" \
)
```

Then run: `bash script/iosCuckooGenerateMocks.sh`

## Output Format

Present generated code in this order:
1. Tracking enum + ViewModel protocol changes
2. ViewModelImpl subject
3. StreamFlow wiring
4. Middleware class (complete file)
5. Coordinator wiring snippet
6. View trigger points (show where each goes)
7. Middleware unit tests (complete file)
8. Mock generation script update

For each file, clearly indicate whether it's a **new file** or **additions to
existing file** (with the target file path).

# Examples

## Example 1: TransitionalPage (from SavingDashboard)

**Input:**
- CSV: filtering `page_purpose = transitionalPage`
- Events: screenShowed, tapOnGoalSaveCard, tapOnFixedDepositCard, back, error
- KMM functions: `transitionalPageEnter()`, `goalSaveCardTapped(tapX:tapY:)`,
  `fixedDepositCardTapped(tapX:tapY:)`, `backButtonTapped(tapX:tapY:)`,
  `transitionalPageError()`
- Prefix: `SD`, Screen: `TransitionalPage`

**Generated Enum:**
```swift
enum SDTransitionalPageTrackingEvent {
    case screenShowed
    case tapOnGoalSaveCard(CGPoint)
    case tapOnFixedDepositCard(CGPoint)
    case back(CGPoint)
    case error
}
```

**Generated Middleware getPayload:**
```swift
private func getPayload(with event: SDTransitionalPageTrackingEvent) -> [String: Any] {
    switch event {
    case .screenShowed:
        return SVTrackingPayload.shared.transitionalPageEnter()
    case .tapOnGoalSaveCard(let point):
        return SVTrackingPayload.shared.goalSaveCardTapped(tapX: .init(point.x), tapY: .init(point.y))
    case .tapOnFixedDepositCard(let point):
        return SVTrackingPayload.shared.fixedDepositCardTapped(tapX: .init(point.x), tapY: .init(point.y))
    case .back(let point):
        return SVTrackingPayload.shared.backButtonTapped(
            tapX: .init(int: Int32(point.x)),
            tapY: .init(int: Int32(point.y))
        )
    case .error:
        return SVTrackingPayload.shared.transitionalPageError()
    }
}
```

**Generated Test (one of many):**
```swift
func testTrackTapOnGoalSaveCard() {
    let onStartTrackingSubject = PublishSubject<SDTransitionalPageTrackingEvent>()
    stubViewModel(onStartTracking: onStartTrackingSubject.asDriver(onErrorDriveWith: .empty()))
    stubTrackingManager()
    sut = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)

    onStartTrackingSubject.onNext(.tapOnGoalSaveCard(CGPoint(x: 10, y: 20)))

    verify(trackingManager).track(
        event: "exploreSavingsOptions_transitionalPage_buttonTap",
        properties: any()
    )
}
```

## Example 2: Multiple events sharing same event_name

When CSV has multiple rows with the same `event_name` but different
`event_description` (e.g., "Client taps CTA button" vs "Client taps Back button"
both under `exploreFixedDeposit_introScreen_buttonTap`), generate **separate
enum cases** that map to **different KMM functions** but may produce the same
`event_name` string. The middleware `getPayload` switch handles them independently.

# Constraints

- **Never pass `.zero` or hardcoded `CGPoint` values** for button tap events.
  Always capture the actual tap coordinates from the gesture callback. If a
  button tap event doesn't have a real CGPoint from a gesture, wire the
  tracking through `onTrackButtonTap` or `.customView?.onTrackButtonTap`
  to capture it properly.
- **Nav bar button tracking**: Always use `.customView?.onTrackButtonTap`
  (NOT `.onTrackBarButtonTap`). Always check the `TymeXNavigationBarStylistV2`
  config to determine if the button is `leftBarButtonItem` or
  `rightBarButtonItem`.
- Never hardcode event name strings or property values in the middleware — always
  call the KMM `shared` module payload functions. The KMM layer is the source of
  truth for tracking payloads.
- Always use `[weak self]` in RxSwift subscribe/drive closures to prevent retain
  cycles.
- Middleware must extend `BaseMiddleware<{ViewModel}>` and use the convenience
  init pattern with `trackingManager` parameter for testability.
- Unit tests use `properties: any()` by default. Only add specific property
  matchers if the user explicitly requests stricter verification.
- If the ViewModel protocol file, ViewModelImpl, or StreamFlow file already exist,
  show additions only — do not regenerate the entire file.
- The Middleware file and Middleware test file are always new files in their
  respective directories (`Middleware/` and `Tests/.../Middleware/`).
- Always remind the user to update `iosCuckooGenerateMocks.sh` and regenerate
  mocks before running tests.

<!-- Generated by Skill Creator Ultra v1.0 -->
