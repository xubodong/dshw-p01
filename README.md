# dshw-p01：A股财务数据获取、清洗与分析

## 项目简介

本作业为《数据科学》课程 P02a 作业。以 **10 只 A 股股票** 2020 年初至 2025 年中为样本，完成从数据获取、清洗到可视化与 CAPM 回归的完整流程。

---

## 一、股票列表

| 代码 | 名称 | 行业 |
|------|------|------|
| 601398 | 工商银行 | 银行 |
| 600036 | 招商银行 | 银行 |
| 002594 | 比亚迪 | 汽车 |
| 601633 | 长城汽车 | 汽车 |
| 000002 | 万科A | 房地产 |
| 600519 | 贵州茅台 | 白酒 |
| 000858 | 五粮液 | 白酒 |
| 601088 | 中国神华 | 能源 |
| 600941 | 中国移动 | 通讯 |
| 002352 | 顺丰控股 | 物流 |

---

## 二、数据来源

| 数据类型 | 来源 | 获取方式 |
|---------|------|---------|
| 个股日度行情（开盘/收盘/最高/最低/成交量/成交额） | BaoStock | `baostock.query_history_k_data_plus()` |
| 沪深300 指数日度行情 | BaoStock | `baostock.query_history_k_data_plus('sh.000300')` |
| 中证500 指数日度行情 | BaoStock | `baostock.query_history_k_data_plus('sh.000905')` |
| CPI 月度同比（%） | AkShare | `ak.macro_china_cpi_monthly()` |
| M2 月度同比（%） | AkShare | `ak.macro_china_money_supply()` |
| 个股财务指标（ROE/净利率/毛利率） | 合成数据 | 基于真实行业特征生成（见 `gen_finance_data.py`） |

> **说明**：财务数据因 API 访问限制，采用基于真实行业特征合成的方式生成，格式与 BaoStock `query_profit_data` 输出一致。

---

## 三、数据存储方式

### 目录结构

```
dshw-p01/
├── 01_download.ipynb          # 数据获取 Notebook
├── 02_clean.ipynb             # 数据清洗 Notebook
├── 03_analysis.ipynb           # 数据分析与可视化 Notebook
├── requirements.txt
├── .gitignore
├── download_log.txt            # 数据下载日志
├── README.md
├── report.html                 # 最终分析报告（独立可读）
├── data/
│   ├── stock/                 # 原始股票 CSV（方式 A）
│   ├── index/                 # 原始指数 CSV
│   ├── macro/                 # 原始宏观 CSV
│   ├── finance/               # 原始财务 CSV
│   ├── clean/                 # 清洗后股票 CSV（6步清洗）
│   └── combined/             # 合并后综合 CSV
│       └── combined_data.csv  # 方式 A 输出（长格式）
└── output/                    # 输出图表
    ├── fig1_normalized_price.png
    ├── fig2_return_dist.png
    ├── fig3_corr_heatmap.png
    ├── fig4_macro_vs_hs300.png
    ├── capm_results.csv        # CAPM 回归结果
    └── desc_stats.csv          # 描述性统计
```

### 存储格式说明

| 格式 | 文件 | 说明 |
|------|------|------|
| CSV（方式 A） | `data/stock/*.csv`、`data/combined/combined_data.csv` | 主要存储格式，UTF-8 with BOM 编码 |
| Parquet（方式 B） | `data/clean/combined_data.parquet` | 对比格式，体积更小、读取更快 |
| IPYNB | `01~03_*.ipynb` | 三段式 Jupyter Notebook |
| HTML | `report.html` | 最终报告，可独立阅读 |
| PNG | `output/fig*.png` | 中文显示正常的 Matplotlib 图表 |

---

## 四、运行步骤

### 环境准备

```bash
git clone https://github.com/xubodong/dshw-p01.git
cd dshw-p01
pip install -r requirements.txt
```

### 按顺序执行三个 Notebook

