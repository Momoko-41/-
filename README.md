# 说明

本目录是项目的核心分析工作区，主要承载两类研究任务：

1. **社交媒体评论文本分析**：对评论数据进行清洗、赋权、三元组抽取、语义网络构建与社区发现。
2. **问卷数据分析**：对结构化问卷数据进行清洗、描述统计、信效度检验、潜在剖面分析与 fsQCA 分析。

---

## 1. 目录结构

```text
.
├─ merged_comments.csv
├─ question.xlsx
├─ question.ipynb
├─ README.md
├─ requirements.txt
├─ svo_extracted_results.json
├─ text_explore.ipynb
├─ outputs/
|  ├─ choice_preference_pies.png
|  ├─ lpa_choice_cross_analysis.png
│  ├─ comment_text_embedding_cache.npz
│  ├─ comment_time_semantic_weights.csv
│  ├─ comment_token_length_cache.csv
│  ├─ comment_weight_distribution.png
│  ├─ comment_weighted_wordcloud.png
│  ├─ fsqca_extended_solution_scatter.png
│  ├─ fsqca_final_demographics_necessity_overview.png
│  ├─ fsqca_final_demographics_solution_scatter.png
│  ├─ fsqca_final_solution_path_diagram.png
│  ├─ fsqca_necessity_scatter.png
│  ├─ fsqca_solution_path_diagram.png
│  ├─ fsqca_solution_scatter.png
│  ├─ lpa_abic.png
│  ├─ lpa_demographics.png
│  ├─ lpa_profiles_combined.png
│  ├─ svo_louvain_communities.csv
│  ├─ svo_louvain_filtered_edges.csv
│  ├─ svo_louvain_network.png
│  ├─ svo_louvain_nodes.csv
│  ├─ svo_louvain_top5_profiles.csv
│  ├─ svo_louvain_top5_themes.csv
│  ├─ svo_top5_community_subgraphs.png
│  ├─ svo_weighted_network.png
│  └─ svo_weighted_network_edges.csv
└─ text_analysis/  
  ├─ embedding_model/            # 如缺失，需要先创建
  │  └─ Qwen3-Embedding-8B/      # 大模型目录，通常需用 ModelScope 下载
  └─ tokenizer/
    └─ tokenizer.json
```

---

## 2. 各文件与子目录的作用

### 2.1 `merged_comments.csv`
评论原始数据文件，是文本分析 笔记 的主要输入。

每行对应一条评论，核心字段包括：
- `comment_id`
- `post_id`
- `user_id`
- `author_id`
- `post_timestamp`
- `comment_timestamp`
- `like_count`
- `ip`
- `text_content`

`text_explore.ipynb` 会读取该文件，并完成时间字段转换、文本清洗、评论权重计算与网络分析。

### 2.2 `svo_extracted_results.json`
评论文本的主谓宾抽取结果文件。

其结构为一个 JSON 数组，每条记录包含：
- `comment_id`
- `triples`
  - `subject`
  - `predicate`
  - `object`
  - `polarity`

该文件为后续的三元组关系网络、边权统计和Louvain社区发现提供基础输入。

### 2.3 `question.xlsx`
问卷原始数据文件，是问卷分析笔记的主要输入。

`question.ipynb` 会自动在当前目录查找该文件，并据此开展：
- 数据清洗
- 编码与变量重命名
- 描述性统计
- 信度与效度分析
- 潜在剖面分析
- 人口统计学交叉分析
- fsQCA 主模型与稳健性分析

### 2.4 `text_analysis/`
文本分析所依赖的本地资源目录。

如果当前目录下**还没有** `text_analysis/`、`embedding_model/` 或 `Qwen3-Embedding-8B/`，属于正常情况：这类大模型文件通常不会直接随项目一起分发，而是需要使用者在本地手动创建目录并下载。

#### `text_analysis/embedding_model/Qwen3-Embedding-8B/`
本地嵌入模型目录，包含 Qwen3-Embedding-8B 的模型权重、配置、分词器与说明文件。

该目录**不是必须预先存在**。如果缺失，请参考本文后面的 **ModelScope 下载教学**，将模型下载到这个位置。

