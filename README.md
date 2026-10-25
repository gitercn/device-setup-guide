# 装机与刷机指南

> 个人设备的装机 / 刷机流程记录：**手机（一加 13）** 与 **电脑（Windows 11）**。
>
> 固件版本、熔断状态、救砖直链、软件包名等信息均有时效性，记录于 **2026** 年前后，使用前请自行核对最新情况（直链尤其可能失效）。

## 目录

**第一部分 · 手机：一加 13 选购与刷机 / Root**

- [一、选购建议](#一选购建议)
- [二、前置准备](#二前置准备)
- [三、重装前备份清单](#三重装前备份清单)
- [四、解锁 Bootloader](#四解锁-bootloader)
- [五、刷机工具与固件](#五刷机工具与固件)
- [六、9008 救砖与熔断规避](#六9008-救砖与熔断规避)
- [七、刷入 Recovery](#七刷入-recovery)
- [八、获取 Root](#八获取-root)
- [九、Root 后常用模块](#九root-后常用模块)
- [十、TEE / 密钥状态说明](#十tee--密钥状态说明)
- [十一、精简 ColorOS](#十一精简-coloros)
- [十二、推荐 App 与技巧](#十二推荐-app-与技巧)
- [十三、刷回官方 / 重新上锁](#十三刷回官方--重新上锁)
- [致谢与来源](#致谢与来源)

**第二部分 · 电脑：Windows 11 全新安装与配置**

- [一、系统镜像下载](#一系统镜像下载)
- [二、Scoop 装机脚本](#二scoop-装机脚本)
- [三、Ungoogled Chromium 配置](#三ungoogled-chromium-配置)
- [四、Everything 与 7z 右键菜单](#四everything-与-7z-右键菜单)
- [五、系统优化工具](#五系统优化工具)

---

# 第一部分 · 手机：一加 13 选购与刷机 / Root 指南

> 本部分针对**中国版 PJZ110（ColorOS）**。

## 一、选购建议

一加 13 升级到 **16.0.3.500** 版本后加入了 **9008 熔断**，导致普通用户进不去 9008 救砖模式。建议购买时选择 **16.0.2.403 或更早**的版本。

对于喜欢玩刷机的人，或者将来考虑用来安装 PostmarketOS 之类系统的，9008 救砖模式仍然是需要的，更应避开已熔断的固件。

---

## 二、前置准备

刷机前请准备好：

- **电脑端 platform-tools**：包含 `adb` 与 `fastboot`，从 [Android 官方](https://developer.android.com/tools/releases/platform-tools) 下载并加入 PATH。
- **USB 驱动**：见下方 [解锁 Bootloader](#四解锁-bootloader) 一节的驱动排障。
- **优质数据线**：能稳定传输数据（非纯充电线）。
- **电量充足**：建议 50% 以上。
- **完成备份**：解锁 Bootloader 会**清空全部数据**，务必先按 [备份清单](#三重装前备份清单) 备份。

---

## 三、重装前备份清单

如果要重新刷机，推荐检查并备份以下文件夹 / 数据：

- **Pictures**
  - douyin 下载
  - 拼多多 / 淘宝保存照片
  - 微信保存文件
- **DCIM**
  - Camera
  - Screenshot
- **Downloads**
- **Documents**
- **MIUI**（如从小米机型迁移的文件夹）
  - Recording：通话 / 微信录音文件
- **Aegis**：密钥导出备份（两步验证）
- **酷控**：智能红外遥控备份

---

## 四、解锁 Bootloader

1. 连续点几下「设置 → 关于手机 → 版本信息 → 版本号」，输入锁屏密码开启开发者选项。
2. 进入「设置 → 其他设置 → 开发者选项」，开启「**OEM 解锁**」和「**USB 调试**」。
3. 用数据线连接电脑，弹窗选择「文件传输 / Android Auto」，授权电脑调试。
4. 执行解锁：

   ```bash
   adb reboot bootloader
   fastboot flashing unlock
   ```

   用「音量键」选择 **UNLOCK BOOTLOADER**，按「电源键」确认，手机自动重启格式化后即完成基础解锁。

### 完整解锁（含 unlock_critical）

上面只解锁了一项，还有 `unlock_critical` 也需要解锁才算完全解锁。正确做法：

1. 在常规解锁最后按「电源键」确认后，**手机黑屏的瞬间马上按住「音量减键」**，让手机重新进入 Bootloader 模式。
2. 执行：

   ```bash
   fastboot flashing unlock_critical
   ```

   同样用音量键选择 **UNLOCK BOOTLOADER**，电源键确认，等待自动重启格式化。
3. 重新进系统，再次开启「USB 调试」并授权。

### 验证解锁状态

```bash
fastboot oem device-info
```

```
(bootloader) Verity mode: true
(bootloader) Device unlocked: true
(bootloader) Device critical unlocked: true
(bootloader) Charger screen enabled: false
```

### 驱动排障（Windows）

如果 `fastboot devices` 连不上设备，且设备管理器里出现 `USB\VID_18D1&PID_D00D&REV_0100` 这种装不上驱动的情况：

- 检查 **Windows 更新 → 高级选项 → 可选更新**，看是否有 *Google, Inc. - Other hardware - Android Bootloader Interface*，安装该驱动。
- 也可尝试**禁用驱动签名强制**后手动安装。
- 参考：<https://developer.android.com/studio/run/win-usb?hl=zh-cn>

---

## 五、刷机工具与固件

一加没有像小米那种官方线刷工具，只有社区制作的刷机工具：

- 社区刷机工具：<https://roms.danielspringer.at/index.php?dir=Oneplus+13%2FSuper+Flashers>

---

## 六、9008 救砖与熔断规避

### 9008 救砖

- 9008 救砖刷机工具：<https://github.com/salokrwhite/OplusEdlTool>
- 救砖包（直链，可能失效）：`https://dfs-serverauto-in.allawnofs.com/dfs/25/07/31/873730e63cba4d3592a9bcf0d0fe32de.zip`

### 避免 9008 熔断

下载 500 全量包后解包，把 **400 全量包**里的以下分区替换到 **500 全量包**里，再开始线刷：

- `abl.img`
- `xbl.img`
- `xbl_config.img`
- `xbl_ramdump.img`

参考：<https://xdaforums.com/t/rom-official-lineageos-23-weeklies-for-oneplus-13.4766200/post-90475185>

---

## 七、刷入 Recovery

以 OrangeFox 为例（一加 13 为 A-only 设备）：

```bash
fastboot flash recovery recovery.img
```

参考：<https://wiki.orangefox.tech/en/guides/installing_orangefox>

---

## 八、获取 Root

**一加 13 推荐：[SukiSU Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra)**（基于 KernelSU 的内核级 Root，支持一加 13）。

- 官方文档（中文）：<https://github.com/SukiSU-Ultra/SukiSU-Ultra/blob/main/docs/zh/README.md>
- 图文教程参考：<https://m.romjd.com/jiaocheng/content/27954>

大致流程（具体以官方文档为准）：

1. 从对应固件解包出 `init_boot.img`。
2. 用 SukiSU Ultra 给 `init_boot.img` 打补丁，得到 `patched.img`。
3. 刷入：

   ```bash
   fastboot flash init_boot patched.img
   ```

4. 重启，安装 SukiSU Ultra 管理器 App 完成配置。

> **注意**：SukiSU Ultra 需要 GKI 内核，**不支持一加 8**。一加 8 推荐用 [APatch](https://apatch.dev/zh_CN/install.html)。

---

## 九、Root 后常用模块

- **ReZygisk** —— Zygisk 实现：<https://github.com/PerformanC/ReZygisk>

  Zygisk 环境（底层搬运工）：它是运行在 C/C++ 层面（Native 层）的底层通道。它只负责一件事——把你的模块「偷运」进 App 的进程里。它本身并不擅长去修改安卓的 Java 逻辑。

- **LSPosed**（JingMatrix 维护版）：<https://github.com/JingMatrix/LSPosed> ，使用说明：<https://v2ex.com/t/1064160>

  LSPosed 环境（Java 层手术刀）：它是基于 Zygisk 运行的。Zygisk 把 LSPosed 偷运进微信之后，LSPosed 就开始大展神威，它专门负责去修改（Hook）App 里的 Java/Kotlin 代码逻辑。

  Zygisk 只是开了个门；LSPosed 则是走进去对代码做外科手术的医生。你只有先装了 Zygisk 开门，才能装 LSPosed 进去做手术。

- **MagiskHluda**
  - 旧版：<https://github.com/Exo1i/MagiskHluda>
  - fork 中较新的：<https://github.com/cergo666/MagiskHluda>

- **Thanox** —— 应用管控 / 后台治理：<https://github.com/Tornaco/Thanox>

---

## 十、TEE / 密钥状态说明

> **重要**：解锁 Bootloader 后，部分硬件级密钥会失效（见下表 `SOTER / Attestation / RKP` 由「成功」变「失败」）。但除了 SOTER key 之外，其他几个可以修复。SOTER key 是腾讯的，目前只影响微信指纹支付。

### 解锁前 Key 状态

| Key | 状态 |
| --- | --- |
| Rpmb key | 成功 |
| SOTER key | 成功 |
| FAA key | 成功 |
| Crypto key | 成功 |
| WidevineL1 key | 不支持 |
| HDCP key | 不支持 |
| Attestation key | 成功 |
| FIDO key | 成功 |
| PKI cert | 成功 |
| PKI Group cert | 成功 |
| FIDO2 key | 成功 |
| RKP default | 成功 |
| RKP widevine | 成功 |
| StrongBox key | 成功 |

### 解锁后 Key 状态

| Key | 状态 |
| --- | --- |
| Rpmb key | 成功 |
| **SOTER key** | **失败(无法真正修复)** |
| FAA key | 成功 |
| Crypto key | 成功 |
| WidevineL1 key | 不支持 |
| HDCP key | 不支持 |
| **Attestation key** | **失败(可修复)** |
| FIDO key | 成功 |
| PKI cert | 成功 |
| PKI Group cert | 成功 |
| FIDO2 key | 成功 |
| **RKP default** | **失败(可修复)** |
| **RKP widevine** | **失败(可修复)** |
| StrongBox key | 成功 |

测试参考：<https://wuxianlin.com/2025/11/12/oneplus-13-15-attestation-rkp-test/>

### 恢复 Attestation / RKP（需先 Root）

```bash
# 先备份原始 persist 分区
adb pull /sdcard/persist_orig.img
# 输出示例：1 file pulled, 0 skipped. 39.2 MB/s (67108864 bytes in 1.632s)

adb shell
su
LD_LIBRARY_PATH=/vendor/lib64/hw KmInstallKeybox Keybox_file Device_ID true rkp
```

成功输出示例：

```
qmi_client_send_msg_sync success for get dev_ser_num 0
Imei: xxxxxx
qmi_client_send_msg_sync success for get dev_ser_num 1
Imei: xxxxxx
Brand: OnePlus
Device: OP5D0DL1
Product: PJZ110
SerialNum: xxxxxx
Manufacturer: OnePlus
Model: PJZ110
TEE done
InstallKeybox is done!
```

---

## 十一、精简 ColorOS

卸载预装 / 无用组件示例（按需，卸前确认用途）：

```bash
pm uninstall --user 0 com.nearme.instant.platform
```

---

## 十二、推荐 App 与技巧

### 开源 Android App

- 开源应用合集：[Oh My FOSS Android](https://github.com/xlucn/oh-my-foss-android)
  - **Fennec**（基于 Firefox 的浏览器）

### 通话录音

- Cube Call Recorder ACR Premium：<https://prosmart.by/android/soft_android/internet_android/19006-cube-call-recorder-acr-premium-2069.html>

### 应用多开（Island）

[Island (F-Droid)](https://f-droid.org/en/packages/com.oasisfeng.island.fdroid/) —— 通过工作资料实现多开 / 隔离：

```bash
adb -d shell
pm create-user --profileOf 0 --managed Island
pm install-existing --user 11 com.oasisfeng.island.fdroid
pm set-profile-owner --user 11 com.oasisfeng.island.fdroid/com.oasisfeng.island.IslandDeviceAdminReceiver
am start-user 11
```

---

## 十三、刷回官方 / 重新上锁

> 退坑、送保修或出二手前，通常需要刷回官方固件并重新锁定 Bootloader。
>
> - 刷回官方：用 [社区刷机工具](#五刷机工具与固件) 或 [9008 救砖](#六9008-救砖与熔断规避) 刷入完整官方全量包。
> - 重新上锁：`fastboot flashing lock`（**会再次清空数据**；务必先刷回完全原厂状态，否则可能无法开机）。

---

## 致谢与来源

本部分整理自社区资料，感谢以下作者与项目：

- [salokrwhite/OplusEdlTool](https://github.com/salokrwhite/OplusEdlTool)
- [SukiSU-Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra)
- [PerformanC/ReZygisk](https://github.com/PerformanC/ReZygisk)
- [JingMatrix/LSPosed](https://github.com/JingMatrix/LSPosed)
- [Tornaco/Thanox](https://github.com/Tornaco/Thanox)
- [APatch](https://apatch.dev/)
- [OrangeFox Recovery](https://wiki.orangefox.tech/)
- [danielspringer.at Super Flashers](https://roms.danielspringer.at/)
- [xlucn/oh-my-foss-android](https://github.com/xlucn/oh-my-foss-android)
- XDA Forums、[wuxianlin.com](https://wuxianlin.com/) 等社区分享

---

# 第二部分 · 电脑：Windows 11 全新安装与配置

> 记录一台新机器从装系统到常用软件就位的流程。偏个人习惯，命令按需增删。

## 一、系统镜像下载

- [RG-Adguard](https://files.rg-adguard.net/version/f0bd8307-d897-ef77-dbd6-216fefbe94c5) —— 微软服务器直链聚合，可选版本 / 语言 / 架构
- [Massgrave](https://massgrave.dev/windows_11_links.html) —— 各版本官方 ISO 直链清单

当前在用的镜像：

```
zh-cn_windows_11_business_editions_version_23h2_updated_jan_2024_x64_dvd_fee59269.iso
```

> Business 版（专业版 / 企业版）比 Consumer 版少一批预装的消费级应用，装完更干净。

## 二、Scoop 装机脚本

Scoop 装在用户目录下，全程不需要管理员权限，重装系统后恢复环境最省事。整段按顺序执行即可。

```powershell
# 1. 允许当前用户运行脚本，并安装 Scoop
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

# 2. 下载加速与 git（bucket 依赖 git）
scoop install aria2
scoop install git
scoop bucket add extras

# 3. 常用软件
scoop install synctrayzor
scoop install beyondcompare
scoop install everything vlc vscode keepassxc ungoogled-chromium idea-ultimate

# 4. 环境自检
scoop checkup

# 5. sudo + 解除 260 字符路径限制
scoop install sudo
sudo Set-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name 'LongPathsEnabled' -Value 1

# 6. 版本管理 / Python 环境
scoop install nvm mambaforge
conda init powershell
```

几点说明：

- `aria2` 装好后 Scoop 会自动用它做多线程下载，拉大包明显更快。
- `synctrayzor` 是 Syncthing 的托盘壳；`beyondcompare` 做文件 / 目录对比。
- `LongPathsEnabled` 建议早点开，否则后面 `git clone` 深目录、编译 Node 原生模块时容易报路径过长。
- `mambaforge` 上游已停止维护，新装机建议改用 Miniforge（装法同理）。
- `conda init powershell` 只对当前用户生效，需要重开终端才生效。

## 三、Ungoogled Chromium 配置

装完是干净的，但有两件事要手动补：搜索引擎和扩展商店。

- [添加 Google 搜索](https://github.com/ungoogled-software/ungoogled-chromium/discussions/1488)
- [安装扩展（chromium-web-store）](https://github.com/NeverDecaf/chromium-web-store)

**用户数据目录**：Scoop 装的 Chromium 默认把 `User Data` 放在 `%localappdata%` 下，`scoop update` 或卸载重装时配置容易丢。先确认实际位置：

```
chrome://version/
```

再把目录重定向到 Scoop 的 persist 目录（`<username>` 换成自己的用户名）：

```cmd
mklink /D "%localappdata%\Chromium\User Data" "C:\Users\<username>\scoop\persist\ungoogled-chromium\User Data"
```

最后改快捷方式的目标，把数据目录固定下来：

```
"C:\Users\<username>\scoop\apps\ungoogled-chromium\current\chrome.exe" --user-data-dir="C:\Users\<username>\scoop\apps\ungoogled-chromium\current\User Data" --single-argument %1
```

参考：<https://github.com/ScoopInstaller/Extras/issues/13975>

## 四、Everything 与 7z 右键菜单

Scoop 装的 7-Zip 不会自动注册右键菜单。用 Everything 搜 `7zfm.exe` 找到 7-Zip 的实际安装路径，再进 7-Zip 里设置文件关联 / 右键菜单。

- [Scoop 使用笔记](https://hackettyu.com/2020/0507/windows-scoop/)

## 五、系统优化工具

- [Optimizer](https://github.com/hellzerg/optimizer) —— 关遥测、清右键菜单、开关系统服务，图形界面
- [Sophia Script for Windows](https://github.com/farag2/Sophia-Script-for-Windows) —— 脚本化调优，粒度比 Optimizer 细

> 两者都会改注册表，跑之前建议先建一个系统还原点。
