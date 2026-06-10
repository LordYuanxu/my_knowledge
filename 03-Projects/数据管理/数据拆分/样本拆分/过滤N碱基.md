# N 碱基过滤技术方案

| 项目   | 内容         |
| ---- | ---------- |
| 文档版本 | v1.0.0     |
| 修订日期 | 2026-06-09 |
| 作者   | 徐园         |

---

## 1. 背景与问题

bcl-convert 拆分后的原始 FASTQ 数据中存在较多 N 碱基，主要来源：
- read 3' 末端质量自然衰减
- RTA3 输出的 `0 qual + nonzero base` 被 bcl-convert 强制 mask 为 `N`

在向客户交付原始数据前，需要对 FASTQ 进行 N 碱基过滤：**PE reads 中任意一端（R1 或 R2）N 碱基数量 > 5，则整条 pair 丢弃**。

---

## 2. 需求与约束

| 需求项 | 内容 |
|--------|------|
| 过滤规则 | PE reads 任一端 N > 5，则丢弃该 pair |
| 序列处理 | **不修改任何碱基**，只做 pass/fail 判断 |
| 其他过滤 | 不开启质量剪切、adapter 去除、长度过滤 |
| 失败 reads | 建议保留，便于审计和问题追溯 |
| 验证方式 | `seqkit stats -a` 对比输入输出 read 数量 |

---

## 3. 基线统计

以样本 `MJ260602P_J69`（Lane 1）为例，先量化问题规模。**注意：必须同时统计 R1 和 R2**，因为 PE300 测序中 R2 末端质量通常显著差于 R1，N 碱基比例可能远高于单端统计结果。

```bash
# 统计 R1 中 N > 5 的 reads
zcat /mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R1_001.fastq.gz | awk '
  NR%4==2 {
    n = gsub(/N/, "", $0)
    total++
    if (n > 5) bad++
  }
  END {
    print "R1 Total reads: " total
    print "R1 Reads with N>5: " bad
    print "R1 Percentage: " (bad/total*100) "%"
  }
'

# 统计 R2 中 N > 5 的 reads
zcat /mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R2_001.fastq.gz | awk '
  NR%4==2 {
    n = gsub(/N/, "", $0)
    total++
    if (n > 5) bad++
  }
  END {
    print "R2 Total reads: " total
    print "R2 Reads with N>5: " bad
    print "R2 Percentage: " (bad/total*100) "%"
  }
'
```

**基线统计结果（MJ260602P_J69）**：

| 端 | Total reads | Reads with N>5 | 比例 |
|---|------------|---------------|------|
| R1 | 3,601,012 | 26,261 | 0.729% |
| R2 | 3,601,012 | **791,340** | **21.9755%** |

**实际过滤结果（按 pair 统计）**：

| 样本 | Total pairs | 过滤 pairs | 过滤比例 |
|------|------------|-----------|---------|
| MJ260602P_J69 | 3,601,012 | 817,558 | **22.70%** |
| MJ260602P_J70 | 1,406,547 | 323,606 | **23.01%** |

> ⚠️ **重要**：单看 R1 的 N 比例会严重低估实际过滤量。本例中 R2 的 N>5 比例高达 21.98%，实际 pair 级别过滤率（~23%）主要由 R2 贡献。

---

## 4. 方案调研

### 4.1 方案一：fastp

#### 初步尝试（失败）

直觉上，fastp 的 `--n_base_limit 5` 参数可以直接实现需求。以下以样本 `MJ260602P_J69` 为例：

```bash
SAMPLE="MJ260602P_J69"
INPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--${SAMPLE}"
OUTPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n"

fastp \
  -i ${INPUT_DIR}/${SAMPLE}_S1_L001_R1_001.fastq.gz \
  -I ${INPUT_DIR}/${SAMPLE}_S1_L001_R2_001.fastq.gz \
  -o ${OUTPUT_DIR}/${SAMPLE}.fastp.R1.fastq.gz \
  -O ${OUTPUT_DIR}/${SAMPLE}.fastp.R2.fastq.gz \
  --n_base_limit 5 \
  --disable_adapter_trimming \
  --disable_trim_poly_g \
  --disable_quality_filtering \
  --disable_length_filtering
```

