# 剥离简报（QuizSyncProtocol / versions/v1-input 四份文档共用）

## 0. 背景

QuizSyncAI 仓库（现有 Flutter 1.1.0 实现）的 `docs/` 下四份文档，是从**现有实现反推出来的现状规范**。
现在要把它们复制进新仓库 `D:\ZCode\QuizSyncProtocol\versions\v1-input\`，作为**语言中立的规范输入材料**，
未来会重写成 `versions/v1-snapshot.md`。

新仓库只放「纸面与数据」；现有实现是 Flutter/Dart，下一代是 Kotlin/Compose（安卓）与 C#/WinUI（Windows）+ C# 服务端。
所以成品里**不能出现任何一方的技术栈词汇**，但要**逐字保住全部语义**。

## 1. 铁律（违反即不合格）

1. **语义零改动**：不新增需求、不删条款、不改任何数字（上限 / 超时 / 限流 / 时长 / 阈值 / 百分比 / 内存与体积指标）、
   字段名、JSON 键名、表名、列名、路径、状态码、错误码、默认值、单位、配色值、UI 文案原文；
   不得把 MUST 降级成 SHOULD，不得把「必须」改成「建议」。
2. **最小编辑**：以原文为底稿，**逐行**过一遍，只替换命中第 2 节清单的片段。其余文字（含标点、括号里的
   `M46 第 1 条` / `用户需求 4` / `用户反馈 M15 第 3 条` 这类编号、表格结构、章节顺序与标题、
   JSON/SQL/prompt 全文代码块）**逐字保留**。
3. **长段落不许缩写、不许合并、不许改语气**。改写绑定片段时，把「怎么做（类 / 方法 / 库 / 文件 / 框架机制）」
   换成「必须表现出什么行为」，**信息量不许减少**（例如原来写「用 X 接口设置 Y」，改写后必须仍能看出
   「必须设置 Y，且不能用 Z 做法」）。
4. **拿不准 → 原样保留**，并在完成报告里列入「未处理清单」，不要自作聪明。
5. 不改写产品名与业务术语：`AI 双端搜题`、`QuizSync AI`、`QuizSyncAI`、`com.quizsync.android`、
   `session` / `question` / `collection` / `task` 等一律保留。

## 2. 必须剥离 → 改为行为 / 能力描述

### A. 语言、框架、框架专属机制名
`Dart`、`Flutter`、`Kotlin`、`Compose`、`WinUI`、`XAML`、`WPF`，以及框架专有名词：
`Widget`、`isolate`、`MethodChannel`、平台通道、`Provider` / `ChangeNotifierProvider` / `provider.select` /
`FutureProvider` / `StreamProvider`、`GlobalKey`、`TextPainter`、`PictureRecorder` / `Picture.toImage`、
`PipelineOwner` / `RenderView` / `RenderRepaintBoundary` / `RenderPositionedBox`、`OverflowBox`、
`Transform.scale`、`SizedBox`、`MediaQuery.padding.top`、`AnnotatedRegion`、`AppBarTheme.systemOverlayStyle`、
`Stack` 孩子顺序、`Positioned.fill`、`Opacity` / `withValues(alpha:)`、`IconButton`、`Tooltip`、
`MenuItem.checkbox` / `MF_CHECKED`、`Expanded` / `softWrap`、`showLicensePage`、`LaunchTheme` / `NormalTheme`、
`PlatformPlugin`、`enableEdgeToEdge()`、`WindowInsetsControllerCompat` / `isAppearanceLightStatusBars` /
`isAppearanceLightNavigationBars`、`window.setStatusBarColor`、`setSystemUiVisibility`、
`_latestStyle` 缓存、`TextPainter.dispose()` / `ui.Paragraph`、`Picture.toImage`、`jsonDecode`。

改法示例：
- 「`ThemedSystemUi` 在顶部安全区（`MediaQuery.padding.top`）画一条条带」→「应用自己在顶部安全区（系统状态栏所占高度）画一条条带」
- 「根节点给下来的紧约束会把 `Transform.scale` 里的 `SizedBox(虚拟尺寸)` 夹回窗口尺寸」→「界面根节点拿到的紧约束会把虚拟画布夹回窗口尺寸」

### B. 第三方库 / 包名 → 能力描述
`shelf` / `shelf_router` / `shelf_web_socket`（→ HTTP 服务 + 路由 + WebSocket 能力）、
`drift` / `sqlite3_flutter_libs`（→ 数据库层 / 数据库迁移钩子）、`dio` / `web_socket_channel`
（→ HTTP 客户端 / WebSocket 客户端）、`flutter_riverpod` / `riverpod`（→ 状态管理 / 状态订阅）、
`qr_flutter` / `mobile_scanner`（→ 二维码生成 / 摄像头扫码）、
`window_manager` / `tray_manager` / `hotkey_manager` / `win32`（→ 窗口控制库 / 托盘库 / 全局热键库）、
`flutter_secure_storage`（→ 系统安全存储）、`flutter_math_fork`（→ 公式排版库）、
`KaTeX`（→ 公式渲染所用字形集合）。
注意：出现库名的地方，改写成能力描述后**原来的能力/限制语义必须保留**。例：
「`window_manager` 0.4.3 把「应用要深色」与「系统当前是深色」做了 AND」→
「窗口控制库的「设置窗口明暗」接口把「应用要深色」与「系统当前是深色」做了 AND」。

### C. 仓库内路径 / 产物路径 / 行号引用
`packages/...`、`apps/...`、`server/lib/...`、任何 `*.dart` 文件、`assets/...`、`icon/...`、
`tool/make_icons.dart`、`packages/quizsync_core/test/...`、`apps/desktop/test/...`、
`dist\...zip` / `dist\...exe`（安装包与便携版产物路径**保留原样**，它们是发布约定）、
`windows/runner/resources/...`、脚本名（`build_all.ps1` / `package_windows.ps1` / `installer.iss` 等）。
→ 一律换成角色词或行为描述：「客户端」「服务端」「共享核心」「应用图标源图」「托盘图标源图」
「图标生成脚本」「构建 / 打包脚本」「数据库迁移测试」「回环（loopback）集成测试」。
行号引用（`xxx.dart:214-216`）只留行为描述。

跨文档互引例外：`SPEC.md`、`protocol.md`、`data-model.md`、`ai-contract.md` 四个**同目录同名**文件的互引**保留**；
指向原仓库特有文件的引用（`DECISIONS.md`、`server/README.md`、`milestones.md`、`progress.md`、
`docs/manual-test-android.md`、`docs/optimization-plan.md`）改成「原仓库的决策记录 / 原仓库的服务端说明 / 原仓库的人工测试清单」这类描述。

### D. 应用内部标识符（类名 / 方法名 / 常量名 / Dart 驼峰字段名）→ 行为描述
例：`AnalysisWorkflow.retrySession`、`CaptureCoordinator`、`CoreRepository.mirrorCollections`、
`pullFromPeer`、`SyncEngine`、`applyRemoteOp`、`applyRemoteCollectionDeletes`、`MathText`、`QuestionCard`、
`HighlightColors` / `HighlightResult.needsReview` / `computeHighlight`、`Question.displayTitle`、
`Question.hasStructuredAnswer`、`AppSettings.multiPageLimit`、`kHardMaxPagesPerTask`、
`kAndroidLocalImageLimit`、`kDefaultBallEnabled`、`kUiScalePresets`、`kBaseMinimumWindowSize`、
`kBallStrokeInset`、`kRecognitionSlowNote`、`kAppVersion`、`kAppName`、`_readI32`、
`applyStoredUiScaleToWindow`、`normalizeForDialog`、`overlayHiders`、`hideWindowWhile`、`findAppWindow`、
`gatedTrigger`、`serverPillLabel`、`FloatFrame.hitRegions`、`FloatNode.painter`、`layoutFloatWindow` /
`layoutQuestionContent` / `renderFloatWindow` / `floatParagraph`、`contentSignature`、`intoBody`、
`rawRgba` / `rawStraightRgba` / `ImageByteFormat`、`scheme.surface`、`subtleFill`、`AppRadius.control`、
`MigrationStrategy` / `onUpgrade`、`IntColumn get xxx` / `copyWith` / `Companion` / `watchDevices()`、
`AnalysisWorkflow`、`WindowsSecureStore`、`BallGestureTracker`、`CaptureService.stop` / `releaseCapture()` /
`availability()`、`kHardMaxPagesPerTask`。
改法：换成「该上限 / 该默认值 / 该文案 / 该行为」的中文描述，**数值一个都不能丢**。
例：`kHardMaxPagesPerTask = 6` → 「页数硬上限 = 6」；`kAndroidLocalImageLimit = 20` → 「安卓端固定只保留最近 20 张」；
`kRecognitionSlowNote` → 「该提示文案」并把文案原文保留。
`schemaVersion` 保留（数据库版本术语）。

### E. 安装器 / 打包工具 DSL
`Inno Setup`、`ISCC.exe`、`DefaultDirName={autopf}\QuizSyncAI`、`UsePreviousAppDir=no`、`AppId`、
`Runner.rc`、`#pragma code_page(65001)`、`--no-tree-shake-icons`、`--split-per-abi`、`flutter build ...`
→ 工具 DSL 与命令行换成行为描述（安装目录固定为英文 `QuizSyncAI`、必须显式关闭「沿用上次安装目录」、
构建必须关闭图标字体裁剪…）；**要求、目录名、版本号、产物文件名一律保留**。

