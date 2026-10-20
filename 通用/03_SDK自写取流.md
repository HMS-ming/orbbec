# 03 SDK 自写取流

> **位置**：奥比中光深度相机通用上手 WIKI · 通用篇（取流代码与平台无关，Windows / Ubuntu 通用）
>
> **前置**：任一平台的 [02 SDK 示例上手](../Windows平台/02_SDK示例上手.md)（以能编译运行为准）
>
> **相关**：[Windows / 02](../Windows平台/02_SDK示例上手.md) · [Ubuntu / 02](../Ubuntu平台/02_SDK示例上手.md)（两篇 §5 的取流代码与本文一致）
>
> **本步目标**：不依赖示例程序，**写一个自己的取流程序**，拿到深度与彩色帧数据
>
> **示例环境**：示例设备 **Astra 2**，安装包 **OrbbecSDK v2.10.6**；下文的 `<SDK 安装目录>`（Ubuntu）/ `<安装目录>`（Windows）即该版本的安装位置

## 0. 结论

✅ 取流只需要四步 API（与各平台篇 [02](../Windows平台/02_SDK示例上手.md) §5 的 `quick_start` 相同，代码完全一致）：

```cpp
ob::Pipeline pipe;                          // ① 创建数据流管线
pipe.start();                               // ② 启动数据流
auto frameSet = pipe.waitForFrameset();     // ④ 等待一帧复合数据
pipe.stop();                                // ⑥ 停止
```

与示例的唯一差别在 **④ 与 ⑥ 之间**：`quick_start` 把帧送进窗口显示，**应用程序在这里处理帧数据**。

本文提供 **C++ 与 Python 两份等价实现**（同一逻辑，均为完整可运行程序）：

| 语言                                  | 章节    | 内容                 |
| ----------------------------------- | ----- | ------------------ |
| C++（`my_stream.cpp`）                | 第 1 节 | 代码、讲解、两平台编译命令与输出示例 |
| Python（`my_stream.py`，pyorbbecsdk2） | 第 2 节 | 安装前提、代码、运行         |
| 共同注意事项                              | 第 3 节 | 两种语言都适用的三个注意点      |
| 进阶方向                                | 第 4 节 | 面向机器人集成的参考示例       |

## 1. C++ 版本

### 1.1 完整示例代码（my_stream.cpp）

```cpp
#include <libobsensor/ObSensor.hpp>
#include <iostream>

int main() try {
    // ① 创建数据流管线，② 以默认配置启动（默认开启深度 + 彩色）
    ob::Pipeline pipe;
    pipe.start();
    std::cout << "[1] Pipeline started (default config)" << std::endl;

    // ③ 循环取 10 帧复合数据
    for (int i = 0; i < 10; ++i) {
        // ④ 等待一帧复合数据（包含所有已开启的流；超时返回空指针）
        auto frameSet = pipe.waitForFrameset();
        if (frameSet == nullptr) {
            std::cout << "frame " << i << ": timeout" << std::endl;
            continue;
        }

        std::cout << "frame " << i << ":";

        // ⑤ 取深度帧：转为 DepthFrame（= 视频帧 + 深度值缩放）
        auto depth = frameSet->getFrame(OB_FRAME_DEPTH);
        if (depth) {
            auto d   = depth->as<ob::DepthFrame>();
            auto buf = static_cast<const uint16_t *>(d->data());   // Y16：每像素 2 字节
            uint32_t w = d->width(), h = d->height();
            // 深度值 = 像素值 × valueScale（结果为毫米，见头文件注释）
            float scale = d->getValueScale();
            uint16_t center = buf[(h / 2) * w + (w / 2)];
            std::cout << " depth " << w << "x" << h
                      << " | center " << center * scale << " mm";
        } else {
            std::cout << " depth: none";
        }

        // ⑥ 取彩色帧（启动初期可能还没有，见第 3 节注意事项）
        auto color = frameSet->getFrame(OB_FRAME_COLOR);
        if (color) {
            auto v = color->as<ob::VideoFrame>();
            std::cout << " | color " << v->width() << "x" << v->height();
        } else {
            std::cout << " | color: none";
        }
        std::cout << std::endl;
    }

    // ⑦ 停止数据流
    pipe.stop();
    std::cout << "done" << std::endl;
    return 0;
}
catch (ob::Error &e) {   // SDK 以异常报告错误，处理方式与 SDK 示例一致
    std::cerr << "function:" << e.getFunction() << "\nargs:" << e.getArgs()
              << "\nmessage:" << e.what() << "\nstatus:" << e.getStatus()
              << "\ntype:" << e.getExceptionType() << std::endl;
    return 1;
}
```

### 1.2 代码讲解