**结果**：输出与输入 read 数基本一致，`--n_base_limit` 未生效。

**原因**：`--disable_quality_filtering` 在 fastp 内部会连带禁用 `--n_base_limit`（N 过滤被归类为质量过滤逻辑的一部分）。

#### 修正方案（成功）

不要直接用 `--disable_quality_filtering`，而是将质量阈值显式置 0：

```bash
fastp \
  -i /mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R1_001.fastq.gz \
  -I /mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R2_001.fastq.gz \
  -o /mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/6a2264de138eaf134c4cbf63--MJ260602P_J69.fastp.R1.fastq.gz \
  -O /mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/6a2264de138eaf134c4cbf63--MJ260602P_J69.fastp.R2.fastq.gz \
  --qualified_quality_phred 0 \
  --unqualified_percent_limit 0 \
  --average_qual 0 \
  --n_base_limit 5 \
  --disable_adapter_trimming \
  --disable_trim_poly_g \
  --disable_length_filtering
```

**结果**：N 过滤生效，输出 read 数减少约 26,261，符合预期。

#### 保留失败 reads

fastp 支持 `--failed_out`，但 PE reads 输出为 **interleaved FASTQ**（R1/R2 交替），需要用 `seqtk` 或 `awk` 拆分为独立 R1/R2。

> ⚠️ 不推荐 `seqkit split2`，实测对 fastp 输出的 interleaved FASTQ 兼容性不佳（read name 格式识别问题）。

```bash
SAMPLE="MJ260602P_J69"
INPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--${SAMPLE}"
OUTPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n"

fastp \
  -i ${INPUT_DIR}/${SAMPLE}_S1_L001_R1_001.fastq.gz \
  -I ${INPUT_DIR}/${SAMPLE}_S1_L001_R2_001.fastq.gz \
  -o ${OUTPUT_DIR}/${SAMPLE}.fastp.R1.fastq.gz \
  -O ${OUTPUT_DIR}/${SAMPLE}.fastp.R2.fastq.gz \
  --failed_out ${OUTPUT_DIR}/${SAMPLE}.failed.fastp.fastq.gz \
  --qualified_quality_phred 0 \
  --unqualified_percent_limit 0 \
  --average_qual 0 \
  --n_base_limit 5 \
  --disable_adapter_trimming \
  --disable_trim_poly_g \
  --disable_length_filtering

# 方法 1：用 seqtk 拆分 interleaved（推荐）
seqtk seq -1 ${OUTPUT_DIR}/${SAMPLE}.failed.fastp.fastq.gz | gzip > ${OUTPUT_DIR}/${SAMPLE}.failed.R1.fastq.gz
seqtk seq -2 ${OUTPUT_DIR}/${SAMPLE}.failed.fastp.fastq.gz | gzip > ${OUTPUT_DIR}/${SAMPLE}.failed.R2.fastq.gz

# 方法 2：用 awk 拆分（不依赖 seqtk）
zcat ${OUTPUT_DIR}/${SAMPLE}.failed.fastp.fastq.gz | awk -v prefix="${OUTPUT_DIR}/${SAMPLE}.failed" '
  NR%8==1 || NR%8==2 || NR%8==3 || NR%8==4 {print | "gzip > " prefix ".R1.fastq.gz"}
  NR%8==5 || NR%8==6 || NR%8==7 || NR%8==0 {print | "gzip > " prefix ".R2.fastq.gz"}
'
```

---

### 4.2 方案二：bbduk（BBMap）

bbduk 通过 `maxns` 参数直接支持 N 碱基过滤，逻辑更清晰，且 `outm1/outm2` 可直接输出失败的 R1/R2。

```bash
/home/ubuntu/app/bioinfo/datasplit_v3/BBMap/current/bbduk.sh \
  -Xmx16g \
  in1=/mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R1_001.fastq.gz \
  in2=/mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R2_001.fastq.gz \
  out1=/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/6a2264de138eaf134c4cbf63--MJ260602P_J69.R1.fastq.gz \
  out2=/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/6a2264de138eaf134c4cbf63--MJ260602P_J69.R2.fastq.gz \
  outm1=/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/failed.R1.fastq.gz \
  outm2=/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/failed.R2.fastq.gz \
  maxns=5 \
  minlength=1 \
  minavgquality=0 \
  minbasequality=0 \
  qtrim=f \
  stats=/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/bbduk_stats.txt
```

