# PurchasesObserver 引导页面观察者方法详解

## 概述

该类实现了 `AdaptyUIOnboardingsEventsObserver` 接口，负责监听和处理引导页面视图的所有事件，包括加载状态、用户操作和分析事件。

## 视图生命周期事件

### 引导页面加载完成

```dart 400:405:example/lib/purchase_observer.dart
  @override
  void onboardingViewDidFinishLoading(
    AdaptyUIOnboardingView view,
    AdaptyUIOnboardingMeta meta,
  ) {
    print('#Example# onboardingViewDidFinishLoading of $view, meta = $meta');
  }
```

引导页面内容加载完成时的回调，包含视图和元数据信息。

### 引导页面加载失败

```dart 407:412:example/lib/purchase_observer.dart
  @override
  void onboardingViewDidFailWithError(
    AdaptyUIOnboardingView view,
    AdaptyError error,
  ) {
    print('#Example# onboardingViewDidFailWithError of $view, error = $error');
  }
```

引导页面加载或初始化失败时的错误处理。

## 用户操作处理

### 关闭操作

```dart 414:426:example/lib/purchase_observer.dart
  @override
  void onboardingViewOnCloseAction(
    AdaptyUIOnboardingView view,
    AdaptyUIOnboardingMeta meta,
    String actionId,
  ) {
    print('#Example# onboardingViewOnCloseAction of $view, meta = $meta, actionId = $actionId');

    if (view.isNativeRendering) {
      view.dismiss();
    }
  }
```

处理关闭引导页面的操作：

- 只有在原生渲染模式下才执行关闭操作
- 记录操作详情用于调试

### 支付墙操作

```dart 428:437:example/lib/purchase_observer.dart
  @override
  void onboardingViewOnPaywallAction(
    AdaptyUIOnboardingView view,
    AdaptyUIOnboardingMeta meta,
    String actionId,
  ) {
    print('#Example# onboardingViewOnPaywallAction of $view, meta = $meta, actionId = $actionId');

    _presentPaywall(actionId);
  }

  Future<void> _presentPaywall(String placementId) async {
    try {
      final paywall = await Adapty().getPaywall(placementId: placementId);
      final paywallView = await AdaptyUI().createPaywallView(paywall: paywall);
      await paywallView.present();
    } catch (e) {
      print('#Example# _presentPaywall error $e');
    }
  }
```

处理从引导页面跳转到支付墙的操作：

1. 根据位置ID获取支付墙配置
2. 创建支付墙视图
3. 显示支付墙
4. 错误处理确保流程稳定

### 自定义操作

```dart 439:456:example/lib/purchase_observer.dart
  @override
  void onboardingViewOnCustomAction(
    AdaptyUIOnboardingView view,
    AdaptyUIOnboardingMeta meta,
    String actionId,
  ) {
    print('#Example# onboardingViewOnCustomAction of $view, meta = $meta, actionId = $actionId');
  }
```

处理自定义操作，目前仅记录日志，可以根据业务需求扩展逻辑。

## 状态更新事件

### 状态更新操作

```dart 458:492:example/lib/purchase_observer.dart
  @override
  void onboardingViewOnStateUpdatedAction(
    AdaptyUIOnboardingView view,
    AdaptyUIOnboardingMeta meta,
    String elementId,
    AdaptyOnboardingsStateUpdatedParams params,
  ) {
    print('#Example# onboardingViewOnStateUpdatedAction of $view, meta = $meta, elementId = $elementId');

    switch (params) {
      case AdaptyOnboardingsSelectParams(id: final id, value: final value, label: final label):
        print('#Example# onboardingViewOnStateUpdatedAction select $id $value $label');
        break;
      case AdaptyOnboardingsMultiSelectParams(params: final params):
        final paramsString = params.map((e) => '(id: ${e.id}, value: ${e.value}, label: ${e.label})').join(', ');
        print('#Example# onboardingViewOnStateUpdatedAction multiSelect: [$paramsString]');
        break;
      case AdaptyOnboardingsInputParams(input: final input):
        switch (input) {
          case AdaptyOnboardingsTextInput(value: final value):
            print('#Example# onboardingViewOnStateUpdatedAction text $value');
            break;
          case AdaptyOnboardingsEmailInput(value: final value):
            print('#Example# onboardingViewOnStateUpdatedAction email $value');
            break;
          case AdaptyOnboardingsNumberInput(value: final value):
            print('#Example# onboardingViewOnStateUpdatedAction number $value');
            break;
        }
        break;
      case AdaptyOnboardingsDatePickerParams(day: final day, month: final month, year: final year):
        print('#Example# onboardingViewOnStateUpdatedAction datePicker $day $month $year');
        break;
    }
  }
```

