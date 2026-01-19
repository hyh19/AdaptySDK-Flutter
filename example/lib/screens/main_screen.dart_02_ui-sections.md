# MainScreen UI 界面构建分析

## 总体布局结构

### 主 build 方法

```dart 139:165:example/lib/screens/main_screen.dart
  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        CupertinoPageScaffold(
          navigationBar: const CupertinoNavigationBar(
            middle: Text('Welcome to Adapty Flutter!'),
          ),
          child: SafeArea(
            child: ListView(
              children: [
                _buildProfileIdSection(),
                _buildInstallationDetailsSection(),
                _buildIdentifySection(),
                _buildProfileInfoSection(),
                _buildExampleABTestSection(),
                _buildCustomPaywallSection(),
                _buildOtherActionsSection(),
                _buildRefundSaverSection(),
                _buildLogoutSection(),
              ],
            ),
          ),
        ),
        if (this.loading) _buildLoadingDimmingWidget(),
      ],
    );
  }
```

界面采用层叠布局（Stack）：

- **底层**: 主内容区域，使用 `CupertinoPageScaffold` 提供 iOS 风格的导航栏和安全区域
- **顶层**: 当 `loading` 为 true 时显示加载遮罩

### 导航栏

```dart 143:145:example/lib/screens/main_screen.dart
          navigationBar: const CupertinoNavigationBar(
            middle: Text('Welcome to Adapty Flutter!'),
          ),
```

使用标准的 iOS 风格导航栏，标题显示欢迎信息。

## 各个功能区块

### 1. 档案 ID 显示区块

```dart 211:225:example/lib/screens/main_screen.dart
  Widget _buildProfileIdSection() {
    return ListSection(
      headerText: 'Adapty Profile Id',
      footerText: '👆🏻 Tap to copy',
      children: [
        ListActionTile(
          title: this.adaptyProfile != null ? '${this.adaptyProfile!.profileId}' : 'null',
          isActive: this.adaptyProfile != null,
          onTap: () {
            Clipboard.setData(ClipboardData(text: this.adaptyProfile!.profileId));
          },
        ),
      ],
    );
  }
```

显示用户的 Adapty Profile ID，支持点击复制到剪贴板功能。

### 2. 安装详情区块

```dart 105:136:example/lib/screens/main_screen.dart
  Widget _buildInstallationDetailsSection() {
    switch (_installationStatus) {
      case AdaptyInstallationStatusNotDetermined():
        return ListSection(
          headerText: 'Installation Details',
          children: [
            ListTextTile(title: 'Status', subtitle: 'notDetermined'),
            ListActionTile(title: 'Update', onTap: () => observer.callGetCurrentInstallationStatus()),
          ],
        );
      case AdaptyInstallationStatusNotAvailable():
        return ListSection(
          headerText: 'Installation Details',
          children: [
            ListTextTile(title: 'Status', subtitle: 'notAvailable'),
            ListActionTile(title: 'Update', onTap: () => observer.callGetCurrentInstallationStatus()),
          ],
        );
      case AdaptyInstallationStatusDetermined(details: final details):
        return ListSection(
          headerText: 'Installation Details',
          children: [
            ListTextTile(title: 'Status', subtitle: 'determined'),
            ListTextTile(title: 'ID', subtitle: details.installId),
            ListTextTile(title: 'Install Time', subtitle: details.installTime.toIso8601String()),
            ListTextTile(title: 'App Launch Count', subtitle: details.appLaunchCount.toString()),
            ListTextTile(title: 'Payload', subtitle: details.payload),
            ListActionTile(title: 'Update', onTap: () => observer.callGetCurrentInstallationStatus()),
          ],
        );
    }
  }
```

根据安装状态显示不同的信息：

- **NotDetermined**: 未确定状态
- **NotAvailable**: 不可用状态  
- **Determined**: 已确定状态，显示详细的安装信息

### 3. 用户身份识别区块

