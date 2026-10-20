<h1 align="center">油价分析 · Oil Price Analysis</h1>

<p align="center"><strong>从价格与库存的时间序列，到事件文本的语义空间。</strong><br>把预测结果、误差和实验过程放在一起看，而不只给出一个数字。</p>

<p align="center">
  <a href="#快速体验">运行看板</a> ·
  <a href="#已保存的实验结果">查看实验</a> ·
  <a href="#复现与路径配置">复现流程</a> ·
  <a href="NLP/README(1).md">NLP 模块</a>
</p>

本项目包含两个独立的分析模块：用 **GRU / LSTM 时序模型**研究下一期价格变化，用 **Word2Vec**探索中文石油事件文本。中英双语 Streamlit 看板把已保存的指标、预测曲线、诊断图和导出文件串起来。

![已保存实验的价格变化真实值与预测值，横轴为 2025 年 10 月至 2026 年 2 月的历史日期](ML/outputs_lstm_wf/final_result_holdout.png)

*仓库中的真实实验图，不是实时行情，也不是交易信号。图中的历史窗口、模型配置与误差见下文。*

## 先看什么

| 你想了解 | 项目提供 |
| --- | --- |
| 模型怎样处理时序数据？ | 周频处理、8 期回看、walk-forward 验证、GRU / LSTM 搜索与集成 |
| 预测错在哪里？ | 真实/预测曲线、残差、校准散点、分折指标和 holdout 诊断 |
| 事件文本里有哪些语义关系？ | jieba 分词、100 维 Word2Vec、PCA 关键词图与相关性分析 |
| 怎样复查结果？ | 已提交的 CSV / JSON 指标、模型包与 Streamlit 导出入口 |

**两个模块目前分开运行。** NLP 词向量并未作为当前 ML 预测器的输入；不要把它理解为已经完成文本与数值特征融合的端到端模型。

## 快速体验

```bash
git clone https://github.com/yuzhang-zhong/oil-price-analysis.git
cd oil-price-analysis
python -m venv .venv

# macOS / Linux
source .venv/bin/activate
# Windows PowerShell: .venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt
python -m streamlit run streamlit_app.py
```

看板默认读取仓库内已保存的结果，不需要先完成长时间训练。四个页面分别是**总览、模型评估、图表看板、导出与文件**，侧边栏可切换中文 / English。

