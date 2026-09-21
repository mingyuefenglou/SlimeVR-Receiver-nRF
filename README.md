# NiNi SlimeNRF Receiver 固件

基于 SlimeNRF 生态的 nRF52833 接收器（dongle/hub）固件。

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

`west.yml` 选定 [jitingcn/sdk-nrf](https://github.com/jitingcn/sdk-nrf) `v3.4-branch`（跟随分支；NCS v3.4.0 基）。构建需 **Zephyr SDK 1.0.1 GNU** + **Python 3.12**；CI 在 Ubuntu 24.04。

```bash
west init -l app
west update
export ZEPHYR_SDK_INSTALL_DIR=/opt/zephyr-sdk-1.0.1
west build -b nini_slimevr_5883rx_uf2 -d build --sysbuild --pristine -s app ---DBOARD_ROOT=$PWD/app
```

注：v3.4.0 起 `MPSL_FEM_GENERIC_TWO_CTRL_PINS_SUPPORT` 由 devicetree 兼容串自动推导（板 dts 已带 `radio-fem-two-ctrl-pins`），无需也不可在配置文件里手工赋值。

## LED 状态指示（三通道）

console 命令（重启保持）：`ledmode daily|debug`（日常=呼吸族·默认 / 调试=闪烁族）、`ledbright 5-100`（全局亮度，一改全改）。

**灯语**：绿=供电在岗；蓝=通讯域；红=异常独占。与 tracker 端形成呼应——两端同时亮起同款错峰双呼吸 = 链路活着。

| 状态 | 日常表（呼吸族） | 调试表（闪烁族） |
|---|---|---|
| 上电在岗、无 tracker | 仅绿慢呼吸 10s | 绿 300ms blip/10s |
| 与 tracker 通讯中（核心态） | 绿呼吸 + 蓝心跳呼吸（错峰 2.5s；**峰值随台数渐满**：1 台 25% → 10 台 55%） | 绿 blip + 蓝连跳（**次数=台数**，1-10 可数） |
| 配对模式（等新 tracker） | 绿呼吸 + 蓝双短呼吸 | 绿 blip + 蓝快闪 |
| 新 tracker 入网 | 蓝渐亮确认 | 蓝连闪 |
| USB 通讯/调参会话 | 绿呼吸 + 蓝 15% 低常亮 | 绿 blip + 蓝 20% 常亮 |
| 错误 | 红（独占三灯）5s 深呼吸 | 红→绿→蓝三色轮播 |
| 断电 | 全彩渐灭 | 渐灭 |

## 许可

沿袭上游 Apache-2.0，见仓库 `LICENSE`。

