# 油价分析 (Oil Price Analysis)

融合机器学习时序预测与 NLP 文本语义建模的油价分析项目，附带 Streamlit 交互式分析前端（支持中英双语）。

<!-- TODO: add screenshot -->

---

## 功能

- **ML 时序预测**：用 LSTM 预测下一期油价变化（`price_change_t1`）；输入严格限定为 `WTI_T` / `Stock_T` / `Price_Change` / `Stock_Change` 及其时序衍生特征；采用 walk-forward 多折验证、Top-N 模型集成、近期窗口专模与 Ridge 误差校正
- **NLP 文本建模**：对中文石油事件文本做 jieba 分词，训练 Word2Vec 词向量（100 维），用 PCA 可视化关键词语义分布，并分析词向量各维度与油价变动的相关性
- **Streamlit 前端**：中英双语切换；四个标签页——总览（核心指标与最新预测）、模型评估（holdout 真实/预测曲线、残差分布、校准图、各折指标表）、图表看板（训练产物 PNG 一览）、导出与文件（关键结果下载）；侧边栏可一键刷新预测、重生成图表、触发完整重训

---

## 快速开始

```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
```

---

## 安装

需要 Python 3 与 pip：

```bash
pip install -r requirements.txt
```

> LSTM 训练依赖 PyTorch；有 GPU 会自动使用 CUDA，否则回退到 CPU。

---

## 用法

### 启动交互前端（推荐）

```bash
streamlit run streamlit_app.py
```

在侧边栏 `Language / 语言` 切换中文 / English；侧边栏按钮分别触发预测刷新、图表重生成与完整重训（底层调用 `ML/main.py`）。

### 单独运行 ML

```bash
cd ML
python main.py train     # 完整训练流程（walk-forward + 超参搜索 + 结果导出）
python main.py predict   # 用最终模型包做下一期预测
python main.py plots     # 补生成描述性与诊断图表
```

> 注意：`train` 与 `predict` 的默认数据路径为 Windows 绝对路径（`train_lstm_pipeline.py` 的 `Config.data_path` 与 `main.py` 的 `--data` 默认值）；在非 Windows 环境运行时请改用 `--data` 指定 CSV，例如：
>
> ```bash
> python main.py predict --data 4.0_enriched.csv --package-dir outputs_lstm_wf/final_package
> ```

### 单独运行 NLP

```bash
cd NLP
pip install -r requirements.txt
python oil_price_nlp.py
```

流程：中文分词 → Word2Vec 训练 → 关键词向量可视化 → 向量维度与油价变动相关性分析；结果写入 `outputs/`。路径与超参数在 `NLP/config.json` 中配置，模块说明见 `NLP/README(1).md`。

---

## 技术栈

- 机器学习：PyTorch（LSTM）、scikit-learn、numpy、pandas、joblib、statsmodels
- 自然语言处理：jieba、gensim（Word2Vec）
- 可视化：matplotlib、seaborn、Plotly
- 前端：Streamlit
- 数据读写：openpyxl、requests

---

## 项目结构

```
oil-price-analysis/
├── streamlit_app.py          # Streamlit 交互前端（中英双语）
├── requirements.txt          # 统一依赖清单
├── LICENSE                   # MIT 协议
├── .gitignore
├── ML/
│   ├── main.py               # ML 统一入口：train / predict / plots
│   ├── train_lstm_pipeline.py# LSTM 训练、超参搜索、walk-forward 验证与结果导出
│   ├── generate_plots.py     # 描述性与诊断图表生成
│   ├── 4.0_enriched.csv      # ML 训练输入数据
│   ├── requirements.txt      # ML 依赖（根目录 requirements.txt 已覆盖）
│   ├── README.md             # ML 模块详细说明
│   └── outputs_lstm_wf/      # 训练产物：指标、预测、图表与最终模型包 final_package/
└── NLP/
    ├── oil_price_nlp.py      # NLP 主脚本：分词、Word2Vec、可视化、相关性分析
    ├── config.json           # 路径、列名、Word2Vec 超参数配置
    ├── data.xlsx             # 中文事件文本输入数据
    ├── requirements.txt      # NLP 依赖（根目录 requirements.txt 已覆盖）
    ├── README(1).md          # NLP 模块说明
    └── outputs/              # NLP 产物：词向量模型、向量矩阵、相关性图表
```

---

## License

本项目采用 MIT 协议，详见 [LICENSE](LICENSE)。
