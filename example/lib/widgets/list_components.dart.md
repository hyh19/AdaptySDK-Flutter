# list_components.dart 组件说明

## 概述

`list_components.dart` 文件定义了一系列用于构建 iOS 风格列表界面的 Flutter 组件。这些组件主要使用 Cupertino 设计语言，提供了一致的列表项展示和交互体验。该文件包含 6 个主要组件类，均为无状态组件（StatelessWidget）。

## 组件结构

### ListSection 组件

```dart 7:32:example/lib/widgets/list_components.dart
class ListSection extends StatelessWidget {
  final String? headerText;
  final String? footerText;
  final List<Widget> children;

  const ListSection({Key? key, this.headerText, this.footerText, this.children = const <Widget>[]}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return CupertinoFormSection.insetGrouped(
      header: this.headerText != null
          ? Padding(
              padding: const EdgeInsets.symmetric(horizontal: 20.0),
              child: Text(headerText!.toUpperCase()),
            )
          : null,
      footer: this.footerText != null
          ? Padding(
              padding: const EdgeInsets.symmetric(horizontal: 20.0),
              child: Text(footerText!),
            )
          : null,
      children: children,
    );
  }
}
```

`ListSection` 是基础的列表分组组件，用于包装其他列表项：

- **headerText**: 分组标题，会自动转换为大写字母显示
- **footerText**: 分组底部说明文字
- **children**: 子组件列表，通常包含各种 ListTile 组件

该组件使用 `CupertinoFormSection.insetGrouped` 提供分组样式，带有圆角和内边距效果。

### ListProductTile 组件

```dart 34:125:example/lib/widgets/list_components.dart
class ListProductTile extends StatelessWidget {
  final AdaptyPaywallProduct product;
  final void Function()? onTap;
  const ListProductTile({Key? key, required this.product, this.onTap}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final theme = CupertinoTheme.of(context).textTheme;
    return GestureDetector(
      onTap: onTap,
      child: Padding(
        padding: const EdgeInsets.all(8.0),
        child: Container(
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(10),
            color: Colors.grey.shade100,
          ),
          child: CupertinoFormRow(
            prefix: Padding(
              padding: const EdgeInsets.fromLTRB(0, 8, 0, 8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product.localizedTitle,
                    style: theme.actionTextStyle,
                  ),
                  Row(
                    children: [
                      Text(
                        "Offer:",
                        style: theme.textStyle.copyWith(color: CupertinoColors.systemGrey2),
                      ),
                      const SizedBox(width: 10),
                      if (product.subscription?.offer != null)
                        Text(
                          product.subscription!.offer!.phases.map((e) => e.paymentMode).join(', '),
                          style: theme.textStyle.copyWith(color: CupertinoColors.systemGrey2),
                        ),
                      if (product.subscription?.offer == null)
                        Text(
                          'No offer',
                          style: theme.textStyle.copyWith(color: CupertinoColors.systemGrey2),
                        ),
                    ],
                  ),
                  FutureBuilder(
                    future: PurchasesObserver().callCreateWebPaywallUrl(product),
                    builder: (context, snapshot) {
                      return Row(
                        children: [
                          Text(
                            'Web URL:',
                            style: theme.textStyle.copyWith(color: CupertinoColors.systemGrey2),
                          ),
                          const SizedBox(width: 10),
                          Text(
                            snapshot.data ?? 'null',
                            style: theme.textStyle.copyWith(color: CupertinoColors.systemGrey2),
                          ),
                        ],
                      );
                    },
                  ),
                  Column(
                    children: [
                      CupertinoButton(
                        onPressed: () {
                          PurchasesObserver().callOpenWebPaywall(product: product);
                        },
                        child: const Text('Open Web Paywall (with product)'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
            // helper: Text(title),
            child: Padding(
              padding: const EdgeInsets.fromLTRB(0, 8, 8, 8),
              child: Text(
                product.price.localizedString ?? 'null',
                textAlign: TextAlign.right,
                style: theme.textStyle.copyWith(color: CupertinoColors.systemGrey2),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

`ListProductTile` 是专门用于展示 Adapty 付费墙产品的复杂组件：

- **product**: AdaptyPaywallProduct 对象，包含产品信息
- **onTap**: 点击回调函数

该组件显示：

- 产品本地化标题
- 订阅优惠信息（payment modes）
- Web 支付墙 URL（通过 FutureBuilder 异步获取）
- 打开 Web 支付墙的按钮
- 产品价格（右对齐显示）

组件具有灰色背景和圆角装饰，集成了支付相关的业务逻辑。

### ListTextTile 组件

```dart 127:153:example/lib/widgets/list_components.dart
class ListTextTile extends StatelessWidget {
  final String title;
  final String? subtitle;
  final Color? subtitleColor;