### F. 原仓库的历史 bug 叙述
原文里大量「原来怎么错、现在怎么改」的叙述**整体保留**，但把其中的类名 / API 调用 / 库名 / 文件路径
按上面规则改写成行为描述。历史 bug 的**因果与结论不许删**（它们是验收口径）。

## 3. 允许 / 必须保留（不是实现绑定）

- 端口与端口范围（8765、8766–8770）、`/api/v1/...` 与 `/ws` 路径、HTTP 头名（`X-QS-*`、`Authorization`）、
  HTTP 状态码、错误码取值、JSON 字段名、SQLite 表名与列名、协议版本号、`protocol_version: 1`。
- 应用名 `AI 双端搜题`、项目名 `QuizSync AI` / `QuizSyncAI`、包名 `com.quizsync.android`、
  tag / 版本号 `1.1.0`、`F8` / `F9` / `Ctrl+V` / `Win+Shift+S` 等键名与快捷键。
- **平台 API 与平台事实**（两条产品线都跑在同一平台上，且承载验收口径，故保留）：
  Win32 函数与常量（`SetWindowPos`、`SWP_NOACTIVATE` / `SWP_SHOWWINDOW`、`ShowWindow(SW_SHOW)`、
  `SetForegroundWindow`、`UpdateLayeredWindow`、`WS_EX_LAYERED` / `TOPMOST` / `TOOLWINDOW` / `NOACTIVATE` /
  `WS_POPUP`、`DWMWA_*` 与属性编号、`RegisterHotKey` / `UnregisterHotKey`、`WM_TIMER` / `WM_HOTKEY` /
  `WM_LBUTTONUP`、`SetTimer` / `KillTimer`、`PeekMessageW`、`FindWindow`、`GlobalLock` / `GlobalSize`、
  `CF_DIB` / `CF_DIBV5` / `BI_RGB` / `BI_BITFIELDS` / `biHeight`、`CryptProtectData` / DPAPI、
  `PrintWindow`、`CreateDIBSection`、`GetSaveFileNameW` / `FNERR_INVALIDFILENAME`、`DwmSetWindowAttribute`）；
  Android 系统 API 与清单键（`MediaProjection` / `VirtualDisplay` / `ImageReader` /
  `MediaProjection.Callback.onStop`、`AccessibilityService.takeScreenshot()`、`SYSTEM_ALERT_WINDOW`、
  `foregroundServiceType="mediaProjection"`、`REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`、`FLAG_SECURE`、
  `allowBackup` / `dataExtractionRules`、`network_security_config.xml` / `cleartextTrafficPermitted`、
  `EncryptedSharedPreferences` / Keystore、`API 30` / `API 35` / `Android 14` / `Android 15`）；
  SQLite（`FTS5` / `LIKE` / `fts5`）。