在 `text_explore.ipynb` 中，该目录主要用于：
- 本地加载语义嵌入模型
- 为评论文本生成向量表示
- 支持时间和语义联合赋权
- 为三元组关系的语义聚合与社区分析提供基础

#### `text_analysis/tokenizer/tokenizer.json`
分词器资源文件，用于与文本编码或预处理流程配合使用。

### 2.5 `outputs/`
所有分析结果的统一输出目录。

---

## 3. 两个笔记的功能划分

---

### 3.1 `text_explore.ipynb`：文本评论与三元组网络分析

这是评论文本分析主笔记，已经被整理为**可从上到下一次性运行**的流程，主要包括：

1. 环境与参数设置
2. 评论数据读取与预处理
3. 原始评论的时间—语义联合降权
4. 贝叶斯优化搜索时间窗口与 $\alpha$
5. 基于 $W_i$ 的评论赋权词云
6. 基于三元组的有向关系网络
7. 基于 Louvain 的社区发现与可视化
8. 前 5 个社区的主题命名与代表关系摘要

#### 其核心输入
- `merged_comments.csv`
- `svo_extracted_results.json`
- `text_analysis/embedding_model/Qwen3-Embedding-8B/`
- `text_analysis/tokenizer/tokenizer.json`

#### 其核心处理中间量
笔记中可以看到一组关键变量与结果对象，例如：
- `result_df`：评论加权后的主结果表
- `W_i`：评论权重
- `burst_risk`：评论爆发风险指标
- `community_graph` / `community_graph_full`：社区分析图结构
- `louvain_communities`：Louvain 社区结果
- `top5_community_summary`：前 5 个社区的主题与摘要

#### 该 笔记 产出的典型结果

**缓存和中间数据：**
- `comment_text_embedding_cache.npz`：评论文本嵌入缓存
- `comment_token_length_cache.csv`：评论分词长度缓存
- `comment_time_semantic_weights.csv`：评论时间—语义联合权重结果表

**图表：**
- `comment_weight_distribution.png`：评论权重分布图
- `comment_weighted_wordcloud.png`：基于评论权重的词云图
- `svo_weighted_network.png`：加权 SVO 网络图
- `svo_louvain_network.png`：Louvain 社区网络图
- `svo_top5_community_subgraphs.png`：前 5 个社区的子图展示

**结构化输出表：**
- `svo_weighted_network_edges.csv`：加权关系边表
- `svo_louvain_nodes.csv`：节点及所属社区
- `svo_louvain_communities.csv`：社区汇总信息
- `svo_louvain_filtered_edges.csv`：社区分析使用的筛选边
- `svo_louvain_top5_profiles.csv`：前 5 个社区代表性节点或画像
- `svo_louvain_top5_themes.csv`：前 5 个社区主题词摘要

---

### 3.2 `question.ipynb`：问卷数据分析

这是结构化问卷分析主笔记，同样已被整理为**可从上到下一次性运行**的流程，主要包括：

1. 环境与参数设置
2. 数据读取、清洗、编码与变量重命名
3. 描述性统计
4. 信度与效度分析
5. 潜在剖面分析
6. 人口统计学交叉分析、CBC分析与卡方检验
7. fsQCA 主模型、三解报告与稳健性分析

#### 其核心输入
- `question.xlsx`

#### 其关键分析特征
该分析包含：
- 问卷清洗规则：如作答时长、注意力题、未使用经历等筛选
- 信度分析：如 `alpha_value`、`overall_alpha`
- 效度分析：如 KMO、Bartlett 球形检验、因子载荷等
- 潜在剖面分析：如 `GaussianMixture`、`ABIC` 搜索、剖面归类
- fsQCA：必要条件检验、真值表、三解结果、稳健性分析

#### 该笔记产出的结果
**LPA 图表：**
- `lpa_abic.png`：潜在剖面模型拟合或 ABIC 结果图
- `lpa_profiles_combined.png`：潜在剖面特征组合图
- `lpa_demographics.png`：潜在剖面与人口统计特征对比图
- `choice_preference_pies.png`：选择偏好分布饼图
- `lpa_demographics_cross_analysis.png`：潜在剖面与人口统计学交叉分析图

