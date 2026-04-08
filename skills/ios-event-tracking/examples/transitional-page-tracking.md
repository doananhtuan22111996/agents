# Example: TransitionalPage Tracking Implementation

## Input Provided

**CSV events (filtered for transitionalPage):**
| event_name | event_description | event_type_cd |
|---|---|---|
| exploreSavingsOptions_transitionalPage_pageEnter | Client enters the savings transitional page | PAGE_ENTER |
| exploreSavingsOptions_transitionalPage_buttonTap | Client taps GoalSave card | BUTTON_TAP |
| exploreSavingsOptions_transitionalPage_buttonTap | Client taps Fixed Deposit card | BUTTON_TAP |
| exploreSavingsOptions_transitionalPage_buttonTap | Client taps Back button | BUTTON_TAP |
| exploreSavingsOptions_transitionalPage_pageEnter | Client enters Transitional Page error screen | PAGE_ENTER |

**KMM/Android code provided:**
```kotlin
fun transitionalPageEnter(): Map<String, Any>
fun goalSaveCardTapped(tapX: Int, tapY: Int): Map<String, Any>
fun fixedDepositCardTapped(tapX: Int, tapY: Int): Map<String, Any>
fun backButtonTapped(tapX: Int?, tapY: Int?): Map<String, Any>
fun transitionalPageError(): Map<String, Any>
```

**User inputs:** Prefix = `SD`, Screen = `TransitionalPage`, Module = `SavingDashboard`

---

## Generated Output

### 1. Tracking Enum (add to SDTransitionalPageViewModel.swift)

```swift
enum SDTransitionalPageTrackingEvent {
    case screenShowed
    case tapOnGoalSaveCard(CGPoint)
    case tapOnFixedDepositCard(CGPoint)
    case back(CGPoint)
    case error
}
```

### 2. ViewModel Protocol Additions (add to SDTransitionalPageViewModel.swift)

```swift
// Add to SDTransitionalPageInput:
var onTracking: AnyObserver<SDTransitionalPageTrackingEvent> { get }

// Add to SDTransitionalPageOutput:
var onStartTracking: Driver<SDTransitionalPageTrackingEvent> { get }
```

### 3. ViewModelImpl Subject (add to SDTransitionalPageViewModelImpl.swift)

```swift
let onSDTrackingSubject = PublishSubject<SDTransitionalPageTrackingEvent>()
```

### 4. StreamFlow Wiring (add to SDTransitionalPageViewModelImpl+StreamFlow.swift)

```swift
// In SDTransitionalPageInput conformance:
var onTracking: AnyObserver<SDTransitionalPageTrackingEvent> {
    onSDTrackingSubject.asObserver()
}

// In SDTransitionalPageOutput conformance:
var onStartTracking: Driver<SDTransitionalPageTrackingEvent> {
    onSDTrackingSubject.asDriver(onErrorJustReturn: .screenShowed)
}
```

### 5. Middleware (NEW FILE: Middleware/SDTransitionalPageMiddleware.swift)

```swift
import BusinessAnalytics
import Foundation
import shared
import TymeXCore

final class SDTransitionalPageMiddleware: BaseMiddleware<SDTransitionalPageViewModel> {
    convenience init(
        _ viewModel: SDTransitionalPageViewModel,
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

    private func getPayload(with event: SDTransitionalPageTrackingEvent) -> [String: Any] {
        switch event {
        case .screenShowed:
            return SVTrackingPayload.shared.transitionalPageEnter()
        case .tapOnGoalSaveCard(let point):
            return SVTrackingPayload.shared.goalSaveCardTapped(
                tapX: .init(point.x), tapY: .init(point.y)
            )
        case .tapOnFixedDepositCard(let point):
            return SVTrackingPayload.shared.fixedDepositCardTapped(
                tapX: .init(point.x), tapY: .init(point.y)
            )
        case .back(let point):
            return SVTrackingPayload.shared.backButtonTapped(
                tapX: .init(int: Int32(point.x)),
                tapY: .init(int: Int32(point.y))
            )
        case .error:
            return SVTrackingPayload.shared.transitionalPageError()
        }
    }
}
```

### 6. Coordinator Wiring (add to SDTransitionalPageCoordinator.swift init)

```swift
// Add property:
private let middleware: SDTransitionalPageMiddleware
var trackingManager: BusinessManagerProtocol = BusinessManager.shared

// In init, after viewModel is resolved:
middleware = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)
```

### 7. View Trigger Points (add to SDTransitionalPageViewController.swift)

