# NiNi 5883 接收器固件（SlimeNRF receiver）

基于 SlimeNRF 生态的 nRF52833 接收器（dongle/hub）固件，为自研 5883 接收板定制。

## 项目来源

- [SlimeVR/SlimeVR-Tracker-nRF-Receiver](https://github.com/SlimeVR/SlimeVR-Tracker-nRF-Receiver) —— 官方上游
- [jitingcn/SlimeVR-Tracker-nRF-Receiver](https://github.com/jitingcn/SlimeVR-Tracker-nRF-Receiver) —— 本仓直接基底（dev @ 5b0e736，含远程命令、USB HID、ESB OTA、采集工具链、tracker 事件订阅等）

配套追踪器固件：[mingyuefenglou/SlimeVR-Tracker-nRF](https://github.com/mingyuefenglou/SlimeVR-Tracker-nRF)（协议特性需配套使用）。

## 与 jiting 版的主要差异

- **新板 `nini_slimevr_5883rx_uf2`**：三颗共阴 LED（红 P0.28 / 绿 P0.30 / 蓝 P0.29）、AT2401C 射频前端（通用双控制脚绑定，ctx P0.17 / crx P0.15，增益起步 22/12 dB）、DCDC 供电（旧板为 LDO）

## SDK 与编译环境

`west.yml` 选定 [jitingcn/sdk-nrf](https://github.com/jitingcn/sdk-nrf) `v3.4-branch`（pin ab62f8df，NCS v3.4.0 基）。构建需 **Zephyr SDK 1.0.1 GNU** + **Python 3.12**；CI 在 Ubuntu 24.04。

```bash
west init -l app
west update
export ZEPHYR_SDK_INSTALL_DIR=/opt/zephyr-sdk-1.0.1
west build -b nini_slimevr_5883rx_uf2 -d build --sysbuild --pristine -s app -- \
  -DBOARD_ROOT=$PWD/app
```

注：v3.4.0 起 `MPSL_FEM_GENERIC_TWO_CTRL_PINS_SUPPORT` 由 devicetree 兼容串自动推导（板 dts 已带 `radio-fem-two-ctrl-pins`），无需也不可在配置文件里手工赋值。

## 许可

沿袭上游 Apache-2.0，见仓库 `LICENSE`。
