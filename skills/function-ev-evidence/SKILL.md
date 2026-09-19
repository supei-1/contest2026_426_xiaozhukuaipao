---
name: function-ev-evidence
description: "审计 ESP32-P4 Function-EV 移植和 LVGL 作品的源码、构建、JTAG、UART 与 LCD/触摸证据边界。适用于提交前复核，避免把静态或烧录结果误报为运行成功。"
---

# Function-EV Evidence Review

用于本作品提交前的证据审计。目标是输出可复核的事实，不替用户扩大硬件支持范围。

## 必须区分的五类结论

按顺序检查并分别记录：

1. **源码实现**：目标文件、配置、分支和提交存在；不等于已经运行。
2. **构建验证**：使用目标工作树的实际工具链生成镜像，记录大小、SHA-256 和构建日志。
3. **JTAG 烧录验证**：重新核对芯片、端口、镜像、地址和偏移，记录 OpenOCD 结果；`Verify OK` 只证明写入校验。
4. **UART0 运行验证**：从启动日志确认 `nsh>`，执行目标命令、保存完整原始日志，并单独验证 Ctrl+C 返回 `nsh>`。
5. **LCD/触摸视觉验证**：只有实体屏幕观察到画面并实际点击后，才能标记视觉或触摸闭环通过。

## Function-EV 约束

- 默认工作树是 `openvela-426-l0`；不要访问另一个历史 bring-up 工作树。
- 保留 GT911、LVGL touchscreen、SC2336/V4L2/CSI 和 Ctrl+C 既有修改，不使用 reset、checkout、clean 或删除来“整理”现场。
- `lvgldemo widgets` 是回归基线；新增 dashboard/出库台不能改变它的入口和退出行为。
- 出库台当前是纯 LVGL 模拟业务，不得把 Camera `BLOCKED`、真实车辆动作、音频或 RTC 读数写成已实现硬件功能。

## 输出格式

用表格列出每个能力的“源码 / 构建 / JTAG / UART / 视觉”状态，并为 PASS 给出日志路径；缺少实体观察时明确写 `PENDING`。对失败尝试保留原始日志，说明它发生在烧录前还是运行中。
