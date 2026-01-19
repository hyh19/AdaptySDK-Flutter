# main.dart 代码详解

## 概述

`main.dart` 是 Adapty Flutter SDK 示例应用的入口文件。这个文件定义了应用的根组件 `MyApp`，实现了一个带有底部导航栏的 iOS 风格应用，包含三个主要功能模块：通用功能、付费墙展示和引导流程展示。

## 代码结构分析

### 导入语句

```dart 1:7:example/lib/main.dart
import 'package:adapty_flutter_example/purchase_observer.dart';
import 'package:flutter/cupertino.dart';
import 'package:toastification/toastification.dart';

import 'screens/main_screen.dart';
import 'screens/onboardings_view.dart';
import 'screens/paywalls_view.dart';
```

- `purchase_observer.dart`: 导入购买观察器，用于处理应用内购买相关逻辑
- `cupertino.dart`: Flutter 的 iOS 风格组件库
- `toastification.dart`: 用于显示 Toast 通知的第三方库
- 三个屏幕组件：主屏幕、付费墙视图和引导视图

### 应用入口

```dart 9:11:example/lib/main.dart
void main() {
  runApp(MyApp());
}
```

这是 Flutter 应用的入口函数，创建并运行 `MyApp` 组件。

### MyApp 组件

```dart 13:16:example/lib/main.dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}
```

`MyApp` 是一个有状态组件，因为应用需要在初始化时执行一些操作（如设置购买观察器）。

### 状态管理类

```dart 18:24:example/lib/main.dart
class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    PurchasesObserver().initialize();

    super.initState();
  }
```

在组件初始化时，调用 `PurchasesObserver().initialize()` 来设置购买观察功能。这确保了应用启动时就能监听购买相关事件。

### 错误对话框方法

```dart 26:48:example/lib/main.dart
  Future<void> _showErrorDialog(BuildContext context, String title, String message, String? details) {
    return showCupertinoDialog(
      context: context,
      builder: (ctx) => CupertinoAlertDialog(
        title: Text(title),
        content: Column(
          children: [
            Text(message),
            if (details != null) Text(details),
          ],
        ),
        actions: [
          CupertinoButton(
              child: const Text('OK'),
              onPressed: () {
                // close dialog
                Navigator.pop(ctx);
                // Navigator.of(context).pop();
              }),
        ],
      ),
    );
  }
```

这是一个通用的错误对话框显示方法：

- 使用 iOS 风格的 `CupertinoAlertDialog`
- 支持可选的详细信息显示
- 提供 OK 按钮关闭对话框

### 应用界面构建

```dart 50:118:example/lib/main.dart
  Widget build(BuildContext context) {
    return ToastificationWrapper(
      config: ToastificationConfig(
        maxTitleLines: 2,
        maxDescriptionLines: 6,
        marginBuilder: (context, alignment) => const EdgeInsets.fromLTRB(0, 16, 0, 110),
      ),
      child: CupertinoApp(
        theme: CupertinoThemeData(
          brightness: Brightness.light,
          scaffoldBackgroundColor: CupertinoColors.systemGroupedBackground,
        ),
        home: CupertinoTabScaffold(
          tabBar: CupertinoTabBar(
            items: [
              BottomNavigationBarItem(
                icon: Icon(CupertinoIcons.home),
                label: 'General',
              ),
              BottomNavigationBarItem(
                icon: Icon(CupertinoIcons.money_dollar_circle_fill),
                label: 'Paywalls',
              ),
              BottomNavigationBarItem(
                icon: Icon(CupertinoIcons.doc_on_clipboard),
                label: 'Onboardings',
              ),
            ],
          ),
          tabBuilder: (context, index) {
            switch (index) {
              case 0:
                return CupertinoTabView(
                  builder: (context) {
                    return CupertinoPageScaffold(
                      child: MainScreen(adaptyErrorCallback: (e) => _showErrorDialog(context, 'Error code ${e.code}!', e.message, e.detail), customErrorCallback: (e) => _showErrorDialog(context, 'Unknown error!', e.toString(), null)),
                    );
                  },
                );
              case 1:
                return CupertinoTabView(
                  builder: (context) {
                    return CupertinoPageScaffold(
                      child: PaywallsView(
                        adaptyErrorCallback: (e) => _showErrorDialog(context, 'Error code ${e.code}!', e.message, e.detail),
                        customErrorCallback: (e) => _showErrorDialog(context, 'Unknown error!', e.toString(), null),
                      ),
                    );
                  },
                );
              case 2:
                return CupertinoTabView(
                  builder: (context) {
                    return CupertinoPageScaffold(
                      child: OnboardingsView(
                        adaptyErrorCallback: (e) => _showErrorDialog(context, 'Error code ${e.code}!', e.message, e.detail),
                        customErrorCallback: (e) => _showErrorDialog(context, 'Unknown error!', e.toString(), null),
                      ),
                    );
                  },
                );
              default:
                return const SizedBox.shrink();
            }
          },
        ),
      ),
    );
  }
```

