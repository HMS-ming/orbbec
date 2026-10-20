# 02 SDK 示例上手（Windows）

> **位置**：奥比中光深度相机通用上手 WIKI · Windows 平台
>
> **前置**：[Windows / 01 下载安装与准备](01_下载安装与准备.md)（SDK 已安装、元数据注册已完成）
>
> **相关**：[通用 / 01 产品准备](../通用/01_产品准备.md)（设备准备与连接）
>
> **本步目标**：了解 `examples\` 中提供的示例 → 编译示例 → 运行示例确认设备可用
>
> **示例环境**：示例设备 **Astra 2**，安装包 **OrbbecSDK v2.10.6**；下文的 `<安装目录>` 即该版本的安装位置

## 0. 结论

> ✅ SDK 示例源码位于 **`<安装目录>\examples\src\`**，使用安装目录自带的一键构建脚本编译：

```powershell
cd "<安装目录>"
.\build_examples.ps1
```

编译产物输出到 **`bin\`** 目录，可执行文件名为 `ob_enumerate.exe`、`ob_imu.exe` 等。

本文以两个示例为主线：**`0.basic.enumerate`** 确认设备与流参数（不依赖 OpenCV，见第 4 节）；**`0.basic.quick_start`** 跑通视频流（需 OpenCV，见第 5 节）。

## 1. 示例程序一览

`examples\src\` 下共 **37 个编号示例目录**，另有 C 示例（`c_examples\`）与 LiDAR 示例（`lidar_examples\`）。按用途分组如下：

| 用途       | 示例目录 → 生成的可执行文件                                                                                           | 功能说明                                                                              |
| -------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **设备信息** | `0.basic.enumerate` → `ob_enumerate.exe`                                                                  | 列出设备型号 / PID / 序列号 / 接口，以及各传感器支持的格式、分辨率、帧率                                        |
| 基础取流     | `0.basic.quick_start` → `ob_quick_start.exe`                                                              | 以默认配置取流，并开窗口显示                                                                    |
| 综合用法     | `3.advanced.common_usages`                                                                                | 查看设备信息、设置参数并显示视频流的综合示例                                                            |
| 单流取流     | `1.stream.depth` · `1.stream.color` · `1.stream.infrared` · `1.stream.confidence` · `1.stream.decimation` | 获取单一路数据流并显示                                                                       |
| 取流（无窗口）  | `1.stream.imu` → `ob_imu.exe`                                                                             | 获取 IMU（加速度计与陀螺仪）数据并打印到控制台                                                         |
| 回调式取流    | `1.stream.callback`                                                                                       | 在回调函数中处理帧数据（自定义处理时推荐的方式）                                                          |
| 多流       | `1.stream.multi_streams`                                                                                  | 同时开启多路数据流                                                                         |
| 设备控制     | `2.device.control`                                                                                        | 激光开关、激光强度、白平衡等参数设置                                                                |
| 设备预设     | `3.advanced.preset`                                                                                       | 读取与设置设备预设值                                                                        |
| 网络设备     | `2.device.forceip`                                                                                        | 通过 ForceIP 命令为网络设备配置 IP 地址                                                        |
| 录制与回放    | `2.device.record`（带窗口）· `2.device.record.nogui`（命令行）                                                      | 将数据流录制为 ROS bag（`.bag`）；`2.device.playback` 用于回放                                  |
| 固件升级     | `2.device.firmware_update` · `2.device.optional_depth_presets_update`                                     | 读取 BIN 文件升级固件 / 预设                                                                |
| 热插拔      | `2.device.hot_plugin`                                                                                     | 设备热插拔回调与断流处理                                                                      |
| 对齐与同步    | `3.advanced.sync_align` · `3.advanced.hw_d2c_align`                                                       | 深度与彩色流对齐、同步                                                                       |
| 后处理      | `3.advanced.post_processing` · `3.advanced.enhanced_depth_filter` · `3.advanced.hdr`                      | 滤波、深度增强、HDR 合成                                                                    |
| 激光交错     | `3.advanced.laser_interleave`                                                                             | 开启 / 关闭激光帧交错与 SequenceId 滤波                                                       |
| **点云**   | `3.advanced.point_cloud` → `ob_point_cloud.exe`                                                           | 生成点云并保存为 `.ply` 文件                                                                |
| 坐标变换     | `3.advanced.coordinate_transform`                                                                         | 不同坐标系之间的转换                                                                        |
| 多设备      | `3.advanced.multi_devices` · `3.advanced.multi_devices_sync`                                              | 多相机同时取流与同步                                                                        |
| 多设备硬件触发  | `3.advanced.multi_devices_sync_gmsltrigger`                                                               | 通过 `/dev/camsync` 向 GMSL 设备发送硬件同步信号（**仅 Linux / GMSL 平台，需 NVIDIA Xavier / Orin**） |
| 图像保存     | `4.misc.save_to_disk`                                                                                     | 将帧数据保存到磁盘                                                                         |
| 日志与元数据   | `4.misc.logger` · `4.misc.metadata`                                                                       | 设置日志级别与输出路径；逐流读取元数据                                                               |
| 第三方库集成   | `5.wrapper.opencv` · `5.wrapper.pcl` · `5.wrapper.open3d`                                                 | 与 OpenCV / PCL / Open3D 的集成示例（PCL 与 Open3D 示例需在 CMake 中开启对应选项，默认不构建）              |

> **OpenCV 为可选依赖**：未安装时，依赖它的示例不会生成，构建过程会给出 `OpenCV not found, some examples may not be built …` 警告。安装要求见第 2 节，示例缺失时的处理见第 4 节。
>
> `utils\` 为示例公用工具（带窗口显示的 `CVWindow` 位于 `utils_opencv.*`）。各示例的代码说明见其所在目录下的 `README.md`，总览见 `examples\src\README.md`。

## 2. 工具链要求

| 组件                | 要求                                                                                                            |
| ----------------- | ------------------------------------------------------------------------------------------------------------- |
| **Visual Studio** | 2017 / 2019 / 2022 任一版本，需包含 C++ 生成工具。仅需编译示例时，安装「**Visual Studio 生成工具**」并勾选「**使用 C++ 的桌面开发**」工作负载即可，无需安装完整 IDE |
| **CMake**         | 需可在命令行中直接调用。安装时勾选 **Add CMake to the system PATH**，或安装后手动添加到 PATH                                             |
| **OpenCV**        | 可选。仅带窗口显示的示例需要，且必须是 **C++ 版本**（安装目录中含 `OpenCVConfig.cmake`）。若安装在非标准位置，需设置 `OpenCV_DIR` 环境变量供 CMake 定位         |

> ⚠️ 两类常见混淆：
>
> - **Visual Studio Code 与 Visual Studio 不是同一产品**。前者为代码编辑器，不包含 C++ 编译器，不能替代 Visual Studio 或 Visual Studio 生成工具。
> - **Python 版 OpenCV 与 C++ 版 OpenCV 不通用**。通过 `pip` 安装的 `opencv-python` 仅供 Python 使用，无法满足 C++ 示例的编译需求。

**环境检查**：以下两条命令均需有输出，方满足构建要求。

```powershell
cmake --version
& "C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
```

> 第二条无任何输出，表示未安装 Visual Studio 或缺少 C++ 生成工具组件。若提示 `vswhere.exe` 不存在，表示本机未安装过 Visual Studio 安装程序。

**安装指引**

| 组件                | 下载位置                                                                                                                             | 安装注意事项                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Visual Studio** | [Visual Studio 下载页](https://visualstudio.microsoft.com/zh-hans/downloads/) → 「Visual Studio 2022 生成工具」（需完整 IDE 可选择 Community 版本） | 在安装器中勾选「**使用 C++ 的桌面开发**」工作负载（含 MSVC 编译器与 Windows SDK）。未勾选该工作负载时，构建脚本仍会报找不到编译器 |
| **CMake**         | [CMake 下载页](https://cmake.org/download/) → Windows x64 Installer                                                                 | 安装向导中勾选 **Add CMake to the system PATH**                                       |

> ⚠️ 安装完成后需**关闭并重新打开终端**，PATH 的变更仅对新打开的终端生效。

**构建脚本报错对照**（按脚本内的检查顺序排列）

| 报错信息                                                    | 原因与处理                                                                                                                |
| ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `No Visual Studio with the C++ build tools was found.`  | 未检测到包含 C++ 工具的 Visual Studio。安装 Visual Studio 生成工具或 Community 版本并勾选「使用 C++ 的桌面开发」；如已安装，可通过 Visual Studio 安装程序补装该工作负载 |
| `CMake was not found in PATH. Install CMake and retry.` | 命令行中没有可用的 `cmake`。安装 CMake 并勾选加入 PATH，或重新打开终端后重试                                                                     |
| `vswhere.exe was not found.`                            | 本机没有 Visual Studio 安装程序，表示从未安装过 Visual Studio，按上表安装                                                                  |

> 也可通过 `-Generator` 参数跳过自动探测，例如 `.\build_examples.ps1 -Generator "Visual Studio 17 2022"`。该参数仅跳过探测步骤，**编译器本身仍须已安装**。

## 3. 构建示例

```powershell
cd "<安装目录>"
.\build_examples.ps1
```

脚本执行内容：探测 Visual Studio 版本 → 使用 CMake 配置 `examples\` → 编译 → 将可执行文件安装到 `bin\` → 将 `tools\extensions\` 复制到 `bin\`（免去手动复制动态库）→ 删除临时目录 `examples\build\`。

| 参数               | 默认值       | 说明                                             |
| ---------------- | --------- | ---------------------------------------------- |
| `-Configuration` | `Release` | 也可指定 `Debug` / `RelWithDebInfo` / `MinSizeRel` |
| `-Architecture`  | `x64`     | 仅支持 x64                                        |
| `-Generator`     | 自动探测      | 需要时手动指定，如 `-Generator "Visual Studio 17 2022"` |

编译完成后，`bin\` 目录中将生成 `ob_enumerate.exe`、`ob_imu.exe` 等文件。

> 手动构建（不使用脚本）：
>
> ```powershell
> cmake -S examples -B examples/build
> cmake --build examples/build --config Release
> cmake --install examples/build --config Release --prefix .
> ```
>
> `cmake --install` 不可省略。若省略，可执行文件将保留在 `examples\build\bin`，该目录中不含 `OrbbecSDK.dll`，运行时将报缺少动态库。

## 4. 运行示例：ob_enumerate

**功能说明**：列出已连接的设备（型号 / PID / 序列号 / 接口类型），并可按提示选择设备与传感器，列出该传感器支持的全部流格式。

**操作步骤**

1. 连接设备，确认设备管理器中可见（元数据注册应已完成，见 [01 第 5 节](01_下载安装与准备.md)）。
2. 进入 `bin\` 目录运行（**必须在 `bin\` 目录下运行**，动态库与可执行文件位于同一目录）：

```powershell
cd "<安装目录>\bin"
.\ob_enumerate.exe
```

1. 按提示输入设备序号并回车，列出该设备的传感器。
2. 输入传感器序号并回车，列出该传感器支持的流格式。
3. 按 `ESC` 逐层返回或退出程序。

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

> 上述输出可用于确认设备支持的数据流参数（格式、分辨率、帧率），是编写取流程序时选择参数的依据。  
> `connection type` 显示 `USB3.0` 时，表示连接速率符合推荐要求。

**故障处理**

| 现象                     | 处理                                                                                                        |
| ---------------------- | --------------------------------------------------------------------------------------------------------- |
| 提示 `No device found!`  | 设备未连接或未被识别。参见 [01 第 4 节](01_下载安装与准备.md) 或 [通用 / 01 产品准备](../通用/01_产品准备.md) 第 4、5 节排查                      |
| 找不到某个 `ob_*.exe`       | 构建未成功，或该示例依赖 OpenCV 而本机未安装。重新执行第 3 节的构建流程，并检查输出中是否有 `OpenCV not found …` 警告；如需要该示例，请安装 C++ 版 OpenCV 后重新构建 |
| 运行时报缺少 `OrbbecSDK.dll` | 未在 `bin\` 目录下运行。进入 `bin\` 目录后重新运行，或将 `bin` 目录添加到 PATH                                                     |

## 5. 跑通视频流：ob_quick_start

`ob_enumerate` 只确认「设备在、支持哪些流」；**真正取流**用入门示例 `0.basic.quick_start`（`ob_quick_start.exe`，需已安装 OpenCV 才会生成）。

**它做什么**（依据示例源码）：以默认配置启动 Pipeline（数据流管线，默认开启深度与彩色流），创建一个 **1280×720、标题为 QuickStart** 的窗口，循环等待复合帧并显示 —— 即**实时显示深度与彩色画面**；关闭窗口即停止数据流并退出程序。

**获取视频流的关键代码**（摘自 `quick_start.cpp`，完整文件位于 `<安装目录>\examples\src\0.basic.quick_start\`）：

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

> ①②④⑥ 是 **SDK 的关键函数**：创建管线 → 启动数据流 → 等帧 → 停止。③⑤ 使用的 `ob_smpl::CVWindow` 是**示例自带的显示工具**（位于 `examples\src\utils\`，基于 OpenCV 实现），**不属于 SDK API** —— 自己的程序里可以用任意方式处理 `frameSet` 中的帧数据。
>
> 这段取流代码在 **Windows 与 Ubuntu 上完全一致**（同一份示例源码）—— SDK 的取流接口是跨平台的，两个平台的差异只在编译方式与运行时环境（见各自平台 01）。

**运行**（仍在 `bin\` 目录）：

```powershell
.\ob_quick_start.exe
```

> 运行后应弹出标题为 **QuickStart** 的窗口，按单行排列显示深度与彩色画面；**关闭窗口即退出程序**。
>
> ⚠️ 刚启动的前一两帧，**深度或彩色可能只有其一**（两个传感器的就绪有先后，缺哪个并不固定），属正常现象 —— 自写程序时不要把「某一类帧为空」当作错误处理。
>
> 默认开启的数据流可在 SDK 配置文件中调整（`bin\OrbbecSDKConfig.xml`）。
>
> ⚠️ 该示例依赖 **OpenCV 与桌面环境**：未安装 OpenCV 时不会生成；无显示环境下无法运行 —— 无窗口取流可改用 `2.device.record.nogui`（把数据流录制为 `.bag` 文件），控制台数据可运行 `1.stream.imu`。

## 6. 其他示例

| 使用目的                  | 示例                                                                                                    |
| --------------------- | ----------------------------------------------------------------------------------------------------- |
| 查看深度 / 彩色 / 红外图像（带窗口） | `0.basic.quick_start` · `1.stream.depth` / `.color` / `.infrared`                                     |
| 获取数据（无窗口输出）           | `1.stream.imu`（控制台输出）· `3.advanced.point_cloud`（导出 `.ply` 文件）· `2.device.record.nogui`（录制为 `.bag` 文件） |
| 编写自有取流程序              | 参考 `examples\src\README.md` 与各示例目录下的 `README.md`（含逐段代码说明）；API 头文件位于 `include\libobsensor\`            |

> 完整清单（各示例的功能说明）见第 1 节。  
> 示例程序均使用 `ob::Error` 异常进行错误处理。运行出错时，控制台会输出 `function` / `args` / `message` / `type` 四项信息，可据此定位问题。

## 7. 后续步骤

- 使用 SDK 编写取流程序 → [03 SDK 自写取流](../通用/03_SDK自写取流.md)
- 其他平台 → [Ubuntu · 01 下载安装与准备](../Ubuntu平台/01_下载安装与准备.md) · [Ubuntu · 02 SDK 示例上手](../Ubuntu平台/02_SDK示例上手.md)

---

版本：v1.0（2026-10-16）
