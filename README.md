# MF703 BTC 预测项目

采用简单协作方式：一人准备公共数据，三人独立完成 AR、Ridge、LightGBM，分别交预测结果，最后用 compare.py 统一比较。下面是组内实验约定，并非教授要求的原文。

## 项目目录

下图展示应完成的结构。**每个个人文件夹至少提交一份 README 和一份可运行代码**，代码可用 `.ipynb` 或 `.py`。标注“待添加/待生成”的文件仅为示例，目前尚未创建。

```text
MF703-Project-Ebaybarebay/
├── README.md                       # 全组协作约定
├── requirements.txt                # 共用依赖
├── config.json                     # 统一配置
├── data/
│   ├── raw/                        # 原始数据，从网盘下载
│   └── processed/                  # 公共数据及评价时间表
├── notebooks/
│   ├── README.md                   # 个人工作区总说明
│   ├── data/
│   │   ├── README.md               # 清洗、特征与划分、运行方法
│   │   ├── prepare_data.ipynb   # 待添加：可运行代码，也可用 .py
│   │   └── *.py                    # 可选：辅助脚本
│   ├── ar/
│   │   ├── README.md               # AR 方法与参数、运行方法
│   │   ├── ar.ipynb             # 待添加：可运行代码，也可用 .py
│   │   └── *.py                    # 可选：辅助脚本
│   ├── ridge/
│   │   ├── README.md               # Ridge 方法与参数、运行方法
│   │   ├── ridge.ipynb          # 待添加：可运行代码，也可用 .py
│   │   └── *.py                    # 可选：辅助脚本
│   └── lightgbm/
│       ├── README.md               # LightGBM 方法与参数、运行方法
│       ├── lightgbm.ipynb       # 待添加：可运行代码，也可用 .py
│       └── *.py                    # 可选：辅助脚本
├── results/
│   ├── README.md                   # 格式及版本说明
│   ├── ar_predictions.csv          # 待添加：AR 预测
│   ├── ridge_predictions.csv       # 待添加：Ridge 预测
│   └── lightgbm_predictions.csv    # 待添加：LightGBM 预测
├── compare.py                      # 待添加：统一评价、记录及绘图
└── comparison/
    ├── README.md                   # 比较记录与结论
    ├── metrics.csv                 # 待生成：RMSE、MAE 比较表
    └── metrics.png                 # 待生成：指标对比图
```

例如，Ridge 负责人需要提交 `notebooks/ridge/README.md` **以及** `ridge.ipynb`（或 `ridge.py`）。


## 全组必须统一的内容

### ① 统一预测任务

令 t 为预测形成时刻，Pₜ 为此时已完成分钟线的收盘价。统一预测期限为十五分钟：

$$
\boxed{\displaystyle y_t=\ln \left(\frac{P_{t+\Delta}}{P_t}\right)}
\qquad \Delta=15 \mathrm{min}
$$

| 要素 | 全组约定 |
|:---|:---|
| 数据版本 | 同一 `raw-v1` 和 `processed-v1`，由数据负责人统一发布 |
| 时间标准 | 全部使用 **UTC**；数据负责人核实原始时间戳含义，统一转换为数据可用时刻 |
| 预测频率 | 每小时 **00、15、30、45 分钟**各预测一次 |
| 目标单位 | **小数对数收益率**，不乘 $100$，不交价格或标准化值 |
| 公共答案 | 数据负责人发布 `evaluation_index.csv`，统一日期、真实值和 `split` |
| 缺口处理 | 按真实十五分钟配对，有效价格才能生成目标，不能跨缺口直接移动十五行 |

### ② 统一训练边界

$$
\underbrace{\mathrm{Train}}_{\mathrm{fit}} \longrightarrow \underbrace{\mathrm{Validation}}_{\mathrm{select}} \longrightarrow \underbrace{\mathrm{Test}}_{\mathrm{report}}
$$

| 环节 | 必须遵守 |
|:---|:---|
| 信息范围 | 只使用预测时刻 $t$ 及之前已经可用的数据 |
| 预处理 | 标准化、填补参数及特征选择只在各折训练期拟合 |
| 参数选择 | 只根据验证结果选参数，不查看测试表现后修改方案 |
| 最终训练 | 用 **2017–2023 年**重训并冻结，测试期不再拟合或调参 |
| 预测样本 | 完整覆盖统一评价时间表，不能自己删除难预测的样本；有缺失先修复，不能静默取交集 |
| 随机性 | 如适用，设 `random_state=42`，记录依赖及参数版本 |

### ③ 统一结果与比较

$$
\boxed{\mathrm{RMSE}\downarrow\qquad\mathrm{MAE}\downarrow}
\qquad \hat y_t^{ \mathrm{baseline}}=0
$$