- 字体与许可（`MiSans`、微软雅黑 / Segoe UI、`Material Icons` 的 Apache-2.0）、
  Windows 数据目录约定（`<exe 所在目录>\userdata`、不可叫 `data` 的理由、`%LOCALAPPDATA%`）、
  第三方 AI 服务与模型名（`DeepSeek`、`deepseek-flash`、`https://api.deepseek.com`、`gpt-4o-mini`、
  `qwen-vl-max`、`glm-4v-plus`、`claude-sonnet`、`gemini-2.x`）及其 HTTP 端点与字段名
  （`chat/completions`、`image_url`、`inline_data`、`v1/messages`、`v1beta/models/...:generateContent`）。
- 全部 UI 文案原文、配色值（`#16A34A` 等）、尺寸与阈值、`M*` / `用户需求 N` / `用户反馈 N` 编号（溯源标记）。

## 4. 硬门禁：这些词在成品里出现次数必须是 0（不区分大小写）

`shelf`、`drift`、`quizsync_core`、`quizsync_ui`、`quizsync_android`、`pubspec`、`.dart`、
`riverpod`、`dio`、`web_socket_channel`、`flutter`、`kotlin`、`packages/`、`apps/`、`server/lib`

注意：
- `com.quizsync.android`（带点）**不算** `quizsync_android`，它是包名，必须保留。
- `QuizSyncAI` / `QuizSync AI` 不含 `quizsync_core` / `quizsync_ui` / `quizsync_android`，可保留。
- `data-model.md` / `protocol.md` / `SPEC.md` / `ai-contract.md` 的互引里不含禁用词，可保留。
- 唯一允许出现的例外是**统一前言第 2 行**（来源行，任务书指定逐字保留），它含有技术栈名；
  正文里一个都不许有。

