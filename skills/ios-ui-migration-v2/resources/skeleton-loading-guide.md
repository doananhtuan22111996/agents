# Skeleton Loading Guide (UI 2.0)

## Overview
Every screen that has async data loading should implement a skeleton loading view to provide visual feedback while data loads.

## Dependencies
```swift
import SkeletonView   // Third-party pod for animated gradient skeletons
import DesignToken    // TymeX.* tokens
import SnapKit
import TymeXCore
```

## Architecture

Create a dedicated `*SkeletonView` class per screen:

```
Screens/[Screen]/View/SubViews/SkeletonView/
  FD[Screen]SkeletonView.swift
```

## Two Skeleton Color Schemes

### 1. On Key Screen (gradient/accent background)
Used when the skeleton sits on a gradient or colored background:
```swift
view.backgroundColor = TymeX.colorBackgroundDefault
view.showAnimatedGradientSkeleton(
    usingGradient: .init(colors: [
        TymeX.colorBackgroundDefault,
        SkeletonOnKeyScreen.skeletonStop2,  // rgb(237, 248, 250)
        SkeletonOnKeyScreen.skeletonStop3,  // rgb(237, 248, 250)
        TymeX.colorBackgroundDefault
    ]),
    transition: .crossDissolve(1.5)
)
```

### 2. On Default Background
Used on standard white/light backgrounds:
```swift
view.backgroundColor = TymeX.patternColorBackgroundInfoBase
view.showAnimatedGradientSkeleton(
    usingGradient: .init(colors: [
        TymeX.patternColorBackgroundInfoBase,
        SkeletonOnDefault.skeletonStop2,  // rgb(233, 243, 245)
        SkeletonOnDefault.skeletonStop3,  // rgb(233, 243, 245)
        TymeX.patternColorBackgroundInfoBase
    ]),
    transition: .crossDissolve(1.5)
)
```

## Skeleton Color Constants

Define in Constants.swift:
```swift
enum SkeletonOnKeyScreen {
    static let skeletonStop2: UIColor = .init(red: 237/255, green: 248/255, blue: 250/255, alpha: 1)
    static let skeletonStop3: UIColor = .init(red: 237/255, green: 248/255, blue: 250/255, alpha: 1)
}

enum SkeletonOnDefault {
    static let skeletonStop2: UIColor = .init(red: 233/255, green: 243/255, blue: 245/255, alpha: 1)
    static let skeletonStop3: UIColor = .init(red: 233/255, green: 243/255, blue: 245/255, alpha: 1)
}
```

## Complete Skeleton View Template

```swift
import DesignToken
import SkeletonView
import SnapKit
import TymeXCore
import UIKit

final class FDMyScreenSkeletonView: UIView {
    // Plain UIViews as placeholder lines
    private let line1: UIView = {
        let view = UIView()
        view.mxCornerRadius = TymeX.patternRadiusDefault
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()

    private let line2: UIView = {
        let view = UIView()
        view.mxCornerRadius = TymeX.patternRadiusDefault
        view.translatesAutoresizingMaskIntoConstraints = false
        return view
    }()

    private let stackView: UIStackView = {
        let sv = UIStackView()
        sv.axis = .vertical
        sv.spacing = TymeX.patternGapElementToElement
        sv.alignment = .leading
        sv.translatesAutoresizingMaskIntoConstraints = false
        return sv
    }()

    private lazy var skeletonLines: [UIView] = [line1, line2]

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupViews()
        setupConstraints()
        backgroundColor = .clear
    }

    @available(*, unavailable)
    required init(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupViews() {
        addSubview(stackView)
        stackView.addArrangedSubview(line1)
        stackView.addArrangedSubview(line2)
    }

    private func setupConstraints() {
        stackView.snp.makeConstraints { make in
            make.top.equalToSuperview().inset(TymeX.screenGapDefault)
            make.leading.trailing.equalToSuperview().inset(TymeX.sectionPaddingLeftRight)
            make.bottom.lessThanOrEqualToSuperview()
        }

        line1.snp.makeConstraints { make in
            make.height.equalTo(20)
            make.width.equalToSuperview()  // full width
        }

        line2.snp.makeConstraints { make in
            make.height.equalTo(20)
            make.width.equalTo(130)  // medium width
        }
    }

    // MARK: - Public API

    func showLoading() {
        isHidden = false
        for line in skeletonLines {
            line.isSkeletonable = true
            line.backgroundColor = TymeX.patternColorBackgroundInfoBase
            line.showAnimatedGradientSkeleton(
                usingGradient: .init(colors: [
                    TymeX.patternColorBackgroundInfoBase,
                    SkeletonOnDefault.skeletonStop2,
                    SkeletonOnDefault.skeletonStop3,
                    TymeX.patternColorBackgroundInfoBase
                ]),
                transition: .crossDissolve(1.5)
            )
        }
    }

    func hideLoading() {
        isHidden = true
        skeletonLines.forEach { $0.stopSkeletonAnimation() }
    }

    // IMPORTANT: Must call layoutSkeletonIfNeeded in layoutSubviews
    override func layoutSubviews() {
        super.layoutSubviews()
        skeletonLines.forEach { $0.layoutSkeletonIfNeeded() }
    }
}
```