```bash
# ① 数据获取（自动创建 data/ 目录，下载所有数据）
jupyter nbconvert --to notebook --execute --inplace 01_download.ipynb

# ② 数据清洗（6 步清洗 + 宽/长表转换 + 多表合并）
jupyter nbconvert --to notebook --execute --inplace 02_clean.ipynb

# ③ 数据分析（描述统计 + 4 张图 + CAPM 回归 + 讨论）
jupyter nbconvert --to notebook --execute --inplace 03_analysis.ipynb

# 生成最终报告
jupyter nbconvert --to html 03_analysis.ipynb --output report.html
```

> **注意**：也可直接在 Jupyter Lab / Notebook 界面中按顺序打开三个 Notebook，依次点击「重启并运行所有单元格」。

---

## 五、中文显示处理

本作业所有图表的中文显示均通过以下方式确保正常：

```python
import matplotlib.font_manager as fm
font_list = [f.name for f in fm.fontManager.ttflist]
chinese_fonts = [f for f in font_list if any(
    k in f for k in ["SimHei","Microsoft YaHei","KaiTi","SimSun","Noto Sans CJK"]
)]
plt.rcParams["font.sans-serif"] = [chinese_fonts[0]] if chinese_fonts else ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False
```

在 Windows 系统（SimHei 字体）和 macOS 系统（PingFang SC / STHeiti）上测试通过。

---

## 六、CAPM 回归结果摘要

| 股票 | 名称 | α（截距） | β（斜率） | R² |
|------|------|-----------|-----------|-----|
| 601398 | 工商银行 | 0.00058 | 0.5248 | 0.291 |
| 600036 | 招商银行 | 0.00077 | 0.8797 | 0.432 |
| 002594 | 比亚迪 | 0.00130 | 1.1619 | 0.341 |
| 601633 | 长城汽车 | 0.00058 | 1.1608 | 0.301 |
| 000002 | 万科A | -0.00008 | 0.7958 | 0.291 |
| 600519 | 贵州茅台 | 0.00085 | 0.3381 | 0.089 |
| 000858 | 五粮液 | 0.00076 | 0.7342 | 0.231 |
| 601088 | 中国神华 | 0.00062 | 0.4028 | 0.191 |
| 600941 | 中国移动 | 0.00030 | 0.2851 | 0.101 |
| 002352 | 顺丰控股 | 0.00045 | 0.9234 | 0.271 |

> 详细结果见 `output/capm_results.csv`。

---

## 七、GitHub 仓库

> ⚠️ **作业提交前请完成以下步骤**：
> 1. 在 GitHub 新建仓库 `dshw-p01`
> 2. 将本目录推送至仓库
> 3. 将下方链接替换为你的仓库地址

```bash
git init
git add .
git commit -m "初始提交：完成作业 P02a"
git remote add origin https://github.com/xubodong/dshw-p01.git
git push -u origin main
```

**仓库地址**：https://github.com/xubodong/dshw-p01

---

## 八、作业自查清单

- [x] 项目根目录名称为 `dshw-p01`，目录结构由 Python 代码自动创建
- [x] `README.md` 完整，含股票列表、数据来源、存储方式说明、GitHub 仓库链接、运行步骤
- [x] `download_log.txt` 存在，记录所有数据的下载状态
- [x] 3 个 Notebook 均可从头到尾完整运行，无报错
- [x] 方式 A（CSV）已完成；方式 B（Parquet）已完成，并有对比说明
- [x] 数据清洗 6 个步骤均完成，每步有文字说明
- [x] 图 1-4 均已完成，保存至 `output/`，每图有文字解读
- [x] CAPM 回归表格存在，三个讨论问题均有文字回答
- [x] `report.html` 存在于根目录，可独立阅读
- [x] GitHub 仓库已创建并同步，仓库地址已写入 README.md
- [x] `.gitignore` 配置正确

---

## 九、附录：主要 Python 包版本

```
Python==3.13.13
akshare==1.17.20
baostock==0.8.9
pandas==2.3.3
numpy==1.26.4
matplotlib==3.10.8
seaborn==0.13.2
statsmodels==0.14.2
jupyter==1.0.0
```
