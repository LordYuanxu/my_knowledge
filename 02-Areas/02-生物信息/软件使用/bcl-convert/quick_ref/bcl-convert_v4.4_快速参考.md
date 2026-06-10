# bcl-convert v4.4 快速参考

> 适用版本：v4.4.4 ~ v4.4.6  
> 最后更新：基于 v4.4.6 (2025-09-17) 发布文档

---

## 一、最简运行命令

```bash
bcl-convert \
  --bcl-input-directory <BCL_ROOT_DIR> \
  --output-directory <OUTPUT_DIR> \
  --sample-sheet <SampleSheet.csv>
```

---

## 二、关键命令行参数速查

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--bcl-input-directory` | BCL 输入目录（必需） | - |
| `--output-directory` | 输出目录（必需） | - |
| `--sample-sheet` | 样本表路径 | BCL 目录下查找 |
| `--force` | 允许输出目录已存在 | false |
| `--bcl-only-lane <N>` | 仅转换指定 lane | 全部 |
| `--no-lane-splitting` | 不按 lane 拆分 FASTQ | false |
| `--bcl-sampleproject-subdirectories` | 按 Sample_Project 输出子目录 | false |
| `--sample-name-column-enabled` | 使用 Sample_Name 命名 FASTQ | false |
| `--bcl-only-matched-reads` | 不输出 Undetermined FASTQ | false |
| `--create-fastq-for-index-reads` | 为 index reads 输出 FASTQ | false |
| `--fastq-gzip-compression-level` | gzip 压缩级别 0-9 | 1 |
| `--num-unknown-barcodes-reported` | Top Unknown Barcodes 数量 | 1000 |
| `--output-legacy-stats` | 输出 bcl2fastq2 兼容统计 | false |
| `--bcl-validate-sample-sheet-only` | 仅验证，不生成 FASTQ | false |
| `--no-sample-sheet` | 无样本表模式（全部进 Undetermined） | false |
| `--strict-mode` | 文件缺失时中止 | false |
| `--first-tile-only` | 仅处理第一个 tile（调试用） | false |
| `--tiles <regex>` | 白名单正则过滤 tile | - |
| `--exclude-tiles <regex>` | 黑名单正则过滤 tile | - |
| `--bcl-num-parallel-tiles` | 并行 tile 数 | 1 |
| `--bcl-num-conversion-threads` | 转换线程数/til | CPU 线程数 |
| `--bcl-num-compression-threads` | 压缩线程数/til | CPU 线程数 or HW+12 |
| `--bcl-num-decompression-threads` | 解压缩线程数/tile | ½ CPU 线程数 or HW+8 |

---

## 三、v4.4 新增/变化特性（对比 4.3）

### 3.1 样本表新设置

| 设置 | 说明 | 示例 |
|------|------|------|
| `OverrideReads` | **全局覆盖** RunInfo 的读段结构（长度 + 方向） | `OverrideReads,Y151;I10;I10r;Y151r` |
| `Index2ColumnReverseComplement` | 显式指定 index2 方向（Y=反向互补, N=正向） | `Y` 或 `N`（大小写敏感） |
| `custom_*` 字段 | 自定义列，v2 样本表**忽略不报错**，用于样本管理 | `custom_LabID` 等 |

> **OverrideReads vs OverrideCycles 区别**：
> - `OverrideCycles`：在 RunInfo 结构内调整（mask/UMI）
> - `OverrideReads`：**完全重新定义**读段结构，覆盖 RunInfo。若指定，OverrideCycles 必须在其框架内操作。
> - 格式：`Y151;I10;I10r;Y151r`（r 后缀表示 paired-end turn 后的读段，影响 index2 解释）

### 3.2 输出文件更新

| 新增/更新文件 | 说明 |
|--------------|------|
| `Demultiplex_Detailed_Stats.csv` | **新增**：cycle & transition error details（错配容忍 demux 的 miscall metrics） |
| `Demultiplex_Tile_Metrics.csv` | **扩展列**：包含与聚合 `Demultiplex_Stats.csv` 相同的派生统计 |
| `--bcl-only-matched-reads` 时 | **仍生成 Undetermined 统计**（不输出 FASTQ 文件） |

### 3.3 ORA 直接压缩（BCL → ORA 跳过 FASTQ.GZ）

```bash
# BCL 直接转 ORA，跳过中间 FASTQ.GZ，提速 ~30%
# 限制：每 lane 最多 40 个样本，不能与 interleaved 同时使用
bcl-convert \
  --bcl-input-directory <DIR> \
  --output-directory <DIR> \
  --sample-sheet <Sheet> \
  --fastq-compression-format dragen \
  --ora-reference <path/to/oradata> \
  --bcl-ora-direct true
