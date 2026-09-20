# Function-EV 适配与验证说明

## 代码归属

本作品采用比赛规定的多仓边界：

| 内容 | 权威位置 | 提交状态 |
| --- | --- | --- |
| ESP32-P4 Function-EV 板级目录、defconfig、底层兼容适配 | [`supei-1/nuttx`](https://github.com/supei-1/nuttx/tree/codex/esp32p4-function-ev-l0-l2) | [上游 PR #393](https://github.com/open-vela/nuttx/pull/393)，CLA 已通过 |
| LVGL 9.2.1 dashboard、内嵌 NSH、出库台和字库资产 | [`supei-1/nuttx-apps`](https://github.com/supei-1/nuttx-apps/tree/codex/esp32p4-function-ev-lvgl) | [上游 PR #131](https://github.com/open-vela/nuttx-apps/pull/131)，CLA 已通过 |
| manifest、作品说明、证据分级 Skill、AI 日志 | 本仓 | 本仓 PR 提交 |

专属仓不复制 `nuttx` 或 `apps` 的生产目录；`openvela.xml` 在 PR 合入前固定到个人 fork 分支，保证当前提交可复现。

## 已实现能力

- L0 启动链、UART0、NSH、GPIO/PWM、32 MB PSRAM、I2C0、SPI2、RTC、Watchdog。
- SPI Flash MTD、SmartFS、MIPI-DSI、EK79007AD LCD、1024x600 RGB565、PSRAM framebuffer、`/dev/fb0`。
- LVGL 9.2.1 fbdev、GT911 `/dev/input0`、LVGL touchscreen、触摸坐标 180° 修正。
- `lvgldemo dashboard` 主界面、内嵌 NSH 和 P4 出库台纯 LVGL 业务流程；`lvgldemo widgets` 保持原路径。
- SC2336/V4L2/CSI 适配和调试代码保留在公共仓 PR，但摄像头业务在本阶段显示为阻塞，不把控制面成功当作真实帧接收成功。

## 业务边界

出库台包含首页、扫码/订单核验、车辆选择/派发和任务状态页。扫码结果、空闲车辆、任务阶段和异常提示均为应用层模拟状态；本阶段不访问真实摄像头、音频、车辆控制、LED、背光或 RTC。

## 证据分级

每项结论必须按下列类别单独报告：

1. **源码实现**：文件、配置、分支和提交存在；不等于已经运行。
2. **构建验证**：使用目标工作树的实际工具链生成镜像，记录大小、SHA-256 和构建日志。
3. **JTAG 烧录验证**：重新核对芯片、端口、镜像、地址和偏移，记录 OpenOCD 结果；`Verify OK` 只证明写入校验。
4. **UART0 运行验证**：从启动日志确认 `nsh>`，执行目标命令、保存完整原始日志，并单独验证 Ctrl+C 返回 `nsh>`。
5. **LCD/触摸视觉验证**：只有实体屏幕观察到画面并实际点击后，才能标记视觉或触摸闭环通过。

## 评委快速检查

```text
nsh> lvgldemo dashboard
# 点击主界面入口进入出库台，按流程推进
# 发送 Ctrl+C，应回到 nsh>
nsh> lvgldemo widgets
# 原有 widgets 应保持可启动，并可 Ctrl+C 返回 nsh>
```

构建、烧录和 UART 原始证据保存在活动工作树 `openvela-426-l0/evidence/` 对应日期目录中；提交报告时只引用实际存在的日志和实测结论。
