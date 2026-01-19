# PurchasesObserver 类核心结构分析

## 类概述

`PurchasesObserver` 是一个单例类，用于管理 Adapty SDK 的所有购买相关操作和事件监听。它实现了两个重要的观察者接口，负责处理支付墙和引导页面的各种事件。

```dart 7:22:example/lib/purchase_observer.dart
class PurchasesObserver implements AdaptyUIPaywallsEventsObserver, AdaptyUIOnboardingsEventsObserver {
  void Function(AdaptyError)? onAdaptyErrorOccurred;
  void Function(Object)? onUnknownErrorOccurred;
  void Function(AdaptyInstallationStatus)? onInstallationStatusUpdated;

  final adapty = Adapty();
  AdaptyInstallationStatus installationStatus = AdaptyInstallationStatusNotDetermined();

  static final PurchasesObserver _instance = PurchasesObserver._internal();

  factory PurchasesObserver() {
    return _instance;
  }

  PurchasesObserver._internal();
```

## 单例模式实现

该类使用经典的 Dart 单例模式实现，确保整个应用中只有一个实例：

- `_instance`：静态私有实例变量
- `_internal()`：私有构造函数
- 工厂构造函数返回已创建的实例

## 主要属性

- `onAdaptyErrorOccurred`：Adapty 错误回调函数
- `onUnknownErrorOccurred`：未知错误回调函数  
- `onInstallationStatusUpdated`：安装状态更新回调函数
- `adapty`：Adapty SDK 实例
- `installationStatus`：当前安装状态

## 初始化方法详解

```dart 23:75:example/lib/purchase_observer.dart
  Future<void> initialize() async {
    try {
      Adapty().setLogLevel(AdaptyLogLevel.debug);

      var isActivated = false;

      if (kDebugMode) {
        isActivated = await Adapty().isActivated();
      } else {
        isActivated = false;
      }

      if (!isActivated) {
        await Adapty().activate(
          configuration: AdaptyConfiguration(apiKey: 'public_live_iNuUlSsN.83zcTTR8D5Y8FI9cGUI6')
            ..withLogLevel(AdaptyLogLevel.debug)
            ..withObserverMode(false)
            // ..withCustomerUserId(
            //   "test_1234567890",
            //   iosAppAccountToken: '1234567890',
            //   androidObfuscatedAccountId: '1234567890',
            // )
            ..withIpAddressCollectionDisabled(false)
            ..withAppleIdfaCollectionDisabled(false)
            ..withGoogleAdvertisingIdCollectionDisabled(false)
            ..withGoogleEnablePendingPrepaidPlans(false)
            ..withAppleClearDataOnBackup(false)
            ..withActivateUI(true),
        );

        _setFallbackPaywalls();
      } else {
        Adapty().setupAfterHotRestart();
      }

      AdaptyUI().setPaywallsEventsObserver(this);
      AdaptyUI().setOnboardingsEventsObserver(this);

      Adapty().onUpdateInstallationDetailsSuccessStream.listen((details) {
        print('#Example# onUpdateInstallationDetailsSuccessStream $details');
        installationStatus = AdaptyInstallationStatusDetermined(details);
        onInstallationStatusUpdated?.call(AdaptyInstallationStatusDetermined(details));
      });

      Adapty().onUpdateInstallationDetailsFailStream.listen((error) {
        print('#Example# onUpdateInstallationDetailsFailStream $error');
      });

      await callGetPaywallForDefaultAudience('example_ab_test');
    } catch (e) {
      print('#Example# activate error $e');
    }
  }
```

### 初始化流程

1. **设置日志级别**：在调试模式下启用详细日志
2. **检查激活状态**：
   - 调试模式：检查 SDK 是否已激活
   - 生产模式：直接设为未激活
3. **SDK 激活**（如果未激活）：
   - 配置 API Key
   - 设置各种数据收集选项
   - 启用 UI 功能
   - 设置备用支付墙
4. **设置观察者**：将当前实例注册为支付墙和引导页面的观察者
5. **监听安装状态更新**：订阅安装详情更新流
6. **初始化默认支付墙**：获取 A/B 测试用的支付墙

### 配置选项说明

- `withObserverMode(false)`：关闭观察者模式
- `withIpAddressCollectionDisabled(false)`：启用 IP 地址收集
- `withAppleIdfaCollectionDisabled(false)`：启用 Apple IDFA 收集
- `withGoogleAdvertisingIdCollectionDisabled(false)`：启用 Google Advertising ID 收集
- `withActivateUI(true)`：启用 Adapty UI 功能

## 错误处理机制

```dart 77:94:example/lib/purchase_observer.dart
  Future<T?> _withErrorHandling<T>(
    Future<T> Function() body, {
    bool suppressError = false,
  }) async {
    try {
      return await body();
    } on AdaptyError catch (adaptyError) {
      if (!suppressError) {
        onAdaptyErrorOccurred?.call(adaptyError);
      }
    } catch (e) {
      if (!suppressError) {
        onUnknownErrorOccurred?.call(e);
      }
    }

    return null;
  }
```

这是一个通用的错误处理包装器：

- 区分 `AdaptyError` 和其他异常
- 支持抑制错误（`suppressError` 参数）
- 返回 `null` 表示操作失败
- 通过回调函数向上层报告错误

这种设计确保了错误处理的一致性和可靠性，同时给调用者提供了灵活的错误处理控制。