```

### 3.4 内存优化

- 新增内部设置降低**大样本批次**的内存使用

---

## 四、样本表设置（[BCLConvert_Settings]）

| 设置 | 默认值 | 可选值 | 说明 |
|------|--------|--------|------|
| `AdapterBehavior` | trim | trim, mask | adapter 处理方式 |
| `AdapterRead1` | None | ACGT 序列 | Read 1 adapter 序列 |
| `AdapterRead2` | None | ACGT 序列 | Read 2 adapter 序列 |
| `AdapterStringency` | 0.9 | 0.5~1.0 | 滑动窗口匹配严格度 |
| `BarcodeMismatchesIndex1` | 1 | 0,1,2 | i7 允许错配数 |
| `BarcodeMismatchesIndex2` | 1 | 0,1,2 | i5 允许错配数 |
| `MinimumTrimmedReadLength` | min(35,最短读长) | 0~最短读长 | 低于此长度则 mask |
| `MinimumAdapterOverlap` | 1 | 1,2,3 | 最短 adapter 匹配长度 |
| `MaskShortReads` | min(22,MinTrimLen) | 0~MinTrimLen | 短读 mask 长度 |
| `OverrideCycles` | None | Y/I/U 字符串 | 指定 UMI 和 mask |
| `OverrideReads` | None | Y/I + 长度 + r | 覆盖 RunInfo 读段结构 |
| `TrimUMI` | true | true/false (1/0) | 是否从 FASTQ 中修剪 UMI |
| `CreateFastqForIndexReads` | false | true/false (1/0) | 输出 index reads FASTQ |
| `NoLaneSplitting` | false | true/false | 所有 lane 合并为一个 FASTQ |
| `FastqCompressionFormat` | gzip | gzip/dragen/dragen-interleaved | 压缩格式 |
| `FindAdaptersWithIndels` | false | true/false | 单 indel 检测适配器修剪 |
| `IndependentIndexCollisionCheck` | 空 | 1~lane 数 | **严格模式**：任一索引碰撞即报错 |
| `Index2ColumnReverseComplement` | 空 | Y/N | 显式指定 index2 方向 |
| `LibraryInputVolume` | 空 | 实数 | 启用 LibraryRebalancing_Stats.csv |

> ⚠️ **索引碰撞默认策略（v4.4）**：
> - **默认 relaxed（兼容 bcl2fastq2）**：两个索引同时碰撞才报错
> - 启用 `IndependentIndexCollisionCheck` 后：**任一索引碰撞即报错**

---

## 五、OverrideCycles / OverrideReads 示例

### OverrideCycles（在 RunInfo 结构内调整）

```
# Agilent XT HS: index2 为 10bp UMI
OverrideCycles,Y151;I8;U10;Y151

# IDT Dual Index + UMI: index1 包含 index+9bp UMI
OverrideCycles,Y151;I8U9;I8;Y151