| 代码                                     | 说明                                                                                                                                 |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `ob::Pipeline pipe;` → `pipe.start();` | 创建管线并启动数据流。默认配置开启深度与彩色流，可在 SDK 配置文件中调整                                                                                             |
| `pipe.waitForFrameset()`               | 阻塞等待一帧**复合数据**（`FrameSet`，包含所有已开启流的帧）。**超时（默认 1000ms）会返回空指针**，必须先判空再解引用                                                            |
| `frameSet->getFrame(OB_FRAME_DEPTH)`   | 从复合帧中取指定类型的帧；该流未就绪时返回空                                                                                                             |
| `frame->as<ob::DepthFrame>()`          | ⚠️ `getFrame` 返回的是基类 `Frame` —— **转成 `DepthFrame`（继承自 `VideoFrame`）才有宽高、像素数据与 `getValueScale()`**；彩色帧转 `VideoFrame` 即可。用错子类会直接编译失败 |
| `d->data()`                            | 帧数据的内存地址（帧对象存活期间有效）。深度帧为 **Y16 格式**：每像素 2 字节，`uint16_t` 数组                                                                         |
| `d->getValueScale()`                   | **深度值 = 像素值 × valueScale（结果为毫米）**，见头文件注释。打印真实距离前先乘上它                                                                               |
| `catch (ob::Error &e)`                 | SDK 以异常报告错误。输出 `function` / `args` / `message` / `status` / `type` 五项，与 SDK 示例的处理方式一致                                              |

### 1.3 编译与运行

代码与平台无关，差别只在编译命令。头文件与库均在 SDK 安装目录下：Ubuntu 以 **`<SDK 安装目录>`** 表示（约定见 [Ubuntu / 01](../Ubuntu平台/01_下载安装与准备.md) 示例环境），Windows 以 **`<安装目录>`** 表示（约定见 [Windows / 01](../Windows平台/01_下载安装与准备.md) 示例环境）。

#### Ubuntu

```bash
g++ -std=c++11 -DOB_STATIC_DEFINE \
    -I<SDK 安装目录>/include my_stream.cpp \
    -L<SDK 安装目录>/lib -lOrbbecSDK \
    -Wl,-rpath,<SDK 安装目录>/lib \
    -o my_stream
./my_stream
```

> - `-DOB_STATIC_DEFINE`：使 SDK 头文件中的导出宏展开为空。该宏**未做平台区分**，Linux 下同样需要。
> - `-Wl,-rpath,<SDK 安装目录>/lib`：将库目录写入可执行文件的运行时搜索路径，加载 `libOrbbecSDK.so` 时无需设置 `LD_LIBRARY_PATH`。

#### Windows

```powershell
g++ -std=c++11 -DOB_STATIC_DEFINE -I<安装目录>\include my_stream.cpp <安装目录>\lib\OrbbecSDK.lib -o my_stream.exe
```

> 编译产物放到 `<安装目录>\bin\` 中运行（该目录已有 `OrbbecSDK.dll` 与 `extensions\`），或将这两个文件复制到可执行文件所在目录。  
> 使用 Visual Studio + CMake 亦可编译（工具链要求见 [Windows / 02](../Windows平台/02_SDK示例上手.md) 第 2 节）。

#### 输出示例

C++ 与 Python 两个版本的输出格式相同：

```text
[1] Pipeline started (default config)
frame 0: depth: none | color 1280x720            ← 启动初期，深度传感器尚未就绪
frame 1: depth 1600x1200 | center 0 mm | ...     ← 深度值 0 = 该像素无效（超出量程 / 无回波）
frame 3: depth 1600x1200 | center 1234 mm | ...  ← 对准量程内的物体后即为真实距离
...
done
```

> 分辨率与帧率取决于设备与默认配置，以实际输出为准；`center` 为画面中心点的距离测量值（毫米）。

## 2. Python 版本（pyorbbecsdk2）

同一取流逻辑的 Python 实现。**注意包名**：PyPI 上的 `pyorbbecsdk` 是 **SDK v1** 的绑定；**SDK v2 的绑定为 `pyorbbecsdk2`**，二者对应不同的 SDK 世代，注意区分。

### 2.1 安装前提

| 项      | 要求                                                     |
| ------ | ------------------------------------------------------ |
| Python | **3.8 – 3.13**（轮子覆盖 cp38 – cp313；⚠️ Python 3.14 暂无轮子）  |
| 平台     | Windows x64 · Linux x86_64 与 **aarch64** · macOS arm64 |
| 依赖     | `numpy`（读取帧数据所需）                                       |

```bash
pip install pyorbbecsdk2 numpy
```

> 轮子内已包含 SDK 动态库与 extensions，**无需另行安装 SDK，也无需复制 DLL** —— 导入时自动加载。这与 C++ 路线不同。

### 2.2 完整示例代码（my_stream.py）

```python
# my_stream.py —— 与 my_stream.cpp 相同逻辑的 Python 版本
# 四步取流：创建管线 → 启动 → 等帧 → 停止
import numpy as np
from pyorbbecsdk import OBError, Pipeline