```swift
// Import at top:
import BusinessAnalyticsUtilities

// In viewWillAppear:
viewModel.input.onTracking.onNext(.screenShowed)

// In setupNavigation, after setting up back button:
navigationItem.leftBarButtonItem?.onTrackBarButtonTap { [weak self] point in
    self?.viewModel.input.onTracking.onNext(.back(point))
}

// In cell creation (onTrackScrollItemTap callback):
// For GoalSave card:
viewModel.input.onTracking.onNext(.tapOnGoalSaveCard(point))
// For FixedDeposit card:
viewModel.input.onTracking.onNext(.tapOnFixedDepositCard(point))

// In fetchProduct error handling (ViewModelImpl):
self?.onSDTrackingSubject.onNext(.error)
```

### 8. Middleware Tests (NEW FILE: Tests/TransitionalPage/Middleware/SDTransitionalPageMiddlewareTests.swift)

```swift
import Cuckoo
@testable import SavingDashboard
import RxCocoa
import RxSwift
import shared
import XCTest

final class SDTransitionalPageMiddlewareTests: XCTestCase {
    var sut: SDTransitionalPageMiddleware!
    var viewModel: MockSDTransitionalPageViewModel!
    var trackingManager: MockBusinessManagerProtocol!

    override func setUpWithError() throws {
        try super.setUpWithError()
        viewModel = MockSDTransitionalPageViewModel()
        trackingManager = MockBusinessManagerProtocol()
        stubTrackingManager()
    }

    override func tearDownWithError() throws {
        reset(viewModel)
        viewModel = nil
        try super.tearDownWithError()
    }

    func testInit() {
        stubViewModel()
        sut = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)
        XCTAssertNotNil(sut.viewModel)
    }

    func testTrackEnterScreen() {
        let subject = PublishSubject<SDTransitionalPageTrackingEvent>()
        stubViewModel(onStartTracking: subject.asDriver(onErrorDriveWith: .empty()))
        sut = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)

        subject.onNext(.screenShowed)

        verify(trackingManager).track(
            event: "exploreSavingsOptions_transitionalPage_pageEnter",
            properties: any()
        )
    }

    func testTrackTapOnGoalSaveCard() {
        let subject = PublishSubject<SDTransitionalPageTrackingEvent>()
        stubViewModel(onStartTracking: subject.asDriver(onErrorDriveWith: .empty()))
        sut = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)

        subject.onNext(.tapOnGoalSaveCard(CGPoint(x: 10, y: 20)))

        verify(trackingManager).track(
            event: "exploreSavingsOptions_transitionalPage_buttonTap",
            properties: any()
        )
    }

    func testTrackTapOnFixedDepositCard() {
        let subject = PublishSubject<SDTransitionalPageTrackingEvent>()
        stubViewModel(onStartTracking: subject.asDriver(onErrorDriveWith: .empty()))
        sut = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)

        subject.onNext(.tapOnFixedDepositCard(CGPoint(x: 10, y: 20)))

        verify(trackingManager).track(
            event: "exploreSavingsOptions_transitionalPage_buttonTap",
            properties: any()
        )
    }

    func testTrackTapOnBack() {
        let subject = PublishSubject<SDTransitionalPageTrackingEvent>()
        stubViewModel(onStartTracking: subject.asDriver(onErrorDriveWith: .empty()))
        sut = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)

        subject.onNext(.back(.init(x: 0, y: 0)))

        verify(trackingManager).track(
            event: "exploreSavingsOptions_transitionalPage_buttonTap",
            properties: any()
        )
    }

    func testTrackError() {
        let subject = PublishSubject<SDTransitionalPageTrackingEvent>()
        stubViewModel(onStartTracking: subject.asDriver(onErrorDriveWith: .empty()))
        sut = SDTransitionalPageMiddleware(viewModel: viewModel, trackingManager: trackingManager)

        subject.onNext(.error)

        verify(trackingManager).track(
            event: "exploreSavingsOptions_transitionalPage_pageEnter",
            properties: any()
        )
    }

    // MARK: - Helpers

    private func stubViewModel(
        onStartTracking: Driver<SDTransitionalPageTrackingEvent> = .empty()
    ) {
        let outputMock = MockSDTransitionalPageOutput()
        let inputMock = MockSDTransitionalPageInput()
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

### 9. Update Mock Generation Script

Add to `script/iosCuckooGenerateMocks.sh`:
```bash
mainFiles=( \
    ...
    "SDTransitionalPageViewModel.swift" \
)
```

Then run: `bash script/iosCuckooGenerateMocks.sh`