# 顺序无关格式（v4.3.6+）
OverrideCycles,R1:Y151;R2:Y151;I1:I8;I2:U10
```

### OverrideReads（完全覆盖 RunInfo）

```
# 重新定义 4 个读段：Y151, I10, I10(反向), Y151(反向)
OverrideReads,Y151;I10;I10r;Y151r
```

> `r` 后缀表示该读段发生在 paired-end turn 之后，BCL Convert 会据此正确解释 index2。

---

## 六、Per-Sample 设置（[BCLConvert_Data] 列）

v4.1+ 支持在 Data 部分逐样本指定以下设置：

| 列名 | 说明 |
|------|------|
| `OverrideCycles` | 逐样本读段覆盖 |
| `BarcodeMismatchesIndex1` | 逐样本 i7 错配数 |
| `BarcodeMismatchesIndex2` | 逐样本 i5 错配数 |
| `AdapterRead1` | 逐样本 R1 adapter |
| `AdapterRead2` | 逐样本 R2 adapter |
| `AdapterBehavior` | 逐样本 adapter 行为 |
| `AdapterStringency` | 逐样本匹配严格度 |

> 要求：v2 样本表，全局 [BCLConvert_Settings] 中不能同时出现该设置，不适用项填 `na`

---

## 七、v4.4.6 关键修复（对比 v4.4.4）

| 问题 | 状态 |
|------|------|
| AWS EBS/Lustre 输出极慢（4天→3小时/lane） | ✅ 修复 |
| `bcl.bgzf` + `NoLaneSplitting` + `bcl-validate-sample-sheet-only=true` 段错误 | ✅ 修复 |
| NovaX 高样本数（>10K）运行极慢 | ✅ 修复 |
| v4.4.4 demuxmap 完全 masked 索引性能回归 | ✅ 修复 |
| **灵活 no-lane-splitting**：允许 Lane 列存在 | ✅ 新增 |
| Rebalancing stats 报告相同 i5 | ✅ 修复 |

> ⚠️ **v4.4.6 新增已知问题**：
> - **不建议用于 proteomics prep 样本**（运行时间极长）
> - BCL→ORA one-step + NoLaneSplitting 不报错（实际不支持）
> - `bcl-only-matched-reads=true` + ORA 压缩报错（用 `--force` 规避）

---

## 八、与 bcl2fastq2 的差异（legacy stats）

| 项目 | 说明 |
|------|------|
| `AdapterTrimming.txt` | cycle 数不同（bcl2fastq2 多列了额外 cycle，值设为 0） |
| `PercentageOfBases` | bcl2fastq2 计算方式有误（把 OverrideCycles 修剪的碱基计入总数） |
| `ConversionStats.xml` Raw | Undetermined 和 all 样本的 Raw cluster 不同（bcl2fastq2 用统计猜测，非确定性） |
| Top Unknown Barcodes | 序列和数值不同（bcl2fastq2 非确定性） |
| `CreateFastqForIndexReads` | 启用时 bcl2fastq2 有时会翻倍 Raw 计数（bcl2fastq2 bug） |

> **结论**：bcl-convert 的统计计算更准确，差异均属预期。

---

## 九、持续已知问题（所有版本）

| 问题 | 规避方法 |
|------|---------|
| `--sample-sheet` 误传**目录**会挂起 | 务必传入**文件路径** |
| `Sample_Project` = "Logs"/"Reports" 无法创建子目录 | 避免使用这两个名称 |
| `--first-tile-only` 不支持 SP flow cells | 用 `--tiles` 替代 |
| `--force` 会追加 FASTQ（多次运行同一输出目录） | 不要对同一目录多次运行 |
| `ExcludeTiles` 排除的 lane 中若存在索引碰撞仍会报错 | 无规避 |

---

## 十、快速决策表

| 场景 | 推荐设置 |
|------|---------|
| 标准双端测序 demux | `--bcl-input-directory` + `--output-directory` + `--sample-sheet` |
| 多 lane 合并为一个 FASTQ | `--no-lane-splitting true`（v4.4.6+ 允许 Lane 列存在） |
| 不需要 Undetermined FASTQ | `--bcl-only-matched-reads true` |
| 需要 index reads 的 FASTQ | `--create-fastq-for-index-reads true` |
| 大样本数（>1000）性能优化 | 默认线程，确保高核心数无 64 线程限制（v4.0.5+ 已修复） |
| 直接输出 ORA（跳过 FASTQ.GZ） | `--fastq-compression-format dragen --ora-reference <path> --bcl-ora-direct true` |
| 兼容 bcl2fastq2 统计格式 | `--output-legacy-stats true` |
| 混合单/双索引样本在同一 lane | Per-Sample `OverrideCycles`，不启用 `IndependentIndexCollisionCheck` |
| 重新定义读段结构（非标准 RunInfo） | `OverrideReads` 全局设置 |
| 需要 BAM RG 标签传播 | 在 [BCLConvert_Data] 添加 `RGID`、`RGSM`、`RGPU`、`RGPL`、`RGLB`、`RGCN`、`RGPM` 列 |
