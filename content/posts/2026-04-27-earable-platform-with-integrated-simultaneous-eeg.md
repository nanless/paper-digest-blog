---
title: "Earable Platform with Integrated Simultaneous EEG Sensing and Auditory Stimulation"
date: 2026-04-27
draft: false
tags: [音频事件检测, 信号处理, 多通道, 时频分析]
categories: [论文速递]
description: "音频事件检测 | 5.5/10"
hiddenInHomeList: true
---

# 📄 Earable Platform with Integrated Simultaneous EEG Sensing and Auditory Stimulation

#音频事件检测 #信号处理 #多通道 #时频分析

📝 **5.5/10** | 后50% | #音频事件检测 | #信号处理 | #多通道 #时频分析 | [arxiv](https://arxiv.org/abs/2604.22137v1)

学术质量 5.0/7 | 选题价值 1.5/2 | 复现加成 -1.0 | 置信度 中


### 👥 作者与机构

- 第一作者：Min Suk Lee (UC San Diego, Shu Chien-Gene Lay Department of Bioengineering)
- 通讯作者：Yuchen Xu (yux013@ucsd.edu), Gert Cauwenberghs (gcauwenberghs@ucsd.edu)
- 作者列表：
    - Min Suk Lee (UC San Diego, Shu Chien-Gene Lay Department of Bioengineering)
    - Abhinav Uppal (UC San Diego, Shu Chien-Gene Lay Department of Bioengineering)
    - Ananya Thota (UC San Diego, Shu Chien-Gene Lay Department of Bioengineering)
    - Chetan Pathrabe (UC San Diego, Shu Chien-Gene Lay Department of Bioengineering)
    - Rommani Mondal (UC San Diego, Shu Chien-Gene Lay Department of Bioengineering)
    - Akshay Paul (UC San Diego, Institute for Neural Computation)
    - Yuchen Xu (UC San Diego, Institute for Neural Computation)
    - Gert Cauwenberghs (UC San Diego, Shu Chien-Gene Lay Department of Bioengineering; Institute for Neural Computation)

### 💡 毒舌点评

亮点在于其将定制化耳道模型与Ag/AgCl干电极喷涂技术相结合，显著提升了信号质量和佩戴舒适度，为长期脑电监测提供了实用方案。短板是验证仅限于单个受试者，且其中一个对侧通道表现出显著噪声，这使得“稳健”、“长期”等宣称的普适性大打折扣，更像一个精心调校的原型机演示。

### 📌 核心摘要

本文旨在解决传统头皮脑电图（EEG）设备笨重、不便携、存在社会污名化的问题，提出一种个性化的耳戴式EEG监测（IEEM）平台。该平台通过定制耳印模和3D打印实现与用户耳道解剖结构的精确贴合，并在同一设备中集成了EEG电极和音频驱动器。与通用耳戴设备相比，其核心创新在于通过个性化定制保证了电极与皮肤的稳定接触和高保真信号采集。实验结果表明，该平台成功检测到了眼电（EOG）、眨眼、下颌紧咬、40 Hz听觉稳态响应（ASSR）和alpha波调制等生理信号，电化学阻抗谱（EIS）显示其阻抗值（例如，在10 Hz时同侧配置平均阻抗为424 kΩ）与传统干电极相当。该集成方案为未来的闭环神经调控应用（如基于EEG的听觉神经反馈）奠定了基础，但主要局限性在于验证实验仅使用了一名受试者，且部分通道噪声较大，定制化流程也限制了其规模化部署。

### 💡 核心创新点

1.  **高度个性化的耳戴硬件平台**：创新在于将从耳模取样到3D打印、定制喷涂电极的全流程工程化，打造了与单个用户耳道解剖结构完美贴合的设备。这解决了通用耳戴设备在可靠信号采集上的根本矛盾。
2.  **EEG感知与听觉刺激的深度集成**：在同一耳内空间同时实现了高质量的多通道EEG记录和音频播放。这种一体化设计为实时、原位的闭环神经调控（如EEG驱动的听觉反馈）提供了最直接的硬件基础。
3.  **适用于耳道的Ag/AgCl干电极工艺**：在定制化的3D打印耳壳上，通过可剥离掩膜和喷涂技术直接制作生物兼容的Ag/AgCl电极。这种方法既保证了电极与耳道曲面的紧密贴合，又避免了导电凝胶的使用，提升了长期佩戴的舒适性和稳定性。
4.  **便携式高保真验证平台**：集成无线采集板（weDAQ）和耳戴设备，构建了一个完整、可移动的验证系统，能够在受控环境中模拟日常佩戴场景，进行多种生理信号（EMG, EOG, EEG）的初步验证。

### 🔬 细节详述

- **训练数据**：未说明。本研究为硬件原型验证，未涉及机器学习模型的训练。
- **损失函数**：未说明。
- **训练策略**：未说明。
- **关键超参数（硬件参数）**：
    - EEG采集：采样率500 Hz，电压增益24，使用主动驱动右腿（DRL）电路，伪单极导联。
    - 滤波处理：实验数据使用不同阶数的Butterworth滤波器。例如，EMG实验使用4阶陷波滤波器（60 Hz）和4阶带通滤波器（2-40 Hz）；ASSR+Alpha实验中，对信号进行了8-13 Hz和40±1 Hz的带通滤波。
    - 刺激参数：听觉刺激为40 Hz振幅调制的白噪声，声压级约为50 dBA。
