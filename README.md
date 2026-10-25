# 一加 13 选购与刷机 / Root 指南

> 本指南针对**中国版 PJZ110（ColorOS）**。
>
> 固件版本、熔断状态、救砖直链等信息均有时效性，记录于 **2026-06** 前后，使用前请自行核对最新情况（直链尤其可能失效）。

## 目录

- [一、选购建议](#一选购建议)
- [二、前置准备](#二前置准备)
- [三、重装前备份清单](#三重装前备份清单)
- [四、解锁 Bootloader](#四解锁-bootloader)
- [五、刷机工具与固件](#五刷机工具与固件)
- [六、9008 救砖与熔断规避](#六9008-救砖与熔断规避)
- [七、刷入 Recovery](#七刷入-recovery)
- [八、获取 Root（推荐 SukiSU Ultra）](#八获取-rootsukisu-ultra)
- [九、Root 后常用模块](#九root-后常用模块)
- [十、TEE / 密钥状态说明](#十tee--密钥状态说明)
- [十一、精简 ColorOS（Debloat）](#十一精简-coloros)
- [十二、推荐 App 与技巧](#十二推荐-app-与技巧)
- [十三、刷回官方 / 重新上锁](#十三刷回官方--重新上锁)
- [致谢与来源](#致谢与来源)

---

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

**一加13推荐：[SukiSU Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra)**（基于 KernelSU 的内核级 Root，支持一加 13）。

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

** SukiSU Ultra需要GKI内核，不支持一加8，一加8推荐：[APatch](https://apatch.dev/zh_CN/install.html)**

---

## 九、Root 后常用模块


- **ReZygisk** —— Zygisk 实现：<https://github.com/PerformanC/ReZygisk>
Zygisk 环境（底层搬运工）： 它是运行在 C/C++ 层面（Native 层）的底层通道。它只负责一件事——把你的模块“偷运”进 App 的进程里。它本身并不擅长去修改安卓的 Java 逻辑。

- **LSPosed**（JingMatrix 维护版）：<https://github.com/JingMatrix/LSPosed> ，使用说明：<https://v2ex.com/t/1064160>
LSPosed 环境（Java 层手术刀）： 它是基于 Zygisk 运行的。Zygisk 把 LSPosed 偷运进微信之后，LSPosed 就开始大展神威，它专门负责去修改（Hook）App里的 Java/Kotlin 代码逻辑。

Zygisk 只是开了个门；LSPosed 则是走进去对代码做外科手术的医生。你只有先装了 ZygiskNext 开门，才能装 LSPosed 进去做手术。

- **MagiskHluda**
https://github.com/Exo1i/MagiskHluda （旧版）
https://github.com/cergo666/MagiskHluda （fork中最新的）

- **Thanox** —— 应用管控 / 后台治理：<https://github.com/Tornaco/Thanox>

---

## 十、TEE / 密钥状态说明

> **重要**：解锁 Bootloader 后，部分硬件级密钥会失效（见下表 `SOTER / Attestation / RKP` 由「成功」变「失败」）。但除了SOTER key之外，其他几个可以修复。SOTER key是腾讯的，目前只影响微信指纹支付。。
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
>


---

## 致谢与来源

本指南整理自社区资料，感谢以下作者与项目：

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