```dart 227:266:example/lib/screens/main_screen.dart
  Widget _buildIdentifySection() {
    return ListSection(
      headerText: 'Customer User Id',
      footerText: null,
      children: [
        if (adaptyProfile?.customerUserId != null) ListTextTile(title: 'Current', subtitle: adaptyProfile!.customerUserId!),
        ListTextFieldTile(
          placeholder: 'Enter Customer User Id',
          onChanged: (txt) => setState(() {
            _enteredCustomerUserId = txt;
          }),
          onSubmitted: (txt) {
            setState(() {
              _enteredCustomerUserId = txt;
              if (_enteredCustomerUserId != null && _enteredCustomerUserId!.isNotEmpty) {
                observer.callIdentifyUser(
                  _enteredCustomerUserId!,
                  iosAppAccountToken: null,
                  androidObfuscatedAccountId: null,
                );
              }
            });
          },
        ),
        ListActionTile(
          title: 'Identify',
          isActive: _enteredCustomerUserId?.isNotEmpty ?? false,
          onTap: () {
            if (_enteredCustomerUserId != null && _enteredCustomerUserId!.isNotEmpty) {
              observer.callIdentifyUser(
                _enteredCustomerUserId!,
                iosAppAccountToken: null,
                androidObfuscatedAccountId: null,
              );
            }
          },
        ),
      ],
    );
  }
```

提供用户身份识别功能：

- 显示当前客户用户 ID（如果已设置）
- 文本输入框用于输入新的用户 ID
- 识别按钮用于提交身份信息

### 4. 档案信息区块

```dart 268:292:example/lib/screens/main_screen.dart
  Widget _buildProfileInfoSection() {
    final premium = adaptyProfile?.accessLevels['premium'];

    if (adaptyProfile?.accessLevels['premium']?.isActive ?? false) {}

    return ListSection(
      headerText: 'Profile',
      children: [
        ListTextTile(title: 'Is Test', subtitle: (adaptyProfile?.isTestUser ?? false) ? 'true' : 'false'),
        ListTextTile(
          title: 'Premium',
          subtitle: (premium?.isActive ?? false) ? 'Active' : 'Inactive',
          subtitleColor: (premium?.isActive ?? false) ? CupertinoColors.systemGreen : CupertinoColors.systemRed,
        ),
        ListTextTile(title: 'Is Lifetime', subtitle: (premium?.isLifetime ?? false) ? 'true' : 'false'),
        if (premium != null) ListTextTile(title: 'Activated At', subtitle: _dateTimeFormattedString(premium.activatedAt)),
        if (premium != null && premium.renewedAt != null) ListTextTile(title: 'Renewed At', subtitle: _dateTimeFormattedString(premium.renewedAt!)),
        if (premium != null && premium.expiresAt != null) ListTextTile(title: 'Expires At', subtitle: _dateTimeFormattedString(premium.expiresAt!)),
        ListTextTile(title: 'Will Renew', subtitle: (premium?.willRenew ?? false) ? 'true' : 'false'),
        ListTextTile(title: 'Subscriptions: ${adaptyProfile?.subscriptions.length ?? 0}'),
        ListTextTile(title: 'NonSubscriptions: ${adaptyProfile?.nonSubscriptions.length ?? 0}'),
        ListActionTile(title: 'Update', onTap: () => _reloadProfile()),
      ],
    );
  }
```

显示用户档案的详细信息：

- 测试用户状态
- Premium 订阅状态和相关信息
- 订阅和非订阅产品的数量
- 更新按钮用于刷新档案信息

### 5. 示例 A/B 测试区块

```dart 383:424:example/lib/screens/main_screen.dart
  Widget _buildExampleABTestSection() {
    final paywall = this.examplePaywall;

    if (paywall == null) {
      return ListSection(
        headerText: 'Example A/B Test',
        children: [
          ListTextTile(
            title: examplePaywallId,
            subtitle: 'Loading...',
            subtitleColor: CupertinoColors.systemBlue,
          ),
        ],
      );
    } else {
      return ListSection(
        headerText: 'Example A/B Test',
        children: [
          ListTextTile(
            title: examplePaywallId,
            subtitle: 'OK',
            subtitleColor: CupertinoColors.systemGreen,
          ),
          _fetchPolicySelector(_examplePaywallFetchPolicy, (value) {
            setState(() {
              this._examplePaywallFetchPolicy = value;
            });
          }),
          ..._paywallContents(
            paywall,
            examplePaywallProducts,
            (p) => _purchaseProduct(p),
            () => observer.callLogShowPaywall(paywall),
          ),
          ListActionTile(
            title: 'Refresh',
            onTap: () => _loadExamplePaywall(),
          ),
        ],
      );
    }
  }
```

演示 A/B 测试功能：

- 显示付费墙加载状态
- 提供获取策略选择器
- 显示付费墙内容和产品列表
- 支持刷新操作

