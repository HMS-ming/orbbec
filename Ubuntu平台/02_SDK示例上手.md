# 02 SDK 示例上手（Ubuntu）

> **位置**：奥比中光深度相机通用上手 WIKI · Ubuntu 平台
>
> **前置**：[Ubuntu / 01 下载安装与准备](01_下载安装与准备.md)（SDK 已安装、udev 规则已配置）
>
> **相关**：[Windows · 02 SDK 示例上手](../Windows平台/02_SDK示例上手.md)（示例集合相同）
>
> **本步目标**：编译 SDK 示例 → 运行示例确认设备可用
>
> **示例环境**：示例设备 **Astra 2**，安装包 **OrbbecSDK v2.10.6**；下文的 `<SDK 安装目录>` 即该版本的安装位置

## 0. 结论

> ✅ 示例随 SDK 一同安装，使用 SDK 自带的构建脚本编译：

```bash
cd <构建脚本所在目录>
./build_examples.sh
```

> 编译产物输出到 **`<SDK 安装目录>/bin/`**（含 C 与 C++ 两类示例），可执行文件名为 `ob_enumerate`、`ob_imu` 等（Linux 下不带扩展名）。
>
> 本文以 **`0.basic.enumerate`** 为例说明运行流程。该示例不依赖 OpenCV，可用于确认设备及各传感器支持的流参数。

## 1. 示例程序一览

SDK 的示例源码与 Windows 平台为**同一套**（按「层级.用途」编号，共 **37 个编号示例目录**，另有 C 示例与 LiDAR 示例），因此**完整清单（各示例的功能说明）见 [Windows / 02 SDK 示例上手](../Windows平台/02_SDK示例上手.md) 第 1 节**，本文不重复列出。

Linux 平台需注意以下两点：

| 说明 | 影响 |
| --- | --- |
| `3.advanced.multi_devices_sync_gmsltrigger` **仅适用于 Linux / GMSL 平台** | 该示例仅在 Linux 平台参与构建，且需支持 GMSL 的设备（如 NVIDIA Xavier / Orin 系列） |
| 带窗口显示的示例依赖 **OpenCV** | 未安装 OpenCV 时这些示例不会生成，构建过程会给出警告（其余示例不受影响） |

> 各示例的代码说明见其所在目录下的 `README.md`，总览见示例目录中的 `README.md`。

## 2. 工具链要求

| 组件 | 要求 |
| --- | --- |
| **GCC** | ≥ 7.5。可用系统包管理器安装构建工具链，如 `sudo apt install build-essential` |
| **CMake** | ≥ 3.15，需可在命令行中直接调用（`cmake --version` 有输出）。如 `sudo apt install cmake` |
| **OpenCV** | 可选。仅带窗口显示的示例需要，且必须是 **C++ 版本**（安装后含 `OpenCVConfig.cmake`）。可用系统包管理器安装，如 `sudo apt install libopencv-dev`；若安装在非标准位置，需设置 `OpenCV_DIR` 供 CMake 定位 |

**环境检查**：以下两条命令均需有输出，方满足构建要求。

```bash
gcc --version
cmake --version
```

## 3. 构建示例

示例源码**随 SDK 一同安装**，构建脚本由 SDK 提供（不在 GitHub 源码仓库中，可用 `dpkg -L orbbecsdk` 定位）：

```bash
dpkg -L orbbecsdk | grep -i build_examples
```

进入该脚本所在目录后运行：

```bash
cd <构建脚本所在目录>
./build_examples.sh
```

> 编译产物输出到 **`<SDK 安装目录>/bin/`**。**脚本参数以脚本自身的输出为准**（本文不列出参数）。
>
> ⚠️ 若因**权限不足**而失败：使用一体安装包方式时，SDK 安装目录位于 `/opt` 下、属 root 所有，此时可改用 `sudo ./build_examples.sh` 运行。

**故障处理**

