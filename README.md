# BatteryWhitelist (电池白名单守护者)

![BatteryWhitelist](assets/ic_launcher.png)

[![release](https://github.com/batterywhitelist/BatteryWhitelist/actions/workflows/release.yml/badge.svg)](https://github.com/batterywhitelist/BatteryWhitelist/actions/workflows/release.yml)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)]()
[![Root](https://img.shields.io/badge/Root-KernelSU%20%7C%20Magisk-red.svg)]()

> “买的设备是自己的，自己拥有对该设备的一切权利，厂商无权干涉。”
> —— 一个诞生于对抗流氓系统“爹味”管理的硬核 LSPosed 模块。

## 🚨 极高危警告（小白必读）

**本模块涉及系统底层深度修改，绝非普通应用！**

> ❌ **严禁**将 Android 系统核心组件（如 `com.android.systemui`, `com.android.settings`, `com.android.phone` 等）加入白名单！
> 
> 即使 UI 层提供了警告，若您执意操作，**极大概率会导致系统 UI 崩溃、死机、甚至无限重启（变砖）**。
> 
> 开发者已在代码层面对高危应用进行拦截，并在《用户许可协议》(EULA) 中尽到最高级别警示义务。任何因不按规范操作、盲目勾选导致的设备损坏，**均由您自行承担全部后果**。小白用户请勿轻易尝试！

## 📖 背景故事

在 Android 16 (ColorOS) 的时代，系统对后台应用的“爹味”管控极其霸道。即使你手动将应用设为“无限制”，系统依然会在重启、切换网络或云控下发指令时，悄悄把它改回“优化”并强行冻结，导致手表断连、消息收不到、后台任务被杀。

本模块专为解放这一痛点而生。它不仅在内存中拦截系统底层的篡改行为，还结合了 Root 脚本，形成了一套“四位一体”的终极防御体系，彻底把选择权还给用户。

## ✨ 核心功能

*   **内存层动态狙杀 (Xposed Hook)**：精准切入 `system_server`，拦截原生 `DeviceIdleController`，并针对 ColorOS 独有的 `OplusHansManager` 与 `IOplusDeviceIdleHelper` 动态代理进行强制拦截。
*   **内存守护线程**：在系统框架内部开启守护线程，每 60 秒主动发送底层系统命令（`cmd deviceidle whitelist` / `appops`），强行纠正系统偏差。
*   **开机自启脚本 (Root)**：通过 UI 端自动向 `/data/adb/service.d/` 写入守护脚本，在系统启动后以 Root 权限无限循环执行强制纠正。
*   **防呆安全拦截**：代码内置高危包名黑名单，UI 层面强制拦截 `systemui` 等核心组件的勾选，导入配置时自动剔除高危项，从物理层面防止“手贱变砖”。
*   **极致轻量**：拒绝 AndroidX 臃肿生态，纯原生 + Support 库瘦身，APK 体积仅 **812KB**，极致纯净。
*   **动态 UI 管理**：抛弃写死包名，内置极简应用列表，勾选后自动同步至底层并持久化生效。
*   **固定签名与自动化分发**：基于 GitHub Actions 云端编译并固定签名，支持 Cover Install 无感升级。用户端 App 自动请求 `version.json` 检查更新，配合 Cloudflare 绕过缓存，实现秒级下载新版本。

## 🛡️ “四位一体”防御体系

| 防御层级 | 技术实现 | 核心逻辑 |
| :--- | :--- | :--- |
| **第一层：原生拦截** | Hook `DeviceIdleController` | 拦截系统将应用移出 Doze 白名单的底层操作 |
| **第二层：ColorOS 狙杀** | Hook `OplusHansManager` | 伪造保活状态与可见性，拦截后台清理指令 |
| **第三层：动态代理阻断** | Hook `IOplusDeviceIdleHelper$Default/$I` | 拦截动态代理，拒绝忽略临时白名单，强制注入 Doze 白名单 |
| **第四层：底层兜底** | Root 脚本 + 内存守护线程 | 每 60 秒/10 秒强行执行 `appops` 与 `deviceidle` 命令纠正状态 |

## 📲 安装与使用

### 前提条件
*   设备已解锁 Bootloader。
*   已刷入 KernelSU (或 Magisk) 并具备 Root 环境。
*   已正确安装并激活 Zygisk Next 与 LSPosed (API 102)。
*   **系统版本**：建议 Android 14 及以上（ColorOS / OxygenOS 效果最佳）。

### 安装步骤
1. 前往 [Releases](../../releases) 下载最新构建的 `app-release.apk`。
2. 在手机上安装该 APK，首次打开请务必仔细阅读并同意《用户许可协议》。
3. 打开 Root 管理器 (KernelSU / Magisk)，在应用列表中找到 `BatteryWhitelist`，开启超级用户权限开关。
4. 打开 BatteryWhitelist App，在列表中搜索并勾选你需要保护的应用（例如：小米运动健康、微信等）。
5. 退出 App 后再次打开，此时 App 会通过 Root 权限自动向 `/data/adb/service.d/` 部署守护脚本。
6. 在 LSPosed 管理器中启用本模块，作用域勾选 **“Android 系统 (android)”**。
7. 重启手机。

### 验证成功
重启后，使用 MT 管理器或 Root 终端检查以下位置：
*   `/data/system/BatteryWhitelist.log` 是否出现 `内存守护线程已启动` 和 `成功连接 UI 配置`。
*   `/data/adb/service.d/battery_guard.sh` 是否存在且权限为 `755`。
*   在终端执行 `dumpsys deviceidle whitelist`，查看受保护应用是否在其中。

## 🔬 核心技术细节

*   **反射黑魔法**：由于 libxposed API 102 AAR 包在编译时存在接口缺陷，项目创造性地使用了运行时反射 (`hookMethod()`) 动态调用 `XposedInterface.hook()` 绕过编译器的类型检查。
*   **跨进程通信**：Xposed 模块通过反射 `ActivityThread.currentActivityThread().getSystemContext()` 获取系统上下文，再 `createPackageContext` 读取 UI 应用写入的 `SharedPreferences`。
*   **反向突破**：深度剖析 `OplusHansManager` 和 `OplusDeviceIdleHelper` 的 `getNewWhiteList`、`whiteListChangedHandle` 等关键方法，实现内存篡改。
*   **数据完整性校验**：UI 端利用 SHA-256 校验用户协议（EULA）是否被篡改，强制用户阅读高危警告；利用 MD5 校验守护脚本，避免重复写入 Root 目录。

## ⚠️ 法律与免责声明

1. **按原样提供**：本项目基于 **Apache License 2.0** 协议开源。本软件按“原样 (AS IS)”提供，不附带任何明示或暗示的担保。
2. **风险自担**：本项目涉及 Android 底层的 `system_server` 修改。请勿使用 MT 管理器直接修改 `/data/system/` 下的系统 XML 配置文件。因违规操作导致的系统无限重启、数据丢失等后果，开发者不承担任何赔偿责任。
3. **商标与二次分发**：严禁将本软件（含源码及编译后的 APK）用于任何商业倒卖。基于本项目的二次修改版，必须在显著位置标明“修改版”，且严禁使用原作者 (MikotoNetwork) 名义进行虚假宣传。

## 诞生故事

> 某天早餐后的头脑风暴，让我想起了这个问题
> 之后我就设了总体架构以及内部的技术细节
> 感谢那个早上的自己，维护项目至今……我从未忘记初心

## 👨‍💻 开发者手记

这个项目诞生于无数个搞机熬夜的夜晚与对流氓系统的愤怒之中。从 GitHub Actions 踩坑，到反编译 `OplusHansManager`，最终用反编译和 Root 脚本重塑底线。

它是一个模块，更是夺回数字主权的缩影。只要设备在我们手里，一切规则就应由我们自己书写。

*(本项目基于 Apache LICENSE V2.0 协议开源)*