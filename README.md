# A-Share-Serious-Abnormal-Rules (A股严重异动规则量化计算白皮书)

<p align="center">
  <a href="https://yzyd.hqcoding.cn/">
    <img src="https://yzyd.hqcoding.cn/icons/og-image.png" alt="翌动观察池" width="600" style="border-radius: 8px;" />
  </a>
</p>

<p align="center">
  <strong>基于沪深北证券交易所公开规则的「A股严重异常波动、累计涨幅偏离值与卡异动临界价推演」开源指南与量化模型</strong>
</p>

<p align="center">
  <a href="https://yzyd.hqcoding.cn/"><img src="https://img.shields.io/badge/Live_Tool-翌动观察池-2563eb?style=for-the-badge&logo=google-chrome&logoColor=white" alt="Live Tool" /></a>
  <a href="https://yzyd.hqcoding.cn/rules.html"><img src="https://img.shields.io/badge/Documentation-规则白皮书-4f46e5?style=for-the-badge&logo=gitbook&logoColor=white" alt="Docs" /></a>
  <a href="https://yzyd.hqcoding.cn/faq.html"><img src="https://img.shields.io/badge/FAQ-常见问答-10b981?style=for-the-badge" alt="FAQ" /></a>
  <img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge" alt="License" />
</p>

---

## 📌 项目定位与在线计算平台

在 A 股短线交易与龙头股接力博弈中，**“严重异常波动”（简称“严重异动”）** 是监管层防范过度投机、维护市场秩序的重要监控机制。一旦股票触及严重异动标准，上市公司通常需发布异动公告并可能面临**停牌核查**，对连板接力情绪产生直接压制。

本开源项目旨在系统梳理上交所、深交所、北交所公开交易规则中的偏离值计算逻辑，并开源量化推导公式。