> **先浏览，再重算。** 当前侧边栏的预测刷新、图表重生成和重训按钮仍会调用带有作者本机 Windows 默认路径的脚本。在其他目录或系统上，先按[路径配置](#复现与路径配置)处理；能打开看板不代表这些按钮已经完成跨平台配置。

## 已保存的实验结果

### 模型实际做了什么

1. 以 `WTI_T`、`Stock_T`、`Price_Change`、`Stock_Change` 为基础，构造滞后与滚动特征。
2. 按时间顺序做 4 折 walk-forward 验证，每折验证与测试窗口各 20 个样本，搜索 100 组配置。
3. 导出的包保存了 **1 个 GRU + 4 个 LSTM** 的五模型集成，并训练最近 96 个拟合样本上的专模。
4. 当前包选择的最优配置是 **GRU**；验证得到的混合权重为近期 GRU 专模 **1.0**、集成 **0.0**，随后应用 Ridge 校正。因此，不能把最终结果简单称为“LSTM 集成预测”。

文件名中的 `lstm` 是历史命名；真实结构以 [`final_package_summary.json`](ML/outputs_lstm_wf/final_package/final_package_summary.json) 内的 `model_type` 和权重为准。

### 指标与实验边界

| 已保存结果 | MAE | RMSE | R² | 方向准确率 |
| --- | ---: | ---: | ---: | ---: |
| 4 折测试均值 | 3.5757 | 4.1447 | -0.1143 | 47.5% |
| 最终包的 holdout 窗口 | 2.5464 | 3.0798 | 0.0067 | 75.0% |

来源：[walk-forward 汇总](ML/outputs_lstm_wf/walkforward_summary.json) · [逐折指标](ML/outputs_lstm_wf/walkforward_fold_metrics.csv) · [最终包汇总](ML/outputs_lstm_wf/final_package/final_package_summary.json)。

- 最终包记录 **218 个拟合、20 个验证、20 个 holdout 样本**；[holdout CSV](ML/outputs_lstm_wf/final_package/final_holdout_predictions.csv) 的 `Event_Date` 为 **2025-10-10 至 2026-02-20**。
- 两行来自不同评估阶段与模型处理流程，不能直接当作同口径的提升幅度。最终 holdout 与最后一个 walk-forward 测试窗口重合，不是额外独立的新测试集。
- 20 个样本上的 75% 方向准确率不代表稳定预测能力；接近零的 R² 与较差的跨折均值也应一起阅读。
- 原始事件数据被重索引为周频并前向填充，连续周可能重复观测值。样本量不等于独立市场事件数。

<details>
<summary><strong>展开查看：校准、残差与误差时间线</strong></summary>

![已保存 holdout 的校准散点、残差时间线、绝对误差与分位数对齐诊断](ML/outputs_lstm_wf/holdout_diagnostics_dashboard.png)

继续查看[时间切分图](ML/outputs_lstm_wf/walkforward_splits.png)、[训练曲线](ML/outputs_lstm_wf/training_process_curve.png)与[完整产物目录](ML/outputs_lstm_wf/)。

</details>

## NLP：另一条探索路径

中文事件描述 → jieba 分词 → 100 维 Word2Vec → PCA 关键词分布 → 向量维度与价格变动的相关性分析。

![中文石油事件关键词的 Word2Vec 向量二维 PCA 投影](NLP/outputs/word_vectors.png)

*这是语料中的词向量投影，不是因果图，也不表示某个事件会推动油价上涨或下跌。*

```bash
cd NLP
python -m pip install -r requirements.txt
python oil_price_nlp.py
```

输入默认是 `NLP/data.xlsx`，文本列为 `Event_Description`，变化列为 `Change`。路径、关键词和训练参数见 [`NLP/config.json`](NLP/config.json)，输出写入 `NLP/outputs/`。非 Windows 环境若中文标签缺字，需要配置可用的中文字体。

## 复现与路径配置

### 使用已有模型包

从仓库根目录执行：

```bash
cd ML
python main.py predict --data 4.0_enriched.csv --package-dir outputs_lstm_wf/final_package
```

此命令对**提供的数据快照**运行推理并写入 `predict_next_output.json`，不会自动下载最新行情。检查输出的 `latest_sequence_date`；当前提交的预测文件标注的是 **2026-02-20**，不是今天的预测。只加载你信任的 PyTorch / joblib 模型文件。

### 重新训练或生成图表

以下命令仍在 `ML/` 目录执行：

```bash
python main.py train
python main.py plots
```

运行前必须调整：

| 入口 | 当前需要检查的位置 |
| --- | --- |
| `train` | [`train_lstm_pipeline.py`](ML/train_lstm_pipeline.py) 中 `Config.data_path` 和 `Config.out_dir` |
| `plots` | [`generate_plots.py`](ML/generate_plots.py) 的 `main()` 中 `base_dir` |
| `predict` | 使用上面的 `--data` / `--package-dir`，或修改 [`main.py`](ML/main.py) 的默认值 |
| 看板侧边栏按钮 | 使用这些脚本的默认配置，修改路径后再操作 |

**只有 `predict` 接受上述路径参数，`train` 和 `plots` 不接受 `--data`。** 重训会重新生成模型和结果文件，运行成本取决于硬件；PyTorch 在可用时使用 CUDA，否则使用 CPU。

## 项目地图

```text
oil-price-analysis/
├── streamlit_app.py             # 中英双语结果看板
├── requirements.txt
├── ML/
│   ├── main.py                  # train / predict / plots
│   ├── train_lstm_pipeline.py   # GRU / LSTM、walk-forward 与模型包
│   ├── generate_plots.py
│   ├── 4.0_enriched.csv
│   └── outputs_lstm_wf/         # 真实实验指标、图表、final_package/
└── NLP/
    ├── oil_price_nlp.py
    ├── config.json
    ├── data.xlsx
    └── outputs/                # 词向量、矩阵与图表
```

技术栈：PyTorch、scikit-learn、pandas / NumPy、jieba / gensim、Matplotlib / seaborn / Plotly、Streamlit。完整依赖见 [`requirements.txt`](requirements.txt)。

## 使用范围与许可

本项目用于学习、方法探索与复查历史实验，**不构成投资建议，不提供实时市场预测、收益承诺或交易执行**。数据快照、填充方式、较小评估窗口和环境依赖都限制了结果的外推。

采用 [MIT License](LICENSE)。模块补充说明：[ML](ML/README.md) · [NLP](NLP/README(1).md)；入口、配置与已保存结果以当前代码和对应产物为准。
