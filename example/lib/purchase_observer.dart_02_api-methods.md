# PurchasesObserver API 方法详解

## 概述

该类封装了所有 Adapty SDK 的核心 API 调用，每个方法都使用统一的错误处理机制，确保了代码的健壮性和一致性。

## 基础数据获取方法

### 获取当前安装状态

```dart 96:103:example/lib/purchase_observer.dart
  Future<AdaptyInstallationStatus?> callGetCurrentInstallationStatus() async {
    return _withErrorHandling(() async {
      final status = await adapty.getCurrentInstallationStatus();
      installationStatus = status;
      onInstallationStatusUpdated?.call(status);
      return status;
    });
  }
```

获取并更新当前安装状态，同时触发状态更新回调。

### 设置备用支付墙

```dart 105:110:example/lib/purchase_observer.dart
  Future<void> _setFallbackPaywalls() async {
    final assetId = Platform.isIOS ? 'assets/fallback_ios.json' : 'assets/fallback_android.json';
    return _withErrorHandling(() async {
      await adapty.setFallback(assetId);
    });
  }
```

根据平台设置相应的备用支付墙配置文件。

### 获取用户档案

```dart 112:116:example/lib/purchase_observer.dart
  Future<AdaptyProfile?> callGetProfile() async {
    return _withErrorHandling(() async {
      return await adapty.getProfile();
    });
  }
```

获取当前用户的完整档案信息，包括订阅状态、购买历史等。

## 用户身份管理

### 用户身份识别

```dart 118:134:example/lib/purchase_observer.dart
  Future<void> callIdentifyUser(
    String customerUserId, {
    String? iosAppAccountToken,
    String? androidObfuscatedAccountId,
  }) async {
    try {
      await adapty.identify(
        customerUserId,
        iosAppAccountToken: iosAppAccountToken,
        androidObfuscatedAccountId: androidObfuscatedAccountId,
      );
    } on AdaptyError catch (adaptyError) {
      onAdaptyErrorOccurred?.call(adaptyError);
    } catch (e) {
      onUnknownErrorOccurred?.call(e);
    }
  }
```

识别用户身份，支持跨平台用户 ID 关联：

- `customerUserId`：主要用户标识符
- `iosAppAccountToken`：iOS 应用账户令牌
- `androidObfuscatedAccountId`：Android 混淆账户 ID

### 更新用户档案

```dart 136:140:example/lib/purchase_observer.dart
  Future<void> callUpdateProfile(AdaptyProfileParameters params) async {
    return _withErrorHandling(() async {
      await adapty.updateProfile(params);
    });
  }
```

更新用户档案信息，如自定义属性、邮箱等。

### 设置集成标识符

```dart 142:146:example/lib/purchase_observer.dart
  Future<void> callSetIntegrationIdentifier(String key, String value) async {
    return _withErrorHandling(() async {
      await adapty.setIntegrationIdentifier(key: key, value: value);
    });
  }
```

设置第三方集成服务的标识符，用于数据同步。

### 用户登出

```dart 224:228:example/lib/purchase_observer.dart
  Future<void> callLogout() async {
    return _withErrorHandling(() async {
      return await adapty.logout();
    });
  }
```

清除用户身份信息，重置为匿名状态。

## 支付墙管理

### 获取默认受众支付墙

```dart 148:156:example/lib/purchase_observer.dart
  Future<AdaptyPaywall?> callGetPaywallForDefaultAudience(
    String placementId,
  ) async {
    return _withErrorHandling(() async {
      return await adapty.getPaywallForDefaultAudience(
        placementId: placementId,
      );
    });
  }
```

根据位置 ID 获取适用于默认受众的支付墙。

### 获取支付墙（完整参数）

```dart 158:171:example/lib/purchase_observer.dart
  Future<AdaptyPaywall?> callGetPaywall(
    String paywallId,
    String? locale,
    AdaptyPaywallFetchPolicy fetchPolicy,
  ) async {
    return _withErrorHandling(() async {
      return await adapty.getPaywall(
        placementId: paywallId,
        locale: locale,
        fetchPolicy: fetchPolicy,
        loadTimeout: const Duration(seconds: 5),
      );
    });
  }
```

获取指定支付墙，支持本地化、缓存策略和超时设置。

### 获取支付墙产品

