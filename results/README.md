# 模型预测结果

放 ar_predictions.csv、ridge_predictions.csv、lightgbm_predictions.csv，由三位负责人完成后上传。当前没有预测结果。

固定 CSV 字段：timestamp,y_true,y_pred,split。timestamp 为 ISO 8601 UTC 预测形成时刻，split 为 val2021、val2022、val2023、test。真实值与日期必须匹配数据负责人发布的 evaluation_index.csv；预测为小数对数收益率，不是价格、百分数或标准化值。

不自行删评价样本，不包含空值或重复行。个人方法、参数及实验编号记录在 notebooks/ 对应 README。过大的 CSV 用网盘交付，在此记录文件名、版本和组内获取方式；不截断数据。最终评价由 compare.py 完成。
