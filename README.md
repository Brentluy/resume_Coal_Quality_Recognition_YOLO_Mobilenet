# Introduction

Coal Quality Inspection — End-to-End Vision Pipeline (YOLO Detection + MobileNetV2 Classification + Flask API)
## English
This project reconstructs a full production-grade coal-quality inspection system originally deployed in a mining environment. Because the real project is under NDA, all experiments here are reproduced using publicly available coal imagery datasets—but the engineering design, modeling steps, and reasoning strictly follow the real pipeline.

The goal is straightforward: **automate the entire process of detecting coal on conveyor belts and estimating its quality in real time**, replacing inconsistent human inspection with a fast, stable, and reproducible computer-vision workflow.

Coal images captured in mines suffer from extreme variability:
- underground images are often very dark  
- conveyor-belt images are sometimes overexposed  
- dust on lenses, vibration, and shadows introduce noise  
- lighting changes constantly across shifts and mining zones  

To address these challenges, the system is designed as a **two-stage vision pipeline**:

### **1. YOLO (v5 + v8) for coal-region detection**  
We first train YOLO models to locate all coal/gangue regions on each conveyor-belt image.  
This step transforms a complex full-image problem into clean ROIs (Regions of Interest), enabling robust downstream classification.  
Both YOLOv5 and YOLOv8 versions are included:
- VOC → YOLO label conversion  
- train/val/test dataset construction  
- lightweight, high-speed YOLOxN models for real-time usage  

### **2. MobileNetV2 for coal-quality classification**  
Once ROIs are extracted, each crop is classified into four coal-quality levels using a MobileNetV2-based model.  
Because quality is mainly a fine-grained texture task, extensive preprocessing and controlled experiments were necessary:
- CLAHE normalization for uneven lighting  
- edge-enhanced variants for ablation  
- backbone comparison under 100-sample stress tests  
- freeze vs. finetune transfer-learning analysis  
- scaling from 100 → 500 → 5000 samples to evaluate robustness  

Across all experiments, **MobileNetV2 (frozen feature extractor)** consistently provided the best balance of accuracy, stability, and deployment latency.

### **3. Flask API for real-time inference**  
Finally, the YOLO detector and MobileNetV2 classifier are connected into a single stateless REST service:
- `POST /predict` accepts an image  
- YOLO detects all coal regions  
- each ROI is classified by MobileNetV2  
- results are aggregated into a structured JSON response  
- outputs include box locations, per-ROI quality, and overall coal-quality distribution  

Model weights are loaded once at startup, keeping inference responsive (<1s) on a small EC2 instance or edge server.

---

## 中文
本项目完整复现了一个在真实矿场环境中部署过的**煤质智能检测系统**。  
由于原始项目受 NDA 限制，这里使用公共数据集重新跑完整流程，但所有的建模逻辑、工程设计、对比实验和推理架构都与真实生产版本一致。

系统的核心目标是：  
**自动化识别传送带上的煤块，并实时判断煤质等级，替代人工检测的主观性与不稳定性。**

矿区图像具有大量噪声与光照问题：
- 井下环境极暗  
- 皮带区域有时严重过曝  
- 镜头常被灰尘覆盖  
- 机械阴影频繁出现  
- 光照随班次和区域剧烈变化  

为解决上述难题，我们将任务拆成一个 **双阶段视觉系统**：

### **1. YOLO（v5 + v8）进行煤块区域检测**  
首先训练 YOLO 检测模型，定位每张图中的煤/矸石区域。  
这一步的作用是把复杂的整图问题转化成干净的 ROI 裁剪任务，为分类器提供稳定输入。  
本项目同时提供 YOLOv5 与 YOLOv8 的训练流程：
- VOC → YOLO 标签转换  
- 构建 train/val/test 标准数据集  
- 使用 nano/small 模型支持实时部署  

### **2. MobileNetV2 进行煤质分类**  
检测出的 ROI 会被送入 MobileNetV2 分类网络，判断煤块属于四种质量等级。  
由于煤质判断高度依赖**细粒度纹理特征**，我们进行了大量对照实验：
- CLAHE 光照归一化  
- 边缘增强的结构消融  
- 100 张小样本下的 Backbone 压测  
- freeze vs finetune 的迁移学习策略对比  
- 100 → 500 → 5000 数据规模下的全链路性能评估  

最终，**冻结特征层的 MobileNetV2** 在稳定性、推理速度、模型体积与准确率之间达成了最佳平衡。

### **3. Flask API 实现端到端实时推理服务**  
最终将 YOLO + MobileNetV2 封装为一个无状态的 REST 服务：
- `POST /predict` 传入图片  
- YOLO 检测煤块位置  
- 对每个 ROI 做 MobileNetV2 分类  
- 聚合成结构化 JSON 输出  
- 包含每个框的预测结果与整图的总体煤质分布  

模型在服务启动时一次性加载，推理过程保持轻量，延迟通常在 1 秒以内，非常适合 EC2 或边缘设备部署。


