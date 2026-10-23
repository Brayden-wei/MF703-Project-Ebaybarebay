# 模型交付结果

| 文件，模型名为 ar / ridge / lightgbm | 列 | 内容 |
|---|---|---|
| `模型名_metrics.csv` | `model,rmse,mae` | 整个统一测试期的一行成绩 |
| `模型名_predictions.csv` | `timestamp,y_true,y_pred,split` | 每十五分钟一行，供统一核算的中间文件 |

timestamp 用 UTC；y_true、y_pred 用小数对数收益率；split 为 val2021、val2022、val2023、test。时间、真实值、split 由数据负责人统一提供，模型负责人加预测值，不删规定样本。

按根目录公式计算整个测试期 RMSE、MAE，最终排名由 compare.py 统一核算。个人 README 记录测试范围、样本数、数据版本和参数。大文件放网盘，在此补文件名及获取方式。
