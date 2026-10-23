# MF703 BTC 预测项目

采用简单协作方式：一人准备公共数据，三人独立完成 AR、Ridge、LightGBM，分别交预测结果，最后用 compare.py 统一比较。

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
| 时间标准 | 表里统一写 UTC 时间。例如北京时间 `2024-01-01 08:00`，在表里写 `2024-01-01 00:00`；见下方例子 |
| 预测频率 | 每小时 **00、15、30、45 分钟**各预测一次 |
| 目标单位 | **小数对数收益率**，不乘 $100$，不交价格或标准化值 |
| 公共答案 | 数据负责人发布 `evaluation_index.csv`，统一日期、真实值和 `split` |
| 缺口处理 | 按真实十五分钟配对，有效价格才能生成目标，不能跨缺口直接移动十五行 |

**时间举例：** 北京时间比 UTC 快 8 小时。同一次预测，大家都按下表写，不能有人填北京时间、有人填电脑的本地时间：

| 事情 | 北京时间，仅用于理解 | 结果表里使用的 UTC 时间 |
|---|---|---|
| 发出预测 | 2024-01-01 08:00 | 2024-01-01 00:00 |
| 预测十五分钟后的情况 | 2024-01-01 08:15 | 2024-01-01 00:15 |

这一行结果的 `timestamp` 填 `2024-01-01 00:00`，意思是“在这个时刻预测接下来十五分钟”。**如果原始数据已经转成 UTC，不要再减 8 小时。** 原始时间戳的含义由数据负责人核实后统一处理。

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
| 实验记录 | 在个人 README 写清使用的数据、参数、运行方式和生成结果的日期 |

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

## 每个模型要交的结果表

**每个模型交一个 CSV 表格。一行就是一次预测，一共固定四列。** CSV 可以用 Excel 打开，不需要另做复杂文件。

| 列名，第一行照此填写 | 这一列放什么 | 谁来提供 |
|---|---|---|
| `timestamp` | 做出预测的时间，统一用 UTC，格式如 `2024-01-01 00:00` | 数据负责人提供，大家保留 |
| `y_true` | 从这个时刻到十五分钟后，实际发生的对数收益率 | 数据负责人提供，大家保留 |
| `y_pred` | 你的模型预测的这十五分钟对数收益率 | 每位模型负责人自己计算 |
| `split` | 这行属于哪一段：`val2021`、`val2022`、`val2023` 或 `test` | 数据负责人提供，大家保留 |

**例如，Ridge 的结果表如下。数字只是格式示例，不是真实实验结果：**

| timestamp | y_true | y_pred | split |
|---|---:|---:|---|
| 2024-01-01 00:00 | 0.0010 | 0.0008 | test |
| 2024-01-01 00:15 | -0.0005 | -0.0002 | test |
| 2024-01-01 00:30 | 0.0003 | 0.0004 | test |
| … | … | … | … |

- 第一行：00:00 时预测 00:00–00:15 的对数收益率。
- 第二行：00:15 时预测 00:15–00:30 的对数收益率。
- 后面照这个方式，每十五分钟一行。所有规定的验证和测试时点都要有结果。
- 收益率写小数，例如 `0.0010`，不要写成价格、百分数或标准化后的数字。

**最简单的交付办法：** 数据负责人先给大家一份有 `timestamp、y_true、split` 的表，每位模型负责人加上自己的 `y_pred` 列，再按上面的四列顺序保存。时间、真实值和所属阶段不要改，不要随意删行；预测没算出的行先修好再提交。保存时保留数值精度，不要只保留示例中的四位小数。

| 模型 | 保存的文件 |
|---|---|
| AR | `results/ar_predictions.csv` |
| Ridge | `results/ridge_predictions.csv` |
| LightGBM | `results/lightgbm_predictions.csv` |

文件过大不能传 GitHub 时，完整文件放网盘，在 `results/README.md` 记下文件名和组内获取方式，不要删掉部分预测来缩小文件。

## 最后统一比较什么

集成人写一个 `compare.py`，读取三份结果表，检查大家的时间和真实值是否一样，再计算 RMSE、MAE。每个人不必再写一套比较程序。

**最终比较表固定四列：模型、评价阶段、RMSE、MAE。每行是一种模型在某个阶段的成绩。** 例如最终测试部分如下：

| model（模型） | split（评价阶段） | rmse | mae |
|---|---|---:|---:|
| 零收益基准，始终预测 0 | test | 待计算 | 待计算 |
| AR | test | 待计算 | 待计算 |
| Ridge | test | 待计算 | 待计算 |
| LightGBM | test | 待计算 | 待计算 |

验证部分按同样格式填写：`split` 分别为 `val2021`、`val2022`、`val2023`、`validation_all`（三段验证结果合在一起）。连同 `test`，共五个阶段；四种模型/基准各一行，因此完整比较表共 **20 行结果，不含表头**。RMSE 和 MAE 均越低越好。

| 最后保存什么 | 文件位置 |
|---|---|
| 上面的比较表 | `comparison/metrics.csv` |
| RMSE、MAE 对比图，各画一个子图 | `comparison/metrics.png` |
| 用了哪些结果文件、比较日期和主要结论 | `comparison/README.md` |

再次比较时另建一个以日期和编号命名的文件夹保存，避免覆盖旧结果。当前只规定这两个预测指标；如课程还需要交易回测，另行补充。

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
