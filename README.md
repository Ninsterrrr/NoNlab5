# NoNlab · B-S-T-A: Viewport Motion Forge
### (视口动作动画熔炉) - A Low-Level Unity Avatar Motion System

> Thank you for using NoN module set.

## 📖 项目简介
B-S-T-A 并不是一个简单的 BlendShape 格式转换器。它是一个专为 VRChat Avatar 打造、直接从底层重构 Unity 动画工作流的**独立动作熔炉**。为了解决原生 Unity 动画系统在调参时产生的预览卡顿、视口焦点冲突以及权重限制问题，本项目完全使用纯 C# 从零构建了全套 UI 状态机与离屏渲染管线。整个插件解压后仅有 **120KB**，零第三方依赖，拒绝一切代码臃肿。

## 技术架构 (Technical Architecture)

### 1. 彻底突破 BlendShape 极限 (PlayerSettings Bypass)
Unity 默认将 BlendShape 权重锁定在 `0-100`。本工具直接修改 `PlayerSettings.legacyClampBlendShapeWeights`。
*   **无限上限与负数形变**：支持自定义上限，并支持输入 **负数权重** 实现反向物理形变。
*   **±0.01 极高精度**：通过底层的 `Mathf.Round(current * 100f) / 100f` 截断，无视 UI 滑块的像素限制，实现小数点后两位的绝对精准。

### 独立离屏渲染视口 (Independent Offscreen Viewport)
在 EditorWindow 中手搓了一套微型 3D 渲染引擎：
*   基于 `RenderTexture`、独立 `Camera` 与 `Light` 构建专属预览管线。
*   支持 4x MSAA 抗锯齿。
*   **WASD+QE 空间漫游**：接管键盘输入，配合 `Vector3.Lerp` 进行相机焦点的平滑插值（支持 Shift 加速与 Ctrl 减速）。

### 底层原生面捕参数直读 (Native Face Tracking Injection)
无需第三方插件中转，直接在底层抓取并映射 VRChat 原生面捕参数（如 `vrc.v_sil`, `vrc.v_aa`, `vrc.v_oh` 等），并直接映射至底层滑块进行极限校准。

### 交互状态机 (UI State Machine)
放弃了 Unity 原生 GUI 的冗余判断，自研了高度隔离的状态机：
*   **Hover (悬停预览)**：鼠标滑过名称，瞬间读取并赋予 Max 权重，移开即恢复。
*   **Scrub (像素擦洗)**：直接接管鼠标 X 坐标，无视原生 Slider 的手柄物理限制，指哪打哪。
*   **Focus (延迟聚焦)**：悬停 1 秒后，自动调用视口相机平滑飞向目标 Mesh。
*   **互斥逻辑**：Scrub 激活时自动切断 Hover 与 Focus，防止动画冲突。全面支持 `Undo.RecordObject` 完整记录撤销栈。

## 自动化与生态 (Automation & Ecosystem)
*   **一键镜像同步**：智能正则识别 `.L` / `.R` 或 `_L` / `_R` 后缀，一键同步左右权重。
*   **一键生成 FX 控制器**：底层自动创建包含 `Idle` 与动画状态的 AnimatorController，并自动配置 Float 参数与 Greater/Less 过渡条件。
*   **一键恢复初始态**：注册渲染器时自动建立权重字典快照，随时回滚。
*   **五国语言引擎**：基于 `Dictionary` 缓存的中英日俄韩无缝热切换系统。
*   **自研主题引擎**：包含 Default, Dark, Light, Nord, Sakura 预设，支持全部 UI 元素动态自定义换色。

## ⚠️ 警告 (Fatal Warnings)
 **极端权重会导致 Mesh 炸裂！**
当你解除限制并赋予模型极端权重时，如果你的模型 Mesh 顶点没有提前刷好权重，**模型会发生极其严重的物理崩坏**。
**务必在操作前使用 `Ctrl+S` 保存场景。** 若动画片段为只读资产，工具将自动跳过并发出警告。

##  安装 (Installation)
直接将 `.unitypackage` 拖入 Unity 项目。
在顶部菜单栏进入：`NoNlab -> Blend Shape to Animation` 即可启动。
Booth↓
https://ninster.booth.pm/items/9083063

---
*Developed by NoNStudio - Ninster.*
