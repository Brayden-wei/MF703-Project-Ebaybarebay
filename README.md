# MF703 BTC 预测项目

使用 BTC 分钟级 OHLCV，研究历史信息对未来十五分钟对数收益率的预测能力，并将预测转换为价格。当前只提供原始 Parquet 数据、配置和文件夹说明。

## 项目目录结构

下图展示仓库当前结构。除原始数据、配置与依赖文件外，各目录只保留 README.md；Python 代码和 Notebook 由大家自行创建。

```text
MF703-Project-Ebaybarebay/
├── README.md                         # 项目总说明、分工、目标与评价公式
├── requirements.txt                  # 全组共用的 Python 依赖
├── config.json                       # 数据路径、预测期限、日期划分与随机种子
├── .gitignore                        # 排除本地环境、缓存与生成结果
│
├── data/                             # 数据负责人维护
│   ├── README.md                     # 数据目录说明
│   ├── raw/                          # 原始数据，仅做格式转换，保留原样
│   │   ├── README.md                 # 分片说明与合并读取示例
│   └── processed/                    # 后续放清洗数据、公共特征、目标与样本划分
│       └── README.md
│
├── src/                              # 后续放正式、可复用的 Python 代码
│   ├── README.md                     # 清洗、特征、基准、评价与回测代码的职责
│   └── models/                       # 三位模型负责人维护各自模型
│       └── README.md                  # AR、Ridge、LightGBM 的代码存放说明
│
├── notebooks/                        # 后续放个人实验与最终展示 Notebook
│   └── README.md                     # 负责人自行创建 .ipynb 文件
│
└── outputs/                          # 后续放模型运行产生的结果
    ├── README.md
    ├── models/                       # 训练模型、预处理器与参数
    │   └── README.md
    ├── predictions/                  # 按统一日期和字段保存的样本外预测
    │   └── README.md
    ├── tables/                       # 模型比较、特征消融与回测结果表
    │   └── README.md
    └── figures/                      # 探索、预测、净值与回撤图
        └── README.md
```

**数据与代码如何衔接：**

```text
data/raw/ 原始分钟数据
        │ 数据负责人清洗并构造公共特征和目标
        ▼
data/processed/ 公共建模数据
        │ 三位模型负责人读取同一版本
        ├── AR
        ├── Ridge
        └── LightGBM
                │ 统一目标、日期与预测文件格式
                ▼
outputs/predictions/
        │ 统一评价与回测
        ├── outputs/tables/  指标及比较表
        └── outputs/figures/ 结果图表
```

## 文件夹用途与分工

| 目录 | 放什么 | 负责人 |
|---|---|---|
| `data/` | 数据管理说明 | 数据与特征负责人 |
| `data/raw/` | 原始分钟数据的八个 Parquet 分片及读取说明，保留原样 | 数据与特征负责人 |
| `data/processed/` | 清洗数据、公共特征、目标和时间划分 | 数据与特征负责人 |
| `src/` | 正式的数据处理、基准、统一评价和回测代码 | 数据负责人及组内集成人 |
| `src/models/` | AR、Ridge、LightGBM 的专属处理、训练和预测代码 | 三位模型负责人 |
| `notebooks/` | 个人实验及最终展示 Notebook，由负责人自行创建 | 全体成员 |
| `outputs/` | 生成结果的分类管理说明 | 全体成员 |
| `outputs/models/` | 模型和训练期拟合的预处理器 | 模型负责人 |
| `outputs/predictions/` | 按日期对齐的基准及模型预测文件 | 模型负责人 |
| `outputs/tables/` | 比较、消融、分时期和回测表格 | 集成人 |
| `outputs/figures/` | 预测、特征重要性、净值和回撤图 | 全体成员 |

每个目录的 README.md 说明具体存放规则。不预建 `.py` 或 `.ipynb`，文件命名和代码实现由负责人安排。公共数据修正由数据负责人统一完成；模型专属标准化、滞后项和特征处理由模型负责人维护，不覆盖原始或公共数据。

## 数据获取与读取

仓库 `data/raw/` 已包含 `btcusd_1-min_data.part-000.parquet` 至 `part-007.parquet`，可以随 Code → Download ZIP 一次下载。源文件为 btcusd_1-min_data.csv，共 7,206,877 行，字段为 Timestamp、Open、High、Low、Close、Volume。

这是格式转换，没有进行清洗、采样、特征计算或精度缩减。原始单文件压缩后仍约 109 MiB，故按原始行序每一百万行分片，最后一片为余下记录；共八片。不要只读取其中一片作为完整数据。

```python
from pathlib import Path
import pandas as pd

# 从项目根目录运行；Notebook 中请先确定项目根路径。
raw_dir = Path('data/raw')
parts = sorted(raw_dir.glob('btcusd_1-min_data.part-*.parquet'))
df = pd.concat([pd.read_parquet(path) for path in parts], ignore_index=True)
df['datetime'] = pd.to_datetime(df['Timestamp'], unit='s', utc=True)
```

