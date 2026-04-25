---
title: "VoxMind: An End-to-End Agentic Spoken Dialogue System"
date: 2026-04-20
draft: false
tags: [语音对话系统, 语音大模型, 端到端, 数据集]
categories: [论文速递]
description: "端到端语音对话模型在自然交互上进步迅速，但普遍缺乏处理复杂任务的agent能力（工具调用、规划、推理）。本文首先形式化定义了\"端到端语音智能体\"的四大维度——画像（Profile）、记忆（Memory）、规划（Planning）与执行（Action Execution），填补了该领域理论标准的空白。"
hiddenInHomeList: true
---

# 📄 VoxMind: An End-to-End Agentic Spoken Dialogue System

#语音对话系统 #语音大模型 #端到端 #数据集

🔥 **评分：8.5/10** | [arxiv](https://arxiv.org/abs/2604.15710v1)


### 👥 作者与机构

- **共同第一作者**：Tianle Liang（浙江大学；China University of Petroleum-Beijing at Karamay），Yifu Chen（浙江大学），Shengpeng Ji（浙江大学）
- **通讯作者**：Zhou Zhao（浙江大学，zhaozhou@zju.edu.cn）
- **其他作者**：Yijun Chen（China University of Petroleum-Beijing at Karamay），Zhiyang Jia（China University of Petroleum-Beijing at Karamay），Jingyu Lu（浙江大学），Fan Zhuo（浙江大学），Xueyi Pu（浙江大学），Yangzhuo Li（厦门大学）

---

### 💡 毒舌点评

**亮点**：VoxMind把文本Agent那套"先想后说"的套路成功塞进了端到端语音模型里，还顺手用"辅助LLM异步捞工具"治好了工具一多就卡顿的绝症，实验硬到能把Gemini-2.5-Pro按在地上摩擦。

**槽点**：470小时的训练数据全靠TTS合成，遇到真人说话时的"嗯…那个…"、结巴和背景噪音立刻掉7个点；所谓"Think-before-Speak"本质上就是在语音流里硬插了一段文本CoT，延迟该高还是高，作者自己也承认这是"必要的 trade-off"——翻译一下就是"我知道慢，但先忍着"。

---

### 📌 核心摘要

端到端语音对话模型在自然交互上进步迅速，但普遍缺乏处理复杂任务的agent能力（工具调用、规划、推理）。本文首先形式化定义了"端到端语音智能体"的四大维度——画像（Profile）、记忆（Memory）、规划（Planning）与执行（Action Execution），填补了该领域理论标准的空白。在此基础上提出VoxMind框架，引入"Think-before-Speak"机制，使模型在生成语音响应前显式产出结构化推理链（Chain-of-Thought）；并构建470小时的AgentChat数据集，包含工具交互与通用对话数据，且全部标注了推理轨迹与工具调用标签。为解决大规模工具库带来的推理延迟爆炸问题，VoxMind设计了多智能体动态工具管理架构：主agent专注于推理与行动，辅助LLM异步从全局工具池中检索候选工具，仅当主agent判定本地工具不足时才动态扩容局部工具集，从而将推理延迟与工具库规模解耦。实验表明，VoxMind的任务总体完成率达74.57%，较基线StepAudio2（34.88%）相对提升113.79%，并超越闭源模型Gemini-2.5-Pro（71.51%）；同时在VoiceBench通用对话评测上保持了与基线相当的能力。局限在于显式推理引入了额外的推理延迟，且AgentChat数据依赖TTS合成，与真实口语的自发性和不流畅性存在差距。

---

### 🏗️ 模型架构

VoxMind是一个基于StepAudio2微分的端到端语音智能体，其系统状态在时刻t被严格形式化为三元组：
**S_t = (O_t, H_t, A_t)**

- **O_t（观测）**：包含当前用户输入X_t（语音token序列）以及环境/工具返回的结构化反馈O_t^env。
- **H_t（历史）**：累积的多模态交互历史，包含语义记忆与声学记忆。
- **A_t（动作空间）**：包含言语回复V和动态可访问的局部工具子集T_t^local ⊂ T_all。

**完整输入输出流程**：
1. **语音编码**：用户语音输入被编码为离散声学token（基于StepAudio2的tokenizer）。
2. **思考阶段（Think）**：策略π_θ^think根据当前观测o_t、历史H_{t-1}和局部工具集T_t^local，显式采样生成一段Chain-of-Thought推理轨迹c_t。这段推理包含意图理解、上下文分析和任务规划，以文本token形式插入在最终输出之前。
3. **行动阶段（Act）**：策略π_θ^act在条件c_t下，基于当前状态采样下一步动作a_t。动作可以是：
   - 生成语音回复token，最终解码为语音波形；
   - 生成结构化工具调用（JSON格式），包含工具名与参数。
4. **动态工具更新（并行）**：在步骤2-3进行的同时，系统并行启动辅助LLM π_LLM，根据已生成的推理轨迹c_t从全局工具池T_all中检索候选工具T_t^cand。
5. **条件状态转移**：若主agent在步骤3发出的动作是检索动作a_retrieve（即判定当前局部工具不足），则下一时刻局部工具集更新为T_{t+1}^local = T_t^local ∪ T_t^cand；否则保持不变。随后主agent基于更新后的工具集执行下一步决策。

**关键设计选择**：
- **显式CoT的"Think-before-Speak"**：传统端到端模型直接做x→y的映射，VoxMind强制插入中间推理步骤z，变为x→z→y。这使得复杂任务分解和工具参数填充有明确的认知基础，而非盲目模仿。
- **主-辅双智能体架构**：语音模态本身编码声学信息需要远多于文本的token，若每次都将全部工具描述填入prompt，延迟将随工具数线性甚至指数增长。通过辅助LLM异步检索，主agent始终只在紧凑的局部工具空间内推理，延迟被有效控制。

### 💡 核心创新点

**1. 端到端语音智能体的统一形式化定义**
- **是什么**：从Profile（静态画像+动态自适应画像）、Memory（语义+声学双通道短/长期记忆）、Planning（显式中间推理z）、Action Execution（决策+工具选择调用）四个维度，首次严格定义了"端到端语音智能体"应该具备什么。
- **之前的问题**：语音agent领域此前只有零散的功能扩展（如Stream RAG、WavRAG），缺乏统一标准，导致模型设计与评估各行其是。
- **效果**：为后续所有语音agent工作提供了理论基准。

**2. "Think-before-Speak"推理机制与AgentChat数据集**
- **是什么**：在端到端语音模型中强制显式生成结构化CoT推理链，并构建470小时语音数据集进行监督微调。
- **之前的问题**：现有端到端语音模型（如Kimi-Audio、StepAudio2）直接映射输入到输出，缺乏复杂规划能力；且缺乏带agent行为标注的语音数据。
- **机制**：通过反向条件生成（给定Q和A，让LLM生成R）+ 严格质量过滤（0-10分制，阈值7，最多重试3次）+ 文本精炼，构建高质量推理轨迹。模型在语音输入后直��生��<think>...< /think>再生成回复或工具调用。
- **效果**：消融实验显示，引入think后模型总体得分从68.83（w/o think, 1:1）提升至74.57（w/ think, 1:0.5），且通用对话能力（VoiceBench）未退化，而不引入think的模型在减少通用数据时通用能力会崩盘（54.80 vs 59.72）。

**3. 多智能体动态工具管理（Multi-Agent Dynamic Tool Management）**
- **是什么**：通过一个与主模型并行的辅助LLM，异步地从全局工具池检索候选工具，动态维护主agent的局部工具空间。
- **之前的问题**：语音输入token本就冗长，若prompt中塞入大量工具描述，推理延迟随工具数指数上升；若工具描述太少，agent又无法完成复杂任务。
- **机制**：主agent生成CoT后，两条路径并行——(a)主agent基于当前局部工具集生成动作；(b)辅助LLM基于CoT检索全局工具。仅当主agent显式发出a_retrieve时才合并候选工具。这样主agent的推理延迟与全局工具库大小解耦。
- **效果**：图4显示，当工具数从1增至100时，无辅助LLM的单智能体延迟从约1飙升至30+（归一化值），而VoxMind保持在约2以下；任务准确率（FS/PF）在无辅助LLM时随工具数增加从95%/70%暴跌至15%/10%，而VoxMind稳定在95%/65%左右。

**4. 延迟-规模解耦的实验验证**
- **是什么**：通过受控实验量化证明辅助LLM检索的等待开销可被主agent的推理过程完全掩盖。
- **效果**：附录I显示，全局工具100个时辅助LLM检索需2.64秒，但主agent平均等待开销仅0.0053秒，实际接近O(1)任务执行延迟。

### 🔬 细节详述

**训练数据**：
- **AgentChat总时长**：约470小时，由Tool Interaction子集（约109小时，14,805条）和General Dialogue子集（约361小时，38,681条）组成。
- **Tool Interaction来源**：
  - ToolACE（5,582条，26.62小时）
  - APIGen-MT（791条，43.26小时）
  - 自建数据（8,432条，39.19小时），细分为：tool-select（1,237条）、multi-tool-select（1,486条）、para-filled（1,409条）、parallel-call（1,144条）、searchTool（467条，主动请求新工具）、observation（2,465条，环境反馈处理）、obs_searchtools（224条）。
- **General Dialogue来源**：
  - ARC-Challenge（1,167条，12.33小时）、ARC-Easy（1,164条，10.82小时）、GSM8K（1,746条，18.47小时）、SciQ（998条，9.49小时）
  - 中学课本知识衍生的course数据（19,152条，141.91小时）和conversation数据（11,259条，125.46小时）、multi-conversation（3,171条，42.35小时）。
- **语音合成**：使用CosyVoice2进行TTS合成；为增加音色多样性，额外使用SeedTTS的600余种提示音色。
- **数据配比**：探索了1:1（agent数据:通用数据时长比）和1:0.5（通用数据下采样约50%）两种策略。
- **补充数据**（表8）：
  - No-Tool：2,717轮用户语音+助手文本（5.09小时），防止误触发工具调用。
  - Security：556轮纯文本安全/推理链数据。
  - Text：2,500轮纯文本标准对话。
- **文本清洗**：粗粒度规则过滤HTML/Markdown/代码；细粒度使用Qwen-plus模型润色为自然口语风格并过滤不适合语音场景的内容。

**CoT构建流程**：
- 采用反向条件生成：给定问题Q和最终输出A（工具调用或回答），使用LLM采样推理链R ~ p_LLM(R|Q,A)。
- 质量评估：0-10分制，阈值τ=7。未达标则最多重试T=3次。仍不达标则丢弃。
- 精炼：使用LLM在保留核心逻辑流的前提下压缩并标准化格式，输出严格单行JSON `{"think": "..."}`。

**损失函数**：
- 论文未显式给出损失函数公式。基于StepAudio2微调，采用标准的**自回归next-token prediction交叉熵损失**，对语音token、文本token（含CoT和工具调用）统一建模。

**训练策略与超参数**：
- **硬件**：2 × NVIDIA H20-NVLink GPU
- **框架**：PyTorch 2.6.0，CUDA 12.4，Python 3.10
- **优化器**：AdamW
- **学习率**：1e-5，采用cosine learning rate scheduler
- **Batch size**：1（per device），gradient accumulation steps = 8，等效batch size = 16
- **正则化**：weight decay = 0.01，max gradient norm clipping = 1.0
- **精度与加速**：bfloat16，DeepSpeed ZeRO-3策略，gradient checkpointing
- **训练时长/轮数**：论文未明确给出总训练步数或epoch数。

**推理细节**：
- 论文未明确给出temperature、top-p、beam search等解码超参数。
- THINK token在语音输出场景中平均占88.0个token，在文本输出场景中平均84.4个token。

### 📊 实验结果

**核心Agent能力评估（对应论文Table 2）**：

| 模型 | Single Task<br>TS / PF | Task Decomp<br>TS / PF | Parallel<br>TS / PF | Contextual<br>TS / PF | Proactive<br>TU | Result<br>FC | **Overall** |
|---|---|---|---|---|---|---|---|
| Gemini-2.5-pro | 90.98 / 75.19 | 82.54 / 52.38 | 88.57 / 69.52 | 84.25 / 61.64 | 26.87 | 4.16 | 71.51 |
| Gemini-2.5-flash | 92.48 / 77.44 | 61.90 / 31.22 | 86.67 / 68.25 | 86.99 / 65.75 | 31.34 | 4.10 | 68.40 |
| GPT-4o-audio | 85.71 / 70.68 | 23.81 / 15.87 | 84.76 / 61.90 | 71.23 / 49.32 | 0.00 | 4.22 | 54.77 |
| Qwen3-8B+Whisper | 94.99 / 68.42 | 82.54 / 41.27 | 85.71 / 46.67 | 84.25 / 47.72 | 7.46 | 4.05 | 64.00 |
| Kimi-Audio | 78.45 / 56.89 | 48.15 / 22.75 | 79.05 / 55.24 | 76.03 / 46.80 | 13.64 | 3.62 | 54.94 |
| Qwen2.5-Omni | 78.70 / 35.84 | 38.62 / 3.17 | 65.40 / 28.57 | 65.75 / 26.03 | 0.00 | 2.82 | 39.85 |
| StepAudio2 | 78.70 / 48.87 | 60.32 / 26.98 | 53.33 / 33.33 | 4.34 / 1.60 | 3.12 | 1.91 | 34.88 |
| **VoxMind** | **98.50 / 72.18** | **95.24 / 38.10** | **89.52 / 61.59** | **80.82 / 62.33** | **68.66** | **3.94** | **74.57** |

- **Overall提升**：VoxMind（74.57）相比基线StepAudio2（34.88）相对提升113.79%，超过最强闭源模型Gemini-2.5-Pro（71.51）3.06个百分点。

**消融实验（对应论文Table 3）**：

| 配置 | Single Task<br>TS / PF | Task Decomp<br>TS / PF | Parallel<br>TS / PF | Contextual<br>TS / PF | Proactive<br>TU | Result<br>FC | **Overall** |
|---|---|---|---|---|---|---|---|
| w/o think (1:1) | 88.72 / 70.68 | 95.24 / 39.68 | 80.00 / 45.71 | 86.99 / 73.29 | 31.34 | 3.83 | 68.83 |
| w/o think (1:0.5) | 90.23 / 71.68 | 93.65 / 36.51 | 80.00 / 59.05 | 86.30 / 75.34 | 37.31 | 3.98 | 70.97 |
| w/ think (1:1) | 90.98 / 68.42 | 94.71 / 44.44 | 80.95 / 51.43 | 84.93 / 65.75 | 59.70 | 3.92 | 71.97 |
| **w/ think (1:0.5)** | **98.50 / 72.18** | **95.24 / 38.10** | **89.52 / 61.59** | **80.82 / 62.33** | **68.66** | **3.94** | **74.57** |

- 关键发现：引入think机制后，减少通用数据比例（1:0.5）不仅提升了agent任务表现（74.57 vs 71.97），且通用能力未受损；而无think时减少通用数据会导致agent任务增益微弱（68.83→70.97）且通用能力显著下降。

**VoiceBench通用对话能力（对应论文Table 4）**：

| 模型 | AlpacaEval | CommonEval | WildVoice | SD-QA<br>(USA)/Panda | SD-QA<br>(USA)/GPT | MMSU | OBQA | BBH | IFEval | AdvBench | **Overall** |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Step-Audio-2 | 4.19 | 3.12 | 3.36 | 55.15 | 52.80 | 50.82 | 68.13 | 58.53 | 39.64 | 92.88 | 64.15 |
| w/o think (1:0.5) | 3.38 | 3.43 | 3.02 | 49.73 | 38.34 | 36.88 | 56.70 | 50.66 | 20.74 | 87.69 | 54.80 |
| w/o think (1:1) | 3.77 | 3.75 | 3.42 | 48.28 | 39.24 | 47.69 | 68.79 | 50.25 | 23.61 | 84.62 | 59.72 |
| w/ think (1:1) | 4.08 | 4.03 | 3.79 | 51.90 | 44.48 | 51.61 | 65.49 | 56.31 | 17.40 | 95.58 | 63.62 |
| **w/ think (1:0.5)** | **3.98** | **3.94** | **3.69** | **49.73** | **44.85** | **53.04** | **71.87** | **54.69** | **18.83** | **100.00** | **64.21** |

- VoxMind最佳配置（w/ think, 1:0.5）Overall 64.21，不仅超过基线Step-Audio-2（64.15），更远优于无think配置，证明agent训练在正确机制下不会牺牲通用对话能力。

**真实语音鲁棒性（附录H）**：

| 输入类型 | FS | PF |
|---|---|---|
| TTS Speech | 93.33% | 67.33% |
| Real Speech | 86.00% | 60.67% |

- 真实语音相较TTS在FS上下降约7.3%，PF下降约6.7%，但在含口吃、犹豫、噪音等条件下仍保持86%的任务成功率。

**延迟-规模解耦（附录I，Table 10）**：

| 全局工具数 | Aux LLM检索延迟(s) | 主agent等待开销(s) |
|---|---|---|
| 10 | 1.3131 | 0.0000 |
| 25 | 1.5731 | 0.0000 |
| 50 | 1.8996 | 0.0154 |
| 75 | 2.3782 | 0.0132 |
| 100 | 2.6426 | 0.0053 |
| **平均** | — | **<0.015** |

**Token级开销（附录J，Table 11）**：

| 输出模式 | THINK Tokens(avg) | Answer Tokens(avg) | THINK/Answer |
|---|---|---|---|
| Speech Output | 88.0 | 701.2 | 12.6% |
| Text Output | 84.4 | 52.6 | 160.5% |

- 语音输出时思考token仅占12.6%，额外开销可忽略；思考token数量稳定在80-90之间，不随工具库规模增长。

**动态工具管理图表（图4）**：
- **图4(a) 推理效率**：无Aux LLM时，工具数1→100对应的归一化延迟从约1指数增长至30+；有Aux LLM时全程稳定在约2以下。
- **图4(b) 任务性能**：无Aux LLM时，FS从95%（1工具）暴跌至约18%（100工具），PF从约70%暴跌至约12%；有Aux LLM时，FS稳定在约95%，PF稳定在约65-70%。

### ⚖️ 评分理由

**创新性：8.5/10**
- 首次为端到端语音agent建立系统的形式化定义，将CoT推理与动态工具管理引入语音模态，是该领域的重要基准工作。但"显式CoT推理"和"工具调用"在文本LLM agent领域已高度成熟，方法论层面的原创性更多体现在"语音化适配"与"系统整合"上，而非底层范式创新。

**实验充分性：9.0/10**
- 评估维度极为全面：涵盖6项核心agent能力、10项通用对话指标、真实语音鲁棒性、延迟-规模解耦量化、token级开销分析；对比基线覆盖闭源（Gemini-2.5-Pro/Flash, GPT-4o-audio）、开源端到端（Kimi-Audio, Qwen2.5-Omni, StepAudio2）与级联系统（Qwen3+Whisper）共7个模型；消融实验清晰验证了think机制与数据比例的作用。扣分点仅在于未报告训练收敛曲线与部分超参数（如具体epoch数）。

**实用价值：8.5/10**
- 动态工具管理直接命中语音agent落地的延迟痛点，完整开源代码和数据集对社区推动力强。但推理延迟trade-off尚未解决，且训练数据依赖TTS合成，距离直接部署到真实场景仍需真实语音数据的进一步迭代。

**灌水程度：2.0/10（分数越低越好）**
- 论文内容密度高，方法、数据、实验、理论定义环环相扣，无明显冗余或夸大。自我剖析的局限性（延迟、TTS数据gap）诚恳且具体。

---

### 🔗 开源详情

- **代码**：完全开源，GitHub地址为 https://github.com/MM-Speech/VoxMind。论文未给出具体stars数量与框架版本依赖细节。
- **模型权重**：基于开源模型StepAudio2进行监督微调。论文未明确说明是否将微调后的权重上传至HuggingFace等平台，但代码仓库公开通常暗示可复现。
- **数据集**：开源AgentChat数据集，总规模约470小时。包含：
  - AgentChat-Tool（约109小时，14,805条）：覆盖单工具选择、多工具选择、参数填充、并行调用、主动检索、环境反馈观察等场景。
  - AgentChat-Normal（约361小时，38,681条）：覆盖常识推理（ARC/SciQ）、数学推理（GSM8K）、课本知识与开放域对话。
  - 补充数据：No-Tool跨模态数据（5.09小时）、Security安全数据、Text纯文本数据。
- **预训练权重**：基于StepAudio2基座模型。
- **在线Demo**：论文中未提及在线体验地址。
- **依赖工具/模型**：PyTorch, DeepSpeed, CosyVoice2（语音合成）, SeedTTS（音色多样化）, Qwen-plus（数据清洗、CoT生成与质量评估）, Gemini-2.5-Flash（自动评估器）。

---

### 🖼️ 图片与表格

**图片保留建议**：
- **图1**: VoxMind统一框架概念图（展示Profile/Memory/Planning/Action四大维度） | 保留: 否 - 纯概念性框图，文字定义已足够清晰，无定量信息。
- **图2**: VoxMind系统架构详图（状态S_t、think策略、act策略、动态工具管理流程） | 保留: 是 - 核心方法流程图，对理解主-辅双agent交互至关重要。
- **图3**: 核心agent能力示意图（single-task/decomposition/parallel/proactive/feedback/contextual六宫格） | 保留: 否 - 纯能力枚举示意图，无具体数据。
- **图4(a)**: 推理效率对比（w/ vs w/o Auxiliary LLM随工具数变化的延迟曲线） | 保留: 是 - 关键定量结果，直接证明延迟-规模解耦。
- **图4(b)**: 任务性能随工具数量变化（FS/PF在有无Aux LLM下的对比曲线） | 保留: 是 - 关键定量结果，证明动态管理不仅快而且准。
- **图5**: 数据词云（Tool/General对话词汇分布） | 保留: 否 - 次要可视化。
- **图6**: 工具交互数据训练样例（完整多轮对话示例） | 保留: 否 - 示例性内容，附录文字已复述。
- **图7-13**: CoT构建/评估/清洗的系统提示词截图 | 保留: 否 - 提示词文本已在附录中完整给出。

**关键表格数据完整输出**：

*核心能力主结果表（Table 2）*
- Gemini-2.5-pro: Overall 71.51（TS 90.98, PF 75.19, TU 26.87, FC 4.16）
- Gemini-2.5-flash: Overall 68.40（TS 92.48, PF 77.44, TU 31.34, FC 4.10）
- GPT-4o-audio: Overall 54.77（TS 85.71, PF 70.68, TU 0.00, FC 4.22）
- Qwen3-8B+Whisper: Overall 64.00（TS 94.99, PF 68.42, TU 7.46, FC 4.05）
- Kimi-Audio: Overall 54.94（TS 78.45, PF 56.89, TU 13.64, FC 3.62）
- Qwen2.5-Omni: Overall 39.85（TS 78.70, PF 35.84, TU 0.00, FC 2.82）
- StepAudio2: Overall 34.88（TS 78.70, PF 48.87, TU 3.12, FC 1.91）
- **VoxMind: Overall 74.57（TS 98.50, PF 72.18, TU 68.66, FC 3.94）**

*消融实验表（Table 3）*
- w/o think (1:1): Overall 68.83
- w/o think (1:0.5): Overall 70.97
- w/ think (1:1): Overall 71.97
- **w/ think (1:0.5): Overall 74.57**

*VoiceBench表（Table 4）*
- Step-Audio-2: Overall 64.15
- w/o think (1:0.5): Overall 54.80
- w/o think (1:1): Overall 59.72
- w/ think (1:1): Overall 63.62
- **w/ think (1:0.5): Overall 64.21**

*TTS vs Real Speech（附录H）*
- Real Speech: FS 86.00%, PF 60.67%
- TTS Speech: FS 93.33%, PF 67.33%

*延迟-规模解耦（附录I, Table 10）*
- 工具数10: Aux LLM 1.31s, 等待开销 0.00s
- 工具数25: Aux LLM 1.57s, 等待开销 0.00s
- 工具数50: Aux LLM 1.90s, 等待开销 0.015s
- 工具数75: Aux LLM 2.38s, 等待开销 0.013s
- 工具数100: Aux LLM 2.64s, 等待开销 0.005s

*Token开销（附录J, Table 11）*
- Speech输出: THINK 88.0 tokens, Answer 701.2 tokens, 占比 12.6%
- Text输出: THINK 84.4 tokens, Answer 52.6 tokens, 占比 160.5%

### 📸 论文图片

![figure](https://arxiv.org/html/2604.15710v1/x1.png)

![figure](https://arxiv.org/html/2604.15710v1/x2.png)

![figure](https://arxiv.org/html/2604.15710v1/x3.png)


---

[← 返回 2026-04-20 论文速递](/audio-paper-digest-blog/posts/2026-04-20/)
