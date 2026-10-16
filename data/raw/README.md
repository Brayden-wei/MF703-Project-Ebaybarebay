# 原始分钟数据

存放 btcusd_1-min_data.part-000.parquet 至 part-007.parquet，共八片、7,206,877 行，按原始 CSV 行序划分。转换保留 Timestamp、Open、High、Low、Close、Volume 及全部数值精度，没有清洗或采样。

单文件超过 GitHub 普通文件上限，因此分片存储。下载仓库后按文件名排序合并读取：

```python
from pathlib import Path
import pandas as pd
parts = sorted(Path("data/raw").glob("btcusd_1-min_data.part-*.parquet"))
df = pd.concat([pd.read_parquet(p) for p in parts], ignore_index=True)
```

从项目根目录运行。不要只读一片，也不要将本目录 README 当作数据读取。数据负责人统一处理缺口、零成交量和异常。
