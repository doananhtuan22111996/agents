# Image & Resource Handling Guide (UI 2.0)

## Primary Image Loading: `tymeXNamedWithFallback`

The standard way to load images and Lottie animations in UI 2.0. It searches through DesignToken bundles with a fallback mechanism.

### UIImage
```swift
let image = UIImage(tymeXNamedWithFallback: "icon_name")
```

### LottieAnimationView
```swift
// IMPORTANT: tymeXNamedWithFallback returns Optional - always use `?? .init()` fallback
let animationView = LottieAnimationView(tymeXNamedWithFallback: "anim_name") ?? .init()
animationView.loopMode = .playOnce  // or .loop
animationView.play()
animationView.contentMode = .scaleAspectFit
animationView.translatesAutoresizingMaskIntoConstraints = false
```

## How Fallback Works
1. First tries to load from DesignToken bundle (shared design system assets)
2. Falls back to module bundle or main bundle if not found
3. Returns nil if not found anywhere (hence `?? .init()` for Lottie)

## Image/Asset Naming Conventions

### Icons (from DesignToken)
- Format: `ic_global_[name]`
- Examples:
  - `ic_global_add` - Plus/add icon
  - `ic_global_edit` - Edit/pencil icon
  - `ic_global_check_circle` - Checkmark in circle
  - `TymeX().iconExit` - Close/exit icon (accessed as property)

### Pictograms (larger illustrations)
- Format: `pictogram_local_[name]`
- Example: `pictogram_local_fixed_usdt_deposit`

### Lottie Animations
- Format: `anim_global_[name]`
- Available animations:
  - `anim_global_gotyme_loading_black` - Loading spinner (24x24)
  - `anim_global_gradient_pill` - Decorative gradient pill
  - `anim_global_success` - Success result animation
  - `anim_global_warning` - Warning/error result animation
  - `anim_global_in_progress` - Processing/in-progress animation

## Usage Patterns

### UIImageView
```swift
let imageView = UIImageView(image: UIImage(tymeXNamedWithFallback: "pictogram_name"))
imageView.contentMode = .scaleAspectFit
imageView.translatesAutoresizingMaskIntoConstraints = false
```

### Icon in CardList
```swift
let iconImage = UIImage(tymeXNamedWithFallback: "ic_global_check_circle")
let item = TymeXCardListItemV2(
    leadingStatus: .icon(iconImage, false),  // image, isCircular
    leadingContent: TymeXLeadingContentV2(title: "Title"),
    listType: .standard
)
```

### Lottie Inline Loading Spinner
```swift
let loadingView = LottieAnimationView(
    tymeXNamedWithFallback: "anim_global_gotyme_loading_black"
) ?? .init()
loadingView.loopMode = .loop
loadingView.translatesAutoresizingMaskIntoConstraints = false
// Don't call play() at init — call it in showLoading()
loadingView.snp.makeConstraints { make in
    make.center.equalToSuperview()
    make.size.equalTo(24)  // Standard inline loading size
}
```

### Reusable Lottie Loading Component (Preferred Pattern)

Based on `FDLoadMore.swift` — wrap Lottie in a self-contained UIView with `showLoading()`/`hideLoading()` API:

```swift
import DesignToken
import Lottie
import TymeXCore
import TymeXUIComponent
import UIKit

final class FDLoadMore: UIView {
    lazy var animationView = LottieAnimationView(
        tymeXNamedWithFallback: Constants.Animation.gotymeLoadingBlack
    ) ?? .init()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupView()
    }

    @available(*, unavailable)
    required init(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func hideLoading() {
        isHidden = true
        animationView.stop()
    }

    func showLoading() {
        isHidden = false
        animationView.play()
    }

    private func setupView() {
        animationView.loopMode = .loop
        animationView.translatesAutoresizingMaskIntoConstraints = false
        addSubview(animationView)
        animationView.snp.makeConstraints { make in
            make.top.equalToSuperview().inset(TymeX.sectionGapDefault)
            make.bottom.equalToSuperview()
            make.centerX.equalToSuperview()
            make.width.height.equalTo(24)
        }
    }
}
```

**Key patterns:**
- Use `lazy var` for the animation view (allows `self` reference if needed)
- Set `loopMode = .loop` in setup, NOT at declaration
- Only call `.play()` in `showLoading()`, `.stop()` in `hideLoading()`
- Toggle `isHidden` to show/hide the entire container
- Standard size: 24x24 for inline loading spinners

### Lottie as Inline Property (Closure Pattern)

For embedding Lottie inside another view (e.g., button with loading state):

```swift
private let animationView: LottieAnimationView = {
    let view = LottieAnimationView(
        tymeXNamedWithFallback: Constants.Animation.gotymeLoadingBlack
    ) ?? .init()
    view.loopMode = .loop
    view.translatesAutoresizingMaskIntoConstraints = false
    return view
}()

// Control playback:
func displayLoading(_ isShow: Bool) {
    if isShow {
        loadMoreView.isHidden = false
        animationView.play()
    } else {
        loadMoreView.isHidden = true
        animationView.stop()
    }
}
```

### Lottie Result Screen Animation
```swift
let animationView = LottieAnimationView(
    tymeXNamedWithFallback: Constants.Animation.success
) ?? .init()
animationView.loopMode = .playOnce
animationView.play()
animationView.contentMode = .scaleAspectFit
animationView.translatesAutoresizingMaskIntoConstraints = false
```

### Lottie in Coordinator (Result/Guide Screens)
```swift
func makeLoadingView(name: String) -> LottieAnimationView? {
    let loadingAnimation = LottieAnimationView(tymeXNamedWithFallback: name)
    loadingAnimation?.play(toFrame: .zero, loopMode: .playOnce)
    loadingAnimation?.translatesAutoresizingMaskIntoConstraints = false
    return loadingAnimation
}
// Note: Returns Optional (no `?? .init()`) — caller handles nil
```

### Navigation Bar Icon
```swift
// Close/exit icon from DesignToken:
right: .icon(TymeX().iconExit)
```

## Module Bundle Access
Each module has its own bundle class for accessing local resources:
```swift
final class BundleFixedDeposit {
    static let bundle: Bundle = Bundle(for: BundleFixedDeposit.self)
}
```

## Constants for Resource Names
Define animation/image constants centrally:
```swift
enum Constants {
    enum Animation {
        static let gotymeLoadingBlack = "anim_global_gotyme_loading_black"
        static let gradientPill = "anim_global_gradient_pill"
        static let inProgress = "anim_global_in_progress"
        static let warning = "anim_global_warning"
        static let success = "anim_global_success"
    }

    enum Pictogram {
        static let fixedUsdtDeposit = "pictogram_local_fixed_usdt_deposit"
    }
}
```

## Localization (SwiftGen)
All user-facing strings via SwiftGen:
```swift
// Module-specific:
Localization.Fd.Introduction.Button.explore
Localization.Fd.Button.confirm
Localization.Fd.Common.newFixedDeposit(productName)  // with parameters

// Core/shared:
CoreLocalization.General.Button.gotIt
CoreLocalization.General.Button.exit
```

## Best Practices
1. **Always use `tymeXNamedWithFallback`** - never load from specific bundles directly
2. **Use constants** for animation/image names - no magic strings
3. **Handle nil** for Lottie views with `?? .init()`
4. **Set contentMode** explicitly for UIImageViews
5. **Always set `translatesAutoresizingMaskIntoConstraints = false`**