## Skeleton Line Sizes (from Figma)

| Type | Width | Height |
|------|-------|--------|
| Full width | `equalToSuperview()` | 16-20 |
| Long | 240 | 20 |
| Medium | 130 | 20 |
| Short | 84 | 16 |
| Circle | 48x48 | 48 |

## ViewController Integration

```swift
final class FDMyViewController: BaseViewController<FDMyViewModel>, TymeXLoadingViewable {
    private let skeletonView: FDMyScreenSkeletonView = .init()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(skeletonView)
        skeletonView.snp.makeConstraints { make in
            make.top.equalTo(view.safeAreaLayoutGuide.snp.top)
            make.leading.trailing.bottom.equalToSuperview()
        }
    }

    override func bindLoadingState(viewModel: BaseViewModel) {
        viewModel.baseOutput.onLoadingProgress
            .drive(onNext: { [weak self] isLoading in
                guard let self else { return }
                if isLoading, self.isLoadAnimation {
                    mainContentView.isHidden = true
                    skeletonView.showLoading()
                } else {
                    skeletonView.hideLoading()
                    mainContentView.isHidden = false
                }
            })
            .disposed(by: disposeBag)
    }
}
```

## Skeleton Card View

For card-shaped skeletons on key screens:
```swift
let cardContainer = UIView()
cardContainer.backgroundColor = TymeX.cardColorBackgroundOnKeyScreen
cardContainer.layer.cornerRadius = TymeX.cardRadiusDefault

// Lines inside card use cardPaddingDefault
innerStack.snp.makeConstraints { make in
    make.edges.equalToSuperview().inset(TymeX.cardPaddingDefault)
}
```

## Reusable Skeleton Group

A repeatable group of 3 lines for content sections:
```swift
final class FDSkeletonGroupView: UIView {
    // fullLine (width=parent), shortLine1 (180), shortLine2 (180)
    // spacing: TymeX.patternGapElementToElement
    // heights: 20 each

    func showSkeleton() { /* animate all lines */ }
    func hideSkeleton() { /* stop all animations */ }
}
```

## Other Loading Patterns

### Full-screen overlay loading:
```swift
// From TymeXLoadingViewable protocol:
mxShowFullScreenLoadingView(duration: .zero)
tymeXHideLoadingView()
```

### Semi-transparent overlay:
```swift
mxShowOverlayLoadingView(duration: .zero)
tymeXHideLoadingView()
```

### Button loading:
```swift
button.showLoading { /* completion */ }
button.hideLoading()
```

### Inline Lottie loading spinner (24x24):
```swift
let loadingView = LottieAnimationView(tymeXNamedWithFallback: "anim_global_gotyme_loading_black") ?? .init()
loadingView.loopMode = .loop
loadingView.play()
loadingView.snp.makeConstraints { make in
    make.center.equalToSuperview()
    make.size.equalTo(24)
}
```