**关键参数说明**：
- `maxns=5`：N 数量 > 5 的 reads 丢弃
- `minlength=1`：关闭 bbduk 默认的 10bp 长度过滤
- `minavgquality=0` / `minbasequality=0`：关闭质量过滤
- `qtrim=f`：关闭质量修剪
- `outm1/outm2`：直接输出失败的 R1/R2，无需 interleaved 拆分

---

## 5. 方案对比

| 对比项 | fastp | bbduk |
|--------|-------|-------|
| 核心参数 | `--n_base_limit 5` | `maxns=5` |
| 参数复杂度 | 中等，需显式把质量阈值置 0 | 低，直接设置 `maxns` 即可 |
| 隐藏陷阱 | `--disable_quality_filtering` 会禁用 N 过滤 | 默认 `minlength=10` 会额外过滤短 reads |
| 失败 reads 输出 | `--failed_out` 输出 interleaved，需拆分 | `outm1/outm2` 直接输出独立 R1/R2 |
| 是否需要后处理 | 需要 `seqtk seq -1/-2` 或 `awk` 拆分 | 不需要 |
| 统计报告 | stdout 输出 | `stats=` 文件 |
| 推荐度 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 6. 推荐方案与实现

**推荐：bbduk 方案。**

理由：
1. `maxns` 逻辑清晰，不受其他参数干扰
2. `outm1/outm2` 天然支持保留失败 reads，无需拆分
3. `stats=` 可直接生成审计报告
4. 更适合批量自动化流程

### 6.1 单样本命令

```bash
/home/ubuntu/app/bioinfo/datasplit_v3/BBMap/current/bbduk.sh \
  -Xmx16g \
  in1=${SAMPLE}_R1_001.fastq.gz \
  in2=${SAMPLE}_R2_001.fastq.gz \
  out1=${SAMPLE}.clean.R1.fastq.gz \
  out2=${SAMPLE}.clean.R2.fastq.gz \
  outm1=${SAMPLE}.failed.R1.fastq.gz \
  outm2=${SAMPLE}.failed.R2.fastq.gz \
  maxns=5 \
  minlength=1 \
  minavgquality=0 \
  minbasequality=0 \
  qtrim=f \
  stats=${SAMPLE}.bbduk_stats.txt
```

### 6.2 批量处理脚本

```bash
#!/bin/bash
# filter_n_bulk.sh - 批量 N 碱基过滤
# 用法：bash filter_n_bulk.sh

set -euo pipefail

INPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/Fastq"
OUTPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n"
LOG_DIR="${OUTPUT_DIR}/logs"
BBDUK="/home/ubuntu/app/bioinfo/datasplit_v3/BBMap/current/bbduk.sh"

# 前置检查
if [[ ! -x "${BBDUK}" ]]; then
    echo "ERROR: bbduk 未找到或没有执行权限: ${BBDUK}" >&2
    exit 1
fi

if [[ ! -d "${INPUT_DIR}" ]]; then
    echo "ERROR: 输入目录不存在: ${INPUT_DIR}" >&2
    exit 1
fi

mkdir -p "${OUTPUT_DIR}" "${LOG_DIR}"

echo "[$(date '+%Y-%m-%d %H:%M:%S')] 开始批量 N 碱基过滤 ..."
echo "输入目录: ${INPUT_DIR}"
echo "输出目录: ${OUTPUT_DIR}"

# 计数器
total=0
success=0
failed=0

for r1 in "${INPUT_DIR}"/*/*_R1_001.fastq.gz; do
    # 防止 glob 不匹配时进入循环
    [[ -e "${r1}" ]] || continue

    total=$((total + 1))
    sample=$(basename "${r1}" _R1_001.fastq.gz)
    dir=$(dirname "${r1}")
    r2="${dir}/${sample}_R2_001.fastq.gz"

    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [${total}] 处理 ${sample} ..."

    # 检查 R2 是否存在
    if [[ ! -f "${r2}" ]]; then
        echo "WARNING: R2 文件不存在，跳过: ${sample}" >&2
        failed=$((failed + 1))
        continue
    fi

    # 执行 bbduk
    if "${BBDUK}" \
        -Xmx16g \
        in1="${r1}" \
        in2="${r2}" \
        out1="${OUTPUT_DIR}/${sample}.clean.R1.fastq.gz" \
        out2="${OUTPUT_DIR}/${sample}.clean.R2.fastq.gz" \
        outm1="${OUTPUT_DIR}/${sample}.failed.R1.fastq.gz" \
        outm2="${OUTPUT_DIR}/${sample}.failed.R2.fastq.gz" \
        maxns=5 \
        minlength=1 \
        minavgquality=0 \
        minbasequality=0 \
        qtrim=f \
        stats="${OUTPUT_DIR}/${sample}.bbduk_stats.txt" \
        2> "${LOG_DIR}/${sample}.bbduk.log"; then

        echo "[$(date '+%Y-%m-%d %H:%M:%S')] ${sample} 完成 ✓"
        success=$((success + 1))
    else
        echo "ERROR: ${sample} 处理失败，日志: ${LOG_DIR}/${sample}.bbduk.log" >&2
        failed=$((failed + 1))
    fi
done

echo "[$(date '+%Y-%m-%d %H:%M:%S')] 批量处理结束"
echo "总计: ${total}, 成功: ${success}, 失败: ${failed}"

if [[ ${failed} -gt 0 ]]; then
    echo "请检查 ${LOG_DIR} 目录中的日志文件定位失败原因" >&2
    exit 1
fi
```