```dart 173:177:example/lib/purchase_observer.dart
  Future<List<AdaptyPaywallProduct>?> callGetPaywallProducts(AdaptyPaywall paywall) async {
    return _withErrorHandling(() async {
      return await adapty.getPaywallProducts(paywall: paywall);
    });
  }
```

获取支付墙中包含的所有产品信息。

## 购买相关方法

### 执行购买

```dart 179:189:example/lib/purchase_observer.dart
  Future<AdaptyPurchaseResult?> callMakePurchase(
    AdaptyPaywallProduct product,
    AdaptyPurchaseParameters? parameters,
  ) async {
    return _withErrorHandling(() async {
      return await adapty.makePurchase(
        product: product,
        parameters: parameters,
      );
    });
  }
```

执行产品购买，支持自定义购买参数。

### 恢复购买

```dart 191:195:example/lib/purchase_observer.dart
  Future<AdaptyProfile?> callRestorePurchases() async {
    return _withErrorHandling(() async {
      return await adapty.restorePurchases();
    });
  }
```

恢复用户之前的购买记录。

## 归因和分析

### 更新归因数据

```dart 197:207:example/lib/purchase_observer.dart
  Future<void> callUpdateAttribution(
    Map<dynamic, dynamic> attribution,
    String source,
  ) async {
    return _withErrorHandling(() async {
      await adapty.updateAttribution(
        attribution,
        source: source,
      );
    });
  }
```

更新用户归因数据，用于营销分析。

### 记录支付墙展示

```dart 209:213:example/lib/purchase_observer.dart
  Future<void> callLogShowPaywall(AdaptyPaywall paywall) async {
    return _withErrorHandling(() async {
      return await adapty.logShowPaywall(paywall: paywall);
    });
  }
```

记录支付墙展示事件，用于分析用户行为。

### 报告交易

```dart 215:222:example/lib/purchase_observer.dart
  Future<void> callReportTransaction({
    required String transactionId,
    String? variationId,
  }) async {
    return _withErrorHandling(() async {
      return await adapty.reportTransaction(transactionId: transactionId, variationId: variationId);
    });
  }
```

报告外部交易数据，用于收入追踪。

## 退款和优惠

### 展示代码兑换界面

```dart 230:234:example/lib/purchase_observer.dart
  Future<void> callPresentCodeRedemptionSheet() async {
    return _withErrorHandling(() async {
      return await adapty.presentCodeRedemptionSheet();
    });
  }
```

显示优惠码兑换界面。

### 更新退款数据收集同意状态

```dart 236:240:example/lib/purchase_observer.dart
  Future<void> callUpdateCollectingRefundDataConsent(bool consent) async {
    return _withErrorHandling(() async {
      return await adapty.updateCollectingRefundDataConsent(consent);
    });
  }
```

设置是否同意收集退款相关数据。

### 更新退款偏好

```dart 242:246:example/lib/purchase_observer.dart
  Future<void> callUpdateRefundPreference(AdaptyRefundPreference refundPreference) async {
    return _withErrorHandling(() async {
      return await adapty.updateRefundPreference(refundPreference);
    });
  }
```

设置用户的退款偏好选项。

## 网页支付

### 创建网页支付墙URL

```dart 248:252:example/lib/purchase_observer.dart
  Future<String?> callCreateWebPaywallUrl(AdaptyPaywallProduct product) async {
    return _withErrorHandling(() async {
      return await adapty.createWebPaywallUrl(product: product);
    }, suppressError: true);
  }
```

为产品创建网页支付墙链接，错误被抑制（返回null）。

### 打开网页支付墙

```dart 254:265:example/lib/purchase_observer.dart
  Future<void> callOpenWebPaywall({
    AdaptyPaywall? paywall,
    AdaptyPaywallProduct? product,
  }) async {
    return _withErrorHandling(() async {
      return await adapty.openWebPaywall(
        paywall: paywall,
        product: product,
        openIn: AdaptyWebPresentation.inAppBrowser,
      );
    });
  }
```

在应用内浏览器中打开网页支付墙。

## 方法设计特点

1. **统一错误处理**：所有方法都使用 `_withErrorHandling` 包装器
2. **类型安全**：使用 Dart 的强类型系统确保参数和返回值类型正确
3. **异步操作**：所有网络和I/O操作都是异步的
4. **灵活配置**：支持可选参数以适应不同使用场景
5. **平台适配**：自动处理 iOS 和 Android 平台的差异
