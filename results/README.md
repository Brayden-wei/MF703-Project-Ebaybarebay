# 各模型的交付结果

## 每个模型交什么结果

**对外展示的是一行模型成绩；逐时预测文件是供代码计算和核对的中间数据。** 两者都保留，但用途不同。

### 1. 模型成绩表：一行就够

例如 Ridge 负责人交 `results/ridge_metrics.csv`，列名固定为 `model,rmse,mae`：

| model（模型） | rmse（整个测试期） | mae（整个测试期） |
|---|---:|---:|
| Ridge | 待计算 | 待计算 |

AR 和 LightGBM 分别交 `ar_metrics.csv`、`lightgbm_metrics.csv`，也是各一行，把模型名称换成自己的即可。这里填实际计算得到的数字；“待计算”只表示目前还没有实验结果。个人不填写排名，最后由公共比较代码统一排名。

RMSE、MAE 按根目录公式，对整个测试期全部规定预测一起计算。在个人 README 写明测试起止日期、样本数、数据版本及参数。验证期用于选参数，成绩可以另记在 Notebook 中。

### 2. 逐时预测文件：供统一计算，不是最终展示表

每位负责人同时保留对应的 `*_predictions.csv`。只需要四列：

| 列名 | 简单说明 |
|---|---|
| `timestamp` | 哪个时刻做出的预测，使用 UTC，例如 `2024-01-01 00:00` |
| `y_true` | 接下来十五分钟实际发生的对数收益率 |
| `y_pred` | 你的模型预测的这十五分钟对数收益率 |
| `split` | 这行属于 `val2021`、`val2022`、`val2023` 还是 `test` |

每十五分钟一行。数据负责人先给大家统一的时间、真实值和所属阶段，模型负责人加上自己的预测值即可。不要改真实值、随意删行或填价格/百分数；预测用小数对数收益率，并保留数值精度。最终比较程序只用其中 `split=test` 的全部行计算主表；验证行保留供核对。

| 模型 | 展示成绩，一行 | 计算用的逐时预测，中间文件 |
|---|---|---|
| AR | `results/ar_metrics.csv` | `results/ar_predictions.csv` |
| Ridge | `results/ridge_metrics.csv` | `results/ridge_predictions.csv` |
| LightGBM | `results/lightgbm_metrics.csv` | `results/lightgbm_predictions.csv` |

逐时预测过大时，完整文件放网盘，在 `results/README.md` 记录文件名和组内获取方式，不要删掉部分预测来缩小文件。
