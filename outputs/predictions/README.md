# 样本外预测

存放基准、AR、Ridge、LightGBM 的验证与最终测试预测。统一字段为 timestamp,target_end,model,y_true,y_pred,split。y_true 和 y_pred 均为十五分钟对数收益率，时间统一 UTC。保持相同评价日期，缺失预测先查原因。