try:
    # ① 创建数据流管线，② 以默认配置启动（默认开启深度 + 彩色）
    pipe = Pipeline()
    pipe.start()
    print("[1] Pipeline started (default config)")

    # ③ 循环取 10 帧复合数据
    for i in range(10):
        # ④ 等待一帧复合数据（超时返回 None）
        frames = pipe.wait_for_frames(1000)
        if frames is None:
            print(f"frame {i}: timeout")
            continue

        print(f"frame {i}:", end="")

        # ⑤ 取深度帧：读分辨率与中心点距离
        depth = frames.get_depth_frame()
        if depth is not None:
            w, h = depth.get_width(), depth.get_height()
            scale = depth.get_depth_scale()   # 深度值 = 像素值 × depth_scale（毫米）
            data = np.frombuffer(depth.get_data(), dtype=np.uint16).reshape(h, w)
            print(f" depth {w}x{h} | center {data[h // 2, w // 2] * scale:.0f} mm", end="")
        else:
            print(" depth: none", end="")

        # ⑥ 取彩色帧（启动初期可能还没有，见第 3 节注意事项）
        color = frames.get_color_frame()
        if color is not None:
            print(f" | color {color.get_width()}x{color.get_height()}", end="")
        else:
            print(" | color: none", end="")
        print()

    # ⑦ 停止数据流
    pipe.stop()
    print("done")
except OBError as e:   # SDK 以异常报告错误
    print("SDK error:", e)
```

### 2.3 运行

```bash
python my_stream.py
```

输出格式与 [1.3 的输出示例](#13-编译与运行)一致。

### 2.4 与 C++ 版的对应关系

| C++（1.1 / 1.2 节）                           | Python（本节）                                         |
| ------------------------------------------ | -------------------------------------------------- |
| `ob::Pipeline pipe;` + `pipe.start();`     | `pipe = Pipeline()` + `pipe.start()`               |
| `pipe.waitForFrameset()`                   | `pipe.wait_for_frames(1000)`（超时返回 `None`）          |
| `frameset->getFrame(OB_FRAME_DEPTH)`       | `frames.get_depth_frame()`                         |
| `as<ob::DepthFrame>()` + `getValueScale()` | `depth.get_depth_scale()`                          |
| `depth->data()`（Y16，`uint16_t` 数组）         | `np.frombuffer(depth.get_data(), dtype=np.uint16)` |
| `catch (ob::Error &e)`                     | `except OBError as e`                              |

## 3. 通用注意事项

以下三点对 C++ 与 Python 两种实现同样适用：

1. ⚠️ **刚启动的前一两帧，深度或彩色可能只有其一**（两个传感器的就绪有先后，缺哪个并不固定），属正常现象 —— 不要把「某一类帧为空」当作错误处理。
2. ⚠️ **深度值 0 表示该像素无效**（超出量程或无回波）。若画面中心恒为 0：可能超出量程或被遮挡，把相机对准 1～3 m 内的实际物体再观察。
3. ⚠️ **彩色帧通常是编码数据**（体积远小于同分辨率原始图像），拿到的是压缩字节流，不能直接当像素读；需要图像时先解码 —— SDK 示例是借助 OpenCV 完成的。深度帧（Y16）无此问题。

## 4. 进阶参考：面向机器人集成

本文拿到的是「单帧数据 + 控制台打印」。机器人程序中通常需要**持续取流并交给算法**，下一步可参考 SDK 示例目录中的这些示例（均附 `README.md`）：

| 需求          | 参考示例                                                      |
| ----------- | --------------------------------------------------------- |
| 持续取流、不阻塞主循环 | `1.stream.callback`（注册回调，帧到达即处理）                          |
| 深度图与彩色图对齐   | `3.advanced.sync_align` · `3.advanced.hw_d2c_align`       |
| 输出点云        | `3.advanced.point_cloud`（导出 `.ply`）                       |
| 数据落盘供离线分析   | `2.device.record.nogui`（录制 `.bag`）· `4.misc.save_to_disk` |

## 5. 后续步骤

- 任务②「在机器人中接入深度相机」的具体整合方式（与上位机 / 底盘控制器的数据通路）按项目需求另行展开
- 各平台环境与示例的上手 → [Windows / 01](../Windows平台/01_下载安装与准备.md) · [Ubuntu / 01](../Ubuntu平台/01_下载安装与准备.md)

---

版本：v1.1（2026-10-20）