| 现象 | 处理 |
| --- | --- |
| 提示 `cmake: command not found` | 未安装 CMake，参见第 2 节 |
| 提示找不到编译器 | 未安装 GCC 或构建工具链，参见第 2 节 |
| 构建输出中出现 `OpenCV not found …` 警告 | 未安装 OpenCV，依赖它的示例不会生成（其余示例不受影响） |
| 权限不足、无法写入 | 见上方提示，改用 `sudo` 运行脚本 |

## 4. 运行示例：ob_enumerate

**功能说明**：列出已连接的设备（型号 / PID / 序列号 / 接口类型），并可按提示选择设备与传感器，列出该传感器支持的全部流格式。

**操作步骤**

1. 连接设备（udev 规则应已配置完成，见 [01 第 4 节](01_下载安装与准备.md)）。
2. 进入 `bin/` 目录运行：

```bash
cd <SDK 安装目录>/bin
./ob_enumerate
```

> 使用一体安装包方式安装时，示例可**直接以普通用户运行**，无需 `sudo`，也无需额外设置库搜索路径。

3. 按提示输入设备序号并回车，列出该设备的传感器。
4. 输入传感器序号并回车，列出该传感器支持的流格式。
5. 按 `ESC` 逐层返回或退出程序。

**输出示例**

```text
enumerated devices:
  0- device name: <设备型号>, device pid: 0x<PID> ,device SN: <序列号>, connection type:<接口类型>
Select a device to enumerate its sensors (Input device index or 'ESC' to exit program):
Sensor list:
 - 0.sensor type: DEPTH
 - 1.sensor type: COLOR
 - 2.sensor type: IR
 - 3.sensor type: ACCEL
 - 4.sensor type: GYRO
1.format: Y16, res: 640*480, fps: 30
2.format: Y16, res: 848*480, fps: 30
```

> 该输出可用于确认设备支持的数据流参数（格式、分辨率、帧率），是编写取流程序时选择参数的依据。
> `connection type` 显示 `USB3.0` 时，表示连接速率符合推荐要求。

**故障处理**

| 现象 | 处理 |
| --- | --- |
| 提示 `No device found!` | 设备未连接或未被识别。参见 [01 第 4 节](01_下载安装与准备.md) 或 [通用 / 01 产品准备](../通用/01_产品准备.md) 第 4、5 节排查 |
| 找不到 `ob_enumerate` | 构建未成功。重新执行第 3 节的构建流程并检查输出信息 |
| 运行时报找不到 `libOrbbecSDK.so*` | 动态库未被加载器找到（使用一体安装包方式时一般不会出现）。可将库文件所在目录加入 `LD_LIBRARY_PATH` 后重试 |
| 提示权限不足、无法打开设备 | udev 规则未生效。参见 [01 第 4 节](01_下载安装与准备.md) 的故障处理 |

## 5. 其他示例

| 使用目的 | 示例 |
| --- | --- |
| 查看深度 / 彩色 / 红外图像（带窗口） | `0.basic.quick_start` · `1.stream.depth` / `.color` / `.infrared`（**需安装 C++ 版 OpenCV**） |
| 获取数据（无窗口输出） | `1.stream.imu`（控制台输出）· `3.advanced.point_cloud`（导出 `.ply` 文件）· `2.device.record.nogui`（录制为 `.bag` 文件） |
| 编写自有取流程序 | 参考各示例目录下的 `README.md`（含逐段代码说明）；API 头文件位于 SDK 安装目录内 |

> 完整清单（各示例的功能说明）见 [Windows / 02 SDK 示例上手](../Windows平台/02_SDK示例上手.md) 第 1 节。
> 示例程序均使用 `ob::Error` 异常进行错误处理。运行出错时，控制台会输出 `function` / `args` / `message` / `type` 四项信息，可据此定位问题。

## 6. 后续步骤

- 使用 SDK 编写取流程序 → `03_SDK自写取流.md`（待写）
- 其他平台 → [Windows · 02 SDK 示例上手](../Windows平台/02_SDK示例上手.md)

---

版本：v1.0（2026-10-16）