### 6. 自定义付费墙区块

```dart 432:493:example/lib/screens/main_screen.dart
  Widget _buildCustomPaywallSection() {
    return ListSection(
      headerText: 'Custom Paywall',
      footerText: 'Here you can load any paywall by its id and inspect the contents',
      children: [
        _fetchPolicySelector(_customPaywallFetchPolicy, (value) {
          setState(() {
            this._customPaywallFetchPolicy = value;
          });
        }),
        if (_customPaywall == null) ...[
          ListTextTile(title: 'No Paywall Loaded'),
          ListTextFieldTile(
            placeholder: 'Enter Paywall Locale',
            onChanged: (locale) => setState(() {
              this._customPaywallLocale = locale;
            }),
          ),
          ListTextFieldTile(
            placeholder: 'Enter Paywall Id',
            onChanged: (id) => setState(() {
              this._customPaywallId = id;
            }),
            onSubmitted: (id) {
              this._customPaywallId = id;
              _loadCustomPaywall();
            },
          ),
          ListActionTile(
            title: 'Load',
            isActive: _customPaywallId?.isNotEmpty ?? false,
            onTap: () => _loadCustomPaywall(),
          ),
        ],
        if (_customPaywall != null) ...[
          ListTextTile(title: 'Paywall Id', subtitle: _customPaywall!.placement.id),
          ..._paywallContents(
            _customPaywall!,
            _customPaywallProducts,
            (p) => _purchaseProduct(p),
            () => observer.callLogShowPaywall(_customPaywall!),
          ),
          ListActionTile(
            title: 'Reload',
            isActive: _customPaywallId?.isNotEmpty ?? false,
            onTap: () => _loadCustomPaywall(),
          ),
          ListActionTile(
            title: 'Reset',
            isActive: _customPaywallId?.isNotEmpty ?? false,
            onTap: () {
              setState(() {
                _customPaywall = null;
                _customPaywallProducts = null;
                _customPaywallId = null;
              });
            },
          ),
        ],
      ],
    );
  }
```

允许用户加载任意付费墙：

- 获取策略选择器
- 输入框用于设置付费墙 ID 和语言环境
- 加载、重载和重置功能

### 7. 其他操作区块

```dart 495:533:example/lib/screens/main_screen.dart
  Widget _buildOtherActionsSection() {
    return ListSection(
      headerText: 'Other Actions',
      footerText: null,
      children: [
        ListActionTile(
          title: 'Restore Purchases',
          onTap: () async {
            _setIsLoading(true);

            final profile = await observer.callRestorePurchases();
            if (profile != null) {
              this.adaptyProfile = profile;
            }

            _setIsLoading(false);
          },
        ),
        ListActionTile(
          title: 'Update Profile',
          onTap: () => _updateProfile(),
        ),
        ListActionTile(
          title: 'Set Integration Identifier',
          onTap: () => _setIntegrationIdentifier(),
        ),
        ListActionTile(
          title: 'Update Attribution',
          onTap: () => _updateAttribution(),
        ),
        ListActionTile(
          title: 'Present Code Redemption Sheet',
          onTap: () async {
            await observer.callPresentCodeRedemptionSheet();
          },
        ),
      ],
    );
  }
```

提供各种其他操作功能：

- 恢复购买
- 更新用户档案
- 设置集成标识符
- 更新归因数据
- 显示兑换码表单

### 8. 退款保护区块

```dart 167:209:example/lib/screens/main_screen.dart
  Widget _buildRefundSaverSection() {
    return ListSection(
      headerText: 'Refund Saver',
      footerText: null,
      children: [
        ListActionTile(
          title: 'Update Consent: FALSE',
          isActive: true,
          onTap: () {
            observer.callUpdateCollectingRefundDataConsent(false);
          },
        ),
        ListActionTile(
          title: 'Update Consent: TRUE',
          isActive: true,
          onTap: () {
            observer.callUpdateCollectingRefundDataConsent(true);
          },
        ),
        ListActionTile(
          title: 'Update Preference: NO_PREFERENCE',
          isActive: true,
          onTap: () {
            observer.callUpdateRefundPreference(AdaptyRefundPreference.noPreference);
          },
        ),
        ListActionTile(
          title: 'Update Preference: DECLINE',
          isActive: true,
          onTap: () {
            observer.callUpdateRefundPreference(AdaptyRefundPreference.decline);
          },
        ),
        ListActionTile(
          title: 'Update Preference: GRANT',
          isActive: true,
          onTap: () {
            observer.callUpdateRefundPreference(AdaptyRefundPreference.grant);
          },
        ),
      ],
    );
  }
```

