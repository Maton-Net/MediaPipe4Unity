# MediaPipe4Unity

```markdown
Plugin version 0.11.0, MediaPipe version 0.8.4
```

This plugin is supported by @Google

With this plugin, you can

- Write MediaPipe code in C#.
- Run MediaPipe's official solution on Unity.
- Run your custom `Calculator` and `CalculatorGraph` on Unity.

  **- :warning: Depending on the type of input/output, you may need to write C++ code.**

## :hammer_and_wrench: Installation

This repository does not contain the required libraries (e.g. `libmediapipe_c.so`, `Google.Protobuf.dll`, etc), so you **need to build them** first.

For a step-by-step guide, please refer to the [Installation Guide](https://github.com/matondotnet/VRigRoid.Plugins/blob/main/MediaPipe4Unity/Installation.md) on Wiki.

### Supported Platforms

> :warning: GPU mode is not supported on macOS and Windows.

|                            |       Editor       |   Linux (x86_64)   |   macOS (x86_64)   |   macOS (ARM64)    |  Windows (x86_64)  |
| :------------------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: |
|     Linux (AMD64) [^1]     | :heavy_check_mark: | :heavy_check_mark: |                    |                    |                    |
|         Intel Mac          | :heavy_check_mark: |                    | :heavy_check_mark: |                    |                    |
|        M1 Mac [^2]         | :heavy_check_mark: |                    |                    | :heavy_check_mark: |                    |
| Windows 10/11 (AMD64) [^3] | :heavy_check_mark: |                    |                    |                    | :heavy_check_mark: |

[^1]: Tested on Arch Linux.
[^2]: Experimental, because MediaPipe does not support M1 Mac.
[^3]: Running MediaPipe on Windows is [experimental](https://google.github.io/mediapipe/getting_started/install.html#installing-on-windows).

## :plate_with_cutlery: Try sample app

### Example Solutions

Here is a list of [solutions](https://google.github.io/mediapipe/solutions/solutions.html) that you can try in the sample app.

> :bell: The graphs you can run are not limited to the ones in this list.

|                         |    Linux (GPU)     |    Linux (CPU)     |    macOS (CPU)     |   Windows (CPU)    |
| :---------------------: | :----------------: | :----------------: | :----------------: | :----------------: |
|     Face Detection      | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|        Face Mesh        | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|          Iris           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|          Hands          | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|          Pose           | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|        Holistic         | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|   Selfie Segmentation   | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|    Hair Segmentation    | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|    Object Detection     | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|      Box Tracking       | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Instant Motion Tracking | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|        Objectron        | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
|          KNIFT          |                    |                    |                    |                    |

### UnityEditor

Select `Mediapipe/Samples/Scenes/Start Scene` and play.

### Desktop

If you've built native libraries for CPU (i.e. `--desktop cpu`), select `CPU` for inference mode from the Inspector Window.
![preferable-inference-mode](https://user-images.githubusercontent.com/4690128/134795568-156f3d41-b46e-477f-a487-d04c99300c33.png)
