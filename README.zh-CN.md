# realman-pi05-banana 🍌

**从自己采集的 50 条示教开始，微调 π0.5，在 RealMan 上完成香蕉抓取、放盘、松爪和抬离。**

[English](README.md) · [实验详情](docs/experiment.md) · [流程与接口](docs/reproduction.md)

## 演示

![Expert-only 香蕉抓放演示，4 倍速](assets/expert_pick_place_4x.gif)

**[观看 / 下载 2 倍速 MP4 视频](assets/expert_pick_place_2x.mp4)**

视频为 **2×**，GIF 为 **4×**。播放倍速不代表机器人实际速度或推理性能。

## 本轮现场结果

| 微调方法 | 选用 checkpoint | 成功次数 | 成功比例 |
|---|---:|---:|---:|
| **Expert-only** | **step 4000** | **9/10** | **90%** |
| LoRA，r16 / alpha16 | step 2500 | 0/8 | 0% |

这是 **2026 年 11 月 8 日现场人工观察统计**。测试期间调整过执行器，所有尝试是否使用完全相同的代码版本和起始条件尚未核实。这组结果适用于本轮 checkpoint 与任务，不代表对两类微调方法的普遍结论。

目前使用 Expert-only 作为部署主线。现场还观察到它能适应香蕉位置的小幅变化；完成放盘后人为取走香蕉，它会继续尝试抓取。后者因时间不足未完成验证，暂作为定性现象记录。

## 实验配置

- **任务**：右臂夹起左侧香蕉，放入右侧蓝盘，松开并抬离；无需回到精确关节原点。
- **数据**：50 条成功示教，转换为 LeRobot；TRAIN 40 条 / 4672 帧，VAL 5 条 / 649 帧，TEST 5 条 / 658 帧。
- **输入**：上方、左腕、右腕三路 RGB，14 维双臂状态和固定任务文本。
- **输出**：14 维关节与夹爪目标，执行器只派发右臂部分。关节使用 degree，夹爪保留独立 scalar。
- **训练**：AMD gfx1100，暴露显存 47.98 GiB，ROCm 7.2。
- **部署**：RTX 3060 12 GB。

使用官方预训练 [π0.5 base](https://huggingface.co/lerobot/pi05_base) 做任务微调。Normalizer 仅由训练集计算，TEST 未用于选模型。

## 当前公开内容

本仓库首版包含演示媒体、实验记录和流程接口说明。原始示教、模型权重、实验室 LeRobot fork 和设备执行代码不在本次公开范围内。

下一步固定执行器和起始场景，分别测试固定位置、小幅位移以及物体被移走后的恢复，补齐任务完成与停止判据。LoRA 保留为本轮失败对照，继续离线分析。

## 致谢

基于 [Physical Intelligence OpenPI](https://github.com/Physical-Intelligence/openpi) 和 [Hugging Face LeRobot](https://github.com/huggingface/lerobot)。感谢实验室已有 RealMan、主从遥操作、数据采集及控制平台，以及提供现场协助的同学。
