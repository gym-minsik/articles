# WidgetsFlutterBinding.ensureInitialized() 이해하기

[English](README.md), [한국어](README_kr.md)

## 바쁜 사람들을 위한 요약
WidgetsFlutterBinding.ensureInitialized()를 앱 시작 시점에 호출하면 Flutter 앱이 MethodChannel.invokeMethod와 같은 메서드를 통해 네이티브 플랫폼과 상호작용할 수 있도록 채널을 초기화합니다.

결론적으로 이 작업은 Firebase와 같이 네이티브 플랫폼과 상호작용하는 기능들이 안정적으로 동작할 수 있도록 합니다.

## Overview
Flutter 앱 개발 시, 대부분의 개발자들은 main() 함수에서 WidgetsFlutterBinding.ensureInitialized()를 호출한 경험이 있을 것입니다. 특정 패키지 문서에서 runApp() 호출 전에 이를 사용하라고 권장하고 있죠.

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();

  runApp(...);
}
```

일반적으로 해당 지침을 단순히 따르기만하고 WidgetsFlutterBinding.ensureInitialized()의 내부 작동 원리나 그 필요성에 대해서는 깊이 파고들지 않는 경우가 많습니다. 물론, 이는 비즈니스 로직에 직접적으로 중요한 부분이 아니며, 내부 코드를 세세하게 분석하는 데에는 상당한 시간이 소요될 수 있기 때문에 굉장히 타당한 선택입니다.

본 문서는 여러분 대신 제가 시간을 투자하여 WidgetsFlutterBinding.ensureInitialized()를 사용하는 핵심적인 이유를 최소의 시간을 투자하여 획득하실 수 있도록 정리해 놓았습니다.

## WidgetsFlutterBinding이 무엇인가요?
아래는 WidgetsFlutterBinding 내부 코드입니다.
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
    if (WidgetsBinding._instance == null) {
      WidgetsFlutterBinding(); // the constructor calls initInstances().
    }
    return WidgetsBinding.instance;
  }
}
```

Flutter 공식문서를 보다보면 API에 대한 설명이 다음과 같이 Flutter Framework가 주어로 오는 경우가 있습니다.
> **The framework** creates a State object by calling StatefulWidget.createState.

> **The framework** calls initState.  

> If **the framework** does not reinsert this subtree by the end of the current animation frame, **the framework** will call dispose,


여기서 **the framework**가 `WidgetsFlutterBinding`라고 볼 수 있습니다. 즉, `WidgetsFlutterBinding.ensureInitialized()`의 뜻은 Flutter Framework를 완전히 초기화라는 뜻 입니다.

## Flutter Framework를 먼저 완전히 초기화하는 이유

WidgetsFlutterBinding 구현을 보면 BindingBase를 상속받고 GestureBinding, SchedulerBinding 등 다양한 Binding을 mixin합니다.

특히, WidgetsFlutterBinding.ensureInitialized()가 필수적인 패키지들은 대부분 네이티브 코드 호출을 필요로 하는 기능을 포함하고 있습니다. 이러한 패키지들이 안정적으로 작동하기 위해서는 ServicesBinding과 같은 믹스인이 중요한 역할을 합니다. ServicesBinding은 네이티브 코드와의 통신을 가능하게 하는 핵심적인 요소로, Flutter 애플리케이션과 플랫폼(안드로이드, iOS 등) 사이의 메소드 채널을 관리합니다.

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

이 ServiceBinding이 초기화가 되어야 MethodChannel.invokeMethod을 정상적으로 사용할 수 있는데, 아래는 간략화된 MethodChannel.invokeMethod 구현입니다.

```dart
class MethodChannel {
  final String name;

  BinaryMessenger get binaryMessenger
    => ServiceBinding.instance.defaultBinaryMessenger

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

MethodChannel.invokeMethod의 구현을 살펴보면, ServiceBinding.instance.defaultBinaryMessenger를 사용하여 메소드 호출을 구현하고 있습니다. 이는 ServicesBinding이 제대로 초기화되지 않은 상태에서는 MethodChannel.invokeMethod을 통한 네이티브 코드 호출을 정상적으로 사용할 수 없음을 의미합니다.

# 결론
WidgetsFlutterBinding.ensureInitialized() 호출은 ServiceBinding의 초기화 과정을 포함합니다. 이 과정은 MethodChannel.invokeMethod를 포함한 다양한 플랫폼 채널 통신 기능을 정상적으로 사용할 수 있도록 합니다. 이는 Flutter 애플리케이션에서 네이티브 플랫폼의 코드와 통신하는 기능이 필수적으로 요구되는 경우, 해당 기능들이 올바르게 작동할 수 있는 기반을 마련합니다.
 