---

## 7. 验证结果

使用 `seqkit stats -a` 对比输入输出 read 数量，验证过滤正确性。

```bash
seqkit stats -a \
  /mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R1_001.fastq.gz \
  /mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--MJ260602P_J69/MJ260602P_J69_S1_L001_R2_001.fastq.gz \
  /mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/6a2264de138eaf134c4cbf63--MJ260602P_J69.R1.fastq.gz \
  /mnt/data/result/CF9-20260603PE300-P2-N1/trim_n/6a2264de138eaf134c4cbf63--MJ260602P_J69.R2.fastq.gz
```

**验证数据**（按 pair 统计，R1+R2 总和 ÷ 2）：

| 样本 | 工具 | 输入 pairs | 输出 pairs | 过滤 pairs | 过滤比例 | 状态 |
|------|------|-----------|-----------|-----------|---------|------|
| MJ260602P_J69 | fastp（修正方案） | 3,601,012 | 2,783,454 | 817,558 | **22.70%** | ✅ 通过 |
| MJ260602P_J69 | bbduk | 3,601,012 | 2,783,454 | 817,558 | **22.70%** | ✅ 通过 |
| MJ260602P_J70 | bbduk | 1,406,547 | 1,082,941 | 323,606 | **23.01%** | ✅ 通过 |

> 注：
> 1. 输入 pairs = (R1 reads + R2 reads) / 2；过滤 pairs = 被丢弃的 read 条数 / 2。
> 2. 核心验收标准：输出 pairs = 输入 pairs − 过滤 pairs，误差应 < 1%。
> 3. PE300 数据中 R2 末端 N 碱基比例显著高于 R1，因此实际 pair 级别过滤率（~23%）远高于单端 R1 统计（~0.7%）。

**验证结论**：
- fastp 修正方案和 bbduk 方案过滤结果一致
- 两种方案均满足验收标准
- bbduk 方案无需 interleaved 拆分，更适合批量自动化

**结果截图**：

![fastp 运行结果统计](fastp运行结果统计.png)

**图 1**：fastp 修正方案 `seqkit stats -a` 输出。以 MJ260602P_J69 为例，R1+R2 总计 7,202,024 条 reads，过滤后剩余 5,566,908 条 reads，对应过滤 817,558 pairs（22.70%）。

![bbduk 运行统计结果](bbduk运行统计结果.png)

**图 2**：bbduk 处理 MJ260602P_J69 后的 `seqkit stats -a` 输出，结果与 fastp 一致。

![bbduk 运行统计结果 2](bbduk运行统计结果2.png)

**图 3**：bbduk 处理 MJ260602P_J70 后的 `seqkit stats -a` 输出，验证该方案在多样本上的稳定性。

---