- **训练硬件**：未说明（非机器学习模型）。
- **推理细节**：未说明（非机器学习模型）。
- **正则化或稳定训练技巧**：未说明。

### 📊 实验结果

论文通过多项电生理实验验证了平台的性能。

**1. 电化学阻抗谱 (EIS) 结果：**

| 配置 | 测量频率 | 平均阻抗 (Mean Impedance) | 平均负相位 (Mean Negative Phase) |
| :--- | :--- | :--- | :--- |
| 同侧 (Ipsilateral) | 10 Hz | 424 kΩ | 未给出具体数值 |
| 同侧 (Ipsilateral) | 100 Hz | 121 kΩ | 未给出具体数值 |
| 对侧 (Contralateral) | 10 Hz | 270 kΩ | 未给出具体数值 |
| 对侧 (Contralateral) | 100 Hz | 87.1 kΩ | 未给出具体数值 |

![图2: (a) 同侧和对侧配置的EIS结果。(b) 人体头部轮廓上的同侧和对侧配置示意图。(c) 根据EIS结果估算的电极-皮肤界面Randles模型。](https://arxiv.org/html/2604.22137v1/figures/Fig3.png)
结论：阻抗值在可接受范围内，与传统干电极相当，证实了电极-皮肤接触的稳定性。同侧阻抗高于对侧，可能因参考电极距离更近。

**2. 电生理测量结果：**
- **EMG（下颌紧咬）**：成功检测到信号。频谱图显示，与放松状态相比，下颌紧咬期间（1-100 Hz，除60 Hz外）的平均功率从4.4778 μV²显著增加到20.5651 μV²。
- **眨眼**：成功检测到眨眼事件，对侧通道信号幅值通常大于同侧通道（耳道电极ED例外）。
- **EOG（水平眼动）**：成功检测到左右眼动信号，峰峰值差异约为80 μV²。垂直眼动和同侧信号不明显。
- **ASSR与Alpha波调制**：
    - 实验在受试者闭眼状态下进行。在第30秒播放40 Hz调幅白噪声。
    - 平均Alpha波（8-13 Hz）功率为6.62 μV²。
    - 播放声音后，平均40 Hz（±1 Hz）信号功率为5.76 μV²。
    - 结果表明，平台既能记录内源性的Alpha节律，又能捕捉到外源性听觉刺激引起的40 Hz稳态响应。

![图3: (a) EMG实验的时间序列结果。(b) (a)中同一EMG实验的频谱图结果，显示了3次下颌紧咬的宽频信号特性。](https://arxiv.org/html/2604.22137v1/figures/Fig4.png)
![图4: (a) 眼动实验的时间序列结果。红色色调图为同侧通道，蓝色色调图为对侧通道。带有垂直虚线的眼睛图标叠加表示眨眼发生时刻。(b) EOG实验的时间序列结果。眼睛图标放置在信号变化旁以指示眼球运动方向。](https://arxiv.org/html/2604.22137v1/figures/Fig5_new.png)
![图5: ASSR+Alpha实验，在实验进行到30秒时播放40 Hz调幅白噪声。(a) ASSR+Alpha实验的时间序列结果。(b) 实验的频谱图结果。](https://arxiv.org/html/2604.22137v1/figures/Fig6.png)

### ⚖️ 评分理由

- **学术质量：5.0/7**：论文清晰地阐述了一个完整硬件系统的设计、制造和初步验证流程，技术路线合理，实验设置详细。创新点明确（个性化定制+集成感知刺激），在耳戴EEG领域具有工程价值。主要扣分点在于验证仅限于**单个受试者**，且存在**一个通道噪声显著**的问题，这使得结论的普适性和鲁棒性存疑，属于原型机级别的工作，而非经过充分验证的平台研究。
- **选题价值：1.5/2**：耳戴式EEG是便携、长期脑监测的前沿方向，集成听觉刺激使其更具应用潜力（如睡眠、冥想、康复），具有明确的实用价值和研究前景。分数不高是因为该领域已非全新，且论文未深入探讨其独特应用场景的不可替代性。
- **开源与复现加成：-1.0/1**：论文**未提供任何代码、模型、数据集或详细的电子设计文件**。硬件制造过程虽有描述，但涉及定制模具、特定材料（如Ag/AgCl油墨）和商业设备，完全复现门槛很高。开源信息的缺失严重限制了工作的可复现性和社区贡献。

### 🔗 开源详情

- 代码：论文中未提及代码链接。
- 模型权重：未提及。
- 数据集：未提及。
- Demo：未提及。
- 复现材料：论文描述了从耳模制作、3D扫描、CAD处理、3D打印、电极喷涂到音频组件组装的详细步骤，并列出了关键材料（如特定型号的柔性树脂、Ag/AgCl油墨、动铁驱动器）和设备（如3D扫描仪、打印机、weDAQ板）。然而，这些属于硬件制作指南，并非通常意义上的模型训练细节、配置或检查点。
- 论文中引用的开源项目：论文引用了其团队此前的工作`paul_versatile_2022, paul_versatile_2023, lee_scalable_2023`，其中可能包含了对无线数据采集板`weDAQ`的详细描述，但这些工作本身是否开源未在本文中说明。
- 论文中未提及开源计划。

---

[← 返回 2026-04-27 论文速递](/audio-paper-digest-blog/posts/2026-04-27/)