  const ListTextTile({Key? key, required this.title, this.subtitle, this.subtitleColor}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final theme = CupertinoTheme.of(context).textTheme;
    return CupertinoFormRow(
      prefix: Padding(
        padding: const EdgeInsets.fromLTRB(0, 8, 0, 8),
        child: Text(title),
      ),
      // helper: Text(title),
      child: Padding(
        padding: const EdgeInsets.fromLTRB(0, 8, 8, 8),
        child: Text(
          this.subtitle ?? '',
          textAlign: TextAlign.right,
          style: theme.textStyle.copyWith(color: subtitleColor ?? CupertinoColors.systemGrey2),
        ),
      ),
    );
  }
}
```

`ListTextTile` 是简单的文本展示组件：

- **title**: 主要标题文本
- **subtitle**: 副标题文本（可选）
- **subtitleColor**: 副标题颜色（可选，默认为灰色）

适用于显示键值对信息，副标题右对齐显示。

### ListActionTile 组件

```dart 155:203:example/lib/widgets/list_components.dart
class ListActionTile extends StatelessWidget {
  final titleLengthLimit = 30;

  final String title;
  final Color? titleColor;

  final String? subtitle;

  final bool showProgress;

  final bool isActive;
  final void Function() onTap;

  const ListActionTile({
    Key? key,
    required this.title,
    this.titleColor,
    this.subtitle,
    this.showProgress = false,
    this.isActive = true,
    required this.onTap,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final theme = CupertinoTheme.of(context).textTheme;

    return CupertinoButton(
      onPressed: isActive ? onTap : null,
      padding: const EdgeInsets.fromLTRB(20, 12, 20, 12),
      child: Row(
        children: [
          Expanded(
            child: Text(
              title,
              style: titleColor != null ? theme.actionTextStyle.copyWith(color: titleColor) : null,
            ),
          ),
          if (subtitle != null)
            Text(
              subtitle!,
              style: theme.textStyle.copyWith(color: CupertinoColors.systemGrey2),
            ),
          if (showProgress) const CupertinoActivityIndicator(),
        ],
      ),
    );
  }
}
```

`ListActionTile` 是可点击的操作按钮组件：

- **title**: 按钮标题
- **titleColor**: 标题颜色（可选）
- **subtitle**: 副标题（可选）
- **showProgress**: 是否显示加载指示器
- **isActive**: 是否激活状态
- **onTap**: 点击回调函数

支持进度显示和激活状态控制，当 `isActive` 为 false 时按钮不可点击。

### ListTextFieldTile 组件

```dart 205:234:example/lib/widgets/list_components.dart
class ListTextFieldTile extends StatelessWidget {
  final String? placeholder;
  final Color? textColor;
  final Color? placeholderColor;

  final void Function(String?)? onChanged;
  final void Function(String?)? onSubmitted;

  const ListTextFieldTile({
    Key? key,
    this.placeholder,
    this.textColor,
    this.placeholderColor,
    this.onChanged,
    this.onSubmitted,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return CupertinoTextField(
      placeholder: placeholder,
      placeholderStyle: TextStyle(color: placeholderColor ?? Colors.black26),
      decoration: BoxDecoration(),
      padding: const EdgeInsets.fromLTRB(20, 12, 12, 12),
      clearButtonMode: OverlayVisibilityMode.editing,
      onChanged: onChanged,
      onSubmitted: onSubmitted,
    );
  }
}
```

`ListTextFieldTile` 是文本输入组件：

- **placeholder**: 占位符文本
- **textColor**: 文本颜色（可选）
- **placeholderColor**: 占位符颜色（可选）
- **onChanged**: 文本变化回调
- **onSubmitted**: 提交回调（回车）

使用 `CupertinoTextField` 提供 iOS 风格的输入体验，支持编辑时显示清除按钮。

### ListToggleTile 组件

```dart 236:271:example/lib/widgets/list_components.dart
class ListToggleTile extends StatelessWidget {
  final String title;
  final bool value;
  final void Function(bool) onChanged;

  const ListToggleTile({
    Key? key,
    required this.title,
    required this.value,
    required this.onChanged,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return CupertinoFormRow(
      prefix: Expanded(
        child: Padding(
          padding: const EdgeInsets.fromLTRB(0, 8, 0, 8),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(title),
            ],
          ),
        ),
      ),
      child: Padding(
        padding: const EdgeInsets.fromLTRB(0, 8, 8, 8),
        child: CupertinoSwitch(
          value: value,
          onChanged: onChanged,
        ),
      ),
    );
  }
}
```

`ListToggleTile` 是开关切换组件：

- **title**: 开关标签文本
- **value**: 当前开关状态
- **onChanged**: 状态变化回调

使用 `CupertinoSwitch` 提供原生的 iOS 开关体验。

## 设计特点

### 一致的设计语言

所有组件都遵循 Cupertino 设计规范：

- 使用 `CupertinoFormRow` 和 `CupertinoFormSection` 作为基础
- 统一的内边距和间距设置
- 一致的颜色方案（灰色副文本）

### 响应式布局

组件具有良好的响应式特性：

- 使用 `Expanded` 和 `Column` 进行灵活布局
- 支持不同屏幕尺寸的适配
- 合理的内边距确保在各种设备上的良好显示效果

### 集成业务逻辑

部分组件直接集成了业务逻辑：

- `ListProductTile` 集成了支付墙相关的 API 调用
- 通过 `PurchasesObserver` 单例进行状态管理和 API 交互

### 可访问性和可用性

组件考虑了用户体验：

- 适当的触摸目标大小
- 清晰的视觉层次结构
- 支持禁用状态和加载状态

## 使用示例

这些组件通常在列表页面中使用，如支付墙展示、设置页面等。通过组合使用，可以快速构建功能丰富、样式一致的用户界面。
