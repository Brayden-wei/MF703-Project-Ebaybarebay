# MF703 BTC 预测项目

使用 BTC 分钟级 OHLCV，研究历史信息对未来十五分钟对数收益率的预测能力，并将预测转换为价格。代码与项目说明由 GitHub 管理，原始数据和公共处理数据通过组内网盘共享。各负责人自行创建代码和 Notebook。

## 四人组快速开始

**统一方式：GitHub 上传代码，网盘下载数据。** 本次调整只修改说明和未来上传规则，仓库中已跟踪的数据及历史记录暂时保留，不会因 .gitignore 自动消失。

| 需要分享的内容 | 位置与权限 | 当前状态 |
|---|---|---|
| GitHub 仓库 | [打开仓库](https://github.com/Brayden-wei/MF703-Project-Ebaybarebay)；组员需接受 Collaborator 邀请才能直接上传 | 已建立 |
| 原始数据 raw-v1 | 网盘共享链接，允许组员查看和下载 | **待项目负责人上传并填写链接** |
| 公共处理数据 processed-v1 | 网盘共享链接，数据负责人发布，其他人查看和下载 | **待数据负责人生成并填写链接** |

仓库目前是公开的。若网盘仅供组员使用，请设置“指定人员可访问”，并通过组内聊天发送链接；不要把可直接编辑或访问组内私有资料的链接公开放进 README。无需分享 GitHub 密码、验证码或网盘账号。

### 项目负责人先做什么

1. 在 OneDrive 或 Google Drive 创建 `MF703-data`，其中创建 `raw-v1` 与 `processed-v1`。
2. 把原始 Parquet 上传到 `raw-v1`，只上传完整单文件或整套八个分片中的一种。
3. 在 GitHub 的 Settings → Collaborators → Add people 输入组员用户名，邀请其加入；组员需自行接受。
4. 把仓库链接、网盘下载链接、数据版本和个人负责目录发给组员。
5. 数据负责人生成公共特征、目标和划分后，发布 `processed-v1` 并附字段说明。新版本使用新文件夹，不覆盖旧版。

### 每位组员第一次如何使用

1. 在 GitHub 点击 Code → Download ZIP，解压为本地项目目录。
2. 从网盘下载指定版本的原始数据，放入本地 `data/raw/`。
3. 公共处理数据发布后，下载同一版本到本地 `data/processed/`，避免四个人各用不同数据。
4. 安装 `requirements.txt` 中的依赖，在本地编写并运行自己负责的代码。
5. 网盘文件须实际下载到本地；不能仅保留需要联网取回的云端占位文件。

## 项目目录结构

下图展示约定的工作结构。数据下载到每个人的本地目录，后续新增数据不提交 GitHub；已有历史数据暂时保留。

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
│   │   ├── README.md                 # 网盘下载、版本与读取说明
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
| `data/raw/` | 从网盘下载的原始 Parquet；保留原样 | 数据与特征负责人 |
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

## 组员上传代码的具体步骤

首次上传前，先接受项目负责人发出的 GitHub Collaborator 邀请并登录自己的账号。当前采用最简单的网页上传流程，组员各自维护自己的文件。

| 步骤 | 网页操作 |
|---|---|
| 1 | 打开仓库，先进入文件应保存的目录，例如 `src/models/` |
| 2 | 点击 Add file → Upload files |
| 3 | 拖入自己写好的 `.py` 或 `.ipynb` 文件；只上传该目录应有的文件 |
| 4 | 等待文件上传完成，确认名称与目录正确 |
| 5 | 填写提交说明，例如 `Add Ridge model and preprocessing` |
| 6 | 选择 Commit directly to the main branch，点击 Commit changes |
| 7 | 返回目录，确认文件出现，把本次改动告知组员 |

**示例：** Ridge 负责人进入 `src/models/` 上传 `ridge.py`；实验 Notebook 则另行进入 `notebooks/` 上传。更新同一文件时，在相同目录上传相同文件名。上传前先看最新版本，避免覆盖别人的修改；不要四个人同时修改同一个文件。无需把整个本地项目或 ZIP 当作代码上传。

| 负责人 | 正式代码位置 | 实验位置 |
|---|---|---|
| 数据与特征 | `src/` | `notebooks/` |
| AR | `src/models/` | `notebooks/` |
| Ridge | `src/models/` | `notebooks/` |
| LightGBM | `src/models/` | `notebooks/` |

原始/处理数据、大模型文件放网盘；代码、配置、说明及需要展示的小型表格和图片放 GitHub。清除 Notebook 中不必要的大型输出。模型预测、特征版本及接口保持前文统一规则。网页单文件上传限 25 MiB，参见 [GitHub 文件上传说明](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository)。

建议统一 Python 3.11，安装初始依赖：

```bash
python -m pip install -r requirements.txt
```

目前没有完整运行入口，由负责人实现后补充。后续更新代码时保留自己的本地数据；Download ZIP 仍可能包含仓库先前跟踪的旧数据，它不代表最新网盘版本。

## 发给组员的消息模板

```text
GitHub：https://github.com/Brayden-wei/MF703-Project-Ebaybarebay
请先接受我发送的 Collaborator 邀请，使用自己的 GitHub 账号。

原始数据链接：[填写指定人员可下载的网盘链接]，版本 raw-v1
公共处理数据：[生成后填写链接]，版本 processed-v1

下载代码后，原始数据放 data/raw/，处理数据放 data/processed/。
你的负责内容：[填写分工]
正式代码上传到：[src/ 或 src/models/]
个人实验上传到：notebooks/

上传：进入对应目录 → Add file → Upload files → Commit changes。
数据不要上传 GitHub。公共数据或接口变更先在组内同步。
```

## 参考

- [Ridge 与 LightGBM 基础示例](https://www.kaggle.com/code/cookingfunda/crypto-return-prediction-ridge-lightgbm)
- [第三名训练方案](https://www.kaggle.com/code/sugghi/training-3rd-place-solution)

只借鉴合适的方法。项目采用自己的对数收益率目标、BTC 数据和只用过去训练的验证；采用公开代码时保留署名与许可说明。
