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
> 本文以两个示例为主线：**`0.basic.enumerate`** 确认设备与流参数（不依赖 OpenCV，见第 4 节）；**`0.basic.quick_start`** 跑通视频流（需 OpenCV，见第 5 节）。

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

## 5. 跑通视频流：ob_quick_start

`ob_enumerate` 只确认「设备在、支持哪些流」；**真正取流**用入门示例 `0.basic.quick_start`（`ob_quick_start`，需已安装 OpenCV 才会生成）。

**它做什么**（依据示例源码）：以默认配置启动 Pipeline（数据流管线，默认开启深度与彩色流），创建一个 **1280×720、标题为 QuickStart** 的窗口，循环等待复合帧并显示 —— 即**实时显示深度与彩色画面**；关闭窗口即停止数据流并退出程序。

**获取视频流的关键代码**（摘自 `quick_start.cpp`，完整文件位于 SDK 示例目录 `examples/src/0.basic.quick_start/`，可用 `dpkg -L orbbecsdk | grep quick_start` 定位）：

```cpp
ob::Pipeline pipe;                 // ① 创建数据流管线（Pipeline）
pipe.start();                      // ② 以默认配置启动数据流（默认开启深度 + 彩色）

ob_smpl::CVWindow win("QuickStart", 1280, 720, ob_smpl::ARRANGE_ONE_ROW);
                                   // ③ 创建 1280×720 的显示窗口

while (win.run()) {
    auto frameSet = pipe.waitForFrameset();   // ④ 等待一帧复合数据（含所有已开启的流，超时 1000ms）
    win.pushFramesToView(frameSet);           // ⑤ 将帧送入窗口显示
}

pipe.stop();                       // ⑥ 停止数据流
```

> ①②④⑥ 是 **SDK 的关键函数**：创建管线 → 启动数据流 → 等帧 → 停止。③⑤ 使用的 `ob_smpl::CVWindow` 是**示例自带的显示工具**（位于 `examples/src/utils/`，基于 OpenCV 实现），**不属于 SDK API** —— 自己的程序里可以用任意方式处理 `frameSet` 中的帧数据。
>
> 这段取流代码在 **Windows 与 Ubuntu 上完全一致**（同一份示例源码）—— SDK 的取流接口是跨平台的，两个平台的差异只在编译方式与运行时环境（见各自平台 01）。

**运行**（仍在 `bin/` 目录）：

```bash
./ob_quick_start
```

> 运行后应弹出标题为 **QuickStart** 的窗口，按单行排列显示深度与彩色画面；**关闭窗口即退出程序**。
>
> ⚠️ 刚启动的前一两帧，**深度或彩色可能只有其一**（两个传感器的就绪有先后，缺哪个并不固定），属正常现象 —— 自写程序时不要把「某一类帧为空」当作错误处理。
>
> 默认开启的数据流由 SDK 配置文件决定。
>
> ⚠️ 该示例依赖 **OpenCV 与桌面环境**：未安装 OpenCV 时不会生成；无显示环境（纯 SSH 会话、无头模式）下无法运行 —— 无窗口取流可改用 `2.device.record.nogui`（把数据流录制为 `.bag` 文件），控制台数据可运行 `1.stream.imu`。

## 6. 其他示例

| 使用目的 | 示例 |
| --- | --- |
| 查看深度 / 彩色 / 红外图像（带窗口） | `0.basic.quick_start` · `1.stream.depth` / `.color` / `.infrared`（**需安装 C++ 版 OpenCV**） |
| 获取数据（无窗口输出） | `1.stream.imu`（控制台输出）· `3.advanced.point_cloud`（导出 `.ply` 文件）· `2.device.record.nogui`（录制为 `.bag` 文件） |
| 编写自有取流程序 | 参考各示例目录下的 `README.md`（含逐段代码说明）；API 头文件位于 SDK 安装目录内 |

> 完整清单（各示例的功能说明）见 [Windows / 02 SDK 示例上手](../Windows平台/02_SDK示例上手.md) 第 1 节。
> 示例程序均使用 `ob::Error` 异常进行错误处理。运行出错时，控制台会输出 `function` / `args` / `message` / `type` 四项信息，可据此定位问题。

## 7. 后续步骤

- 使用 SDK 编写取流程序 → [03 SDK 自写取流](../通用/03_SDK自写取流.md)
- 其他平台 → [Windows · 02 SDK 示例上手](../Windows平台/02_SDK示例上手.md)

---

版本：v1.0（2026-10-16）