| 交付项 | 全组约定 |
|:---|:---|
| 预测 CSV | 固定四列：`timestamp,y_true,y_pred,split` |
| 评价指标 | **RMSE 为主，MAE 为辅**，二者均越低越好 |
| 统一比较 | `compare.py` 计算各模型和**零收益基准**的指标、表格及图片 |
| 实验记录 | 数据版本、代码版本、最终参数、运行方式和实验编号 `run_id` |

> **可以各自决定：** 特征、滞后长度、模型参数和模型专属预处理。  
> **必须保持一致：** 预测目标、时间划分、评价日期、真实值和结果格式。

个人修改只保存在自己的文件夹，不覆盖公共数据。不同特征的比较代表完整预测方案的比较，不能只归因于模型种类。

### 固定数据划分

沿用此前约定的时间划分；不增加新的验证流程。

| split | 训练期 | 预测评价期 |
|---|---|---|
| val2021 | 2017–2020 | 2021 |
| val2022 | 2017–2021 | 2022 |
| val2023 | 2017–2022 | 2023 |
| test | 2017–2023 | 2024 年起至数据最后有效目标 |

年份均按 UTC，边界为每年 1 月 1 日 00:00。训练样本的目标结束时刻必须早于下一评价期起点。特征可使用评价期起点之前的历史作为预热。数据负责人根据公共数据质量和公共特征预热规则确定评价时间表并冻结；若某个模型仍有预热困难，先协调修订公共时间表，不能单独删测试样本。测试期可以用当时已知的历史价格构造特征，但不以测试标签调参或重训。

## 目标和两个指标

### 预测目标

$$
y_t=\ln \left(\frac{P_{t+15}}{P_t}\right)
$$

### 评价指标

| 符号 | 含义 |
|:---:|:---|
| $N$ | 同一评价期规定的预测数量 |
| $y_i$ | 公共数据中的真实对数收益率 |
| $\hat y_i$ | 模型对同一时点的预测值 |
| $e_i=\hat y_i-y_i$ | 单次预测误差 |

**RMSE · 均方根误差**：对较大误差惩罚更重，作为主要比较指标。

$$
\boxed{\displaystyle \mathrm{RMSE}=\sqrt{\frac{1}{N}\sum_{i=1}^{N}e_i^2}}
$$

**MAE · 平均绝对误差**：反映每次预测的平均绝对偏差，作为辅助指标。

$$
\boxed{\displaystyle \mathrm{MAE}=\frac{1}{N}\sum_{i=1}^{N}\left|e_i\right|}
$$

两者均以**小数对数收益率**计算，越低越好。分别报告三折验证、合并验证和最终测试；合并验证须拼接全部验证预测后计算，不能简单平均三个 RMSE。测试结果只用于最终报告。

## 每个人交什么

| 负责人 | 个人工作目录 | 必交内容 |
|---|---|---|
| 数据与特征 | notebooks/data/ | 可运行的处理 Notebook 或代码、自己的 README、网盘公共数据及 evaluation_index.csv |
| AR | notebooks/ar/ | 可运行 Notebook 或代码、自己的 README、results/ar_predictions.csv |
| Ridge | notebooks/ridge/ | 可运行 Notebook 或代码、自己的 README、results/ridge_predictions.csv |
| LightGBM | notebooks/lightgbm/ | 可运行 Notebook 或代码、自己的 README、results/lightgbm_predictions.csv |
| 集成人，可由上述一人兼任 | 根目录及 comparison/ | compare.py、最终指标表、图、比较说明 |

**每位负责人都要提交“方法说明 + 可运行代码”。** README 按以下清单填写：

| 说明部分 | 需要写清楚 |
|:---|:---|
| 基本信息 | 负责人、数据版本、代码入口和实验编号 |
| 数据处理 | 缺失值、异常值、标准化方法及拟合时间范围 |
| 特征设计 | 字段、计算公式、滞后长度和滚动窗口 |
| 参数设置 | 最终参数值、候选范围和选择依据 |
| 训练验证 | 日期划分、验证过程及防止未来信息的方法 |
| 如何运行 | Python/依赖版本、执行顺序、输入输出路径 |
| 实验结果 | 预测文件位置、主要发现和局限 |

四个目录已有填写模板。Notebook 或 Python 脚本由负责人添加，未填写项不代表已经完成实验。

## 统一预测文件

每个模型提交一个 UTF-8 CSV，第一行为固定字段名：

```csv
timestamp,y_true,y_pred,split
```

