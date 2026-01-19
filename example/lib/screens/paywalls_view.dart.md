# PaywallsView 代码解析

## 概述

`paywalls_view.dart` 文件实现了一个用于展示 AdaptyUI 支付墙的 Flutter 页面组件。该文件包含两个主要类：`PaywallsViewSharedState`（共享状态管理器）和 `PaywallsView`（UI 组件）。

## PaywallsViewSharedState 类详解

### 基本概念

这是一个单例类，用于在应用中共享和管理支付墙 ID 的状态。

```dart 6:41:example/lib/screens/paywalls_view.dart
class PaywallsViewSharedState {
  static final PaywallsViewSharedState _instance = PaywallsViewSharedState._internal();
  static const String _prefsKey = 'stored_paywall_ids';

  Function(List<String>)? onNeedsUpdateState;

  factory PaywallsViewSharedState() {
    return _instance;
  }

  List<String> paywallsIds = [];

  PaywallsViewSharedState._internal() {
    _restorePaywallIds();
  }

  Future<void> _restorePaywallIds() async {
    final prefs = await SharedPreferences.getInstance();
    paywallsIds = prefs.getStringList(_prefsKey) ?? [];
    if (!paywallsIds.contains('example_ab_test')) {
      paywallsIds.add('example_ab_test');
    }
    onNeedsUpdateState?.call(paywallsIds);
  }

  Future<void> addPaywallId(String id) async {
    if (!paywallsIds.contains(id)) {
      paywallsIds.insert(0, id);
    }

    final prefs = await SharedPreferences.getInstance();
    await prefs.setStringList(_prefsKey, paywallsIds);

    onNeedsUpdateState?.call(paywallsIds);
  }
}
```

### 核心特性

1. **单例模式实现**：
   - 使用私有构造函数 `_internal()` 和工厂构造函数确保全局唯一实例
   - 静态 `_instance` 变量存储单例实例

2. **数据持久化**：
   - 使用 `SharedPreferences` 进行本地数据存储
   - 键名 `_prefsKey = 'stored_paywall_ids'` 用于存储支付墙 ID 列表

3. **状态回调机制**：
   - `onNeedsUpdateState` 回调函数用于通知状态变化
   - 当支付墙 ID 列表更新时，会触发回调通知相关组件

4. **默认数据处理**：
   - 初始化时会检查并添加 `'example_ab_test'` 示例支付墙 ID
   - 确保示例数据始终存在

### 方法说明

- **`_restorePaywallIds()`**: 从本地存储恢复支付墙 ID 列表，并添加默认示例数据
- **`addPaywallId(String id)`**: 添加新的支付墙 ID 到列表开头，并持久化到本地存储

## PaywallsView 组件详解

### 组件结构

```dart 43:51:example/lib/screens/paywalls_view.dart
class PaywallsView extends StatefulWidget {
  final OnAdaptyErrorCallback adaptyErrorCallback;
  final OnCustomErrorCallback customErrorCallback;

  PaywallsView({super.key, required this.adaptyErrorCallback, required this.customErrorCallback});

  @override
  State<PaywallsView> createState() => _PaywallsViewState();
}
```

这是一个有状态的 Flutter 组件，接收两个错误处理回调函数作为必需参数。

### 状态管理类 _PaywallsViewState

```dart 53:97:example/lib/screens/paywalls_view.dart
class _PaywallsViewState extends State<PaywallsView> {
  final sharedState = PaywallsViewSharedState();

  Future<void> _addButtonPressed(BuildContext context) {
    return showCupertinoDialog(
      context: context,
      builder: (ctx) => CupertinoAlertDialog(
        title: const Text('Add and save Paywall'),
        content: Column(
          children: [
            Padding(
              padding: const EdgeInsets.only(top: 8.0),
              child: CupertinoTextField(
                placeholder: 'Enter Placement Id',
                onChanged: (value) {
                  _currentlyAddedPaywallId = value;
                },
              ),
            ),
          ],
        ),
        actions: [
          CupertinoButton(
              child: const Text('OK'),
              onPressed: () {
                Navigator.of(ctx).pop();

                if (_currentlyAddedPaywallId != null) {
                  setState(() {
                    sharedState.addPaywallId(_currentlyAddedPaywallId!);
                  });
                }
              }),
        ],
      ),
    );
  }

  String? _currentlyAddedPaywallId;

  @override
  void initState() {
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return CupertinoPageScaffold(
      navigationBar: CupertinoNavigationBar(
        middle: const Text('AdaptyUI'),
        trailing: CupertinoButton(
          onPressed: () => _addButtonPressed(context),
          child: const Icon(CupertinoIcons.add),
        ),
      ),
      child: PaywallsList(
        adaptyErrorCallback: widget.adaptyErrorCallback,
        customErrorCallback: widget.customErrorCallback,
      ),
    );
  }
}
```

### 主要功能

1. **UI 布局**：
   - 使用 `CupertinoPageScaffold` 提供 iOS 风格的页面框架
   - 导航栏中间显示 "AdaptyUI" 标题
   - 右侧添加按钮用于添加新的支付墙

2. **添加支付墙功能**：
   - 点击添加按钮弹出对话框
   - 使用 `CupertinoAlertDialog` 显示输入表单
   - `CupertinoTextField` 用于输入 Placement Id
   - 点击确定后将新 ID 添加到共享状态

3. **子组件集成**：
   - 页面主体显示 `PaywallsList` 组件
   - 传递错误处理回调给子组件

### 用户交互流程

1. 用户点击导航栏右侧的 "+" 按钮
2. 弹出输入对话框，提示用户输入 Placement Id
3. 用户输入 ID 并点击确定
4. 新 ID 被添加到共享状态的列表开头
5. 同时持久化到本地存储
6. 触发状态更新回调通知相关组件

## 设计模式分析

### 观察者模式

通过 `onNeedsUpdateState` 回调函数实现观察者模式，当共享状态发生变化时通知所有监听者。

### 单例模式

`PaywallsViewSharedState` 使用单例模式确保应用中只有一个状态管理实例，避免数据不一致。

### 状态提升

支付墙 ID 的状态被提升到组件外部的共享状态管理器中，实现跨组件的状态共享。

## 依赖关系

- `shared_preferences`: 用于本地数据持久化
- `cupertino.dart`: 提供 iOS 风格的 UI 组件
- `paywalls_list_screen.dart`: 支付墙列表显示组件

## 使用场景

这个组件主要用于 Adapty SDK 的 Flutter 示例应用中，用于演示如何：

1. 管理多个支付墙的配置
2. 动态添加新的支付墙 ID
3. 在应用重启后恢复之前保存的配置
4. 提供统一的错误处理机制

通过这个设计，用户可以方便地在示例应用中测试不同的支付墙配置，而不需要每次重启应用都重新配置。