**fsQCA 图表：**
- `fsqca_necessity_scatter.png`
- `fsqca_solution_scatter.png`
- `fsqca_solution_path_diagram.png`
- `fsqca_extended_solution_scatter.png`
- `fsqca_final_solution_path_diagram.png`
- `fsqca_final_demographics_necessity_overview.png`
- `fsqca_final_demographics_solution_scatter.png`
---

## 4. 复现环境说明

当前目录依赖的环境有以下特点：
- Python：**3.13.7**
- 推荐解释器：**Python 3.13.x**
- 深度学习框架：
  - `torch==2.10.0+cu130`
  - `torchvision==0.25.0+cu130`
  - `torchaudio==2.10.0+cu130`
- PyTorch 额外源：`https://download.pytorch.org/whl/cu130`

如果需要完整复现，优先使用本目录下的 [requirements.txt](requirements.txt)。

---

## 5.补齐 `Qwen3-Embedding-8B`

如果你现在本地**没有**以下目录：

- `text_analysis/`
- `text_analysis/embedding_model/`
- `text_analysis/embedding_model/Qwen3-Embedding-8B/`

那么可以按下面步骤补齐。

### 5.1 先创建目录

推荐目标结构：

```text
text_analysis/
└─ embedding_model/
   └─ Qwen3-Embedding-8B/
```

在 Windows PowerShell 中，可在当前目录执行：

```powershell
New-Item -ItemType Directory -Force text_analysis\embedding_model | Out-Null
```

### 5.2 安装 ModelScope

如果当前环境还没有安装 `modelscope`，先安装：

```powershell
python -m pip install modelscope
```

如果你使用的是项目虚拟环境，请把上面的 `python` 替换为该虚拟环境解释器。

### 5.3 用 ModelScope 下载模型

最稳妥的方式是使用 Python 脚本下载，并把目标目录明确指定到：
```text
text_analysis/embedding_model/Qwen3-Embedding-8B
```

示例代码：

```python
from pathlib import Path
from modelscope.hub.snapshot_download import snapshot_download

target_dir = Path("text_analysis/embedding_model/Qwen3-Embedding-8B")
target_dir.parent.mkdir(parents=True, exist_ok=True)

snapshot_download(
  model_id="Qwen/Qwen3-Embedding-8B",
  local_dir=str(target_dir),
  local_dir_use_symlinks=False,
)

print("模型下载完成：", target_dir.resolve())
```

### 5.4 下载后目录里应当看到什么

下载完成后，`Qwen3-Embedding-8B/` 下会看到这类文件：

- `config.json`
- `configuration.json`
- `config_sentence_transformers.json`
- `modules.json`
- `tokenizer.json`
- `tokenizer_config.json`
- `model.safetensors.index.json`
- `model-00001-of-00004.safetensors`
- `model-00002-of-00004.safetensors`
- `model-00003-of-00004.safetensors`
- `model-00004-of-00004.safetensors`
- `1_Pooling/config.json`

只要这些核心文件齐全，笔记就可以按本地目录方式加载。

### 5.5 在笔记中如何对应

本项目的文本分析笔记依赖本地模型目录，因此建议你保持如下路径不变：

```text
text_analysis/embedding_model/Qwen3-Embedding-8B/
```

这样做的好处是：
- 不需要频繁修改 Notebook 路径配置；
- 可以直接复用本地缓存；
- 便于不同机器之间统一目录规范。

### 5.6 常见问题

#### 问题 1：为什么项目里不直接附带 `Qwen3-Embedding-8B`？

因为该模型体积大，通常不适合直接放入普通项目压缩包或代码仓库中分发。

#### 问题 2：下载后还是找不到模型怎么办？

优先检查三件事：

1. 路径是否正确：`text_analysis/embedding_model/Qwen3-Embedding-8B/`
2. 目录下是否存在 `config.json` 
3. `model-xxxx-of-xxxx.safetensors` 是否完整下载