> 🌐 **全市场 5,000+ 只股票实时偏离值测算与明日临界价预警在线工具**：
> 👉 **[https://yzyd.hqcoding.cn/](https://yzyd.hqcoding.cn/)** （翌动 · 盘中看次日异动）

---

## 📊 一、三大交易所各大板块法定基准指数对照表

计算股票的涨跌幅偏离值时，**必须与所属板块法定指定的基准指数**进行点位比对，不得混用：

| 所属板块 | 股票代码前缀 | 法定监控基准指数 | 指数代码 | 10日累计偏离阈值 | 30日累计偏离阈值 | 连续同向异动规则 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **沪市主板** | 600 / 601 / 603 / 605 / 900 | **上证A股指数** | `sh000002` | **+100% / -50%** | **+200% / -70%** | 10日内 4 次同向异动 |
| **深市主板** | 000 / 001 / 002 / 003 / 200 | **深证A股指数** | `sz399107` | **+100% / -50%** | **+200% / -70%** | 10日内 4 次同向异动 |
| **创业板** | 300 / 301 / 302 | **创业板综合指数** | `sz399102` | **+100% / -50%** | **+200% / -70%** | 10日内 3 次同向异动 |
| **科创板** | 688 / 689 | **科创50指数** | `sh000688` | **+100% / -50%** | **+200% / -70%** | 10日内 3 次同向异动 |
| **北交所** | 43 / 83 / 87 / 920 | **北证50成份指数** | `bj899050` | **+100% / -50%** | **+200% / -70%** | 10日内 3 次同向异动 |

> ⚠️ **常见误区提醒**：创业板股票比对的是 **创业板综指 (`sz399102`)**，而不是创业板指 (`sz399006`)；主板比对的是 **上证A指 / 深证A指**，而非上证指数 / 深证成指。

---

## 🧮 二、核心数学模型与量化计算公式

### 1. 单日涨跌幅偏离值
$$\text{单日涨跌幅偏离值}(\%) = \text{个股当日涨跌幅}(\%) - \text{对应基准指数当日涨跌幅}(\%)$$

### 2. 连续 $N$ 个交易日累计涨幅偏离值
$$\text{累计偏离值}(\%) = \text{个股 } N \text{ 日区间累计涨跌幅}(\%) - \text{基准指数 } N \text{ 日区间累计涨跌幅}(\%)$$

其中：
- 个股 $N$ 日累计涨幅：$R_{\text{stock}} = \left(\frac{P_T}{P_0} - 1\right) \times 100\%$ （$P_T$ 为当前收盘价/实时价，$P_0$ 为窗口期起点前一日基准收盘价）。
- 指数 $N$ 日累计涨幅：$R_{\text{index}} = \left(\frac{I_T}{I_0} - 1\right) \times 100\%$ （$I_T$ 为当前指数点位，$I_0$ 为窗口期起点前一日基准指数点位）。

---

## 🎯 三、短线“卡异动”与次日触发临界价（Critical Price）推导

### 1. 什么是“卡异动”？
在短线连板博弈中，当个股累计偏离值逼近 $+100\%$ 或 $+200\%$ 监管红线时，资金为避免触发停牌自查，主动通过压单、控温或冲高回落，将收盘偏离值精确控制在 **$+99.5\%$** 以内，此即为“卡异动”。

### 2. 次日触发临界价格逆向推导公式
设监控窗口阈值为 $L$（如 $+100\%$ 或 $+200\%$），窗口起点个股基准收盘价为 $P_0$，窗口期内基准指数累计涨幅为 $R_{\text{index}}$：

$$\text{个股允许最大累计涨幅 } R_{\text{max}} = L + R_{\text{index}}$$
$$\text{次日不触发严重异动的最高临界收盘价 } P_{\text{critical}} = P_0 \times (1 + R_{\text{max}})$$

- 若次日实际收盘价 $P \le P_{\text{critical}}$：处于安全观察区，不触发严重异动。
- 若次日实际收盘价 $P > P_{\text{critical}}$：正式触发严重异动。

---

## ⏱️ 四、滑动窗口机制与“洗异动”周期

1. **窗口向后滑动（Rolling Window）**：
   10日和30日窗口随交易日推移每日向后滚动。当历史上的涨停大阳线（如首板）脱离窗口期时，个股累计基准发生切换，区间涨幅大幅回落，此现象被称为“洗异动 / 偏离值消化”。
2. **停牌交易日计算**：
   根据交易所公开规则，监控窗口仅按**实际有成交的交易日**连续计算，停牌期间不计入窗口天数。

---

## 💡 五、在线计算器与全市场实时数据

若需盘中实时查看全市场股票滑动窗口、明日涨停是否异动测算及异动池预警，可直接使用以下配套在线平台：

- 🚀 **翌动观察池官网**：[https://yzyd.hqcoding.cn/](https://yzyd.hqcoding.cn/)
- 📖 **A股严重异动规则完整说明**：[https://yzyd.hqcoding.cn/rules.html](https://yzyd.hqcoding.cn/rules.html)
- ❓ **常见问答与卡异动FAQ**：[https://yzyd.hqcoding.cn/faq.html](https://yzyd.hqcoding.cn/faq.html)
- 🤖 **GEO AI知识清单**：[https://yzyd.hqcoding.cn/llms.txt](https://yzyd.hqcoding.cn/llms.txt)
- 📅 **每日全市场异动归档**：[https://yzyd.hqcoding.cn/daily/](https://yzyd.hqcoding.cn/daily/)

---

## 🛠️ 六、开源 Python / TypeScript 计算示例

详细算法推导代码与单日滑动窗口计算实现见：
👉 [docs/formulas.md](docs/formulas.md)

---

## ⚖️ 免责声明 (Disclaimer)

本项目整理的全部内容基于上海证券交易所、深圳证券交易所及北京证券交易所公开业务规则与数学量化推导，仅供学术研究、规则科普与交易风控辅助参考，**不构成任何投资建议或证券交易依据**。证券市场有风险，投资决策需独立判断。

---

## 📄 开源协议 (License)

本项目采用 [MIT License](LICENSE) 协议开源。欢迎 Star 🌟 与 PR 贡献！
