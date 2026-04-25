---
title: "Hierarchical Codec Diffusion for Video-to-Speech Generation"
date: 2026-04-20
draft: false
tags: [语音合成, 扩散模型, 多模态模型, 零样本, 跨模态]
categories: [论文速递]
description: "本论文针对 Video-to-Speech（VTS）生成中视觉-语音模态信息不对称的问题，提出现有方法忽略了语音从粗粒度语义到细粒度韵律的层次结构，导致视觉条件无法与语音表示精准对齐。为此，作者提出 HiCoDiT（Hierarchical Codec Diffusion Transformer），"
hiddenInHomeList: true
---

# 📄 Hierarchical Codec Diffusion for Video-to-Speech Generation

#语音合成 #扩散模型 #多模态模型 #零样本 #跨模态

🔥 **评分：8.5/10** | [arxiv](https://arxiv.org/abs/2604.15923v1)


### 👥 作者与机构

- 第一作者：Jiaxin Ye（Fudan University）
- 通讯作者：Hongming Shan（Fudan University，hmshan@fudan.edu.cn）
- 其他作者：
  - Gaoxiang Cong（Institute of Computing Technology, Chinese Academy of Sciences；University of Chinese Academy of Sciences）
  - Chenhui Wang（Fudan University）
  - Xin-Cheng Wen（Harbin Institute of Technology (Shenzhen)）
  - Zhaoyang Li（Fudan University）
  - Boyuan Cao（Fudan University）

---

### 💡 毒舌点评

**亮点**：这篇论文像个严谨的“交通协管员”，终于把 RVQ 不同层级当成了不同的车道——让嘴唇和身份去底层飙内容，让表情去高层管情绪，治好了 VTS 领域长期存在的“视觉条件瞎注入”的拥堵病。

**槽点**：虽然口口声声“首个”层次化离散扩散，但骨子里是 SEDD + MaskGCT Codec + DiT AdaLN 的“学术拼好饭”；更妙的是训练时偷偷用真实音频的 GE2E 特征来 stabilize 模型，推理时却只能看脸硬撑，这算不算一种“开卷考试练出的学霸”？

---

### 📌 核心摘要

本论文针对 Video-to-Speech（VTS）生成中视觉-语音模态信息不对称的问题，提出现有方法忽略了语音从粗粒度语义到细粒度韵律的层次结构，导致视觉条件无法与语音表示精准对齐。为此，作者提出 HiCoDiT（Hierarchical Codec Diffusion Transformer），首次将 RVQ 编解码器的固有层次先验显式引入离散扩散框架：低层 token（VQ 1-2 层）主要由唇动与面部身份条件控制，以生成说话人相关的语义内容；高层 token（VQ 3-12 层）由面部表情情感条件调制，以捕捉细粒度韵律动态。同时，论文设计了双尺度自适应层归一化（Dual-scale AdaLN），通过通道归一化建模全局音色风格、通过时间归一化捕捉局部韵律变化。在 VoxCeleb2 上训练后，模型在零样本的 LRS2 与 LRS3 基准上超越了 FTV、AlignDiT、EmoDubber 等最新 SOTA，取得更优的语音自然度（UTMOS/DNSMOS）、可懂度（WER）与唇音同步性（LSE-C）。消融实验验证了层次化建模与双尺度 AdaLN 的有效性。局限在于训练数据说话人多样性不足时，纯视觉条件下的说话人相似度仍略逊于使用音频引导的对比方案。

---

### 🏗️ 模型架构

HiCoDiT 的整体流程可以概括为：**“视频特征解耦 → RVQ 语音 token 分层 → 层次化离散扩散去噪 → Codec 解码重建波形”**。

**1. 输入与视觉特征解耦**
给定一段静音视频序列，系统并行提取三种解耦的视觉条件：
- **唇动特征** `c_lip`：使用预训练的 **AV-HuBERT-Large** 提取最后一层隐藏状态，经 MLP 投影到维度 `L × C`（序列长度 × 通道维度），用于与低层语音 token 做帧级同步。
- **身份特征** `c_id`：使用 **ArcFace** 从面部图像提取视觉身份嵌入，经 MLP 投影到与 GE2E 语音嵌入相同的维度 `L × C_ge2e`。训练时通过 L1 损失与 **GE2E** 提取的声学身份嵌入对齐，再经 MLP 生成 AdaLN 的调制参数，用于控制音色。
- **情感特征** `c_emo`：使用 **Poster2**（视频表情识别模型）预测每帧情感类别，经 0.5 秒时间窗口平滑降采样到长度 `L_emo`，再通过可学习嵌入层映射为 `L_emo × C`，用于控制高层韵律。

**2. 语音 Tokenization 与分层**
采用预训练的 **RVQ-based Codec（来自 MaskGCT）** 将单声道 16 kHz 语音压缩为 12 层离散 token 序列 `x^{r1:r12}`，每层 codebook 大小为 1024，序列长度为 `L`。根据 RVQ 的残差量化特性，论文将 token 显式划分为：
- **低层（Low-level）**：`x_t^low = x_t^{r1:r2}`（第 1-2 层），编码粗粒度的说话人感知语义与音色。
- **高层（High-level）**：`x_t^high = x_t^{r3:r12}`（第 3-12 层），编码细粒度的韵律与声学细节。

**3. 离散扩散过程（基于 SEDD）**
- **前向过程**：以连续时间离散马尔可夫链对 token 进行加噪，每个 token 以概率 `1 - e^{-σ̄(t)}` 被替换为 [MASK]，其中 `σ̄(t)` 为累积噪声调度（log-linear）。
- **反向过程**：HiCoDiT 作为 **score network**，预测 concrete score（即去噪转移率），参数化从 [MASK] 恢复到各有效 token 的概率。

**4. HiCoDiT Transformer 内部结构**
模型由 **8 个 Low-level blocks** 与 **8 个 High-level blocks** 组成，隐藏维度 `C = 768`，注意力头数 12。
- **Low-level Blocks**：
  - 输入为 masked low-level token 特征 `m_t^low`。
  - **内容条件注入**：将 `m_t^low` 与 `c_lip` 在通道维度拼接，经线性层融合，实现帧级细粒度对齐。
  - **音色条件注入**：将 `c_id` 与时间 `t` 一起输入 MLP，预测 single-scale AdaLN 的通道级 scale/shift 参数 `α, γ, β`，对 attention 与 FFN 的输出进行调制：
    ```
    (1 + γ^i) · LayerNorm(h) + β^i
    ```
- **High-level Blocks**：
  - 输入为 high-level token。
  - **韵律条件注入**：使用 **Dual-scale AdaLN**，同时捕捉全局风格与局部动态：
    - **Channel-level**：使用 pooling 后的情感特征 + 时间特征，经 MLP 预测通道级参数 `α_emo,c, γ_emo,c, β_emo,c`。
    - **Temporal-level**：使用情感序列 `c_emo` 经 Temporal MLP 预测时间级 scale 参数 `γ_emo,t ∈ R^{L_emo}`，再通过 Kronecker 积 `⊗ 1_25`（`1_25 ∈ R^25` 为全 1 向量）上采样到原始长度 `L`（因为 `L_emo = L / 25`）。
    - 最终调制公式：
      ```
      [γ_emo,t ⊗ 1_25] · [(1 + γ_emo,c) · LayerNorm(h) + β_emo,c]
      ```
- **输出层**：12 ��独立的线性 score head，分别对应 RVQ 的 12 个层级，预测每层的 concrete score。

**5. 解码重建**
去噪后的 12 层 RVQ token 输入预训练的 **Codec Decoder**，直接重建出高保真语音波形。

---

### 💡 核心创新点

**创新点 1：层次化离散扩散框架（Hierarchical Codec Diffusion）**
- **定义**：首次将 RVQ 编解码器固有的“低层内容/音色 + 高层韵律”层次结构，显式引入 VTS 任务的离散扩散生成框架。
- **之前的方法**：现有 VTS（如 FTV、AlignDiT、DiffV2S）将语音视为扁平的 mel-spectrogram 或单一层级的 token 序列，视觉条件被全局、纠缠地注入，导致模态对齐模糊。
- **机制**：低层 block 只生成 VQ 1-2 层 token，条件为唇动与身份；高层 block 只生成 VQ 3-12 层 token，条件为情感。让“看什么”与“生成什么层级”严格对应。
- **效果**：消融显示，移除层次化建模后，LRS3 的 WER 从 29.41 升至 30.65，EmoAcc 从 79.41 降至 76.98，证明该先验显著有效。

**创新点 2：解耦视觉条件注入（Disentangled Visual Conditioning）**
- **定义**：将视频输入解耦为 lip、identity、emotion 三个独立适配器，分别通过不同机制注入对应的模型层级。
- **之前的方法**：传统方法通常使用单一视觉编码器或简单拼接，导致内容、音色、情感条件相互干扰。
- **机制**：AV-HuBERT 管内容（与 low-level token 拼接）；ArcFace 管音色（经 GE2E 跨模态对齐后通过 AdaLN 注入）；Poster2 管情感（经时域平滑后通过 Dual-scale AdaLN 注入高层）。
- **效果**：去掉 GE2E 对齐损失后，SpkSim 从 56.78 暴跌至 34.10（Table 7），验证了身份解耦的必要性。

**创新点 3：双尺度自适应层归一化（Dual-scale AdaLN）**
- **定义**：在高层 block 中，同时使用通道尺度（全局音色风格）与时间尺度（局部韵律动态）进行自适应归一化。
- **之前的方法**：vanilla AdaLN（如 DiT）仅使用全局通道级 scale/shift，无法建模韵律随时间变化的局部动态。
- **机制**：Temporal MLP 预测帧级 scale，Channel MLP 预测全局 shift/scale，两者通过 Kronecker 积相乘结合，实现对“全局风格 + 局部起伏”的联合控制。
- **效果**：消融显示，去掉双尺度 AdaLN 后，LRS3 的 MCD 从 9.62 升至 9.75，LSE-C 从 7.15 降至 7.12，韵律同步性下降。

---

### 🔬 细节详述

**训练数据**
- **数据集**：VoxCeleb2（大规模音视频说话人数据集）。
- **预处理流程**：
  1. 音频重采样至 16 kHz；
  2. 使用语音语言识别模型（引用 VoxLingua107 / SpeechBrain）过滤非英语片段；
  3. 使用说话人分割模型（引用 PyAnnote）移除多说话人片段；
  4. 使用 **ClearerVoice** 语音分离模型增强信噪比；
  5. 使用（引用 [43]）过滤文本-语音不对齐的样本。
- **最终规模**：261.5 小时音频，169k 条语句，覆盖 7 种基本情绪，3,438 位说话人。

**损失函数**
- **多层级 DSE 损失**：
  ```
  L_score = Σ_{i=1}^{12} L_DSE(x^{r_i}, t, c)
  ```
  即对 RVQ 全部 12 层分别计算去噪分数熵（Denoising Score Entropy），并求和。
- **身份对齐损失**：
  ```
  L_id = L1(c_id, c_GE2E)
  ```
  其中 `c_id` 为 ArcFace 视觉身份嵌入，`c_GE2E` 为 GE2E 声学身份嵌入。
- **总损失**：
  ```
  L_total = L_score + λ · L_id，λ = 100.0
  ```

**训练策略**
- **优化器**：AdamW
- **学习率**：1e-4（固定，未提及 warmup 或衰减策略）
- **Batch size**：32
- **总迭代数**：200k
- **无分类器引导（CFG）训练**：
  - 每个条件（lip / id / emo）独立地以 10% 概率设为空 `∅`；
  - 所有条件同时以 10% 概率设为空 `∅`。
- **训练技巧**：为保证训练稳定性，identity 与 emotion 特征在训练时使用**真实音频**提取的声学特征（ground truth）作为输入；推理阶段则完全使用视觉特征。

**关键超参数**
- RVQ 层数：12
- Codebook 大小：1,024
- Low-level blocks 数量：8
- High-level blocks 数量：8
- 通道维度 `C`：768
- 注意力头数：12
- 噪声调度：log-linear `σ(t)`
- 推理采样：Euler sampler，64 步
- 引导尺度（LRS3）：`w_all = 2.5`，`w_id = 1.25`，`w_emo = 1.5`，`w_lip = 2.0`
- 引导尺度（LRS2）：`w_all = 2.25`，`w_id = 1.25`，`w_emo = 1.5`，`w_lip = 2.0`

**训练硬件与效率**
- 论文**未提及**使用的 GPU 型号、数量及训练时间。

**推理细节**
- 采用 **enhanced predictor-free guidance** 进行多条件组合引导。
- 从全 [MASK] 序列出发，通过 64 步 Euler 采样迭代去噪。
- 输出 concrete score 后，经 RVQ Codec Decoder 合成波形。

---

### 📊 实验结果

**LRS3 客观指标对比（Table 1）**

| 方法 | WER ↓ | DNSMOS ↑ | UTMOS ↑ | MCD ↓ | LSE-C ↑ | LSE-D ↓ | EmoAcc ↑ | SpkSim ↑ |
|------|-------|----------|---------|-------|---------|---------|----------|----------|
| Ground Truth | 2.29 | 3.29 | 3.57 | 0.00 | 6.66 | 6.89 | 100.00 | 1.0000 |
| Lip2Wav † | 98.68 | 2.47 | 1.29 | 13.43 | 3.37 | 9.85 | 63.11 | 0.4785 |
| MTL | 76.61 | 2.42 | 1.28 | 9.84 | 5.87 | 7.51 | 61.24 | 0.3347 |
| EmoDubber † | 41.52 | 2.95 | 2.83 | 9.25 | 6.88 | 6.85 | 72.01 | 0.6052 |
| AlignDiT | 31.37 | 3.24 | 3.76 | 10.02 | 6.95 | 6.82 | 76.11 | 0.5597 |
| FTV | 30.37 | 3.22 | 3.99 | 10.54 | 7.08 | 6.66 | 73.19 | 0.5981 |
| **HiCoDiT (ours, A✗V✓)** | **29.41** | **3.50** | **3.84** | 9.62 | **7.15** | **6.58** | **79.41** | 0.5678 |
| **HiCoDiT (ours, A✓V✓)** | 28.98 | 3.44 | 3.80 | **8.69** | 7.10 | 6.61 | 77.08 | **0.6715** |

**LRS2 客观指标对比（Table 2）**

| 方法 | WER ↓ | DNSMOS ↑ | UTMOS ↑ | MCD ↓ | LSE-C ↑ | LSE-D ↓ | EmoAcc ↑ | SpkSim ↑ |
|------|-------|----------|---------|-------|---------|---------|----------|----------|
| Ground Truth | 8.93 | 3.14 | 3.05 | 0.00 | 7.20 | 6.67 | 100.00 | 1.0000 |
| Lip2Wav † | 100.05 | 2.47 | 1.31 | 14.09 | 3.83 | 9.80 | 54.38 | 0.4438 |
| MTL | 58.03 | 2.42 | 1.30 | 10.71 | 6.58 | 7.16 | 63.89 | 0.3556 |
| EmoDubber | 47.60 | 2.84 | 2.77 | 7.02 | 7.42 | 6.60 | 66.76 | 0.5252 |
| AlignDiT † | 42.26 | 3.13 | 3.65 | 8.46 | 7.50 | 6.58 | 67.01 | 0.5187 |
| FTV | 38.09 | 3.11 | 3.88 | 12.91 | 7.71 | 6.35 | 67.84 | 0.5368 |
| **HiCoDiT (A✗V✓)** | **39.99** | **3.35** | **3.68** | **8.74** | **7.95** | **6.17** | **68.21** | 0.5222 |
| **HiCoDiT (A✓V✓)** | 40.75 | 3.27 | 3.38 | 8.36 | 7.83 | 6.24 | 65.65 | **0.5954** |

**主观评价（Table 3 & Table 4）**

| 方法 | MOS_nat ↑ | MOS_exp ↑ | MOS_syn ↑ |
|------|-----------|-----------|-----------|
| Ground Truth | 3.07 ± 1.02 | 3.30 ± 1.19 | 3.40 ± 0.93 |
| AlignDiT | 2.47 ± 1.19 | 2.63 ± 1.30 | 3.13 ± 0.75 |
| FTV | 2.80 ± 1.03 | 2.90 ± 1.45 | 3.48 ± 1.02 |
| **HiCoDiT** | **3.17 ± 1.31** | 2.88 ± 1.53 | **3.50 ± 0.86** |

A/B 测试偏好率：
- Ours vs AlignDiT：57.0% / 4.9% / 38.1%
- Ours vs FTV：52.1% / 6.1% / 41.8%
- GT vs Ours：45.5% / 0.6% / 53.9%（即 53.9% 的情况下听众偏好 HiCoDiT 而非真实音频）

**消融实验（Table 5）**

| 数据集 | 消融项 | WER ↓ | DNSMOS ↑ | UTMOS ↑ | MCD ↓ | LSE-C ↑ | LSE-D ↓ | EmoAcc ↑ | SpkSim ↑ |
|--------|--------|-------|----------|---------|-------|---------|---------|----------|----------|
| LRS3 | w/o Hierarchical | 30.65 | 3.36 | 3.73 | 10.07 | 7.02 | 6.75 | 76.98 | 0.5652 |
| LRS3 | w/o Dual Scale AdaLN | 29.60 | 3.45 | 3.92 | 9.75 | 7.12 | 6.60 | 78.55 | 0.5621 |
| LRS3 | **HiCoDiT (full)** | **29.41** | **3.50** | **3.84** | **9.62** | **7.15** | **6.58** | **79.41** | **0.5678** |
| LRS2 | w/o Hierarchical | 44.57 | 3.18 | 3.48 | 9.43 | 7.66 | 6.47 | 64.69 | 0.4946 |
| LRS2 | w/o Dual Scale AdaLN | 41.01 | 3.30 | 3.75 | 9.33 | 7.88 | 6.22 | 68.61 | 0.5155 |
| LRS2 | **HiCoDiT (full)** | **39.99** | **3.35** | **3.68** | **8.74** | **7.95** | **6.17** | **68.21** | **0.5222** |

**OOD 电影数据实验（Table 6）**

| 方法 | WER ↓ | MCD ↓ | DNSMOS ↑ | Emo ↑ | Spk ↑ | LSE-D ↓ |
|------|-------|-------|----------|-------|-------|---------|
| EmoDubber | 88.3 | 9.9 | 2.8 | 76.5 | 45.1 | 7.72 |
| AlignDiT | 80.8 | 11.4 | 3.2 | 75.2 | 58.5 | 8.23 |
| **HiCoDiT** | **58.7** | **9.8** | **3.5** | **82.0** | 50.1 | **7.60** |

**视觉条件消融（Table 7）**

| 消融项 | WER ↓ | MCD ↓ | DNSMOS ↑ | Emo ↑ | Spk ↑ | LSE-D ↓ |
|--------|-------|-------|----------|-------|-------|---------|
| (a) w/o GE2E L_id | 29.38 | 10.18 | 3.41 | 74.47 | 34.10 | 6.71 |
| (b) w/o Poster2 (替换为 Poster) | 29.41 | 9.68 | 3.50 | 76.29 | 55.28 | 6.67 |
| HiCoDiT (full) | 29.41 | 9.62 | 3.50 | 79.41 | 56.78 | 6.58 |

---

### ⚖️ 评分理由

- **创新性：8.5/10** — 首次将 RVQ 层级结构与离散扩散结合用于 VTS，并提出了视觉条件与 token 层级的显式对齐策略，思路清晰且有效。但 SEDD 扩散框架、MaskGCT Codec、AdaLN 均为已有技术，属于“高明的组装创新”，尚未建立全新范式。
- **实验充分性：9/10** — 实验极其全面：两个标准 benchmark（LRS2/LRS3）、OOD 真实电影数据、两组主要消融（层次化、AdaLN）、视觉条件细粒度消融、主观 MOS 与 A/B 测试。唯一缺憾是缺少模型参数量、计算量（FLOPs）及推理实时性的分析。
- **实用价值：7.5/10** — 对静音视频配音、辅助失声人群沟通等场景有直接价值，且离散 token 建模在效率上具潜力。但 VTS 本身受限于“视觉信息不足以完全决定语音”的固有瓶颈，且论文未展示实时推理或低延迟优化，离大规模落地仍有距离。
- **灌水程度：1.5/10** — 方法简洁，实验扎实，claims 基本有数据支撑。仅有个别“first”表述（如“first discrete diffusion framework for VTS”）需要加若干限定词才能严格成立，整体不属于灌水。

---

### 🔗 开源详情

- **代码**：已开源，GitHub 仓库地址：https://github.com/Jiaxin-Ye/HiCoDiT
- **模型权重**：论文中未明确说明是否公开预训练权重文件。
- **数据集**：使用公开数据集 VoxCeleb2，作者进行了多阶段预处理（语言过滤、说话人分割、语音增强、对齐过滤），但预处理后的 261.5 小时数据集未说明是否提供下载。
- **预训练权重**：未提及是否提供基于 VoxCeleb2 训练的 checkpoint。
- **在线 Demo**：论文提到 speech demo 可在项目页面或 GitHub 获取，但未提供具体在线交互网址。
- **依赖的开源项目**：AV-HuBERT（唇动特征）、ArcFace（人脸身份）、Poster2（表情识别）、GE2E（说话人嵌入）、MaskGCT（RVQ Codec）、SEDD（离散扩散框架）、ClearerVoice（语音增强）、SyncNet（唇音同步评估）、ECAPA-TDNN（说话人相似度）、emotion2vec / EmoBox（语音情感评估）、PyAnnote（说话人分割）、SpeechBrain / VoxLingua107（语言识别）。

---

### 🖼️ 图片与表格

- **图 1: HiCoDiT 整体架构图** | 保留: **是** — 该图展示了从视频输入到三个解耦适配器（Lip/Identity/Emotion），再到 Low-level / High-level Blocks、12 个 Score Head，最终到 Codec Decoder 的完整数据流，是理解方法的核心。
- **图 2: (a) RVQ Codec 示意图与 (b) Hierarchy Analysis 曲线** | 保留: **是** — (a) 展示残差向量量化的层级叠加过程；(b) 通过 Content/Timbre/Prosody Score 随 VQ layer 的变化曲线，直观证明了“低层承载内容与音色、高层承载韵律”的先验假设，是整篇论文的立论基石。
- **图 3: 定性语谱图对比** | 保留: **否** — 仅为与其他方法生成的 mel-spectrogram 的目视对比，无定量信息，属于辅助性展示。
- **图 4: OOD 电影数据定性结果** | 保留: **否** — 仅为在 CinePile 数据集上的样例展示，无关键定量数据，次要。

**关键表格数据（已在上文“实验结果”节完整输出）**：
- Table 1（LRS3 客观指标）与 Table 2（LRS2 客观指标）已完整复述所有模型在所有指标上的数值。
- Table 3（主观 MOS）与 Table 4（A/B 测试）已完整复述。
- Table 5（消融实验）已完整复述两个数据集下的所有数值。
- Table 6（OOD 实验）已完整复述。
- Table 7（视觉条件消融）已完整复述。

### 📸 论文图片

![figure](https://arxiv.org/html/2604.15923v1/x1.png)

![figure](https://arxiv.org/html/2604.15923v1/x2.png)

![figure](https://arxiv.org/html/2604.15923v1/x3.png)


---

[← 返回 2026-04-20 论文速递](/audio-paper-digest-blog/posts/2026-04-20/)