## 预测目标

令 $P_t$ 为时点 $t$ 已完成分钟线的收盘价。预测未来十五分钟的对数收益率：

$$
y_t=\ln\left(\frac{P_{t+15}}{P_t}\right)
$$

模型输出 $\hat y_t$ 后，转换为预测价格和普通收益率：

$$
\hat P_{t+15}=P_t\exp(\hat y_t),\qquad
\hat r_t=\exp(\hat y_t)-1
$$

对数收益率与普通收益率不同；二者在变化很小时近似相等。模型训练、RMSE、MAE 和 Pearson 均统一使用对数收益率，不能混用。对数收益率没有百分比单位，0.001 在小幅变化下约对应 0.1% 的普通收益率。

每十五分钟形成一次预测。仅使用当时已经完成的数据；按真实时间配对目标，不能跨缺口直接移动十五行。

## 统计评判标准与公式

设 $N$ 为有效样本外预测数，$y_i$ 为真实对数收益率，$\hat y_i$ 为同一时点的预测；$\bar y$ 与 $\bar{\hat y}$ 分别为其均值。所有模型在相同预测日期评价。

### 主要指标 RMSE

$$
\mathrm{RMSE}=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(\hat y_i-y_i)^2}
$$

越低越好，0 表示预测完全准确；较大的错误受到更大惩罚。它衡量对数收益率预测误差，不是利润或涨跌准确率。

### 辅助指标 MAE

$$
\mathrm{MAE}=\frac{1}{N}\sum_{i=1}^{N}|\hat y_i-y_i|
$$

越低越好，表示对数收益率的平均绝对误差，帮助识别 RMSE 是否受少量大误差影响。

### 辅助指标 Pearson

$$
\rho=\frac{\sum_{i=1}^{N}(y_i-\bar y)(\hat y_i-\bar{\hat y})}{\sqrt{\sum_{i=1}^{N}(y_i-\bar y)^2}\sqrt{\sum_{i=1}^{N}(\hat y_i-\bar{\hat y})^2}}
$$

范围为 −1 至 1，越高表示正向线性关系越强，不保证幅度或方向准确。任一序列为常数时无定义，记 N/A，不记 0。

### 相对基准的误差改善率

$$
\mathrm{Improvement}=\frac{\mathrm{RMSE}_{baseline}-\mathrm{RMSE}_{model}}{\mathrm{RMSE}_{baseline}}\times100
$$

改善率按百分数报告，例如数值 5 表示改善 5%。零对数收益率基准恒预测 0，对应未来价格等于当前价格；其 Pearson 为 N/A。另设训练期平均对数收益率基准。改善率大于 0 表示超过基准；基准 RMSE 为 0 时改善率无定义。

## 数据划分与比较规则

| 阶段 | 训练数据 | 评价数据 |
|---|---|---|
| 验证一 | 2017–2020 年 | 2021 年 |
| 验证二 | 2017–2021 年 | 2022 年 |
| 验证三 | 2017–2022 年 | 2023 年 |
| 最终测试 | 2017–2023 年重训并冻结 | 2024 年起的有效记录 |

样本起点是研究设计，需根据数据质量说明理由。每折预处理只在训练区拟合；训练标签结束时点必须早于验证起点。使用合并的样本外预测计算开发 RMSE，同时展示逐年表现。测试期不再选特征、调参或选择策略阈值。

模型比较：零收益、历史均值、AR、Ridge、LightGBM。Ridge 和 LightGBM 使用相同公共特征做基础比较；个人专属特征作为扩展实验并明确标记。初始特征建议分为历史收益、趋势与K线形态、成交量与波动率三组，做逐组消融。

统一预测文件字段如下，y_true 和 y_pred 均为对数收益率：

```text
timestamp,target_end,model,y_true,y_pred,split
```

## 金融评价

回测采用信号形成后的可执行价格，明确手续费、滑点和仓位规则，收益必须用实际执行区间的普通收益率计算，不直接累乘对数收益率。补充成本后累计收益、Sharpe 和最大回撤，与相同区间的买入并持有比较。

## 组员上传方式

组员接受仓库 Collaborator 邀请后，进入自己负责的目录，用 Add file → Upload files 上传代码。下载可使用 Code → Download ZIP，数据会随仓库下载。公共配置和接口改动先在组内同步。

建议统一 Python 3.11，安装初始依赖：

```bash
python -m pip install -r requirements.txt
```

目前没有完整运行入口，由负责人实现后在本说明补充执行方式。初始依赖版本待实现后统一固定。

## 参考

- [Ridge 与 LightGBM 基础示例](https://www.kaggle.com/code/cookingfunda/crypto-return-prediction-ridge-lightgbm)
- [第三名训练方案](https://www.kaggle.com/code/sugghi/training-3rd-place-solution)

只借鉴合适的方法。项目采用自己的对数收益率目标、BTC 数据和只用过去训练的验证；采用公开代码时保留署名与许可说明。