**写完自己跑一遍**（在 `D:\ZCode\QuizSyncProtocol\versions\v1-input\` 下）：
```
rg -in "shelf|drift|quizsync_core|quizsync_ui|quizsync_android|pubspec|\.dart\b|riverpod|dio\b|web_socket_channel|flutter|kotlin|packages/|apps/|server/lib" <你的文件>
```
命中的**行号必须全部落在第 2 行（前言来源行）**。有任何正文命中就继续改，改到 0。

## 5. 开头统一前言（每份文件都要，逐字照抄，含开头 `<!--` 与本行之前的换行）

```
<!--
来源：QuizSyncAI 仓库（Flutter 1.1.0，tag v1.1.0-flutter）docs/<原文件名>
本文件是「按实现反推的现状规范」的输入材料：已剥离实现绑定（技术栈/包名/文件路径），
语义未改动。规范正文见 versions/v1-snapshot.md（Phase 1 交付）。
-->
```
`<原文件名>` 替换成实际文件名（如 `SPEC.md`）。前言之后空一行，再接原文第一行（`# 标题`）。

## 6. 输出要求

- 用 `write` 工具写整份文件（父目录会自动创建）：UTF-8、LF、Markdown、结尾保留一个换行。
- **写整份完整文件**，不是补丁、不是摘要、不是 diff。
- 完成后报告：① 文件路径 + 行数 + 字节数；② 改动处数（按「语言框架 / 第三方库 / 路径文件 / 内部标识符 /
  安装器 DSL / 其他」分类计数）；③ 禁用词自检结果（命中行号）；④ 「未处理，待 Phase 1 确认」清单
  （原样保留但拿不准的项）。