timestamp 使用 ISO 8601 UTC，例如 2024-01-01T00:00:00Z，表示预测形成时刻；不是目标结束时刻。split 仅取 val2021、val2022、val2023、test。y_true、y_pred 均是原始小数对数收益率，不能存价格、百分数或标准化后的值。每行对应一次预测，按时间排序，timestamp 与 split 组合唯一；无空预测、NaN、无穷值。y_true 和日期以公共 evaluation_index.csv 为准；CSV 往返保存应保留足够数值精度。

数据文件超过网页上传限制时，通过网盘交付，results/ 中记录下载地址或组内获取方式、数据版本、文件名；不要截断或删减预测来减小文件。

## 最后如何比较

compare.py 由集成人后续实现，不要求大家各自写评价框架：

1. 读取公共评价时间表及三份 CSV，核对日期、split、真实值、重复行和缺失预测；不一致时停止并列出问题。
2. 计算各模型和零收益基准的 RMSE、MAE，分折、合并验证、测试分别记录。
3. 保存 comparison/metrics.csv，字段至少含 model、split、n、rmse、mae、data_version、run_id。
4. 生成 comparison/metrics.png，以不同子图显示 RMSE、MAE；可补充固定同一时间窗口的真实/预测对数收益率图。
5. 在 comparison/README.md 记录比较日期、输入文件及各模型提交版本、数据版本、结论。新一轮比较另存新 run_id，不覆盖已有实验记录。

本阶段模型比较只使用上述两个指标；交易回测如课程最终要求需要，另行补充，不能用预测误差直接代表交易收益。

## 数据获取与读取

数据使用网盘分发。源 CSV 有 7,206,877 行，字段为 Timestamp、Open、High、Low、Close、Volume。Parquet 转换未清洗、采样或缩减数值精度。

推荐网盘目录：

```text
MF703-data/
├── raw-v1/          # 原始数据，只读下载
│   └── btcusd_1-min_data.parquet
└── processed-v1/    # 数据负责人生成后发布
    ├── 公共处理数据文件
    └── 数据说明.md  # 字段、时间范围、版本、生成方法
```

将完整 Parquet 下载到本地 `data/raw/btcusd_1-min_data.parquet`。配置已改为优先约定此单文件路径。若网盘提供的是八个旧分片，可以使用下列第二种读取方式；不要混合单文件与分片，否则会重复数据。

```python
from pathlib import Path
import pandas as pd

# 从项目根目录运行；Notebook 请先确定项目根路径。
df = pd.read_parquet("data/raw/btcusd_1-min_data.parquet")

# 仅当使用整套八个分片时，改用以下代码：
# parts = sorted(Path("data/raw").glob("btcusd_1-min_data.part-*.parquet"))
# df = pd.concat([pd.read_parquet(p) for p in parts], ignore_index=True)
df["datetime"] = pd.to_datetime(df["Timestamp"], unit="s", utc=True)
```

组员记录使用的 raw 与 processed 版本。公共数据由数据负责人统一发布；模型标准化等专属处理放在模型代码中。网盘链接尚未提供，本 README 不代表数据已上传到网盘。

## GitHub 和网盘怎么共享

仓库：[MF703-Project-Ebaybarebay](https://github.com/Brayden-wei/MF703-Project-Ebaybarebay)。

| 内容 | 位置 |
|---|---|
| 代码、个人 README、可运行 Notebook | GitHub 对应个人文件夹 |
| 原始数据 raw-v1 | 网盘，链接待负责人上传后提供 |
| 公共处理数据 processed-v1 和评价时间表 | 网盘，待数据负责人生成 |
| 预测 CSV | results/；过大时网盘交付并在 results/ 记录 |
| 最终小型指标表和图片 | comparison/ |

负责人通过 Settings → Collaborators → Add people 邀请组员；每人使用自己的账号接受邀请。仓库公开，组内网盘链接可通过群聊发给指定组员，不必公开私有资料访问链接。

首次使用：Code → Download ZIP → 解压 → 网盘下载对应数据到本地 data/ → 安装 requirements.txt 依赖。

上传：进入自己的目录（例如 notebooks/ridge/）→ Add file → Upload files → 拖入 Notebook、README 和辅助代码 → 填提交说明 → Commit directly to the main branch → Commit changes。预测结果另行上传到 results/。同路径同文件名用于更新，先看最新版本，避免覆盖别人修改。网页上传单文件限 25 MiB；.gitignore 不阻止手动网页上传，请自行排除大数据。

Python 建议统一 3.11。有新增依赖时同步 requirements.txt 并在个人 README 记录版本。

## 参考

- [Ridge 与 LightGBM 基础示例](https://www.kaggle.com/code/cookingfunda/crypto-return-prediction-ridge-lightgbm)
- [第三名训练方案](https://www.kaggle.com/code/sugghi/training-3rd-place-solution)

采用公开代码时保留署名及许可说明。使用自己的 BTC 数据、统一目标和时间划分。
