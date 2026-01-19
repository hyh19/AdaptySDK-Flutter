# PaywallsList 页面组件详解

## 概述

这个文件实现了一个 Flutter 页面组件 `PaywallsList`，用于展示和管理 Adapty SDK 中的付费墙（Paywalls）。它是 Adapty Flutter SDK 示例应用的一部分，主要负责获取、展示和管理付费墙的生命周期。

## 核心架构

### PaywallsList 类

`PaywallsList` 是一个 StatefulWidget，负责展示付费墙列表及其相关操作。

```dart 15:27:example/lib/screens/paywalls_list_screen.dart
class PaywallsList extends StatefulWidget {
  const PaywallsList({
    super.key,
    required this.adaptyErrorCallback,
    required this.customErrorCallback,
  });

  final OnAdaptyErrorCallback adaptyErrorCallback;
  final OnCustomErrorCallback customErrorCallback;

  @override
  State<PaywallsList> createState() => _PaywallsListState();
}
```

该组件接收两个回调函数：

- `adaptyErrorCallback`: 处理 Adapty 特定的错误
- `customErrorCallback`: 处理其他类型的错误

### PaywallsListItem 数据模型

用于存储付费墙数据的辅助类：

```dart 29:39:example/lib/screens/paywalls_list_screen.dart
class PaywallsListItem {
  String id;
  AdaptyPaywall? paywall;
  AdaptyError? error;

  PaywallsListItem({
    required this.id,
    this.paywall,
    this.error,
  });
}
```

## 状态管理

### 核心状态变量

```dart 41:44:example/lib/screens/paywalls_list_screen.dart
class _PaywallsListState extends State<PaywallsList> {
  List<String>? _paywallsIds;

  final Map<String, PaywallsListItem> _paywallsItems = {};
```

- `_paywallsIds`: 存储付费墙 ID 列表
- `_paywallsItems`: 以 ID 为键的付费墙数据映射

### 初始化逻辑

```dart 47:57:example/lib/screens/paywalls_list_screen.dart
@override
void initState() {
  PaywallsViewSharedState().onNeedsUpdateState = (ids) {
    setState(() {
      _paywallsIds = ids;
      _loadPaywalls();
    });
  };
  _loadPaywalls();

  super.initState();
}
```

组件初始化时设置状态更新回调，并立即开始加载付费墙数据。

## 数据加载机制

### 单个付费墙数据加载

```dart 59:74:example/lib/screens/paywalls_list_screen.dart
Future<void> _loadPaywallData(String id) async {
  try {
    _paywallsItems[id] = PaywallsListItem(
      id: id,
      paywall: await Adapty().getPaywall(placementId: id),
    );

    setState(() {});
  } on AdaptyError catch (e) {
    _paywallsItems[id] = PaywallsListItem(id: id, error: e);

    widget.adaptyErrorCallback(e);
  } catch (e) {
    widget.customErrorCallback(e);
  }
}
```

异步加载单个付费墙数据，包含完善的错误处理机制。

### 批量付费墙加载

```dart 76:82:example/lib/screens/paywalls_list_screen.dart
void _loadPaywalls() {
  for (var id in _paywallsIds ?? []) {
    _paywallsItems[id] = PaywallsListItem(id: id);
    setState(() {});
    _loadPaywallData(id);
  }
}
```

遍历所有付费墙 ID，初始化数据结构并异步加载每个付费墙。

## 自定义资源配置

### 自定义标签（Tags）

```dart 87:92:example/lib/screens/paywalls_list_screen.dart
final Map<String, String>? _customTags = {
  'CUSTOM_TAG_NAME': 'Walter White',
  'CUSTOM_TAG_PHONE': '+1 234 567890',
  'CUSTOM_TAG_CITY': 'Albuquerque',
  'CUSTOM_TAG_EMAIL': 'walter@white.com',
};
```

用于在付费墙中替换占位符文本的自定义标签数据。

### 自定义计时器（Timers）

```dart 94:102:example/lib/screens/paywalls_list_screen.dart
final Map<String, DateTime>? _customTimers = {
  'CUSTOM_TIMER_24H': DateTime.now().add(const Duration(seconds: 86400)),
  'CUSTOM_TIMER_10H': DateTime.now().add(const Duration(seconds: 36000)),
  'CUSTOM_TIMER_1H': DateTime.now().add(const Duration(seconds: 3600)),
  'CUSTOM_TIMER_10M': DateTime.now().add(const Duration(seconds: 600)),
  'CUSTOM_TIMER_1M': DateTime.now().add(const Duration(seconds: 60)),
  'CUSTOM_TIMER_10S': DateTime.now().add(const Duration(seconds: 10)),
  'CUSTOM_TIMER_5S': DateTime.now().add(const Duration(seconds: 5)),
};
```

