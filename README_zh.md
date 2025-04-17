[English](README.md) | **中文**

# MGM

[![](https://img.shields.io/pypi/v/microformer-mgm)](https://pypi.org/project/microformer-mgm/) 
[![Downloads](https://pepy.tech/badge/microformer-mgm)](https://pepy.tech/project/microformer-mgm)
![](https://img.shields.io/badge/status-beta-yellow?style=flat) 
![](https://img.shields.io/github/license/HUST-NingKang-Lab/MGM?style=flat) 


**微生物通用模型 (MGM)** 是一个大规模预训练语言模型，专为可解释的微生物组数据分析而设计。MGM 支持多种任务，包括数据准备、模型训练和推理，使其成为微生物组研究的多功能工具。

![MGM Pipeline](pipeline.png)

## 安装

### 从 PyPI 安装

**使用 pip 安装 MGM：**

```bash
pip install microformer-mgm
```

### 从源代码安装

**从源代码安装 MGM：**

```bash
python setup.py install
```

## MicroCorpus-260K

**MicroCorpus-260K** 数据集包含来自 MGnify 的 263,302 个微生物组样本，非常适合训练您自己的 MGM 模型。数据集可在 [OneDrive](https://1drv.ms/f/c/18d7815ed643fcef/EhpVq8yLORVImPIqpsFtHa4BrllBymIa-OWc7jS2vyADzw?e=AzZiLf) 上下载。数据集包括：

- **`MicroCorpus-260K.pkl`**：归一化的微生物组语料库（所有样本的均值和标准差）。
- **`MicroCorpus-260K_unnorm.pkl`**：未归一化的微生物组语料库。
- **`mgnify_biomes.csv`**：数据集中样本的元数据。

### 加载 MicroCorpus-260K

**在 Python 中加载数据集：**

```python
from pickle import load
corpus = load(open('MicroCorpus-260K.pkl', 'rb'))
corpus[0]  # 访问第一个样本（包含 input_ids 和 attention_mask 的字典）
abundance = corpus.data  # 访问丰度数据
```

## 示例笔记本

[mgm_interpretability_guideline.ipynb]（mgm_interpretability_guideline.ipynb）jupyter Notebook提供了一份全面的指南，用于使用MGM执行可解释的微生物组分析。它演示了如何：

*在婴儿微生物组数据集上微调MGM。
*提取样本嵌入以进行可视化。
*计算注意力重量以识别Keystone微生物属。

本笔记本适合寻求MGM的可解释性功能并将其应用于自己的数据集。


## 使用方法

MGM 通过命令行界面 (CLI) 访问，提供多种模式。一般语法为：

```bash
mgm <mode> [options]
```

以下模式分为 **数据准备**、**模型训练** 和 **推理** 三个部分，以便更好地组织。

### 数据准备

#### `construct`

**将丰度数据转换为微生物组语料库，使用系统发育进行归一化，并按属丰度从高到低排序。**

- **输入：** `hdf5`、`csv` 或 `tsv` 格式的丰度数据（特征在行，样本在列）
- **输出：** 包含微生物组语料库的 `.pkl` 文件

**示例：**

```bash
mgm construct -i data/abundance.csv -o data/corpus.pkl
```

> **注意：** 对于 `hdf5` 文件，使用 `-k` 指定键（默认为 `genus`）。

### 模型训练

#### `pretrain`

**使用因果语言建模在微生物组语料库上预训练 MGM 模型。可以选择使用带标签的数据训练生成器。**

- **输入：**
  - 微生物组语料库 (`.pkl`)
  - 可选：标签文件 (`.csv`，两列：样本 ID 和标签)
- **输出：** 预训练的 MGM 模型

**示例：**

```bash
mgm pretrain -i data/corpus.pkl -o models/pretrained_model
mgm pretrain -i data/corpus.pkl -l data/labels.csv -o models/generator_model --with-label
```

> **注意：** 默认加载MicroCorpus-260K上的预训练权重，使用 `--from-scratch` 从头开始训练。如果提供了标签文件训练生成器，tokenizer 和模型嵌入层将被更新。

#### `train`

**使用带标签的数据从头训练 MGM 分类模型。**

- **输入：**
  - 微生物组语料库 (`.pkl`)
  - 标签文件 (`.csv`，两列：样本 ID 和标签)
- **输出：**  MGM 分类模型

**示例：**

```bash
mgm train -i data/corpus.pkl -l data/labels.csv -o models/supervised_model
```

#### `finetune`

**使用带标签的数据对预训练的 MGM 模型进行微调，以适应特定任务。**

- **输入：**
  - 微生物组语料库 (`.pkl`)
  - 标签文件 (`.csv`，两列：样本 ID 和标签)
  - 可选：预训练模型（如果未指定，默认为 MicroCorpus-260K 预训练模型）
- **输出：** 微调后的 MGM 分类模型

**示例：**

```bash
mgm finetune -i data/corpus.pkl -l data/labels.csv -m models/pretrained_model -o models/finetuned_model
```

### 推理

#### `predict`

**使用微调后的 MGM 模型生成预测。可选择与真实标签进行评估。**

- **输入：**
  - 微生物组语料库 (`.pkl`)
  - 可选：标签文件 (`.csv`) 用于评估
  - 监督 MGM 模型
- **输出：** 预测结果 (`.csv`)

**示例：**

```bash
mgm predict -E -i data/corpus.pkl -l data/labels.csv -m models/finetuned_model -o data/predictions.csv
```

> **注意：** 使用 `-E` 和标签文件将预测与真实标签进行比较。

#### `generate`

**使用预训练的 MGM 模型生成微生物组数据。**

- **输入：**
  - 预训练的 MGM 模型
  - 可选：提示文件 (`.txt`，每行一个标签) 用于带标签的生成
- **输出：** 生成微生物组的rank排序 (`.pkl`)

**示例：**

```bash
mgm generate -m models/generator_model -p data/prompt.txt -n 100 -o data/synthetic.pkl
```

> **注意：** 使用 `-n` 指定要生成的样本数量。

#### `reconstruct`

**从排序的语料库中重建丰度数据，可选地训练重构器模型或解码标签。**

- **输入：**
  - 丰度文件（例如 `csv`）用于训练重构器，或训练好的模型检查点
  - 用于重建的排序语料库 (`.pkl`)
  - 可选：生成器模型和提示文件（文本，每行一个标签）用于带标签的数据
- **输出：**
  - 重建的语料库 (`.pkl`)
  - 重构器模型（如果训练）
  - 解码的标签（如果适用）

**示例：**

```bash
mgm reconstruct -a data/abundance.csv -i data/synthetic.pkl -g models/generator_model -w True -o data/reconstructor
mgm reconstruct -r data/reconstructor_model.ckpt -i data/synthetic.pkl -g models/generator_model -w True -o data/reconstructed
```

**更多详细信息，请运行：**

```bash
mgm <mode> --help
```

## 维护者

| 姓名          | 电子邮件                  | 组织                                             |
|---------------|---------------------------|----------------------------------------------------------|
| 张皓鸿 | [haohongzh@gmail.com](mailto:haohongzh@gmail.com)       | 华中科技大学生命科学与技术学院博士生 |
| 康子鑫   | [29590kang@gmail.com](mailto:29590kang@gmail.com)       | 华中科技大学生命科学与技术学院本科生 |
| 宁康     | [ningkang@hust.edu.cn](mailto:ningkang@hust.edu.cn)      | 华中科技大学生命科学与技术学院教授   |