## 界面构建逻辑

### ToastificationWrapper

应用被 `ToastificationWrapper` 包装，提供 Toast 通知功能：

- 配置了最大标题行数和描述行数
- 设置了边距，避免与底部导航栏重叠

### CupertinoApp 主题

```dart 57:61:example/lib/main.dart
        theme: CupertinoThemeData(
          brightness: Brightness.light,
          scaffoldBackgroundColor: CupertinoColors.systemGroupedBackground,
        ),
```

设置 iOS 风格主题：

- 明亮模式
- 使用系统分组背景色

### 底部导航栏

```dart 62:78:example/lib/main.dart
        home: CupertinoTabScaffold(
          tabBar: CupertinoTabBar(
            items: [
              BottomNavigationBarItem(
                icon: Icon(CupertinoIcons.home),
                label: 'General',
              ),
              BottomNavigationBarItem(
                icon: Icon(CupertinoIcons.money_dollar_circle_fill),
                label: 'Paywalls',
              ),
              BottomNavigationBarItem(
                icon: Icon(CupertinoIcons.doc_on_clipboard),
                label: 'Onboardings',
              ),
            ],
          ),
```

创建带有三个标签的底部导航栏：

1. **General** (通用) - 主屏幕，显示一般功能
2. **Paywalls** (付费墙) - 展示付费墙界面
3. **Onboardings** (引导) - 显示引导流程

### 标签页面构建器

```dart 79:114:example/lib/main.dart
          tabBuilder: (context, index) {
            switch (index) {
              case 0:
                return CupertinoTabView(
                  builder: (context) {
                    return CupertinoPageScaffold(
                      child: MainScreen(adaptyErrorCallback: (e) => _showErrorDialog(context, 'Error code ${e.code}!', e.message, e.detail), customErrorCallback: (e) => _showErrorDialog(context, 'Unknown error!', e.toString(), null)),
                    );
                  },
                );
              case 1:
                return CupertinoTabView(
                  builder: (context) {
                    return CupertinoPageScaffold(
                      child: PaywallsView(
                        adaptyErrorCallback: (e) => _showErrorDialog(context, 'Error code ${e.code}!', e.message, e.detail),
                        customErrorCallback: (e) => _showErrorDialog(context, 'Unknown error!', e.toString(), null),
                      ),
                    );
                  },
                );
              case 2:
                return CupertinoTabView(
                  builder: (context) {
                    return CupertinoPageScaffold(
                      child: OnboardingsView(
                        adaptyErrorCallback: (e) => _showErrorDialog(context, 'Error code ${e.code}!', e.message, e.detail),
                        customErrorCallback: (e) => _showErrorDialog(context, 'Unknown error!', e.toString(), null),
                      ),
                    );
                  },
                );
              default:
                return const SizedBox.shrink();
            }
          },
```

每个标签都使用 `CupertinoTabView` 和 `CupertinoPageScaffold` 来构建页面，并传入错误回调函数。

## 错误处理机制

所有子组件都接收两个错误回调：

- `adaptyErrorCallback`: 处理 Adapty SDK 特定的错误
- `customErrorCallback`: 处理其他未知错误

这确保了统一的错误显示体验。

## 设计模式和架构特点

1. **状态管理**: 使用 Flutter 的有状态组件来管理应用初始化
2. **错误处理**: 统一的错误对话框处理机制
3. **组件化**: 将不同功能模块分离到独立的屏幕组件中
4. **iOS 风格**: 完全采用 Cupertino 组件库，保持原生 iOS 体验
5. **回调模式**: 通过回调函数实现错误处理的解耦

## 总结

这个文件是 Adapty Flutter SDK 示例应用的核心入口文件，展示了如何构建一个结构化的 Flutter 应用，包括购买功能初始化、错误处理、以及模块化的界面组织。代码采用了典型的 Flutter 架构模式，注重用户体验和错误处理。
