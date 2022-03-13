# SwiftUI: ScrollView clipping

<div align="center">
	<img src="scrollview-clipping.png" height="200px" />
</div>

## Problem

Some built-in view modifiers like [`.shadow`](https://developer.apple.com/documentation/swiftui/view/shadow(color:radius:x:y:)) _do not_ introduce any additional padding that would increase the intrinsic size of the view.
This can cause problems when views are placed within `UIKit` hierarchies that enable [`.clipsToBounds`](https://developer.apple.com/documentation/uikit/uiview/1622415-clipstobounds).
This is the case with `ScrollView`, which internally uses a [`UIScrollView`](https://developer.apple.com/documentation/uikit/uiscrollview).

## Solution

Extending [`UIScrollView`](https://developer.apple.com/documentation/uikit/uiscrollview) works, because [`.clipsToBounds`](https://developer.apple.com/documentation/uikit/uiview/1622415-clipstobounds) is not read-only.

```swift
import UIKit

extension UIScrollView {
  open override var clipsToBounds: Bool {
    get { false }
    set { }
  }
}
```

One should be aware that this is applied globally.