用于倒计时功能的自定义计时器配置。

### 自定义资源（Assets）

```dart 104:128:example/lib/screens/paywalls_list_screen.dart
final Map<String, AdaptyCustomAsset>? _customAssets = {
  'custom_image_walter_white': AdaptyCustomAsset.localImageAsset(
    assetId: 'assets/images/Walter_White.png',
  ),
  'hero_image': AdaptyCustomAsset.localImageAsset(
    assetId: 'assets/images/landscape.png',
  ),
  'apple_icon_image': AdaptyCustomAsset.localImageData(
    data: base64ImageData,
  ),
  'custom_video_mp4': AdaptyCustomAsset.localVideoAsset(
    assetId: 'assets/videos/demo_video.mp4',
  ),
  'custom_image_landscape': AdaptyCustomAsset.localImageAsset(
    assetId: 'assets/images/landscape.png',
  ),
  'custom_color_orange': AdaptyCustomAsset.color(color: Colors.orange),
  'custom_bright_gradient': AdaptyCustomAsset.linearGradient(
    gradient: LinearGradient(
      colors: [Colors.white.withAlpha(0), Colors.green.withAlpha(128), Colors.yellow],
      begin: Alignment.topCenter,
      end: Alignment.bottomCenter,
    ),
  ),
};
```

支持多种类型的自定义资源：

- 本地图片资源
- Base64 编码图片数据
- 本地视频资源
- 颜色资源
- 渐变资源

## 付费墙展示功能

### 原生付费墙展示

```dart 130:171:example/lib/screens/paywalls_list_screen.dart
Future<void> _createAndPresentPaywallView(
  AdaptyPaywall paywall,
  bool loadProducts,
  AdaptyUIIOSPresentationStyle iosPresentationStyle,
) async {
  setState(() {
    _loadingPaywall = !loadProducts;
    _loadingPaywallWithProducts = loadProducts;
  });

  try {
    final view = await AdaptyUI().createPaywallView(
      paywall: paywall,
      customTags: _customTags,
      customTimers: _customTimers,
      customAssets: _customAssets,
      preloadProducts: loadProducts,
      productPurchaseParams: Map.fromEntries(
        paywall.productIdentifiers.map(
          (e) {
            final parameters = AdaptyPurchaseParametersBuilder();
            // ..setObfuscatedAccountId('123e4567-e89b-12d3-a456-426614174000')
            // ..setObfuscatedProfileId('123e4567-e89b-12d3-a456-426614174000');

            return MapEntry(e, parameters.build());
          },
        ),
      ),
    );

    await view.present(iosPresentationStyle: iosPresentationStyle);
  } on AdaptyError catch (e) {
    widget.adaptyErrorCallback(e);
  } catch (e) {
    widget.customErrorCallback(e);
  } finally {
    setState(() {
      _loadingPaywall = false;
      _loadingPaywallWithProducts = false;
    });
  }
}
```

创建并展示原生付费墙视图，支持：

- 自定义标签、计时器和资源
- 产品预加载选项
- 购买参数配置
- iOS 特定的展示样式

### 平台视图展示

```dart 184:309:example/lib/screens/paywalls_list_screen.dart
Future<void> _showPaywallPlatformView(AdaptyPaywall paywall, bool showToastEvents) async {
  try {
    await Navigator.of(context).push(
      CupertinoPageRoute(
        builder: (BuildContext context) => CupertinoPageScaffold(
          navigationBar: CupertinoNavigationBar(
            middle: Text('Paywall ${paywall.placement.id}'),
          ),
          child: SafeArea(
            child: Container(
              width: double.infinity,
              height: double.infinity,
              color: CupertinoColors.systemBackground,
              child: AdaptyUIPaywallPlatformView(
                paywall: paywall,
                customTags: _customTags,
                customTimers: _customTimers,
                customAssets: _customAssets,
                productPurchaseParams: Map.fromEntries(
                  paywall.productIdentifiers.map(
                    (e) => MapEntry(e, AdaptyPurchaseParametersBuilder().build()),
                  ),
                ),
                onDidAppear: (view) {
                  print('#Example# Platform View onDidAppear');
                  if (showToastEvents) {
                    _showToast('Action: onDidAppear', 'View: $view');
                  }
                },
                // ... 其他事件处理
                onDidPerformAction: (view, action) {
                  print('#Example# Platform View onDidPerformAction: $action');
                  if (showToastEvents) {
                    _showToast('Action: onDidPerformAction', 'Action: $action');
                  }

                  switch (action) {
                    case const CloseAction():
                      Navigator.of(context).pop();
                      break;
                    default:
                      break;
                  }
                },
                // ... 更多事件处理器
              ),
            ),
          ),
        ),
      ),
    );
  } on AdaptyError catch (e) {
    widget.adaptyErrorCallback(e);
  } catch (e) {
    widget.customErrorCallback(e);
  } finally {
    setState(() {
      _loadingPaywall = false;
      _loadingPaywallWithProducts = false;
    });
  }
}
```

