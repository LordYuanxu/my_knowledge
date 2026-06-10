# 安装(R中)
```r
BiocManager::install("sangeranalyseR")
```

# 安装依赖(conda 安装)
```bash
# 1. 激活conda R环境
conda activate r
# 2. mamba补全依赖
## 编译环境
mamba install -y zlib 
mamba install -y bzip2 xz curl openssl libdeflate
## 基础R包
mamba install -y r-seqinr
mamba install -y r-shiny
mamba install -y r-plotly
mamba install -y r-shinydashboard
mamba install -y r-shinyjs
mamba install -y r-shinycssloaders
mamba install -y r-shinywidgets
mamba install -y r-data.table

# 3. R终端内执行
if (!require("BiocManager")) install.packages("BiocManager")
BiocManager::install("sangeranalyseR",update=F,ask=F)
```