提供退款保护相关的设置选项。

### 9. 登出区块

```dart 535:547:example/lib/screens/main_screen.dart
  Widget _buildLogoutSection() {
    return ListSection(
      headerText: null,
      footerText: null,
      children: [
        ListActionTile(
          title: 'Logout',
          titleColor: CupertinoColors.destructiveRed,
          onTap: () => _logout(),
        ),
      ],
    );
  }
```

提供用户登出功能，按钮使用红色强调危险操作。

## 共享 UI 组件

### 付费墙内容构建器

```dart 294:347:example/lib/screens/main_screen.dart
  List<Widget> _paywallContents(
    AdaptyPaywall paywall,
    List<AdaptyPaywallProduct>? products,
    void Function(AdaptyPaywallProduct) onProductTap,
    void Function() onLogShowTap,
  ) {
    return [
      ListTextTile(title: 'Name', subtitle: paywall.name),
      ListTextTile(title: 'Variation', subtitle: paywall.variationId),
      ListTextTile(title: 'Revision', subtitle: '${paywall.placement.revision}'),
      ListTextTile(title: 'Locale', subtitle: '${paywall.remoteConfig?.locale}'),
      if (products == null) ...paywall.productIdentifiers.map((e) => ListTextTile(title: e.vendorProductId)),
      if (products != null)
        ...products.map((p) => ListProductTile(
              product: p,
              onTap: () => onProductTap(p),
            )),
      ListActionTile(
        title: 'Log Show Paywall',
        onTap: () => onLogShowTap(),
      ),
      ListActionTile(
        title: 'Report Transaction',
        onTap: () async {
          await observer.callReportTransaction(
            transactionId: 'test_transaction_id',
            variationId: paywall.variationId,
          );
        },
      ),
      if (paywall.hasViewConfiguration)
        ListActionTile(
          title: 'Present View',
          onTap: () async {
            try {
              final view = await AdaptyUI().createPaywallView(paywall: paywall);
              await view.present();
            } catch (e) {
              print('#Example# createPaywallView error $e');
            }
          },
        ),
      ListActionTile(
        title: 'Open Web Paywall',
        onTap: () async {
          try {
            await observer.callOpenWebPaywall(paywall: paywall);
          } catch (e) {
            print('#Example# createPaywallView error $e');
          }
        },
      ),
    ];
  }
```

通用的付费墙内容展示组件，包含：

- 付费墙基本信息
- 产品列表
- 各种操作按钮（日志记录、交易报告、视图展示等）

### 获取策略选择器

```dart 349:381:example/lib/screens/main_screen.dart
  Widget _fetchPolicySelector(
    DemoPaywallFetchPolicy value,
    void Function(DemoPaywallFetchPolicy) onSelected,
  ) {
    return ListActionTile(
      title: 'Fetch Policy',
      subtitle: value.title(),
      // subtitleColor: CupertinoColors.systemGreen,
      onTap: () => showCupertinoModalPopup(
        context: context,
        builder: (context) {
          return CupertinoActionSheet(
            actions: DemoPaywallFetchPolicy.values
                .map((e) => CupertinoActionSheetAction(
                      child: Text(e.title()),
                      onPressed: () {
                        onSelected.call(e);
                        Navigator.pop(context);
                      },
                    ))
                .toList(),
            cancelButton: CupertinoActionSheetAction(
              child: Text('Cancel'),
              onPressed: () {
                // Perform action
                Navigator.pop(context);
              },
            ),
          );
        },
      ),
    );
  }
```

使用 iOS 风格的动作表单来选择数据获取策略。

## UI 设计特点

1. **模块化设计**: 每个功能区块独立构建，便于维护
2. **响应式状态**: UI 根据数据状态动态变化
3. **iOS 风格**: 使用 Cupertino 组件保持一致的苹果设计语言
4. **交互反馈**: 按钮激活状态、颜色编码等提供清晰的视觉反馈
5. **错误处理**: 通过回调机制向上层传递错误信息

这个界面设计全面展示了 Adapty SDK 的各种功能，并提供了直观的用户交互体验。