展示平台特定的付费墙视图，包含完整的事件处理：

- 视图出现/消失事件
- 用户操作事件（关闭等）
- 产品选择事件
- 购买流程事件
- 恢复购买事件
- 渲染错误事件

## UI 构建逻辑

### 错误状态展示

```dart 311:319:example/lib/screens/paywalls_list_screen.dart
List<Widget> _buildErrorStatusItems() {
  return const [
    ListTextTile(
      title: 'Status',
      subtitle: 'Error',
      subtitleColor: CupertinoColors.systemRed,
    ),
  ];
}
```

当付费墙加载失败时显示错误状态。

### 付费墙信息展示

```dart 321:362:example/lib/screens/paywalls_list_screen.dart
List<Widget> _buildPaywallItems(AdaptyPaywall paywall) {
  return [
    const ListTextTile(
      title: 'Status',
      subtitle: 'OK',
      subtitleColor: CupertinoColors.systemGreen,
    ),
    ListTextTile(
      title: 'Variation Id',
      subtitle: paywall.variationId,
    ),
    ListTextTile(
      title: 'Has View',
      subtitle: paywall.hasViewConfiguration ? 'true' : 'false',
      subtitleColor: paywall.hasViewConfiguration ? CupertinoColors.systemGreen : CupertinoColors.systemRed,
    ),
    if (paywall.hasViewConfiguration) ...[
      if (Platform.isIOS) ...[
        ListActionTile(
          title: 'Present Page Sheet',
          showProgress: _loadingPaywall,
          onTap: () => _createAndPresentPaywallView(paywall, false, AdaptyUIIOSPresentationStyle.pageSheet),
        ),
      ],
      ListActionTile(
        title: 'Present Full Screen',
        showProgress: _loadingPaywall,
        onTap: () => _createAndPresentPaywallView(paywall, false, AdaptyUIIOSPresentationStyle.fullScreen),
      ),
      ListActionTile(
        title: 'Present Platform View',
        showProgress: _loadingPaywall,
        onTap: () => _showPaywallPlatformView(paywall, true),
      ),
      ListActionTile(
        title: 'Load Products and Present',
        showProgress: _loadingPaywallWithProducts,
        onTap: () => _createAndPresentPaywallView(paywall, true, AdaptyUIIOSPresentationStyle.fullScreen),
      ),
    ],
  ];
}
```

构建付费墙信息展示列表，包含：

- 状态信息
- 变体 ID
- 视图配置状态
- 不同展示方式的操作按钮（基于平台和配置）

## 页面布局

```dart 364:379:example/lib/screens/paywalls_list_screen.dart
@override
Widget build(BuildContext context) {
  return SafeArea(
    child: ListView(
      children: (_paywallsIds ?? []).map((paywallId) {
        final item = _paywallsItems[paywallId];

        return ListSection(
          headerText: 'Paywall $paywallId',
          children: item?.paywall == null ? _buildErrorStatusItems() : _buildPaywallItems(item!.paywall!),
        );
      }).toList(),
    ),
  );
}
```

使用 ListView 展示付费墙列表，每个付费墙作为一个独立的区块显示其状态和可用操作。

## 工具方法

### Toast 提示

```dart 173:182:example/lib/screens/paywalls_list_screen.dart
void _showToast(String title, String description) {
  toastification.show(
    context: context,
    type: ToastificationType.info,
    style: ToastificationStyle.minimal,
    title: Text(title),
    description: Text(description),
    autoCloseDuration: const Duration(seconds: 3),
  );
}
```

用于显示用户友好的提示信息。

## 关键特性

1. **异步数据加载**: 支持并发加载多个付费墙数据
2. **错误处理**: 区分 Adapty 错误和其他异常
3. **自定义资源**: 支持丰富的自定义标签、计时器和资源
4. **多种展示方式**: 原生视图、平台视图、全屏和页面表单
5. **事件监听**: 完整的付费墙生命周期事件处理
6. **平台适配**: iOS 特定的展示样式支持
7. **状态管理**: 响应式状态更新和加载指示器
8. **用户反馈**: Toast 提示和视觉状态指示

这个组件展示了 Adapty Flutter SDK 的完整功能，是一个很好的付费墙管理示例。
