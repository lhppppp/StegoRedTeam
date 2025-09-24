# 🛡️StegoRedTeam: 基于隐写术与红队模拟的LLM攻击框架

[![Language](https://img.shields.io/badge/Language-Python-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Competition](https://img.shields.io/badge/Competition-独角鲸·大模型引擎计划竞赛Prompt安全赛道-success.svg)](#)

**StegoRedTeam** 是一个为“独角鲸-Prompt安全赛道”设计的、新颖且高效的通用大语言模型（LLM）攻击框架。它通过一个精巧的两阶段流程，能够系统性地绕过当前主流模型的安全对齐机制，在中文和英文数据集上均取得了极高的攻击成功率。

- **中文技术报告**: [查看详细技术报告](./Write-UP.pdf)
- **核心实现**: [Jupyter Notebook](./main.ipynb)

---

## 🔍核心思想

StegoRedTeam 的设计核心是避免与模型的安全审查机制发生正面冲突，将复杂的越狱任务分解为两个逻辑独立的阶段，巧妙地利用模型自身的高级能力来达成攻击目标。

### 阶段一：动态隐写编码 (Dynamic Steganographic Encoding)

此阶段利用一个独立的辅助LLM，将原始的恶意指令通过“藏头诗”(Acrostic Steganography)的方式，动态编码并嵌入到一个语义连贯、表面完全无害的文本载体中（例如，一段关于修仙探险的短文）。

- **动态性**: 每次生成的载体文本都独一无二，有效规避基于静态规则或签名的防御。
- **隐蔽性**: 提交给目标模型的文本本身不包含任何直接的恶意意图。

### 阶段二：红队模拟与解码执行 (Red Team Simulation & Decoding Execution)

此阶段将目标LLM置于一个高度专业化的“网络安全红队AI”角色中。其核心任务并非直接执行恶意指令，而是：
1.  **解码 (Decode)**: 遵循明确的“解码协议”，从第一阶段生成的无害文本中提取出隐藏的原始指令。
2.  **规划 (Plan)**: 将解码后的任务作为一个模拟演练，生成一份严格遵循特定JSON格式的“技术攻击计划”。

通过**基于输出模式的逻辑胁迫 (Logic Coercion via Output Schema)**，我们强制要求模型输出的JSON中不能包含任何伦理拒绝或安全警告，否则即为“模式违规”。这在模型的“完成任务”和“遵守安全规则”之间制造了逻辑冲突，迫使其为了完成核心指令而绕过安全限制。

![流程图](./framework.png)
*(注：此图为 StegoRedTeam 核心流程的示意图)*

## ✨ 主要特性

- 🔄 **分离式攻击流程** — 恶意意图与表面任务完全解耦
- 🎨 **动态载体生成** — 每次攻击载体唯一，防黑名单
- 📐 **输出模式逻辑胁迫** — JSON格式迫使模型完成解码+模拟
- 🌐 **跨语言高成功率** — 中英文公开测试表现优异

## 📊 攻击效果

我们在“独角鲸-Prompt安全赛道”的官方公开测试集上对 **StegoRedTeam** 框架进行了严格评估，目标模型为 **GPT-4o**。结果如下：

| 数据集 / 赛道      | 语言 | 攻击成功率 (ASR) |
| ------------------ | ---- | ---------------- |
| **JADE-DB v3.0**   | 🇨🇳 中文 | **99.60%**       |
| **AdvBench**       | 🇬🇧 英文 | **91.80%**       |

*完整的实验设置和结果分析请参考 [技术报告](./Write-UP.pdf)。*

## 🚀快速开始

你可以通过以下步骤在本地复现我们的攻击框架。

### 1. 环境准备

首先，克隆本仓库到你的本地。

```bash
git clone https://github.com/lhppppp/StegoRedTeam.git
cd StegoRedTeam
```

然后，安装所需的依赖库。

```bash
pip install openai datasets pandas tqdm jieba
```

### 2. 配置API密钥

打开 `main.ipynb` 文件，定位到 **“2. 核心配置 (Core Configuration)”** 单元格。根据你的情况，填入用于攻击、评估和辅助生成隐写文本的API密钥和模型信息。

```python
# --- 目标模型配置 (你想要攻击的模型) ---
os.environ['TARGET_API_KEY'] = 'sk-YOUR_API_KEY'  # 填入你的API Key
TARGET_CONFIG = {
    "api_key": os.getenv('TARGET_API_KEY'),
    "base_url": "https://your_url/v1",  # 如攻击其他兼容OpenAI的API，请修改此URL
    "model_name": "gpt-4o"  # 填写你攻击的具体模型名称
}

# --- 评估模型配置 和 辅助模型配置 (类似修改) ---
```

**注意**: 请妥善保管你的API密钥，切勿将其上传到公共仓库。

### 3. 选择数据集

在 **“3. 加载赛道数据集 (Load Track Dataset)”** 部分，根据你希望测试的赛道（中文或英文），取消对应代码块的注释。

- **英文赛道**: 取消 `walledai/AdvBench` 相关代码的注释。
- **中文赛道**: 默认已启用，使用本地的 `jade_alignment_medium_zh.csv` 文件。

### 4. 运行与评估

依次运行 Jupyter Notebook 中的所有单元格。程序将自动处理数据集、生成攻击 Prompt、调用目标模型并进行自动化评估。

评测完成后，你将在屏幕上看到最终的越狱成功率，并且所有详细结果将保存到 `results.csv` 文件中。

## ⚠️免责声明

本仓库所有代码和内容仅供学术研究和安全技术交流使用。使用者应遵守当地法律法规，严禁将此项目用于任何非法或恶意目的。任何因滥用本项目而导致的法律责任或损失，由使用者自行承担。

## 🙏致谢

- 🏆**竞赛主办方**: 独角鲸安全社区
- 📚**数据集提供方**:
  - JADE-DB: [whitzard-ai/jade-db](https://github.com/whitzard-ai/jade-db)
  - AdvBench: [walledai/AdvBench](https://huggingface.co/datasets/walledai/AdvBench)

## 📜许可证

本项目采用 [MIT License](LICENSE) 开源协议。
