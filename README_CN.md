# gdgs: Godot Gaussian Splatting

维护者：ReconWorldLab

[English README](README.md)

当前插件版本：`1.1.0`

`gdgs` 是一个基于 `CompositorEffect` 和 compute shader 的 Godot 4 Gaussian Splatting 插件。

它可以导入受支持的 3D Gaussian Splat 资源，通过 `GaussianSplatNode` 放入场景，并结合场景深度与常规 3D 内容进行合成。

## 演示

![演示截图](image.png)

- 视频演示：[Bilibili - BV1NRwFzYEVc](https://www.bilibili.com/video/BV1NRwFzYEVc)

## 功能特性

- 支持导入 `.ply`、`.compressed.ply`、`.splat` 和 `.sog` 格式的 Gaussian 资源。
- 将不同输入格式统一转换为 GPU 可直接使用的 Gaussian 资源。
- 导入时默认对 Gaussian 数据做归中处理。
- `GaussianSplatNode` 默认附带 `-180` 度的 Z 轴旋转。
- 支持在同一场景中渲染一个或多个 `GaussianSplatNode`。
- 通过 `WorldEnvironment.compositor` 与标准 Godot 3D 场景进行合成。
- 基于场景深度缓冲进行遮挡混合。
- 支持编辑器内预览和 gizmo 操作。
- 内置 alpha、颜色、GS 深度、场景深度和深度剔除遮罩等调试视图。

## 环境要求

- Godot `4.4` 或更新版本。
- 使用 `Forward Plus` 渲染后端。
- 支持 compute shader 的桌面 GPU 和驱动。
- 一份受支持格式的 Gaussian 资源文件。

## 安装方法

1. 如果你的 Godot 项目里还没有 `addons` 目录，先创建它。
2. 将本仓库中的 `gdgs` 文件夹复制到项目中，目标路径为 `addons/gdgs`。
3. 用 Godot 打开项目。
4. 进入 `Project > Project Settings > Plugins`。
5. 启用 `gdgs` 插件。

安装完成后，插件根目录应位于 `res://addons/gdgs`。

## 快速开始

1. 将一个受支持的 Gaussian 资源加入项目。本仓库附带了 `demo.ply`、`demo.compressed.ply` 和 `demo.sog` 作为示例。
2. 等待 Godot 将其导入为资源。
3. 在场景中添加一个 `GaussianSplatNode`。
4. 将导入后的资源赋值给 `GaussianSplatNode` 的 `gaussian` 属性。
5. 在场景中添加一个 `WorldEnvironment` 节点。
6. 在 `WorldEnvironment.compositor` 上创建一个 `Compositor` 资源。
7. 在该 `Compositor` 中添加一个 `CompositorEffect`，并将脚本设为 `res://addons/gdgs/postprocess.gd`。
8. 运行场景。

## 场景说明

- `GaussianSplatNode` 只负责保存变换和资源引用，实际渲染由 compositor pass 完成，不走 Godot 标准 mesh 渲染管线。
- 支持多个 `GaussianSplatNode` 同时存在，并在同一个 Gaussian pass 中统一渲染。
- 导入后的 Gaussian 数据会按平均位置做归中处理，因此默认更接近场景原点。
- `GaussianSplatNode` 默认带有 `-180` 度的 Z 轴旋转。如果你的源数据方向已经正确，可以在放入场景后自行调整节点变换。
- 如果你替换了源资源文件内容，请在 Godot 中重新导入，以确保生成资源与源文件保持同步。

## 后处理参数

compositor effect 脚本位于 `res://addons/gdgs/postprocess.gd`。

- `alpha_cutoff`：alpha 低于该阈值的像素会在最终合成时被忽略。
- `depth_bias`：GS 深度与场景深度比较时使用的小偏移量。
- `depth_test_min_alpha`：只有当 GS alpha 高于该阈值时才应用深度剔除。
- `debug_view`：调试输出模式。

`debug_view` 可选项：

- `Composite`：最终合成结果。
- `GS Alpha`：Gaussian alpha 缓冲。
- `GS Color`：Gaussian 颜色缓冲。
- `GS Depth`：Gaussian 深度缓冲。
- `Scene Depth`：场景深度缓冲。
- `Depth Reject Mask`：显示哪些 GS 像素因为深度测试被剔除。

## 支持的格式

### 标准 Gaussian `.ply`

导入器支持二进制小端的 Gaussian Splat `.ply` 文件，要求至少包含以下属性：

- 位置：`x`、`y`、`z`
- DC 颜色系数：`f_dc_0`、`f_dc_1`、`f_dc_2`
- 剩余 SH 系数：`f_rest_0` 到 `f_rest_44`
- 不透明度：`opacity`
- 缩放：`scale_0`、`scale_1`、`scale_2`
- 旋转：`rot_0`、`rot_1`、`rot_2`、`rot_3`

### `.compressed.ply`

- 通过独立的 compressed PLY 解码器导入。
- 可以通过 `.compressed.ply` 后缀或压缩顶点属性自动识别。

### 旧版 `.splat`

- 支持较早期的 Gaussian Splat record 格式资源。

### `.sog`

- 当前支持 SOG `v2` 归档格式。

该导入器面向 Gaussian Splatting 风格资源，不适用于通用点云文件。

## 仓库结构

- `gdgs/`：仓库中的插件根目录。复制到 Godot 项目后应位于 `addons/gdgs`。
- `gdgs/gaussian`：导入器、解码器和 Gaussian 资源定义。
- `gdgs/node`：场景节点和编辑器 gizmo。
- `gdgs/rendering`：渲染管理器、渲染上下文和 compute shader。
- `gdgs/postprocess.gd`：compositor effect 入口。
- `demo.ply`：标准 Gaussian PLY 示例资源。
- `demo.compressed.ply`：compressed Gaussian PLY 示例资源。
- `demo.sog`：SOG 示例资源。

## 已知限制

- 当前仅面向桌面端 `Forward Plus` 渲染。
- 依赖 Godot 的 compositor 与 compute 管线，因此不支持 compatibility 和 mobile 渲染器。
- 当前渲染管理器仍以共享的 root 级运行时管理器存在，复杂的编辑器多场景或多视口工作流仍需要进一步验证。
- 标准 `.ply` 仅支持 Gaussian Splat 所需的二进制小端布局，不支持任意点云属性结构。
- `.sog` 当前仅支持 `v2` 格式。

## 致谢

- 本项目中的 shader 实现参考了 [2Retr0/GodotGaussianSplatting](https://github.com/2Retr0/GodotGaussianSplatting)。感谢 2Retr0 公开该项目。
- 上游 `2Retr0/GodotGaussianSplatting` 仓库采用 MIT License。若你复用与其实现密切相关的衍生内容，请同时检查并保留相应的上游许可说明。
- radix sort 相关 shader 文件也保留了各自的上游来源说明，详见对应 shader 文件头部注释。

## 参考资料

- [2Retr0/GodotGaussianSplatting](https://github.com/2Retr0/GodotGaussianSplatting)
- [3D Gaussian Splatting for Real-Time Radiance Field Rendering](https://arxiv.org/abs/2308.04079)

## 许可证

本项目采用 [MIT License](LICENSE)。
