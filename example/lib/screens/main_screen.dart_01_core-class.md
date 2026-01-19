# MainScreen 核心类结构分析

## 概述

`MainScreen` 是 Adapty Flutter SDK 示例应用的主屏幕组件，展示了一个完整的用户界面，演示了 Adapty SDK 的各种功能特性。这个类继承自 `StatefulWidget`，包含了复杂的状态管理和多个 UI 交互功能。

## 类定义和构造函数

```dart 12:20:example/lib/screens/main_screen.dart
class MainScreen extends StatefulWidget {
  const MainScreen({super.key, required this.adaptyErrorCallback, required this.customErrorCallback});

  final OnAdaptyErrorCallback adaptyErrorCallback;
  final OnCustomErrorCallback customErrorCallback;

  @override
  _MainScreenState createState() => _MainScreenState();
}
```

`MainScreen` 构造函数接收两个回调函数：

- `adaptyErrorCallback`: 处理 Adapty SDK 相关的错误
- `customErrorCallback`: 处理其他自定义错误

## 状态类结构

### 核心状态变量

```dart 22:34:example/lib/screens/main_screen.dart
class _MainScreenState extends State<MainScreen> {
  final observer = PurchasesObserver();

  bool loading = false;
  String? _enteredCustomerUserId;
  AdaptyProfile? adaptyProfile;

  final String examplePaywallId = 'example_ab_test';
  AdaptyPaywall? examplePaywall;
  List<AdaptyPaywallProduct>? examplePaywallProducts;
  AdaptyInstallationStatus _installationStatus = AdaptyInstallationStatusNotDetermined();

  DemoPaywallFetchPolicy _examplePaywallFetchPolicy = DemoPaywallFetchPolicy.reloadRevalidatingCacheData;
```

#### 状态变量说明

1. **`observer`**: `PurchasesObserver` 实例，负责处理所有 Adapty SDK 的 API 调用和事件监听
2. **`loading`**: 布尔值，表示当前是否有异步操作正在进行
3. **`_enteredCustomerUserId`**: 用户输入的客户用户 ID
4. **`adaptyProfile`**: 当前用户的 Adapty 档案信息
5. **`examplePaywallId`**: 示例付费墙的 ID
6. **`examplePaywall`**: 示例付费墙对象
7. **`examplePaywallProducts`**: 示例付费墙的产品列表
8. **`_installationStatus`**: 应用安装状态信息
9. **`_examplePaywallFetchPolicy`**: 示例付费墙的数据获取策略

### 自定义付费墙相关状态

```dart 426:430:example/lib/screens/main_screen.dart
  DemoPaywallFetchPolicy _customPaywallFetchPolicy = DemoPaywallFetchPolicy.reloadRevalidatingCacheData;
  String? _customPaywallId;
  String? _customPaywallLocale;
  AdaptyPaywall? _customPaywall;
  List<AdaptyPaywallProduct>? _customPaywallProducts;
```

这些变量用于管理用户自定义加载的付费墙功能。

## 初始化逻辑

### initState 方法

```dart 37:46:example/lib/screens/main_screen.dart
  @override
  void initState() {
    super.initState();

    _subscribeForEvents();
    _installationStatus = observer.installationStatus;

    Future.delayed(Duration(seconds: 1), () {
      this._initialize();
    });
  }
```

初始化过程：

1. 调用 `_subscribeForEvents()` 设置事件监听器
2. 获取初始安装状态
3. 延迟 1 秒后调用 `_initialize()` 进行数据初始化

### _initialize 方法

```dart 84:91:example/lib/screens/main_screen.dart
  Future<void> _initialize() async {
    try {
      _reloadProfile();
      _loadExamplePaywall();
    } catch (e) {
      print('#Example# activate error $e');
    }
  }
```

初始化方法执行两个核心操作：

1. 重新加载用户档案
2. 加载示例付费墙

## 事件订阅系统

### _subscribeForEvents 方法

```dart 54:82:example/lib/screens/main_screen.dart
  void _subscribeForEvents() {
    observer.onAdaptyErrorOccurred = (error) {
      switch (error.code) {
        case AdaptyErrorCode.paymentCancelled:
          return;
        default:
          break;
      }

      widget.adaptyErrorCallback(error);
    };

    observer.onUnknownErrorOccurred = (error) {
      widget.customErrorCallback(error);
    };

    observer.onInstallationStatusUpdated = (status) {
      setState(() {
        _installationStatus = status;
      });
    };

    Adapty().didUpdateProfileStream.listen((profile) {
      print('#Example# didUpdateProfileStream $profile');
      setState(() {
        adaptyProfile = profile;
      });
    });
  }
```

#### 事件监听器说明

1. **`onAdaptyErrorOccurred`**: 处理 Adapty SDK 错误，对于支付取消错误特殊处理（不显示给用户）
2. **`onUnknownErrorOccurred`**: 处理未知错误
3. **`onInstallationStatusUpdated`**: 监听安装状态变化并更新 UI
4. **`didUpdateProfileStream`**: 监听用户档案更新，通过 Stream 实时更新

## 辅助方法

### 加载状态管理

```dart 48:52:example/lib/screens/main_screen.dart
  void _setIsLoading(bool value) {
    setState(() {
      loading = value;
    });
  }
```

简单的状态管理方法，用于控制加载指示器的显示。

### 加载遮罩 UI

```dart 93:103:example/lib/screens/main_screen.dart
  Widget _buildLoadingDimmingWidget() {
    return Container(
      decoration: BoxDecoration(color: CupertinoColors.black.withAlpha(200)),
      child: Center(
        child: CupertinoActivityIndicator(
          color: CupertinoColors.white,
          radius: 20.0,
        ),
      ),
    );
  }
```

创建半透明的加载遮罩，显示活动指示器。

## 架构特点

1. **观察者模式**: 使用 `PurchasesObserver` 来封装所有 Adapty SDK 的交互逻辑
2. **状态驱动 UI**: 通过 `setState` 更新状态来驱动界面重新渲染
3. **事件驱动**: 使用回调和 Stream 来处理异步事件和状态变化
4. **错误处理**: 统一的错误处理机制，通过回调传递给父组件
5. **模块化设计**: 将不同功能拆分为独立的方法，便于维护和测试

这个核心类结构为整个示例应用提供了稳定的基础架构，支持复杂的用户交互和数据管理功能。