处理各种表单控件的状态更新：

- **单选选择**：记录选择项的ID、值和标签
- **多选选择**：记录所有选中项的详细信息
- **文本输入**：记录文本、邮箱、数字输入的值
- **日期选择**：记录选择的日期（日/月/年）

## 分析事件处理

### 分析事件回调

```dart 494:529:example/lib/purchase_observer.dart
  @override
  void onboardingViewOnAnalyticsEvent(
    AdaptyUIOnboardingView view,
    AdaptyUIOnboardingMeta meta,
    AdaptyOnboardingsAnalyticsEvent event,
  ) {
    switch (event) {
      case AdaptyOnboardingsAnalyticsEventOnboardingStarted():
        print('#Example# onboardingViewOnAnalyticsEvent onboardingStarted, meta = $meta');
        break;
      case AdaptyOnboardingsAnalyticsEventScreenPresented():
        print('#Example# onboardingViewOnAnalyticsEvent screenPresented, meta = $meta');
        break;
      case AdaptyOnboardingsAnalyticsEventScreenCompleted(elementId: final elementId, reply: final reply):
        print('#Example# onboardingViewOnAnalyticsEvent screenCompleted, meta = $meta, elementId = $elementId, reply = $reply');
        break;
      case AdaptyOnboardingsAnalyticsEventSecondScreenPresented():
        print('#Example# onboardingViewOnAnalyticsEvent secondScreenPresented, meta = $meta');
        break;
      case AdaptyOnboardingsAnalyticsEventRegistrationScreenPresented():
        print('#Example# onboardingViewOnAnalyticsEvent registrationScreenPresented, meta = $meta');
        break;
      case AdaptyOnboardingsAnalyticsEventProductsScreenPresented():
        print('#Example# onboardingViewOnAnalyticsEvent productsScreenPresented, meta = $meta');
        break;
      case AdaptyOnboardingsAnalyticsEventUserEmailCollected():
        print('#Example# onboardingViewOnAnalyticsEvent userEmailCollected, meta = $meta');
        break;
      case AdaptyOnboardingsAnalyticsEventOnboardingCompleted():
        print('#Example# onboardingViewOnAnalyticsEvent onboardingCompleted, meta = $meta');
        break;
      case AdaptyOnboardingsAnalyticsEventUnknown(name: final name):
        print('#Example# onboardingViewOnAnalyticsEvent unknown $name');
    }
  }
```

处理引导流程中的各种分析事件：

- **引导开始**：用户开始引导流程
- **屏幕展示**：每个引导屏幕被展示
- **屏幕完成**：用户完成某个屏幕，包含元素ID和回复内容
- **第二屏幕展示**：特定屏幕的展示事件
- **注册屏幕展示**：注册相关屏幕的展示
- **产品屏幕展示**：产品展示屏幕的展示
- **邮箱收集**：成功收集到用户邮箱
- **引导完成**：整个引导流程完成
- **未知事件**：未识别的分析事件

## 设计特点

1. **全面的事件覆盖**：监听引导页面的所有关键事件
2. **详细的状态跟踪**：记录用户在表单中的所有交互
3. **分析数据收集**：为产品优化提供完整的数据支持
4. **灵活的操作处理**：支持自定义操作和标准操作
5. **条件渲染支持**：根据渲染模式（原生/网页）调整行为
6. **错误容错**：支付墙跳转包含完整的错误处理
7. **调试友好**：所有事件都有详细的日志输出

## 实际应用场景

这个观察者实现支持典型的引导流程：

1. **用户引导**：通过多屏引导收集用户信息
2. **表单交互**：支持文本、选择、日期等多种输入类型
3. **商业化转化**：在适当时机展示支付墙
4. **数据分析**：跟踪用户行为和转化 funnel
5. **个性化体验**：根据用户输入调整后续内容

这种设计使得引导页面不仅能收集信息，还能无缝地引导用户完成购买转化。
