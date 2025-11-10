# OpenCE: 开放上下文工程工具箱

[English](https://github.com/sci-m-wang/OpenCE/blob/main/README.md) | [中文](https://github.com/sci-m-wang/OpenCE/blob/main/README_ZH.md)

### 🚀 项目演进：从 `ACE-open` 到 `OpenCE`

您好！感谢您的关注。本项目正在经历一次激动人心的演进。

本仓库最初是 **`ACE-open`**，一个社区驱动的 **Agentic Context Engineering (ACE)** 论文 (arXiv:2510.04618) 的复现项目，因为原论文并未开源。得益于社区的鼎力支持，本项目迅速获得了 **300+** 颗星！(非常感谢\! 🙏)

海量的 Issues、讨论和 Fork 使命题变得清晰：社区需要的不仅仅是一个论文复现，而是一个更健壮、更标准、可扩展的\*\*“上下文工程 (Context Engineering)” 工具箱\*\*。

因此，本项目决定正式升级。我们在此发起 **OpenCE**：一个全新的、社区驱动的开源项目，致力于构建上下文工程领域的“瑞士军刀”，而最初的 ACE 复现将作为它的第一个核心模块。

### 🌟 OpenCE 的愿景

**OpenCE (Open Context Engineering)** 旨在成为一个模块化、功能强大、易于使用的工具箱，帮助开发者和研究者轻松**实现**、**评估**和**组合**各种前沿的 CE 技术。

**我们的核心原则：**

  * **模块化 (Modular):** 轻松插拔、组合不同的 CE 策略（如 RAG、压缩、Prompting）。
  * **评估驱动 (Evaluation-Driven):** 提供标准化基准，用数据衡量 CE 策略的真实效果。
  * **社区所有 (Community-Owned):** 这不是“我”的项目，这是“我们”的项目。

### 🗺️ 路线图 (Roadmap)

  * **[v0.1 - 基础重构]** (进行中)
      * [ ] 将现有 ACE 代码重构为 OpenCE 的第一个核心模块：`opence.ace`。
      * [ ] 建立清晰的 `CONTRIBUTING.md` 贡献指南。
      * [ ] 迁移并解决 `ACE-open` 仓库的遗留 Issues。
  * **[v0.5 - 核心模块]**
      * [ ] 添加 `opence.compression` (上下文压缩) 等新模块。
      * [ ] 引入 `opence.evaluation` (一个基础的 CE 评估框架)。
  * **[v1.0 - 生态扩展]**
      * [ ] 与 LangChain / LlamaIndex 等生态的深度集成。
      * [ ] ... 更多功能，由社区决定！

### 🤝 我们正在寻找你！(Call for Contributions)

一个人走得快，一群人走得远。为了实现 OpenCE 的愿景，我们迫切需要您的帮助。

我们正在寻找：

  * **开发者 (Developers)**: 构建新功能、修复 Bug。
  * **研究者 (Researchers)**: 帮助我们集成最新的 CE 论文。
  * **文档贡献者 (Doc Writers)**: 帮助我们撰写清晰易用的文档。

**如何开始贡献？**

1.  阅读我们的 **[CONTRIBUTING.md](link-to-contributing-guide)** (即将推出)。
2.  寻找 **[Good First Issue](link-to-issues)** (适合新手的任务) 标签。

-----

## 核心模块：ACE 框架 (我们故事开始的地方)

*(This module is the reproduction that started it all)*

本模块是 [Agentic Context Engineering (ACE)](https://arxiv.org/abs/2510.04618) 论文方法的实现框架。

代码设计遵循原论文：

  * 上下文 (Contexts) 是由“条目 (bullet entries)”构成的结构化手册 (playbooks)，每个条目都有“有益/有害”计数器。
  * 三种 Agent 角色 (Generator, Reflector, Curator) 通过增量“Deltas 更新”进行交互。
  * 离线 (Offline) 和在线 (Online) 适应循环支持多轮训练和测试时的持续学习。

关于该方法的精炼总结，请参阅 [docs/method\_outline.md](https://github.com/sci-m-wang/OpenCE/blob/main/docs/method_outline.md)。

### 项目结构

```
ace/         # v0.1 将重命名为 opence/ace: 核心库模块
tests/       # 轻量级回归测试
docs/        # 关于论文方法的技术笔记
scripts/     # (新增) 示例运行脚本
```

### 快速开始

确保您安装了 Python 3.9+ (开发环境使用 3.12)。

(可选) 创建并激活虚拟环境。

运行单元测试：

```bash
python -m unittest discover -s tests
```

### 使用示例

这是一个使用 `DummyLLMClient` (虚拟 LLM) 的最小离线适应循环：

```python
import json
from ace import (
    Playbook, DummyLLMClient, Generator, Reflector, Curator,
    OfflineAdapter, Sample, TaskEnvironment, EnvironmentResult
)

# 定义一个玩具任务环境
class ToyEnv(TaskEnvironment):
    def evaluate(self, sample, generator_output):
        gt = sample.ground_truth or ""
        pred = generator_output.final_answer
        feedback = "correct" if pred == gt else f"expected {gt} but got {pred}"
        return EnvironmentResult(feedback=feedback, ground_truth=gt)

client = DummyLLMClient()

# 为3个 Agent 角色预设好返回的 JSON 响应
client.queue(json.dumps({"reasoning": "...", "bullet_ids": [], "final_answer": "42"}))
client.queue(json.dumps({"reasoning": "...", "error_identification": "", "root_cause_analysis": "",
                         "correct_approach": "", "key_insight": "Remember 42.", "bullet_tags": []}))
client.queue(json.dumps({"reasoning": "...", "operations": [{"type": "ADD", "section": "defaults",
                         "content": "Answer 42 when in doubt.", "metadata": {"helpful": 1}}]}))

adapter = OfflineAdapter(
    playbook=Playbook(),
    generator=Generator(client),
    reflector=Reflector(client),
    curator=Curator(client),
)
samples = [Sample(question="Life?", ground_truth="42")]

adapter.run(samples, ToyEnv(), epochs=1)
```

### 扩展至完整实验

1.  **实现 `LLMClient` 子类**：包装您选择的模型 API (例如 OpenAI, DeepSeek)。
2.  **提供任务特定的 Prompts**：参见 `ace/prompts.py`，或根据您的领域进行定制。
3.  **构建 `TaskEnvironment` 适配器**：运行您的基准测试工作流 (例如 AppWorld ReAct agent, FiNER/Formula 评估)。
4.  **配置循环**：使用 `OfflineAdapter.run` 和 `OnlineAdapter.run`，并按原论文所述配置多轮 (epochs)。
5.  **换用真实 LLM**：例如，使用本地模型权重并指定 GPU：
    ```bash
    CUDA_VISIBLE_DEVICES=2,3 python scripts/run_local_adapter.py
    ```
    (请参阅 `scripts/` 中的最小配置示例。)
