# MF703 BTC 预测项目

一人准备公共数据，三人分别训练 AR、Ridge、LightGBM，最后由 `compare.py` 统一评价。

## 项目结构

个人文件夹必须有 **README + 可运行的 Notebook 或 Python 代码**。下图代码及结果为交付示例，尚待负责人添加。

```text
MF703-Project-Ebaybarebay/
├── README.md
├── requirements.txt
├── config.json                      #共用的设置文件
├── data/
│   ├── raw/                         # 原始 csv文件，网盘下载
│   └── processed/                   # 处理后数据
├── notebooks/
│   ├── README.md
│   ├── data/
│   │   ├── README.md                # 清洗、特征、划分、运行说明
│   │   └── prepare_data.ipynb       # 数据处理代码，也可用 .py
│   ├── ar/
│   │   ├── README.md                # 方法、参数、数据处理
│   │   └── ar.ipynb                 # 训练与预测代码，也可用 .py
│   ├── ridge/
│   │   ├── README.md
│   │   └── ridge.ipynb              # 训练与预测代码，也可用 .py
│   └── lightgbm/
│       ├── README.md
│       └── lightgbm.ipynb           # 训练与预测代码，也可用 .py
├── results/
│   ├── README.md
│   ├── ar_metrics.csv              # 整个测试期的 RMSE、MAE
│   ├── ar_predictions.csv          # 每次预测的真实值与预测值
│   ├── ridge_metrics.csv
│   ├── ridge_predictions.csv
│   ├── lightgbm_metrics.csv
│   └── lightgbm_predictions.csv
├── compare.py                       # 待实现：统一计算、排名、绘图
└── comparison/
    ├── README.md                    # 比较日期、数据版本和结论
    ├── metrics.csv                  # 最终四行比较表
    └── metrics.png                  # RMSE、MAE 对比图
```

可在个人目录添加辅助脚本；运行顺序写入自己的 README。

## 全组统一规则

预测目标是未来十五分钟的对数收益率：

$$
y_t=\ln\left(\frac{P_{t+15}}{P_t}\right)
$$

| 项目 | 统一要求 |
|---|---|
| 数据 | 同一原始版本和公共处理版本；个人修改不覆盖公共数据 |
| 价格 | Pₜ 为预测时刻已完成分钟线的 Close；时间戳含义由数据负责人核实 |
| 时间 | UTC：例如北京时间 2024-01-01 08:00，表里写 2024-01-01 00:00；已是 UTC 则不用再减 8 小时 |
| 频率 | 每小时 00、15、30、45 分钟预测接下来十五分钟 |
| 单位 | 对数收益率小数，不乘 100；按真实时间配对目标，不跨缺口移动十五行 |
| 训练 | 只用预测时刻已知的信息；标准化等只在训练期拟合 |
| 参数 | 在验证期确定；测试前重训并冻结，测试期不调参或重训 |
| 评价样本 | 数据负责人统一发布日期及真实值，所有模型完整覆盖，不单独删行 |
| 个人选择 | 特征、窗口和参数可自行决定，写入个人 README；随机种子如适用设为 42 |

## 数据划分

| 阶段标记 split | 训练期 | 评价期 |
|---|---|---|
| val2021 | 2017–2020 | 2021 |
| val2022 | 2017–2021 | 2022 |
| val2023 | 2017–2022 | 2023 |
| test | 2017–2023 | 2024 年起至最后有效目标 |

年份均按 UTC。训练样本的目标结束时刻须早于下一评价期起点；数据负责人统一处理缺口和特征预热，冻结评价时间表。

## 模型评判标准

**以整个测试期的 RMSE 排名，越低越好；MAE 仅作辅助。** 下面的求和覆盖全部测试预测，因此一个模型得到一个 RMSE。

$$
\boxed{\displaystyle \mathrm{Score}(m)=\mathrm{RMSE}_m=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(\hat y_i^{(m)}-y_i)^2}}
$$

$$
\mathrm{MAE}_m=\frac{1}{N}\sum_{i=1}^{N}\left|\hat y_i^{(m)}-y_i\right|
$$

N 是测试期预测总数，yᵢ 是真实对数收益率，ŷᵢ⁽ᵐ⁾ 是模型 m 的预测。三个模型使用同一批测试样本；零收益基准始终预测 0。

| 模型 | 测试期 RMSE ↓ | 测试期 MAE ↓ | RMSE 排名 |
|---|---:|---:|---:|
| 零收益基准 | 待计算 | 待计算 | 待计算 |
| AR | 待计算 | 待计算 | 待计算 |
| Ridge | 待计算 | 待计算 | 待计算 |
| LightGBM | 待计算 | 待计算 | 待计算 |

每行是一种模型在整个测试期的成绩。同分可并列；验证成绩用于选参数，不放入最终主表。

## 每个人交什么

| 负责人 | 交付内容 |
|---|---|
| 数据 | `notebooks/data/` 的代码、README、公共数据和统一评价时间表 |
| AR / Ridge / LightGBM | 自己的代码、README，以及下面两份 CSV，放入 `results/` |
| 集成人，可兼任 | `compare.py`、最终模型比较表和对比图 |

个人 README 写清数据处理、特征、参数及选择依据、运行方法。

### 以 Ridge 为例：交这两份 CSV

**① `ridge_metrics.csv`：整个测试期的成绩，只有一行。**

| model | rmse | mae |
|---|---:|---:|
| Ridge | 0.0012 | 0.0008 |

上面数字仅作格式示例。实际填入整个测试期计算出的 RMSE、MAE。

**② `ridge_predictions.csv`：每次预测的明细，用于核对成绩。**

| timestamp | y_true | y_pred | split |
|---|---:|---:|---|
| 2024-01-01 00:00 | 0.0010 | 0.0008 | test |
| 2024-01-01 00:15 | -0.0005 | -0.0002 | test |
| 2024-01-01 00:30 | 0.0003 | 0.0004 | test |
| … | … | … | … |

明细数字也仅作格式示例。每十五分钟一行；四列依次是 **UTC 预测时间、真实对数收益率、预测对数收益率、所属阶段**。时间、真实值和阶段由数据负责人提供，你只填写预测值，不删行；收益率保留原始小数精度。

| 你负责的模型 | 需要交的两个文件 |
|---|---|
| AR | `ar_metrics.csv`、`ar_predictions.csv` |
| Ridge | `ridge_metrics.csv`、`ridge_predictions.csv` |
| LightGBM | `lightgbm_metrics.csv`、`lightgbm_predictions.csv` |

`compare.py` 用明细中 `split=test` 的全部行统一核算，生成前面的四行模型比较表及对比图。验证成绩留作选参数，预测文件过大时走网盘。

## 数据与代码共享

- **数据：** 原始 csv文件 下载至 `data/raw/`，处理后公共数据至 `data/processed/`；网盘链接(https://drive.google.com/drive/folders/1oUc_AD97JhNFmT7GM21hw91WWX0XLKNo?usp=drive_link)。
- **下载代码：** GitHub → Code → Download ZIP；安装 `requirements.txt` 依赖，Python 3.11 其余环境包后面慢慢补充。
- **上传代码：** 接受 Collaborator 邀请 → 进入自己的目录 → Add file → Upload files → Commit changes。

