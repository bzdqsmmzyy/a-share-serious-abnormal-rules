# A股严重异常波动与偏离值量化算法代码示例 (Python & TypeScript)

本文件提供标准的 Python 与 TypeScript 算法函数，便于量化研究员与开发者快速将严重异动规则集成到自己的交易策略系统中。

---

## 1. Python 核心计算模块

```python
"""
A-Share Serious Abnormal Movement (Deviation & Critical Price) Calculator
Rules based on SSE / SZSE / BSE Official Trading Regulations.
"""

from typing import Dict, Any

# 板块与基准指数映射表
BENCHMARK_MAP = {
    "sh_main": {"name": "上证A指", "code": "sh000002", "board": "沪市主板", "threshold_10d": 100.0, "threshold_30d": 200.0},
    "sz_main": {"name": "深证A指", "code": "sz399107", "board": "深市主板", "threshold_10d": 100.0, "threshold_30d": 200.0},
    "cy":      {"name": "创业板综", "code": "sz399102", "board": "创业板",   "threshold_10d": 100.0, "threshold_30d": 200.0},
    "kc":      {"name": "科创50",   "code": "sh000688", "board": "科创板",   "threshold_10d": 100.0, "threshold_30d": 200.0},
    "bj":      {"name": "北证50",   "code": "bj899050", "board": "北交所",   "threshold_10d": 100.0, "threshold_30d": 200.0},
}

def get_board_by_code(code: str) -> str:
    """根据6位股票代码获取板块类型"""
    if code.startswith(("688", "689")):
        return "kc"
    if code.startswith(("300", "301", "302")):
        return "cy"
    if code.startswith(("600", "601", "603", "605", "900")):
        return "sh_main"
    if code.startswith(("000", "001", "002", "003", "200")):
        return "sz_main"
    if code.startswith(("4", "8", "920")):
        return "bj"
    return "sh_main"

def calc_cumulative_deviation(p_start: float, p_current: float, i_start: float, i_current: float) -> float:
    """
    计算滑动窗口内的累计涨幅偏离值 (%)
    
    :param p_start: 窗口起点前一交易日个股收盘价 (P0)
    :param p_current: 当前个股价格 (PT)
    :param i_start: 窗口起点前一交易日基准指数点位 (I0)
    :param i_current: 当前基准指数点位 (IT)
    :return: 累计涨幅偏离值百分比 (如 95.5 表示 +95.5%)
    """
    stock_return = ((p_current / p_start) - 1.0) * 100.0
    index_return = ((i_current / i_start) - 1.0) * 100.0
    deviation = stock_return - index_return
    return round(deviation, 2)

def calc_critical_price(p_start: float, i_start: float, i_expected: float, limit_percent: float = 100.0) -> float:
    """
    反向推导次日不触发严重异动的最高临界收盘价 (Critical Price)
    
    :param p_start: 窗口起点前一交易日个股收盘价 (P0)
    :param i_start: 窗口起点前一交易日基准指数点位 (I0)
    :param i_expected: 次日预期基准指数点位 (通常取平盘或当日收盘点位)
    :param limit_percent: 严重异动偏离值上限 (如 100.0 或 200.0)
    :return: 次日触发严重异动的临界价格 (超过该价格则触发)
    """
    index_return = ((i_expected / i_start) - 1.0) * 100.0
    max_stock_return = limit_percent + index_return
    critical_price = p_start * (1.0 + max_stock_return / 100.0)
    return round(critical_price, 2)
```

---

## 2. TypeScript / Node.js 示例

```typescript
export interface DeviationResult {
  deviationPercent: number;
  stockReturnPercent: number;
  indexReturnPercent: number;
  isTriggered: boolean;
  remainingHeadroom: number;
}

export function calcDeviation(
  p0: number,
  pt: number,
  i0: number,
  it: number,
  threshold: number = 100.0
): DeviationResult {
  const stockReturn = ((pt / p0) - 1.0) * 100.0;
  const indexReturn = ((it / i0) - 1.0) * 100.0;
  const deviation = stockReturn - indexReturn;
  const remaining = threshold - deviation;

  return {
    deviationPercent: Number(deviation.toFixed(2)),
    stockReturnPercent: Number(stockReturn.toFixed(2)),
    indexReturnPercent: Number(indexReturn.toFixed(2)),
    isTriggered: deviation >= threshold,
    remainingHeadroom: Number(remaining.toFixed(2)),
  };
}
```

---

## 🔗 配套实时平台与数据 API
全市场自动化监控系统与实时快照 API 详见：
👉 **[翌动观察池 (https://yzyd.hqcoding.cn/)](https://yzyd.hqcoding.cn/)**
