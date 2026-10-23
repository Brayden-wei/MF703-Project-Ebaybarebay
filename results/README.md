# CSV 提交示例

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

最终比较表：零收益基准、AR、Ridge、LightGBM 各一行，列为 model,rmse,mae,rank。
