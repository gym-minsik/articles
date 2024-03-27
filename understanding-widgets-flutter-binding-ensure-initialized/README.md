# Understanding WidgetsFlutterBinding.ensureInitialized()

[English](README.md), [한국어](README_kr.md)

## TL;DR
Invoking `WidgetsFlutterBinding.ensureInitialized()` before `runApp` establishes communication channels with the native platform for methods like `MethodChannel.invokeMethod`.

This step is crucial for ensuring that functionalities, especially those interacting with native platforms like Firebase, operate reliably and smoothly.

## Overview
When developing Flutter apps, it's quite common for developers to call `WidgetsFlutterBinding.ensureInitialized()` in the `main()` function. You might have noticed recommendations in various package documents to use this before calling `runApp()`.

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();

  runApp(...);
}
```

Typically, developers follow this guideline without delving deeply into how `WidgetsFlutterBinding.ensureInitialized()` works internally or understanding its necessity. This approach is quite rational; after all, it's not directly critical to the business logic, and thoroughly analyzing the internal code could be quite time-consuming.

In this post, I've taken the time to dig into the core details of `WidgetsFlutterBinding.ensureInitialized()` for you. The goal is to provide you with the essential information in the least amount of time possible.

## What is WidgetsFlutterBinding?
Below is the internal code for `WidgetsFlutterBinding`.
```dart
class WidgetsFlutterBinding 
  extends BindingBase 
  with GestureBinding, 
    SchedulerBinding,
    ServicesBinding, 
    PaintingBinding, 
    SemanticsBinding, 
    RendererBinding, 
    WidgetsBinding 
{
  static WidgetsBinding ensureInitialized() {
    if (WidgetsBinding.instance == null) {
      WidgetsFlutterBinding(); // The constructor calls initInstances().
    }
    return WidgetsBinding.instance;
  }
}
```

As you delve into the Flutter official documentation, you might encounter descriptions where the Flutter Framework is often mentioned as the subject. For example:
> **The framework** creates a State object by calling StatefulWidget.createState.

> **The framework** calls initState.

> If **the framework** does not reinsert this subtree by the end of the current animation frame, **the framework** will call dispose,

In these contexts, **the framework** refers to `WidgetsFlutterBinding`. Essentially, `WidgetsFlutterBinding.ensureInitialized()` means to fully initialize the Flutter Framework.

## Why It's Crucial to Initialize the Flutter Framework First

When examining the `WidgetsFlutterBinding` implementation, you'll notice it inherits from `BindingBase` and mixes in various Bindings, including `GestureBinding`, `SchedulerBinding`, among others.

Notably, packages that deem `WidgetsFlutterBinding.ensureInitialized()` essential typically include features requiring native code invocation. For these packages to function reliably, `ServicesBinding` play a pivotal role. `ServicesBinding` is a key component that enables communication with native code, managing the method channels between the Flutter application and the platform (e.g., Android, iOS).

```dart
mixin ServicesBinding on BindingBase, SchedulerBinding {
  @override
  void initInstances() {
    _defaultBinaryMessenger = createBinaryMessenger();
    // omitted...
  }

  /// The default instance of [BinaryMessenger].
  ///
  /// This is used to send messages from the application to the platform, and ...
  BinaryMessenger get defaultBinaryMessenger => _defaultBinaryMessenger;
}
```

The initialization of this `ServiceBinding` is crucial for the normal operation of `MethodChannel.invokeMethod`, as illustrated in the simplified implementation below:

```dart
class MethodChannel {
  final String name;

  BinaryMessenger get binaryMessenger
    => ServicesBinding.instance.defaultBinaryMessenger;

  Future<T?> invokeMethod<T>(
    String method, [ 
    dynamic arguments 
  ]) async {
    final ByteData input = codec.encodeMethodCall(MethodCall(method, arguments));
    final ByteData? result = await binaryMessenger.send(name, input);
      
    return codec.decodeEnvelope(result) as T?;
    }
}
```

A closer look at `MethodChannel.invokeMethod` shows it uses `ServicesBinding.instance.defaultBinaryMessenger` for method invocations. This means if `ServicesBinding` isn't properly initialized, native code calls through `MethodChannel.invokeMethod` cannot be reliably used. This underscores the importance of initializing `WidgetsFlutterBinding.ensureInitialized()` to set up a stable foundation for any Flutter app that interacts with native code.

# Conclusion
Calling `WidgetsFlutterBinding.ensureInitialized()` includes the initialization process of `ServicesBinding`. This process enables the normal use of various platform channel communication functionalities, including `MethodChannel.invokeMethod`. It lays the foundation for these features to function correctly in Flutter applications that require essential communication with native platform code. This ensures that when your Flutter application needs to interact with the native platform, the necessary groundwork is established for these interactions.
