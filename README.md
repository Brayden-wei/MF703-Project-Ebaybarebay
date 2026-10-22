# MF703 BTC 预测项目

采用简单协作方式：一人准备公共数据，三人独立完成 AR、Ridge、LightGBM，分别交预测结果，最后用 compare.py 统一比较。下面是组内实验约定，并非教授要求的原文。

## 项目目录

下图是当前采用的结构。只建立文件夹说明；代码、Notebook 和结果文件由负责人完成后添加。

```text
MF703-Project-Ebaybarebay/
├── README.md
├── requirements.txt
├── config.json
├── data/
│   ├── raw/                 # 原始 Parquet，网盘下载
│   └── processed/           # 公共数据、目标、评价时间表，网盘发布
├── notebooks/
│   ├── README.md
│   ├── data/README.md       # 数据负责人：清洗、公共特征及划分
│   ├── ar/README.md         # AR：自己的 Notebook、辅助代码和方法说明
│   ├── ridge/README.md      # Ridge：自己的 Notebook、辅助代码和方法说明
│   └── lightgbm/README.md   # LightGBM：自己的 Notebook、辅助代码和方法说明
├── results/README.md        # 各模型统一格式的预测 CSV
├── compare.py               # 待集成人实现：统一评价、记录、绘图
└── comparison/README.md     # 最后生成的指标表、图和比较说明
```

已有 src/、outputs/ 及旧数据分片暂时保留，作为旧结构；本轮任务以本 README 为准，不要求把个人 Notebook 再拆成 src 模块，也不要求保存 pkl。当前尚未实现训练或比较代码。

## 全组必须统一的内容

| 项目 | 统一约定 |
|---|---|
| 数据 | 同一 raw-v1 和 processed-v1；数据负责人统一发布，不覆盖原始数据 |
| 预测目标 | 未来 15 分钟的对数收益率，以小数表示，不乘 100 |
| 价格和时间 | UTC；P_t 是决策时点 t 已完成分钟线的 Close；原始时间戳代表开盘还是收盘由数据负责人核实并在 README 记录，统一转换为可用时刻 |
| 预测频率 | UTC 每小时 00、15、30、45 分钟形成预测；从 t 预测真实时间 t+15 分钟 |
| 公共目标 | 数据负责人生成 timestamp、y_true、split 的 evaluation_index.csv；目标必须有真实有效价格，不跨缺口直接移动 15 行 |
| 数据划分 | 使用下表固定年份，不能随机打乱切分；所有模型使用相同折 |
| 防止未来信息 | 特征只用 t 时刻及之前已经可用的数据；标准化、填补参数及特征选择只在训练期拟合 |
| 模型训练 | 只在验证期选参数，测试前用 2017–2023 年重训并冻结；测试期不再调整或重新拟合模型 |
| 结果 | 按 evaluation_index.csv 的全部规定时点提交，不能自己删除预测困难的样本；缺预测先修复，不能静默取交集 |
| 评价 | 只使用 RMSE、MAE，越低越好；主要按 RMSE 比较，同时报告 MAE |
| 基准 | compare.py 同时计算恒预测 0 的基准，不需要单独训练 |
| 可复现 | 有随机性的模型设 random_state=42；README 记录库版本、最终参数、数据版本和运行顺序 |

特征、滞后长度、模型参数、模型专属缺失处理可以各自选择，但必须写清楚。个人修改只保存在自己的文件夹，不覆盖公共数据；不能改变统一目标、评价日期和真实值。不同特征的比较代表完整预测方案的比较，不能只归因于模型种类。

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

$$
y_t=\ln\left(\frac{P_{t+15}}{P_t}\right)
$$

设 N 为同一评价期规定的预测数量，y_i 为公共真实值，hat y_i 为预测值：

$$
\mathrm{RMSE}=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(\hat y_i-y_i)^2}
$$

$$
\mathrm{MAE}=\frac{1}{N}\sum_{i=1}^{N}|\hat y_i-y_i|
$$

二者均以小数对数收益率计算。分别报告三折验证、合并验证和最终测试；合并验证须拼接全部验证预测后计算，不能简单平均三个 RMSE。测试结果只用于最终报告。

## 每个人交什么

| 负责人 | 个人工作目录 | 必交内容 |
|---|---|---|
| 数据与特征 | notebooks/data/ | 可运行的处理 Notebook 或代码、自己的 README、网盘公共数据及 evaluation_index.csv |
| AR | notebooks/ar/ | 可运行 Notebook 或代码、自己的 README、results/ar_predictions.csv |
| Ridge | notebooks/ridge/ | 可运行 Notebook 或代码、自己的 README、results/ridge_predictions.csv |
| LightGBM | notebooks/lightgbm/ | 可运行 Notebook 或代码、自己的 README、results/lightgbm_predictions.csv |
| 集成人，可由上述一人兼任 | 根目录及 comparison/ | compare.py、最终指标表、图、比较说明 |

每份个人 README 必须填写：负责人、输入数据版本、清洗/缺失处理、特征与窗口、最终参数及选择理由、训练验证过程、随机种子和依赖、运行步骤、结果文件位置、主要发现。四个目录已经放入对应填写模板，未填写项不代表已经做过实验。

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
