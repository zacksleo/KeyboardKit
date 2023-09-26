# Understanding Callouts

This article describes the KeyboardKit callout engine.

Callouts are an important part of the typing experience, where input callouts show the currently pressed key and action callouts present secondary keyboard actions.

In KeyboardKit, a ``CalloutActionProvider`` can be used to provide secondary actions to an ``Callouts/ActionCallout``, which in turn will update views like ``Callouts/ActionCallout``.

KeyboardKit will bind a ``StandardCalloutActionProvider`` to ``KeyboardInputViewController/calloutActionProvider`` when the keyboard is loaded. It has a base set of actions, but you can inject localized providers into it or replace it with a custom implementation at any time.

[KeyboardKit Pro][Pro] unlocks and registers localized providers for all keyboard locales when you register a valid license key. It also lets you inherit **ProCalloutActionProvider** for more features. Information about Pro features can be found at the end of this article.



## Callout namespace

KeyboardKit has a ``Callouts`` namespace that contains callout-related types.

For instance, a ``Callouts/InputCallout`` shows the currently pressed key while a ``Callouts/ActionCallout`` shows secondary actions when long pressing a key. These callouts are automatically used if you use a ``SystemKeyboard``.

The namespace doesn't contain protocols or open classes, or types that are meant to be top-level ones. It's meant to be a container for types used by top-level types, to make the library easier to overview.



## How to show input and action callouts

You can apply callout-specific view extensions in KeyboardKit, to make any view act as the container of input and action callouts. 

For instance, this will make your custom keyboard view show both input and action callouts:

```swift
MyKeyboard()
    .keyboardCalloutContainer(...)
```

This view extension will bind a ``CalloutContext`` and its input and action contexts to the view, then apply ``Callouts/ActionCallout`` and ``Callouts/InputCallout`` views that will show as these contexts change. 

The ``SystemKeyboard`` and ``KeyboardButton/Button`` will automatically apply this extension and update the callout contexts as you interact with the keyboard.



## How to customize callout actions

In KeyboardKit, a ``CalloutActionProvider`` can be used to provide dynamic callout actions.

You can customize the callout actions by adding localized providers to the default ``StandardCalloutActionProvider``, or by replacing ``KeyboardInputViewController/calloutActionProvider`` with a custom ``CalloutActionProvider``.



## How to create a custom callout action provider

You can create a custom ``CalloutActionProvider`` by either inheriting the ``StandardCalloutActionProvider`` base class and customize the parts you want, or implement the ``CalloutActionProvider`` protocol from scratch.

There is also a base class to make it easy to implement a custom provider. ``BaseCalloutActionProvider`` provides base functionality that you can extend.

For instance, here's a custom provider that inherits ``StandardCalloutActionProvider`` and customizes the secondary actions for the `$` key:

```swift
class CustomCalloutActionProvider: StandardCalloutActionProvider {
    
    override func calloutActions(for action: KeyboardAction) -> [KeyboardAction] {
        switch action {
        case .character(let char):
            switch char {
            case "$": return "$₽¥€¢£₩".chars.map { KeyboardAction.character($0) }
            default: break
            }
        default: break
        }
        return super.calloutActions(for: action)
    }
}
```

To use this provider instead of the standard one, just set ``KeyboardInputViewController/calloutActionProvider`` to this custom provider:

```swift
class KeyboardViewController: KeyboardInputViewController {

    override func viewDidLoad() {
        calloutActionProvider = CustomCalloutActionProvider()
        super.viewDidLoad()
    }
}
```

This will make KeyboardKit use your custom implementation instead of the standard one.



## 👑 Pro features

[KeyboardKit Pro][Pro] unlocks localized ``CalloutActionProvider``s for all ``KeyboardLocale``s in your license and automatically injects them into the ``StandardCalloutActionProvider`` when you register a valid license key.

You can access all providers that your license unlocks like this:

```swift
let providers = License.current.localizedCalloutActionProviders
```

and locale-specific providers like this:

```swift
let provider = try ProCalloutActionProvider.Swedish()
```

The `ProCalloutActionProvider` base class provides more convenience functions for specifying actions. You can inherit it, or any of the available localized versions, to customize the base behavior:

```swift
class CustomCalloutActionProvider: ProCalloutActionProvider.Swedish {

    override func calloutActionString(for char: String) -> String {
        switch char {
        case "s": return "sweden"
        default: return ""
        }
    }
}
```

To use a custom provider with KeyboardKit Pro, make sure to register it *after* registering your license key, otherwise it will be overwritten by the license registration process.

For instance, this is how you would register the custom provider that we created earlier, using the localized providers from your license:

```swift
open func setupKeyboardKit() {
    try? setupPro(withLicenseKey: key, view: keyboardView) { license in
        self.setupCustomServices(with: license)
    }
}

func setupCustomServices(with license: License) {
    calloutActionProvider = CustomCalloutActionProvider(
        keyboardContext: keyboardContext,
        localizedProviders: license.localizedCalloutActionProviders
    )
}
```

You can add a custom initializer to your custom provider if you need additional setup, then just call `super.init` to setup the rest.



[Pro]: https://github.com/KeyboardKit/KeyboardKitPro