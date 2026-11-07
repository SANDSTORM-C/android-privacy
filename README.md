# 对于仍然坚持使用国产手机者的建议

> 面向仍然使用国内定制 ROM（MIUI / HyperOS / ColorOS / OriginOS 等）的用户
> 不要求 root、不要求刷机。需要Windows系统电脑，或于wifi环境中启用无线调试并输入配对码

---

## 一、能与不能

避免读者产生错误的期望。

**可预防：**

- 预装应用在后台持续采集（定位、通讯录、应用列表、使用时长）
- 输入法把按键内容上传
- 应用安装器在安装过程中做额外动作
- OTA 在未经你同意时把已移除的组件装回来
- 应用厂商通过系统无障碍/通知监听读取其它应用内容

**无法解决：**

- 厂商固件层上报通道。通道不经过应用层权限，该方案无能为力
- 运营商侧数据。如基站定位、通话详情等
- 已获授权的应用上传的内容（微信等平台聊天记录）
- 物理与供应链层植入

并非绝对安全。

---

## 二、准备

| 工具 | 用途 | 获取方式 |
|---|---|---|
| Shizuku | 免 root 调用系统 API，替代 adb 常驻操作 | [GitHub: RikkaApps/Shizuku](https://github.com/RikkaApps/Shizuku) |
| Canta | 图形化冻结/卸载系统应用，走 Shizuku | [GitHub: samolego/Canta](https://github.com/samolego/Canta) |
| InstallerX Revived | 替代厂商安装器 | [GitHub: Peleron/InstallerX-Revived](https://github.com/Peleron/InstallerX-Revived) |
| Dhizuku | 申请 Device Owner，管理设备所有者权限 | [GitHub: Rosan-P/dhizuku](https://github.com/Rosan-P/dhizuku) |
| 更好的输入法 | 替代隐私风险输入法 | 见第五节 |

Shizuku可选择借用ADB shell权限而非系统root

---

## 三、Shizuku

国产 ROM 大量系统 API 锁在 `shell` 权限后。Shizuku 可保留`adb shell` 权限并转交给授权的第三方应用。

### 3.1 启动方式

**无线调试**

1. 设置 → 关于手机 → 连点“版本号”7 次开启开发者选项
2. 开发者选项 → 无线调试 → 打开
3. 进入“无线调试” → “使用配对码配对设备”，记下配对码
4. 打开 Shizuku → “通过无线调试启动”，输入配对码

**电脑 ADB**

```bash
adb pair <IP>:<端口>
adb shell sh /storage/emulated/0/Android/data/moe.shizuku.privileged.api/start.sh
```

顾名思义，需要在电脑上安装ADB

### 3.2 后续维护

- 大部分情况下，设备重启后 Shizuku 需重新启动服务。重复上述步骤即可

---

## 四、Canta

Canta 的图形界面提供了详细的应用标签，若加载失败可开启代理

### 4.1 两种操作的区别

| 操作 | 命令 | 效果 | 可恢复 |
|---|---|---|---|
| **冻结** | `pm disable-user <包名>` | 应用仍在，但不再启动或接收广播 | `pm enable <包名>` |
| **用户级卸载** | `pm uninstall --user 0 <包名>` | 该用户下看起来未安装，数据清空 | `pm install-existing --user 0 <包名>`（仅系统应用） |
| **卸载第三方应用** | 正常卸载 | 就只是正常卸载 | 重新安装 |

冻结适合用于希望保留但目前不希望其运行的组件。用户级卸载适合确定移除但一般办法无法移除的预装应用

**重要：** 对系统预装应用做用户级卸载后APK 仍在系统分区，任何人(包括OTA)执行 `pm install-existing` 即可恢复，并非永久删除

### 4.2 值得考虑

**定位与网络**

- 系统级定位融合服务与辅助定位，像是 GNSS 增强和网络定位代理之类的
- 大量厂商会用独立的定位代理包绕过 Android 的定位开关

**统计分析**

- 各种 `*.analytics`、`*.onetrack`、`*.beacon` 类包
- 使用数据上传似乎一般归在这类

**垃圾广告**

- 广告 SDK、推荐引擎、内容聚合
- 通常命名为 `*AdSolution`、`*recommendation`、`*yellowpage`

**云同步**

- 厂商云服务、剪贴板同步、相册自动备份
- 你真的觉得这安全吗

### 4.3 注意保留

- **Settings / SystemUI / 权限控制中心**
- **证书安装器、VPN 对话框、媒体选择器**
- **紧急广播（CellBroadcast）**：地震预警走这个通道
- **安全中心 / 权限管理**：部分 ROM 的第三方权限管理依赖此组件

### 4.4 tips

1. 确认不影响日常使用或后果可接受再考虑卸载，有得必有失
2. 逐项清理，便于排查

### 4.5 参考清单

一份在 HyperOS / Android 16 上可行的用户级卸载清单，可作为起点。其中多数在任何国内定制 ROM 上皆有对应包，但包名前缀也许不同

注意：

- 这是**一种可能的选择**，不是推荐清单。每一项对应的是用户不用这项功能，而非这项功能有害
- 清单里多数是系统预装包，`pm install-existing` 可恢复
- 少数预装在产品分区的商店也在其列，同样无需重新下载。只有第三方应用的复原方法为重装 APK ，下文另作区分
- 逐个核对后清除，同名包在不同 ROM 版本上承担的功能可能存在差异

**系统与工具**

| 包名 | 说明 |
|---|---|
| `com.android.browser` | 厂商自带浏览器，纯区 |
| `com.android.quicksearchbox` | 全局搜索框 你真的需要这个吗 |
| `com.android.safetycenter.resources` | 安全中心资源包 |
| `com.android.traceur` | 系统跟踪工具 开发调试用，可能被用于审计 |
| `com.bsp.catchlog` | 日志抓取服务 |
| `com.miui.bugreport` | 用户反馈与日志上报 |

**反诈中心提醒您**

将近期讨论推上风口的相关组件

| 包名 | 说明 |
|---|---|
| `com.miui.greenguard` | 未成年人保护与内容管控框架 |
| `com.miui.guardprovider` | **小米「系统安全组件」**，含录屏上传检测，见 4.6 |
| `com.miui.misightservice` | 行为分析服务 |
| `com.xiaomi.security.onetrack` | 安全组件埋点统计 |
| `com.mobiletools.systemhelper` | **联通设备管理代理**，见 4.6 |

**云服务**

| 包名 | 说明 |
|---|---|
| `com.miui.cloudservice` | 小米云服务 |
| `com.miui.micloudsync` | 云同步 |
| `com.xiaomi.otrpbroker` | OTP 短信中转 |
| `com.xiaomi.xmsfkeeper` | 推送框架守护 |
| `com.xiaomi.metoknlp` | 网络定位辅助 |

**广告**

| 包名 | 说明 |
|---|---|
| `com.miui.systemAdSolution` | 系统广告 SDK |
| `com.miui.yellowpage` | 生活黄页与推荐 |
| `com.miui.thirdappassistant` | 第三方应用推荐 |
| `com.xiaomi.aireco` | 内容推荐引擎 |
| `com.miui.personalassistant` | 负一屏与个人助理 |
| `com.miui.hybrid` | 快应用运行时 |

**定位**

| 包名 | 说明 |
|---|---|
| `com.xiaomi.location.fused` | 定位融合服务 |
| `com.xiaomi.gnss.polaris` | GNSS 增强、星历下载 |

**AI 助手**

| 包名 | 说明 |
|---|---|
| `com.xiaomi.aiasst.service` | 语音助手服务 |
| `com.xiaomi.aiasst.vision` | 视觉识别 |
| `com.xiaomi.aicr` | 智能场景识别 |

**诊断与遥测**

| 包名 | 说明 |
|---|---|
| `com.miui.daemon` | 后台守护进程 |
| `com.miui.uireporter` | UI 事件上报 |
| `com.miui.wmsvc` | 系统服务桥 |

**内容与输入**

| 包名 | 说明 |
|---|---|
| `com.miui.contentcatcher` | 内容抓取 |
| `com.miui.contentextension` | 内容扩展服务 |
| `com.miui.phrase` | 常用短语，云联想相关 |
| `com.miui.securityinputmethod` | 安全输入法 |
| `com.sohu.inputmethod.sogou.xiaomi` | 搜狗定制输入法 |

**预装在产品分区的商店**

| 包名 | 说明 |
|---|---|
| `com.miui.themestore` | 主题商店，位于 `/product/data-app/` |
| `com.xiaomi.market` | 小米应用商店，位于 `/product/app/` |

APK 在主分区，执行`pm install-existing` 即可恢复，不必重新下载。是否卸载取决于对应功能的使用与否

如希望于多个用户空间内保持一致（例如主账户与工作资料、安全空间），对每个用户分别执行一次：

```bash
adb shell pm uninstall --user 0 <包名>
adb shell pm uninstall --user <编号> <包名>
```

首先确认目标用户里该包状态为 `installed=true`，不要对已为 `false` 的包重复操作

多用户空间的取舍值得思考。全量同步会让所有用户空间看起来一字不差，但这未必是好事。
电幕前留点东西反而更自然。把商店或输入法保留，是没有成本且明智的选择

---

### 4.6 特殊组件单独说明

值得单独列出来。

这两个包在不同品牌、不同 ROM 版本上的文件名和位置并不统一，下面的包名是小米国行 HyperOS 上的实测结果

### com.mobiletools.systemhelper —— 联通设备管理代理

**用途** 文件位于 `/system/priv-app/SystemHelper`，但 `isMiuiPreinstall=false`——它不属于 MIUI 自己的预装体系，而是运营商通过终端厂商定制链路预置进来的。入口类是 `DMApplication`（Device Management，设备管理），服务的 action 为 `com.mobiletools.systemhelper.service.IRegisterHelperService`。最广为人知的

**用途：**

| 权限 | 对应能力 |
|---|---|
| `QUERY_ALL_PACKAGES` | 枚举全部已安装应用 |
| `PACKAGE_USAGE_STATS` | 各应用使用时长 |
| `READ_PHONE_STATE`、`READ_PRIVILEGED_PHONE_STATE` | IMEI、IMSI、ICCID |
| `ACCESS_FINE_LOCATION`、`ACCESS_BACKGROUND_LOCATION` | 常驻定位 |
| `INTERNET`、`CHANGE_WIFI_STATE`、`ACCESS_NETWORK_STATE` | 上报与网络状态判断 |
| `READ_EXTERNAL_STORAGE` | 外部存储 |

接收一侧同样对上， `SIMStateReceiver`（监听 `SIM_STATE_CHANGED`，换卡触发）、`ConnectStateReceiver`（`CONNECTIVITY_CHANGE`）、`OnBootReceiver`（优先级 999 ）。

**已经公开的行为记录。** 2026 年 10 月至 11 月间，有开发者在其小米设备上提取并拆解了该组件，记录到的行为是：统计过去 24 小时内使用过的应用、枚举已安装应用，把每个应用的使用时长与流量与设备的 IMEI、IMSI、ICCID 关联后上报。上报地址属于运营商的设备管理平台，明文 HTTP。该组件在插入非本运营商的 SIM 卡时同样会注册。

另外，在部分厂商的系统属性中能找到对它的直接引用。例如某款手机上存在 `ro.vivo.unicom.dm.name = com.mobiletools.systemhelper`，属性名的含义就是“指定本机的联通设备管理应用”

**要注意的是**：以上行为来自第三方拆解，属于外部资料而非厂商披露。本文记录的方式是：此设备上确实存在该包，其权限与接收器构成与上述记录一致。读者在自己的设备上判断时，建议以权限和组件构成为准，而不是仅凭包名

### com.miui.guardprovider —— 小米「系统安全组件」

**身份。** 文件在 `/product/app/MIUIGuardProvider`，`flags=[SYSTEM HAS_CODE ALLOW_CLEAR_USER_DATA LARGE_HEAP]`，设备上它显示为系统安全组件

**已经公开的行为记录。** 同一批拆解记录中提到，这个组件包含屏幕录制采集：以 720×1280、24 fps 录制约 3 秒，转为 Base64 后加密上传至云端检测，未命中则删除该片段、延时 2 秒后继续下一段，命中则保留片段并提示

需要区分清楚：**录屏隶属于此组件，与上报不同**。前者采集屏幕内容，后者采集应用清单，二者来自不同的包与服务端

同样地，这是第三方拆解的结果，本文不重复其结论，仅记录两件客观事实：这个包在设备上存在，它是构建产物中明确标记为系统级、且被授予 `READ_PHONE_STATE` 的组件

### 处理方式

两者都可以用同一条命令处理，不需要 root

```bash
adb shell pm uninstall --user 0 com.mobiletools.systemhelper
adb shell pm uninstall --user 0 com.miui.guardprovider
```

`pm uninstall --user 0` 会清空该用户下的数据目录 `ceDataInode` 。如果希望保留数据目录，加 `-k` ，日常使用中保留数据没有意义，通常不建议

验证：

```bash
adb shell dumpsys package com.mobiletools.systemhelper | grep -E "User 0:|installed="
```

应当看到 `installed=false`、`ceDataInode=-1`。

**限制：**

1. APK 仍在系统分区，恢复出厂设置或部分 OTA 即可恢复。安全起见，需要同时处理系统更新（见第八节）
2. 卸载只覆盖执行过命令的用户，设备上的其它用户空间需要分别执行
3. 这不是唯一通道，系统里常有其它具备类似权限的组件，努力缩小泄露面吧
4. 最有实际效果的做法仍然是将隐私应用与日常应用分散在不同设备

---

## 五、输入法

输入法能看到你输入的**所有内容**，包括密码框以外的搜索词、聊天内容、地址、账号，云端联想意味着这些内容离开设备

### 5.1 判断标准

- 是否有联网权限。纯本地输入法完全可以不要 `INTERNET` 权限
- 是否默认开启云联想、云同步、用户词库上传
- 是否有语音输入且默认走云端
- 开源、代码可查

### 5.2 可选方案

| 输入法 | 特点 | 注意 |
|---|---|---|
| Gboard | 功能完整，隐私条款相对明确 | 需在设置里关闭分享使用统计和改善输入法 |
| HeliBoard | 开源，基于 AOSP 键盘延伸 | 中文字库需自行导入，说实话体验一般 |
| Fcitx5 for Android | 开源，Rime 引擎 | 可完全离线，中文体验取决于导入的方案 |
| 同文输入法（Trime） | 开源，Rime 引擎 | 学习成本高，配置得自己写 |
| Simple Keyboard | 极简，无联想无联网 | 仅英文，可作为密码输入时的临时键盘 |

### 5.3 设置要点

仍需在系统里确认

```
设置 → 系统 → 语言与输入法 → 输入法管理
```

- 只保留一个默认输入法
- 关闭 允许输入法访问联系人
- 关闭 使用云服务
- 关闭所有 改善输入法体验 类选项

### 5.4 如果必须用厂商输入法

至少

- 撤销联网权限
- 关闭云联想、云同步、用户词库上传
- 关闭语音输入，或改为离线语音包

---

## 六、InstallerX

国产 ROM 的包安装器会在安装时插入广告、推荐、甚至喜闻乐见的安全检测环节，且“可能”把安装信息上报。InstallerX 用 Shizuku 或设备所有者权限直接调用系统安装 API，绕过检测环节
### 6.1 部署

1. 安装 InstallerX Revived（需支持 ABI 为 arm64-v8a）
2. 授予相应的权限
3. 在应用内选择工作模式：
   - **Shizuku 模式**：普通应用，权限来自 Shizuku
   - **Dhizuku 模式**：设备所有者模式，权限更高，可以卸载/安装系统应用
4. 设为默认安装器。一般会弹出打开方式，勾选 始终

### 6.2 注意事项

- 设为默认后，打开 APK 会走 InstallerX。若某次安装失败可断网临时改回系统安装器
- 慎用一切高权限
- 安装时若提示未知来源，在系统设置里授予 InstallerX 对应权限即可

### 6.3 btw

厂商安装器的“安全检测”会读取 APK 内容并可能上传。更换后安装行为数据不再经过厂商服务端

---

## 七、Dhizuku

Dhizuku ，可使应用获得 `Device Owner`（设备所有者）身份。此身份在 Android 体系里权限极高，可：

- 静默安装/卸载应用
- 管理设备策略（禁用摄像头、强制密码策略）
- 控制设备管理 API
- 忘了

### 7.1 风险

这是本节最需要谨慎的部分。

- `Device Owner` 一旦设定，**普通方式无法取消**。取消需要 `adb shell dpm remove-active-admin`，且部分 ROM 会在恢复出厂前拒绝
- 授予后，获得该身份的**应用本身**就成了这台设备的管理者。选择哪个应用获得这个身份，比操作本身更重要
- **避免使用闭源应用申请此权限**

### 7.2 建议

- 只在确有需要时使用
- 给开源项目
- 用完后移除：

```bash
adb shell dpm remove-active-admin com.rosan.dhizuku/.server.DhizukuDAReceiver
```

- 不使用时关闭 Dhizuku 的守护服务

### 7.3 与 Shizuku 的分工

| | Shizuku | Dhizuku |
|---|---|---|
| 权限来源 | ADB shell | Device Owner |
| 生命周期 | 设备重启后需重启服务 | 设定后长期有效 |
| 能力 | 系统 API、包管理、appops | 设备策略、静默安装、系统应用管理 |
| 风险 | 中 | **高** |

日常操作使用 Shizuku 即可

---

## 八、OTA

不被悄无声息的更新。

### 8.1 为什么

系统 OTA 可能：

- 将用户级卸载的系统应用恢复安装（`install-existing`）
- 重置用户设置
- 引入新的采集组件
- 在 A/B 机型上静默写入备用分区，重启后生效

### 8.2 操作

- 关闭自动更新
- 创建系统更新下载文件的同名文件以占位

### 8.3 代价

冻结系统更新后 **收不到安全补丁**，看着办吧。如果使用场景对漏洞补丁敏感，建议保留手动更新能力，并在更新前备份当前精简列表

### 8.4 更新前

决定更新时

1. 手动记录卸载清单
2. 更新完成后逐条核对
3. 检查是否有此前不存在的厂商组件

---

## 九、值得处理

### 9.1 应用权限

- 定位： 仅使用期间 ，关闭后台定位
- 麦克风/摄像头： 每次询问
- 通讯录/短信/通话记录：绝大多数应用不需要
- 通知读取权限：高价值权限，按需取用

### 9.2 无障碍服务

无障碍权限**极其危险**，这也是为什么小米会恶心你十秒钟的原因

只保留输入法（若其需要）与必要的自动化工具

### 9.3 网络层面

- 统一走代理
- 对不需要联网的应用在系统或防火墙层断网
- 部分 ROM 允许在“数据使用”里逐应用限制后台流量，更可靠

*网络这一层的做法单独写在第十节，它是全文里最不容易被上游改动绕过的一环*

### 9.4 开发者选项

```bash
adb shell settings get global adb_enabled
```

不用 adb 时建议关闭，避免 USB 接口暴露调试能力

### 9.5 广告标识符

Android 10 以上可以重置或关闭广告 ID

```
设置 → 隐私 → 广告 → 删除广告 ID
```

---

## 十、网络层

此前的做法都作用在应用层，现在该看看网络层了

它不与任何具体组件绑定，上报域名不变即可长期生效

### 10.1 能与不能

**能：**

- 按域名或 IP 丢弃出站请求，生效范围包括系统进程
- 记录被丢弃的请求，可审计
- 上游更换组件后仍然有效

**不能：**

- 拦截直接使用 IP 地址的请求（排除了网段除外）
- 拦截 VPN 隧道内部建立的通道
- 不改变设备本地已有的数据
- 移动网络下用户态 VPN 的接管并不总是完整的，见 10.5

这一层定位为 **增加成本**，不是制造不可能的障碍。它让把东西送出去这件事变得需要额外工作

### 10.2 两条部署方式的取舍

 Android 二选一

| | 用户态 VPN（Clash / sing-box 等） | 系统 hosts 文件 |
|---|---|---|
| 是否需要 root | 否 | **是** |
| 生效范围 | 绝大多数应用（少数可绕开） | 全系统，含原生代码 |
| 是否可被应用察觉 | 是（`isVpnActive`） | 否 |
| 是否留下日志 | 可审计 | 静默丢弃 |
| 配置复杂度 | 中 | 低 |
| 能否做 IP 段和规则匹配 | 是 | 仅可逐个域名 |
| 移动网络上是否可靠 | 视实现与 ROM 而定 | 是 |

若没有 root 只有用户态 VPN 这一条路

- **应用能检测到 VPN 是否开启。** 有些应用会因此拒绝工作或改变行为
- **同一时间只能有一个 VPN 应用。** 如果已经在用别的代理工具，需合并或者改用分应用代理
- **系统级 VPN 需要用户手动确认** 首次启用时有系统弹窗

如果设备有 root，hosts 更彻底，但解决不了 IP 直连且不留日志。两者并不互斥，同时用最好

### 10.3 需要拦的目标

以下是公开资料中出现过的采集端点，按来源分组。这一部分需要强调来源：它们来自第三方拆解与公开讨论，不是厂商文档，域名可能调整

**运营商设备管理平台**

| 域名 | 说明 |
|---|---|
| `dm.wo.com.cn` | 联通 DM 平台，SystemHelper 的上报目的地 |

这一条在公开拆解记录中有明确出处,运营商 DM 平台的域名通常遵循类似的命名规则，可以按后缀匹配整组

```
DOMAIN-SUFFIX,wo.com.cn,REJECT
```

**小米反诈与安全**

| 域名 | 说明 |
|---|---|
| `fraud-detection.engine.miui.com` | 反诈引擎 |
| `flash.sec.miui.com` | 安全组件上报 |

这两个域名出现在同一批公开讨论中。小米的域名分布很广，逐个列举不可行，更实际的做法是按后缀处理

**按后缀拦截的做法**

```
DOMAIN-SUFFIX,miui.com,REJECT
DOMAIN-SUFFIX,xiaomi.com,REJECT
DOMAIN-SUFFIX,mi.com,REJECT
DOMAIN-SUFFIX,wo.com.cn,REJECT
```

**不要直接照抄这四条** 三个小米域名覆盖了云服务、账号、主题、应用商店、系统更新、推送在内的一切。设备会失去大部分在线功能，系统界面会一直加载。正确做法是

1. 添加明确需拦截的条目，开日志观察一周
2. 审计被反复连接的域名
3. 逐个判断

### 10.4 Clash Meta 的写法

Mihomo（Clash Meta）的规则从上往下匹配，第一条命中的生效。所以：

1. **放行必须保留的**
2. **拦截采集端点**
3. **走默认策略**

一个最小可用的规则集：

```yaml
rules:
  # 1. 明确放行（按需补充）
  - DOMAIN-SUFFIX,xiaomi.com,DIRECT
  - DOMAIN-SUFFIX,miui.com,DIRECT

  # 2. 要拦截的采集端点，必须放在放行规则之前
  - DOMAIN,dm.wo.com.cn,REJECT
  - DOMAIN-SUFFIX,wo.com.cn,REJECT
  - DOMAIN,fraud-detection.engine.miui.com,REJECT
  - DOMAIN,flash.sec.miui.com,REJECT

  # 3. 默认
  - MATCH,DIRECT
```

有趣的是，第 2 组在第 1 组之后，而 `flash.sec.miui.com` 的父域是 `miui.com`，组 1 的 `DOMAIN-SUFFIX,miui.com` 会先命中。**拦截规则要放在放行规则前面**，但拦截规则要写得足够具体。正确顺序：

```yaml
rules:
  # 1. 只拦具体域名
  - DOMAIN,dm.wo.com.cn,REJECT
  - DOMAIN,fraud-detection.engine.miui.com,REJECT
  - DOMAIN,flash.sec.miui.com,REJECT

  # 2. 放行其余小米域名
  - DOMAIN-SUFFIX,miui.com,DIRECT
  - DOMAIN-SUFFIX,xiaomi.com,DIRECT

  # 3. 默认
  - MATCH,DIRECT
```

`REJECT` 会直接断开连接（TCP 发 RST，UDP 丢弃），比 `REJECT-DROP` 更快给出失败。如果某些组件在收到 RST 后会反复重试，改用 `REJECT-DROP` 反而能减少无谓的流量

**日志：**

```yaml
log-level: info
```

在 Mihomo 的界面或日志页看 `REJECT` 条目。细节转 `debug`，代价是日志量极大

**分应用处理** Mihomo 支持按应用（包名）决定是否接管，把需要联网的应用放进去，其余不接管，对系统界面影响最小

### 10.5 移动网络下的注意事项

这一部分和 WiFi 表现不同，容易被忽略。

在使用移动数据时，不少 ROM 会对用户态 VPN 做额外处理。表现为：

- VPN 显示已连接，但个别系统进程的流量仍然不经过
- 切换网络（WiFi ↔ 移动数据）后需要重新建连，期间有短暂的上报窗口
- 部分 ROM 的“智能网络切换”会临时改变路由

**排查方法**：在 VPN 开启状态下，用 `adb shell` 检查是否存在 tun 接口

```bash
adb shell ip -o link show | grep -E "tun|utun"
```

有输出说明隧道已建立。没有则说明 VPN 应用没有真正接管流量

另外，系统级的 VPN 和“按应用代理”是两种模式。后者依赖应用主动配合，想覆盖系统进程只能用前者

### 10.6 如何验证拦截生效

验证。

**一、日志**

在 Mihomo 的日志里搜目标域名，确认出现 `REJECT` 而不是 `DIRECT`

**二、 ADB 主动发起请求**

```bash
adb shell "ping -c 1 dm.wo.com.cn"
```

在 REJECT 规则下，DNS 解析或连接会失败。仍然成功意味着规则并未生效

**三、隧道是否存在**

```bash
adb shell ip -o link show | grep tun
```

没有 tun 接口玩个蛋

### 10.7 关系

两者解决的是不同问题

- **删组件**解决进程运行，但代价是功能损失且无法覆盖系统核心组件
- **拦网络**解决网络上报。覆盖范围更广但依赖规则，且需要定期从日志维护

**两者叠加** 

具体的先后顺序

1. 处理掉 4.6 节的采集组件
2. 加上明确已知的采集域名
3. 观察并补充规则
4. 对于不可避免的泄露，考虑是否接受

### 10.8 抄作业

作为起点。保留小米云服务与推送

```yaml
rules:
  # 采集端点
  - DOMAIN,dm.wo.com.cn,REJECT
  - DOMAIN-SUFFIX,wo.com.cn,REJECT
  - DOMAIN,fraud-detection.engine.miui.com,REJECT
  - DOMAIN,flash.sec.miui.com,REJECT

  # 其余小米域名放行
  - DOMAIN-SUFFIX,miui.com,DIRECT
  - DOMAIN-SUFFIX,xiaomi.com,DIRECT
  - DOMAIN-SUFFIX,mi.com,DIRECT

  - MATCH,DIRECT
```

替换掉 `MATCH,DIRECT` 可接入自己的策略组。如日常用途是分流翻墙，把默认策略换成分组名即可，拦截规则保持不动

---

## 十一、一套可执行的顺序

1. **开启开发者选项 → Shizuku → 启动服务**
2. **Canta**
3. **优先处理 4.6 节的两个采集组件**
4. **换输入法** *优先选无联网权限的开源方案*
5. **装 InstallerX 并设为默认安装器**
6. **逐批冻结厂商组件** *每批不超过 5 个，观察一周*
7. **配置网络层规则并开日志** *从第十节开始*
8. **冻结系统更新** *记录当前版本号*
9. **Dhizuku** *注意仅授权开源项目*

---

## 十二、回滚

每条操作都有对应的回滚方式，建议在开始前保存

```bash
# 查看某个包的状态
adb shell dumpsys package <包名> | grep -E "User 0:|enabled=|installed="

# 恢复冻结
adb shell pm enable <包名>

# 恢复用户级卸载（仅系统应用）
adb shell pm install-existing --user 0 <包名>

# 移除设备所有者
adb shell dpm remove-active-admin com.rosan.dhizuku/.server.DhizukuDAReceiver

# 恢复系统更新
adb shell pm enable com.android.updater
```

---

## 十三、结语

**把决定权从默认开启改成默认关闭，需要时再打开。**

它不是隐私的终点。但对绝大多数日常使用场景来说，这些步骤明显减少被采集面，且代价是可以承受的

如果你的威胁模型更严格（例如考虑的是有组织的定向关注），那么用户层加固的作用有限，应当考虑购买 Pixel 或 GrapheneOS 一类的设备，抱歉不在本文范围内

---

*命令基于 Android 16 / HyperOS 实测环境编写。不同 ROM 的包名与组件路径有差异，操作前使用 `dumpsys package` 确认*
