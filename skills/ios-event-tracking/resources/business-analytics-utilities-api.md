# BusinessAnalyticsUtilities API Reference

Source: `PodLocals/BusinessAnalyticsUtilities/Current/`

## Key Behavior: Pixel Coordinates

All tap callbacks (`onTrackButtonTap`, `onTrackScrollItemTap`, `onTrackBarButtonTap`)
return CGPoint already scaled by `UIScreen.main.scale` and rounded internally.
No manual pixel conversion needed by consumers.

## Functions

### 1. `UIView.onTrackButtonTap(in:_:)` — Extension+UIView.swift

```swift
func onTrackButtonTap(in inView: UIView? = nil, _ callback: @escaping (CGPoint) -> Void)
```

- Standalone buttons, CTA buttons
- Uses `UILongPressGestureRecognizer` (minimumPressDuration=0)
- Bounds check on `.ended`, returns pixel-scaled `CGPoint`

**Usage:**
```swift
ctaButton.onTrackButtonTap { [weak self] point in
    self?.viewModel.input.onTracking.onNext(.tapOnCta(point))
}
```

### 2. `UIView.onTrackScrollItemTap(in:_:)` — Extension+UIView.swift

```swift
func onTrackScrollItemTap(in inView: UIView? = nil, _ callback: @escaping (CGPoint) -> Void)
```

- Items inside UITableView/UICollectionView cells
- 10px distance threshold to distinguish taps from scrolls
- Returns pixel-scaled `CGPoint`

**Usage (inside `cellForRowAt`):**
```swift
cell.onTrackScrollItemTap { [weak self] point in
    self?.viewModel.input.onTracking.onNext(.tapOnCard(point))
}
```

### 3. `UIBarButtonItem.onTrackBarButtonTap(_:)` — Extension+UIBarButtonItem.swift

```swift
func onTrackBarButtonTap(_ action: @escaping (CGPoint) -> Void)
```

- Navigation bar buttons (back, right items)
- Handles both `customView`-based and system bar items internally
- Returns pixel-scaled `CGPoint`

**Usage:**
```swift
navigationItem.leftBarButtonItem?.onTrackBarButtonTap { [weak self] point in
    self?.viewModel.input.onTracking.onNext(.back(point))
}
```

### 4. `UIView.onTrackViewSwipe(delay:axis:disposeBag:handler:)` — Extension+UIView.swift

```swift
@discardableResult
func onTrackViewSwipe(
    delay: RxTimeInterval = .seconds(2),
    axis: SwipeAxis,           // .horizontal or .vertical
    disposeBag: DisposeBag,
    handler: @escaping (SwipeMetric) -> Void
) -> Self
```

- Swipe/pan on any `UIView`
- Handler receives `SwipeMetric` struct (not a dictionary)
- Call `.asDictionary` for `[String: Any]` with snake_case keys
- All coordinates are already in pixel coordinates

**Usage:**
```swift
bannerView.onTrackViewSwipe(axis: .horizontal, disposeBag: disposeBag) { [weak self] metric in
    self?.viewModel.input.onTracking.onNext(.swiped(metric))
}
```

### 5. `UIScrollView.onTrackViewScroll(delay:disposeBag:onScrollEnd:)` — Extension+ScrollView.swift

```swift
@discardableResult
func onTrackViewScroll(
    delay: RxTimeInterval = .seconds(2),
    disposeBag: DisposeBag,
    onScrollEnd: @escaping (UIScrollView) -> Void
) -> Self
```

- Scroll tracking for `UITableView`/`UICollectionView`/`UIScrollView`
- Uses RxSwift delegate proxies (works with all scroll view types)
- Tracks `contentSize` changes via KVO for dynamic content (load more)
- Access `scrollView.scrollMetrics` (`ScrollMetrics` struct) in the callback
- Call `.asDictionary` for `[String: Int]` with snake_case keys

**Usage:**
```swift
tableView.onTrackViewScroll(disposeBag: disposeBag) { [weak self] scrollView in
    let metrics = scrollView.scrollMetrics
    self?.viewModel.input.onTracking.onNext(.scrolled(metrics))
}
```

### 6. `AppLifecycleTimeTracker.shared` — Models/AppLifecycleTimeModel.swift

```swift
AppLifecycleTimeTracker.shared.getCurrentMetrics() -> AppLifecycleTimeMetrics
```

- Singleton tracking app session time
- Must call at appropriate lifecycle events:
  - `onAppOpened()` — when app launches
  - `onAppEnterForeground()` — UIApplication.willEnterForeground
  - `onAppEnterBackground()` — UIApplication.didEnterBackground
- Returns `AppLifecycleTimeMetrics` with:
  - `appTotalTimeSpent: Double` (seconds since app opened)
  - `lastInBackgroundTime: Double` (seconds of last background period)
- Call `.asDictionary` for `[String: Any]`

## Models — Models/ScrollMetricModel.swift

### ScrollMetrics (Codable, Equatable)

```swift
public struct ScrollMetrics: Codable, Equatable {
    public let scrollStartHorizontalCoordinate: Int
    public let scrollStartVerticalCoordinate: Int
    public let scrollEndHorizontalCoordinate: Int
    public let scrollEndVerticalCoordinate: Int
    public let scrollDepthPercentage: Int      // 0-100
    public let scrollPageContentSize: Int       // total content length in pixels
    public let scrollVelocity: Int              // pixels/second
}
```

`.asDictionary` returns `[String: Int]`:
```
scroll_start_horizontal_coordinate, scroll_start_vertical_coordinate,
scroll_end_horizontal_coordinate, scroll_end_vertical_coordinate,
scroll_depth_percentage, scroll_page_content_size, scroll_velocity
```

### SwipeMetric

```swift
public struct SwipeMetric {
    public let direction: String      // "LEFT", "RIGHT", "UP", "DOWN"
    public let startHorizontal: Int   // pixel coordinate
    public let startVertical: Int
    public let endHorizontal: Int
    public let endVertical: Int
    public let velocity: Int          // pixels/second
}
```

`.asDictionary` returns `[String: Any]`:
```
swipe_direction, swipe_start_horizontal_coordinate,
swipe_start_vertical_coordinate, swipe_end_horizontal_coordinate,
swipe_end_vertical_coordinate, swipe_velocity
```

## CGPoint Extensions — Extension+UIView.swift

```swift
public extension CGPoint {
    var pixelCoordinate: CGPoint {
        // Returns CGPoint(x: rounded(x * scale), y: rounded(y * scale))
    }

    var pixelCoordinateInt: (x: Int, y: Int) {
        // Returns (x: Int(x.rounded()), y: Int(y.rounded()))
        // NOTE: Does NOT scale — use on already-scaled CGPoint values
    }
}
```

## Decision Guide: Which Helper to Use

| Event Type | View Type | Helper |
|---|---|---|
| BUTTON_TAP | Standalone button/view | `onTrackButtonTap` |
| BUTTON_TAP | Cell in table/collection view | `onTrackScrollItemTap` |
| BUTTON_TAP | Navigation bar item (back, etc.) | `onTrackBarButtonTap` |
| PAGE_SCROLL | UITableView/UICollectionView | `onTrackViewScroll` |
| PAGE_SCROLL | Generic UIView (carousel/banner) | `onTrackViewSwipe` |
| PAGE_ENTER | Any | Direct call in `viewWillAppear` |
| ERROR | Any | Direct call from ViewModelImpl |
