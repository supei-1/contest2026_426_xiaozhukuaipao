# ESP32-P4 Function-EV 镜像复现说明

本仓用于在另一台 Linux x86_64 电脑上重新拉取当前 Function-EV 源码和构建配置。它复现的是当前 `nuttx.bin` 的源码配置基线，不把 `build/`、`.repo/`、编译缓存或本地 SmartFS 数据当作源码提交。

## 当前基线

| 项目 | 值 |
| --- | --- |
| `nuttx` | `e0b3f6e640c196f2534e7fa8d4c1c81886bcf0f5` |
| `apps` | `b27aa57d9cc3c0315d3394e61c50974d5d71d1b3` |
| 配置快照 | `configs/esp32p4-function-ev-lvgl.config` |
| 工具链 | `riscv-none-elf-gcc 13.4.0` |
| 镜像 | `1,305,572 bytes` |
| 镜像 SHA-256 | `d6ed02a0d05ec212e7f3b7164f7e99b69b0ac450947fa58ba87af1de7f591aa3` |
| 应用偏移 | `0x2000` |
| 芯片 | `ESP32-P4` |

## 重新拉取

需要安装 `repo`、Git、Python 3 和基本 Linux 构建工具。使用本仓提交分支作为 manifest 仓：

```bash
repo init \
  -u https://github.com/supei-1/contest2026_426_xiaozhukuaipao \
  -b dev-ai-contest-2026 \
  -m openvela.xml
repo sync -c -j8
```

`openvela.xml` 已将 `nuttx` 和 `apps` 固定到上表提交，并将 OpenVela 组织的其他项目改为绝对 GitHub 地址，避免从个人 fork 解析相对 remote 失败。

## 构建当前镜像

在 manifest 工作区根目录执行：

```bash
export PATH="$PWD/prebuilts/build-tools/linux-x86_64/bin:$PWD/prebuilts/gcc/linux-x86_64/riscv-none-elf/bin:$PATH"
cp contest2026_426_xiaozhukuaipao/configs/esp32p4-function-ev-lvgl.config nuttx/.config
make -C nuttx olddefconfig
make -C nuttx -j"$(nproc)"
sha256sum nuttx/nuttx.bin
```

输出应生成 `nuttx/nuttx.bin`。如果 SHA-256 不同，先核对 `nuttx`/`apps` 提交、`riscv-none-elf-gcc --version`、主机架构以及是否重新生成过 `.config`；功能构建成功不自动等于字节级 SHA 一致。

## 烧录边界

源码仓只保证生成镜像。烧录仍需按项目 JTAG 流程核对实际芯片、USB-JTAG 序列号、端口和 `0x2000` 偏移；UART0 启动、LCD/触摸视觉和 SmartFS `/data` 内容属于目标板运行状态，不会由 Git 源码自动恢复。