## 8. 注意事项与陷阱

### 8.1 fastp `--disable_quality_filtering` 会禁用 `--n_base_limit`

**错误用法**：
```bash
fastp ... --n_base_limit 5 --disable_quality_filtering
```

**正确用法**：
```bash
fastp ... --n_base_limit 5 --qualified_quality_phred 0 --unqualified_percent_limit 0 --average_qual 0
```

### 8.2 fastp `--failed_out` 对 PE 输出为 interleaved

fastp 的 `--failed_out` 对 PE reads 输出的是 R1/R2 交替的 interleaved FASTQ，不是独立的两个文件。如需独立 R1/R2，需用 `seqtk seq -1/-2` 或 `awk` 拆分，**不推荐 `seqkit split2`**（实测兼容性不佳），或改用 bbduk 避免拆分。

### 8.3 bbduk 默认 `minlength=10` 会额外过滤短 reads

即使只开了 `maxns=5`，bbduk 默认也会过滤掉短于 10bp 的 reads。如需纯 N 过滤，必须显式设置 `minlength=1`。

---

## 9. 备选方案：fastp

如果环境中没有 bbduk，或希望统一使用 fastp 做质控，可使用以下修正命令：

```bash
SAMPLE="MJ260602P_J69"
INPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/Fastq/6a2264de138eaf134c4cbf63--${SAMPLE}"
OUTPUT_DIR="/mnt/data/result/CF9-20260603PE300-P2-N1/trim_n"

fastp \
  -i ${INPUT_DIR}/${SAMPLE}_S1_L001_R1_001.fastq.gz \
  -I ${INPUT_DIR}/${SAMPLE}_S1_L001_R2_001.fastq.gz \
  -o ${OUTPUT_DIR}/${SAMPLE}.clean.R1.fastq.gz \
  -O ${OUTPUT_DIR}/${SAMPLE}.clean.R2.fastq.gz \
  --qualified_quality_phred 0 \
  --unqualified_percent_limit 0 \
  --average_qual 0 \
  --n_base_limit 5 \
  --disable_adapter_trimming \
  --disable_trim_poly_g \
  --disable_length_filtering
```

如需保留失败 reads（注意 fastp 输出为 interleaved，需拆分）：

```bash
fastp \
  -i ${INPUT_DIR}/${SAMPLE}_S1_L001_R1_001.fastq.gz \
  -I ${INPUT_DIR}/${SAMPLE}_S1_L001_R2_001.fastq.gz \
  -o ${OUTPUT_DIR}/${SAMPLE}.clean.R1.fastq.gz \
  -O ${OUTPUT_DIR}/${SAMPLE}.clean.R2.fastq.gz \
  --failed_out ${OUTPUT_DIR}/${SAMPLE}.failed.fastq.gz \
  --qualified_quality_phred 0 \
  --unqualified_percent_limit 0 \
  --average_qual 0 \
  --n_base_limit 5 \
  --disable_adapter_trimming \
  --disable_trim_poly_g \
  --disable_length_filtering

# 用 seqtk 拆分为独立 R1/R2（推荐）
seqtk seq -1 ${OUTPUT_DIR}/${SAMPLE}.failed.fastq.gz | gzip > ${OUTPUT_DIR}/${SAMPLE}.failed.R1.fastq.gz
seqtk seq -2 ${OUTPUT_DIR}/${SAMPLE}.failed.fastq.gz | gzip > ${OUTPUT_DIR}/${SAMPLE}.failed.R2.fastq.gz

# 或用 awk 拆分（零依赖）
zcat ${OUTPUT_DIR}/${SAMPLE}.failed.fastq.gz | awk -v prefix="${OUTPUT_DIR}/${SAMPLE}.failed" '
  NR%8==1 || NR%8==2 || NR%8==3 || NR%8==4 {print | "gzip > " prefix ".R1.fastq.gz"}
  NR%8==5 || NR%8==6 || NR%8==7 || NR%8==0 {print | "gzip > " prefix ".R2.fastq.gz"}
'
```

---

## 10. 参考

- [fastp 官方文档](https://github.com/OpenGene/fastp)
- [BBMap/bbduk 官方文档](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/)
- [seqkit 文档](https://bioinf.shenwei.me/seqkit/)
