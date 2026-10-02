# NiNi SlimeNRF Receiver 固件

> 基于 SlimeNRF 生态的 nRF52833 接收器（dongle/hub）固件 —— ESB 协议、USB HID、三共阴 LED、ESB OTA。

[![build](https://github.com/mingyuefenglou/SlimeVR-Receiver-nRF/actions/workflows/workflow.yml/badge.svg?branch=NiNi_Slime_5883)](https://github.com/mingyuefenglou/SlimeVR-Receiver-nRF/actions/workflows/workflow.yml)
![NCS](https://img.shields.io/badge/NCS-v3.4.1_LTS-00A9CE)
![MCU](https://img.shields.io/badge/MCU-nRF52833-333333)
![license](https://img.shields.io/badge/license-Apache--2.0-lightgrey)

## 特性

| 模块 | 说明 |
|---|---|
| **协议** | SlimeVR ESB 接收（多 tracker 汇聚）· 远程命令（fusion reset / tcal 校准）· USB HID（带卡死自愈冷却）· tracker 事件订阅 · TDMA 槽位偏移监测 |
| **LED** | 三通道，日常（常亮/闪族·本轮重做）/调试（闪烁族）双状态表；蓝=通讯域（配对 20Hz 快闪 / 待命 lub-dub 心跳 / 已连常亮、亮度随台数渐亮）；每命中一台全彩渐亮渐灭；`ledmap` 免重编重绑 |
| **升级** | ESB OTA + UF2 双路；配套 BL 断电/误触多重加固（断电直进 APP，不再卡 BL 须重刷） |
| **工具** | 采集工具链（data_collect CDC 变体）· console 调参 |

## 分支说明

| 分支 | 用途 |
|---|---|
| `main` | 稳定发布线——验证通过的版本（日常刷机用这里） |
| `NiNi_Slime_5883` | 开发线——板级定制与功能开发在此进行，验证后合入 `main` |
| `dev` | jitingcn 上游镜像（本仓不在此开发）——跟进上游更新、比对差异、合并上游修复 |

开发流程：在 `NiNi_Slime_5883` 提交 → 上板验证 → 合入 `main` 发布。

## 项目来源

* [SlimeVR/SlimeVR-Tracker-nRF-Receiver](https://github.com/SlimeVR/SlimeVR-Tracker-nRF-Receiver) —— 官方上游
* [jitingcn/SlimeVR-Tracker-nRF-Receiver](https://github.com/jitingcn/SlimeVR-Tracker-nRF-Receiver) —— 本仓直接基底（dev @ 5b0e736，含远程命令、USB HID、ESB OTA、采集工具链、tracker 事件订阅等）

配套追踪器固件：[mingyuefenglou/SlimeVR-Tracker-nRF](https://github.com/mingyuefenglou/SlimeVR-Tracker-nRF)（协议特性需配套使用）。

## SDK 与编译环境

`west.yml` 选定 [jitingcn/sdk-nrf](https://github.com/jitingcn/sdk-nrf) `v3.4-branch`（跟随分支；NCS v3.4.1 LTS 基）。构建需 **Zephyr SDK 1.0.1 GNU** + **Python 3.12**；CI 在 Ubuntu 24.04。

```bash
west init -l app
west update
export ZEPHYR_SDK_INSTALL_DIR=/opt/zephyr-sdk-1.0.1
west build -b nini_slimevr_rx_uf2 -d build --sysbuild --pristine -s app -- -DBOARD_ROOT=$PWD/app
```

注：v3.4.0 起 `MPSL_FEM_GENERIC_TWO_CTRL_PINS_SUPPORT` 由 devicetree 兼容串自动推导（板 dts 已带 `radio-fem-two-ctrl-pins`），无需也不可在配置文件里手工赋值。

## LED 状态指示（三通道）

console 命令（重启保持）：

| 命令 | 作用 |
|---|---|
| `ledmode` | 查看当前模式 |
| `ledmode daily\|debug` | 切日常（呼吸族·默认）/调试（闪烁族）状态表 |
| `ledbright` | 查看全局亮度 |
| `ledbright 0-100` | 全局亮度=**全域最大亮度**（0=全灭，一改全改）。所有灯效输出都受它缩放、任何图案都不会超过它；**默认 80%** |
| `ledmap` | 查看 LED 绑定：物理位 LED1/2/3 各是什么色 |
| `ledmap LED1 R LED2 G LED3 B` | 全量指派（三位须为 R/G/B 各一次，重复直接拒绝） |
| `ledmap LED1 R` | 单点=交换语义：LED1 与当前占 R 的位对调 |
| `ledmap reset` | 回板默认 LED1=R LED2=G LED3=B |

**LED 绑定（`ledmap`）**：本板默认引脚映射 **红=P0.30、绿=P0.29、蓝=P0.28**（pwm0 通道 1/2/0），共阴 LED、GPIO 经 1kΩ 限流。换用不同色序的灯用 `ledmap` 改路由即可、免重编，重启保持。

**灯语**：绿=供电在岗（每 3s 短闪）；蓝=通讯域（配对 20Hz 快闪 / 待命 lub-dub 心跳 / 已连常亮）；红=异常独占（常亮）。与 tracker 端形成呼应——两端同时呼吸起伏 = 链路活着。

| 状态 | 日常表（常亮/闪族·默认） | 调试表（闪烁族） |
|---|---|---|
| 工作指示（一切状态的底） | 🟢 绿每 3s 短闪一次（300ms 亮） | 🟢 绿 300ms blip/10s |
| 配对模式中（含全新机开机自动搜台） | 🔵 蓝 **20Hz 快闪**（25ms 亮/25ms 灭） | 🔵 蓝 100/900 快闪 |
| 非配对 + 无在线设备（有记录全离线、新机超时后） | 🔵 蓝 **lub-dub 双击心跳**（150 亮→300 灭→150 亮→停 1.2s） | 蓝灭（保持原「无连接=灭」） |
| 配对中每命中一台 | 🌈 全彩渐亮渐灭 1.2s（每台各播一次） | 🔵 蓝连闪 |
| 配对完成、≥1 台在线 | 🔵 蓝**常亮**，亮度随在线台数 1→10 渐亮（50%→95%） | 🔵 蓝连跳（**次数=台数**，1-10 可数） |
| 故障（错误位置位） | 🔴 红**常亮**（蓝绿强制取消） | 🔴→🟢→🔵 三色轮播 |
| 断电 | 全彩渐灭 | 渐灭 |

## 许可

沿袭上游 Apache-2.0，见仓库 `LICENSE`。
