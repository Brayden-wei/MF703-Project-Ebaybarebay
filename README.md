# MF703 BTC Price Prediction

本项目研究历史价格和成交量能否改善 BTC 未来十五分钟收益率预测，并将预测收益率转换成未来价格。当前仓库为项目骨架，模型和数据处理模块由组员逐步实现。

## 文件夹结构

```text
MF703-project/
├── README.md                 # 项目说明、文件用途和协作方式
├── requirements.txt          # 全组共用的 Python 依赖
├── config.json               # 路径、预测期限、数据划分与随机种子
├── .gitignore                # 忽略原始数据、生成文件与本地缓存
├── data/
│   ├── raw/                  # 原始 BTC 分钟 CSV，保留原样
│   └── processed/            # 清洗数据、公共特征和样本划分
├── src/
│   ├── __init__.py
│   ├── data.py               # 数据读取、清洗、时间对齐和划分
│   ├── features.py           # 公共特征与未来十五分钟目标
│   ├── baselines.py          # 零收益率和历史均值基准
│   ├── models/
│   │   ├── __init__.py
│   │   ├── ar.py             # AR 专属处理、训练和预测
│   │   ├── ridge.py          # Ridge 专属处理、训练和预测
│   │   └── lightgbm.py       # LightGBM 专属处理、训练和预测
│   ├── evaluation.py         # 统一 RMSE、MAE、Pearson 与改善率
│   └── backtest.py           # 交易执行、手续费、滑点和风险评价
├── notebooks/
│   ├── data_exploration.ipynb # 数据与特征探索
│   ├── ar_experiments.ipynb   # AR 个人实验
│   ├── ridge_experiments.ipynb# Ridge 个人实验
│   ├── lgbm_experiments.ipynb # LightGBM 个人实验
│   └── main.ipynb             # 最终统一运行和展示入口
└── outputs/
    ├── models/               # 训练后的模型和预处理参数
    ├── predictions/          # 各模型按时间对齐的预测
    ├── tables/               # 模型比较、消融与回测表格
    └── figures/              # 预测、特征重要性、净值与回撤图
```

`.gitkeep` 只用于保留空文件夹；实际数据和结果生成后无需修改它。

## 四人分工

| 负责人 | 主要文件 | 任务 |
|---|---|---|
| 数据与特征负责人 | `src/data.py`、`src/features.py` | 清洗、公共特征、目标和时间划分 |
| AR 负责人 | `src/models/ar.py`、对应实验 Notebook | AR 序列处理、参数选择、预测 |
| Ridge 负责人 | `src/models/ridge.py`、对应实验 Notebook | 标准化、正则化、训练和预测 |
| LightGBM 负责人 | `src/models/lightgbm.py`、对应实验 Notebook | 专属处理、参数选择、训练和预测 |

`baselines.py`、`evaluation.py`、`backtest.py` 和 `main.ipynb` 由组内指定集成人维护。个人探索写入自己的 Notebook；最终计算逻辑整理进对应 Python 模块。不要同时修改别人的文件。

## 数据下载与放置

原始文件为 `btcusd_1-min_data.csv`，列为 Timestamp、Open、High、Low、Close、Volume。原始 CSV 不包含在代码仓库，也不会随 Download ZIP 自动下载。

数据下载链接：**待数据负责人补充共享网盘链接**。

1. 从共享链接下载原始 CSV。
2. 将文件放在本地 `data/raw/btcusd_1-min_data.csv`。
3. 数据负责人实现处理流程后，运行它生成 `data/processed/` 内容。处理后数据的具体文件名与格式在该模块完成时统一。

模型负责人读取公共处理数据；标准化、AR 滞后项和其他模型专属处理写在自己的模型文件内，不覆盖公共数据。公共错误或新增公共特征由数据负责人统一修改。

## 任务与统一评价

- 目标为普通收益率 `y_t = Close_(t+15) / Close_t - 1`；预测价格为 `Close_t * (1 + y_pred)`。
- 时间统一为 UTC；收益率使用小数，0.01 代表 1%。
- 每十五分钟产生一次预测，仅使用当时已经完成的分钟线。
- 主指标为收益率 RMSE，辅助为 MAE、Pearson。零收益率基准对应未来价格等于当前价格；常数预测的 Pearson 记 N/A。
- 评价必须使用相同日期与目标，不能各自删掉难预测样本后比较。
- 标准化和特征筛选只在对应训练窗口拟合；训练标签结束时间必须早于验证开始时间。
- 开发区为 2017–2023 年，按年份向前验证；最终测试为 2024 年起的有效样本。最终方案冻结后使用测试集。

预测 CSV 统一字段：

```text
timestamp,target_end,model,y_true,y_pred,split
```

## 安装与运行顺序

建议全组使用 Python 3.11。初始依赖列于 requirements.txt；实现后统一记录可复现的版本。

```bash
python -m pip install -r requirements.txt
```

计划运行顺序：读取清洗 → 公共特征及目标 → 基准与三个模型 → 统一评价 → 金融回测。`notebooks/main.ipynb` 为最终入口，目前仅有说明占位，尚未实现完整流水线。

## 最简网页上传方式

1. 仓库所有者在 Settings → Collaborators 邀请组员，组员接受邀请。
2. 组员通过 Code → Download ZIP 下载代码，并单独下载原始数据。
3. 上传时先进入自己文件对应的目录，再用 Add file → Upload files 上传，确认目录和文件名正确。
4. 更新自己的模块即可；涉及 config、公共接口或 main.ipynb 的修改先在组内同步。
5. 更新已有本地项目时，只替换需要更新的代码，保留本地 data 与生成结果。

## 代码参考

- [Crypto Return Prediction | Ridge + LightGBM](https://www.kaggle.com/code/cookingfunda/crypto-return-prediction-ridge-lightgbm)
- [training 3rd place solution](https://www.kaggle.com/code/sugghi/training-3rd-place-solution)

参考基础结构、特征构造和训练组织。项目使用自己的 BTC 数据与预测目标；不会直接复制竞赛的多币种残差目标或双向训练验证方式。采用参考代码时保留相应署名和许可说明。
