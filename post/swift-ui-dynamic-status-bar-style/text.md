# SwiftUI: Dynamic status bar style

SwiftUI provides many view modifiers that affect the surrounding view hierarchy, but surprisingly none that alter that status bar style.

## Problem

In the old days (_cough_ SwiftUI 1) this could be accomplished by subclassing [`UIHostingController`](https://developer.apple.com/documentation/swiftui/uihostingcontroller) to override  [`preferredStatusBarStyle`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621416-preferredstatusbarstyle), which is read-only by default, and using the newly created class as the [`rootViewController`](https://developer.apple.com/documentation/uikit/uiwindow/1621581-rootviewcontroller) of the key window. 

```swift 
class HostingController<Content: View>: UIHostingController<Content> {
    override var preferredStatusBarStyle: UIStatusBarStyle {
        return .lightContent
    }
}

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func scene(
      _ scene: UIScene, 
      willConnectTo session: UISceneSession, 
      options connectionOptions: UIScene.ConnectionOptions
    ) {
        guard let windowScene = scene as? UIWindowScene else { return }

        let rootView = RootView()

        let window = UIWindow(windowScene: windowScene)
        window.rootViewController = HostingController(rootView: rootView)
        self.window = window
        window.makeKeyAndVisible()
    }
}
```
<figcaption>Old technique</figcaption>

Said solution worked, because the now discouraged  [`UIWindowSceneDelegate`](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate) life cycle exposed the app's `SceneDelegate`. 

This is not the case anymore.

There are [still ways](https://github.com/xavierdonnellon/swiftui-statusbarstyle/blob/main/StatusBarController.swift) to swap out the [`UIHostingController`](https://developer.apple.com/documentation/swiftui/uihostingcontroller) during runtime, but these tend to break features that depend on the new app life cycle being used (e.g.  [`onOpenURL`](https://developer.apple.com/documentation/swiftui/view/onopenurl(perform:))).

## Solution

### `>=` iOS 16

Use [`toolbarColorScheme(_:for:)`](https://developer.apple.com/documentation/swiftui/view/toolbarcolorscheme(_:for:))

### `<` iOS 16

Since swapping in a custom [`UIHostingController`](https://developer.apple.com/documentation/swiftui/uihostingcontroller) isn't an option, a minimally invasive approach involves swizzling `preferredStatusBarStyle` to point to a computed variable which itself points to a writeable variable.

> Method swizzling is a technique of last resort; you should only use it if you have no other options.
>
> —  [eskimo](https://developer.apple.com/forums/thread/48047?answerId=141282022#141282022) 

```swift
extension UIViewController {
    fileprivate enum Holder {
        static var statusBarStyleStack: [UIStatusBarStyle] = .init()
    }
    
    fileprivate func interpose() -> Bool {
        let sel1: Selector = #selector(
            getter: preferredStatusBarStyle
        )
        let sel2: Selector = #selector(
            getter: preferredStatusBarStyleModified
        )
        
        let original = class_getInstanceMethod(Self.self, sel1)
        let new = class_getInstanceMethod(Self.self, sel2)
        
        if let original = original, let new = new {
            method_exchangeImplementations(original, new)
            
            return true
        }
        
        return false
    }
    
    @objc dynamic var preferredStatusBarStyleModified: UIStatusBarStyle {
        Holder.statusBarStyleStack.last ?? .default
    }
}
```

Some additional scaffolding is required to implement a `.statusBarStyle` view modifier.

```swift
import SwiftUI

enum Interposed {
    case pending
    case successful
    case failed
}

struct InterposedKey: EnvironmentKey {
    static let defaultValue: Interposed = .pending
}

extension EnvironmentValues {
    fileprivate(set) var interposed: Interposed {
        get { self[InterposedKey.self] }
        set { self[InterposedKey.self] = newValue }
    }
}

/// `UIApplication.keyWindow` is deprecated
extension UIApplication {
    var keyWindow: UIWindow? {
        connectedScenes
            .compactMap { $0 as? UIWindowScene }
            .flatMap(\.windows)
            .first {
                $0.isKeyWindow
            }
    }
}

extension UIViewController {
    fileprivate enum Holder {
        static var statusBarStyleStack: [UIStatusBarStyle] = .init()
    }
    
    fileprivate func interpose() -> Bool {
        let sel1: Selector = #selector(
            getter: preferredStatusBarStyle
        )
        let sel2: Selector = #selector(
            getter: preferredStatusBarStyleModified
        )
        
        let original = class_getInstanceMethod(Self.self, sel1)
        let new = class_getInstanceMethod(Self.self, sel2)
        
        if let original = original, let new = new {
            method_exchangeImplementations(original, new)
            
            return true
        }
        
        return false
    }
    
    @objc dynamic var preferredStatusBarStyleModified: UIStatusBarStyle {
        Holder.statusBarStyleStack.last ?? .default
    }
}

struct StatusBarStyle: ViewModifier {
    @Environment(\.interposed) private var interposed
    
    let statusBarStyle: UIStatusBarStyle
    let animationDuration: TimeInterval
    
    private func setStatusBarStyle(_ statusBarStyle: UIStatusBarStyle) {
        UIViewController.Holder.statusBarStyleStack.append(statusBarStyle)
        
        UIView.animate(withDuration: animationDuration) {
            UIApplication.shared.keyWindow?.rootViewController?.setNeedsStatusBarAppearanceUpdate()
        }
    }
    
    func body(content: Content) -> some View {
        content
            .onAppear {
                setStatusBarStyle(statusBarStyle)
            }
            .onChange(of: statusBarStyle) {
                setStatusBarStyle($0)
                UIViewController.Holder.statusBarStyleStack.removeFirst(1)
            }
            .onDisappear {
                UIViewController.Holder.statusBarStyleStack.removeFirst(1)
                
                UIView.animate(withDuration: animationDuration) {
                    UIApplication.shared.keyWindow?.rootViewController?.setNeedsStatusBarAppearanceUpdate()
                }
            }
            // Interposing might still be pending on initial render
            .onChange(of: interposed) { _ in
                UIView.animate(withDuration: animationDuration) {
                    UIApplication.shared.keyWindow?.rootViewController?.setNeedsStatusBarAppearanceUpdate()
                }
            }
    }
}

extension View {
    func statusBarStyle(
        _ statusBarStyle: UIStatusBarStyle,
        animationDuration: TimeInterval = 0.3
    ) -> some View {
        modifier(StatusBarStyle(statusBarStyle: statusBarStyle, animationDuration: animationDuration))
    }
}


@main
struct YourApp: App {
    @Environment(\.scenePhase) private var scenePhase
    
    /// Ensures that interposing only occurs once
    private var interposeLock = NSLock()
    
    @State private var interposed: Interposed = .pending
    
    var body: some Scene {
        WindowGroup {
            VStack {
                Text("Hello, world!")
                    .padding()
            }
            .statusBarStyle(.lightContent)
            .environment(\.interposed, interposed)
        }
        .onChange(of: scenePhase) { phase in
            /// `keyWindow` isn't set before first `scenePhase` transition
            if case .active = phase {
                interposeLock.lock()
                if case .pending = interposed,
                   case true = UIApplication.shared.keyWindow?.rootViewController?.interpose() {
                    interposed = .successful
                } else {
                    interposed = .failed
                }
                interposeLock.unlock()
            }
        }
    }
}
```

The `.\interposed` environment value can used to detect swizzling failure.  A potential fallback could involve filling the safe area beneath the status bar with an adaptive color.

```swift
struct Fallback: View {
  var body: some View {
    GeometryReader { reader in
      Color(uiColor: .systemBackground)
        .opacity(0.4)
        .frame(height: reader.safeAreaInsets.top)
        .edgesIgnoringSafeArea(.top)
    }
  }
}
```



