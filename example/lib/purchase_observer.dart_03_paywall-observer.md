# PurchasesObserver 支付墙观察者方法详解

## 概述

该类实现了 `AdaptyUIPaywallsEventsObserver` 接口，负责监听和处理支付墙视图的所有生命周期事件和用户交互。

## 视图生命周期事件

### 支付墙视图出现

```dart 269:272:example/lib/purchase_observer.dart
  @override
  void paywallViewDidAppear(AdaptyUIPaywallView view) {
    print('#Example# paywallViewDidAppear of $view');
  }
```

当支付墙视图显示在屏幕上时触发，仅用于调试日志记录。

### 支付墙视图消失

```dart 274:277:example/lib/purchase_observer.dart
  @override
  void paywallViewDidDisappear(AdaptyUIPaywallView view) {
    print('#Example# paywallViewDidDisappear of $view');
  }
```

当支付墙视图从屏幕上移除时触发，仅用于调试日志记录。

## 用户操作处理

### 执行操作处理

```dart 279:311:example/lib/purchase_observer.dart
  @override
  void paywallViewDidPerformAction(AdaptyUIPaywallView view, AdaptyUIAction action) async {
    print('#Example# paywallViewDidPerformAction ${action.runtimeType} of $view');

    switch (action) {
      case const CloseAction():
      case const AndroidSystemBackAction():
        view.dismiss();
        break;
      case OpenUrlAction(url: final url):
        final Uri uri = Uri.parse(url);

        final selectedAction = await view.showDialog(
          title: 'Open URL?',
          content: url,
          primaryActionTitle: 'Cancel',
          secondaryActionTitle: 'OK',
        );

        switch (selectedAction) {
          case AdaptyUIDialogActionType.primary:
            print('#Example# paywallViewDidPerformAction primaryAction');
            break;
          case AdaptyUIDialogActionType.secondary:
            print('#Example# paywallViewDidPerformAction secondaryAction');
            launchUrl(uri, mode: LaunchMode.inAppBrowserView);
            break;
        }
        break;
      default:
        break;
    }
  }
```

处理用户在支付墙中的各种操作：

- **关闭操作**：直接关闭支付墙视图
- **系统返回**（Android）：等同于关闭操作
- **打开URL**：显示确认对话框，用户确认后在应用内浏览器中打开

## 购买流程事件

### 购买失败（产品加载失败）

```dart 314:316:example/lib/purchase_observer.dart
  @override
  void paywallViewDidFailLoadingProducts(AdaptyUIPaywallView view, AdaptyError error) {
    print('#Example# paywallViewDidFailLoadingProducts of $view, error = $error');
  }
```

产品信息加载失败时的错误处理，仅记录日志。

### 渲染失败

```dart 318:321:example/lib/purchase_observer.dart
  @override
  void paywallViewDidFailRendering(AdaptyUIPaywallView view, AdaptyError error) {
    view.dismiss();
  }
```

支付墙渲染失败时自动关闭视图，避免显示空白或错误界面。

### 购买完成

```dart 323:338:example/lib/purchase_observer.dart
  @override
  void paywallViewDidFinishPurchase(AdaptyUIPaywallView view, AdaptyPaywallProduct product, AdaptyPurchaseResult purchaseResult) {
    print('#Example# paywallViewDidFinishPurchase of $view');

    switch (purchaseResult) {
      case AdaptyPurchaseResultSuccess(profile: final profile):
        if (profile.accessLevels['premium']?.isActive ?? false) {
          view.dismiss();
        }
        break;
      case AdaptyPurchaseResultPending():
        break;
      case AdaptyPurchaseResultUserCancelled():
        break;
    }
  }
```

处理购买完成事件：

- **成功购买**：检查用户是否获得高级访问权限，如果是则自动关闭支付墙
- **待处理**：购买正在处理中，保持视图打开
- **用户取消**：用户主动取消购买，保持视图打开

### 购买失败

```dart 340:343:example/lib/purchase_observer.dart
  @override
  void paywallViewDidFailPurchase(AdaptyUIPaywallView view, AdaptyPaywallProduct product, AdaptyError error) {
    print('#Example# paywallViewDidFailPurchase ${product.vendorProductId} of $view, error = $error');
  }
```

购买失败时记录详细错误信息，包括产品ID。

## 恢复购买流程

### 开始恢复购买

```dart 345:348:example/lib/purchase_observer.dart
  @override
  void paywallViewDidStartRestore(AdaptyUIPaywallView view) {
    print('#Example# paywallViewDidStartRestore of $view');
  }
```

恢复购买过程开始时的日志记录。

### 恢复购买完成

```dart 350:367:example/lib/purchase_observer.dart
  @override
  void paywallViewDidFinishRestore(AdaptyUIPaywallView view, AdaptyProfile profile) {
    print('#Example# paywallViewDidFinishRestore of $view');

    _handleFinishRestore(view, profile);
  }

  Future<void> _handleFinishRestore(AdaptyUIPaywallView view, AdaptyProfile profile) async {
    await view.showDialog(
      title: 'Success!',
      content: 'Purchases were successfully restored.',
      primaryActionTitle: 'OK',
    );

    if (profile.accessLevels['premium']?.isActive ?? false) {
      await view.dismiss();
    }
  }
```

恢复购买成功后：

1. 显示成功对话框
2. 如果用户获得高级权限，自动关闭支付墙

### 恢复购买失败

```dart 369:378:example/lib/purchase_observer.dart
  @override
  void paywallViewDidFailRestore(AdaptyUIPaywallView view, AdaptyError error) {
    print('#Example# paywallViewDidFailRestore of $view, error = $error');

    view.showDialog(
      title: 'Error!',
      content: error.toString(),
      primaryActionTitle: 'OK',
    );
  }
```

恢复购买失败时显示错误对话框给用户。

## 用户交互事件

### 产品选择

```dart 380:383:example/lib/purchase_observer.dart
  @override
  void paywallViewDidSelectProduct(AdaptyUIPaywallView view, String productId) {
    print('#Example# paywallViewDidSelectProduct $productId of $view');
  }
```

用户选择不同产品时的日志记录。

### 开始购买

```dart 385:388:example/lib/purchase_observer.dart
  @override
  void paywallViewDidStartPurchase(AdaptyUIPaywallView view, AdaptyPaywallProduct product) {
    print('#Example# paywallViewDidStartPurchase ${product.vendorProductId} of $view');
  }
```

购买流程开始时的日志记录，包含产品信息。

## 网页支付事件

### 网页支付导航完成

```dart 390:397:example/lib/purchase_observer.dart
  @override
  void paywallViewDidFinishWebPaymentNavigation(
    AdaptyUIPaywallView view,
    AdaptyPaywallProduct? product,
    AdaptyError? error,
  ) {
    print('#Example# paywallViewDidFinishWebPaymentNavigation of $view, product = $product, error = $error');
  }
```

网页支付导航完成时的回调，包含产品信息和可能的错误。

## 设计特点

1. **事件驱动**：所有方法都是对用户操作或系统事件的响应
2. **用户体验优化**：
   - 成功购买高级内容后自动关闭支付墙
   - 渲染失败时自动关闭避免空白界面
   - 失败时显示用户友好的错误提示
3. **调试支持**：大量日志输出便于问题排查
4. **异步处理**：涉及用户交互的方法使用 async/await
5. **类型安全**：充分利用 Dart 的模式匹配和类型系统
