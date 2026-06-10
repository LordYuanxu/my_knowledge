# bcl-convert v4.5 快速参考

> 适用版本：v4.5.x  
> 最后更新：基于 v4.5.4 (2026-03-31) 发布文档  
> 继承特性：v4.4 的所有功能均保留，以下是 v4.5 的新增/变化内容

---

## 一、最简运行命令

```bash
bcl-convert \
  --bcl-input-directory <BCL_ROOT_DIR> \
  --output-directory <OUTPUT_DIR> \
  --sample-sheet <SampleSheet.csv>
```

---

## 二、v4.5 核心新增特性（对比 v4.4）

### 2.1 AutoDetect 模式（自动样本表生成）⭐

v4.4.7+ 引入，v4.5 修复了 bug 并增强鲁棒性。

**作用**：自动从运行数据中检测索引、识别 adapter 序列、推断 demux 参数、发现样本 → 生成新样本表，**大幅减少手动配置**。

```bash
# AutoDetect 运行流程
bcl-convert --bcl-input-directory <DIR> --output-directory <DIR> --auto-detect true
```

**AutoDetect 会输出**：
1. 检测到的索引序列（新样本）
2. 修正后的样本表（Corrected SampleSheet）
3. 自动识别的 adapter 和 demux 参数

> 详见 DRAGEN 文档中的 BCL Autodetect Flow 图示。

### 2.2 大运行吞吐量改进

| 场景 | 改进 |
|------|------|
| >1,000 样本的大运行 | demux 性能显著提升 |
| 超大样本数 | 内存和线程优化 |

### 2.3 No-Lane-Splitting + Lane 支持 ⭐

v4.5 支持在 **保留 Lane 列** 的同时启用 no-lane-splitting：

- 样本表中的 `Lane` 列可指定该样本属于哪些 lane
- BCL Convert 会将这些 lane 的 FASTQ **合并输出为一对文件**（per-lane FASTQ output）
- 即：一个样本可输出 `Sample_L001-L004_R1_001.fastq.gz`（合并了 lane 1-4）

```
[BCLConvert_Data]
Sample_ID,index,index2,Lane
Sample1,ATCG,TAGC,1
Sample1,ATCG,TAGC,2
Sample1,ATCG,TAGC,3
Sample1,ATCG,TAGC,4
```

> 配合 `--no-lane-splitting true` 使用，输出合并后的 FASTQ。

---

## 三、关键命令行参数速查（继承 + 新增）

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--bcl-input-directory` | BCL 输入目录（必需） | - |
| `--output-directory` | 输出目录（必需） | - |
| `--sample-sheet` | 样本表路径 | BCL 目录下查找 |
| `--auto-detect` | **v4.5 新增**：启用 AutoDetect 模式 | false |
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
| `--no-sample-sheet` | 无样本表模式 | false |
| `--strict-mode` | 文件缺失时中止 | false |
| `--tiles <regex>` | 白名单正则过滤 tile | - |
| `--exclude-tiles <regex>` | 黑名单正则过滤 tile | - |
| `--bcl-num-parallel-tiles` | 并行 tile 数 | 1 |
| `--bcl-num-conversion-threads` | 转换线程数/tile | CPU 线程数 |
| `--bcl-num-compression-threads` | 压缩线程数/tile | CPU 线程数 or HW+12 |
| `--bcl-num-decompression-threads` | 解压缩线程数/tile | ½ CPU 线程数 or HW+8 |

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
| `FindAdaptersWithIndels` | false | true/false | 单 indel 检测 adapter 修剪 |
| `IndependentIndexCollisionCheck` | 空 | 1~lane 数 | 严格模式：任一索引碰撞即报错 |
| `Index2ColumnReverseComplement` | 空 | Y/N | 显式指定 index2 方向 |
| `LibraryInputVolume` | 空 | 实数 | 启用 LibraryRebalancing_Stats.csv |

---

## 五、v4.5 已修复的问题（对比 v4.4）

| 问题 | 状态 |
|------|------|
| 超长索引的 Top Unknown Barcodes 内存溢出 | ✅ 修复 |
| BCL→ORA one-step + NoLaneSplitting + lane 缺失未报错 | ✅ 修复（现在会报错退出） |
| CBCL 数据集缺失文件时工作流未正确退出 | ✅ 修复 |
| `bcl-only-matched-reads=true` + ORA 压缩错误 | ✅ 修复 |
| AutoDetect 模式的 bug 和鲁棒性问题 | ✅ 修复并增强 |

---

## 六、持续已知问题

| 问题 | 规避方法 |
|------|---------|
| `--sample-sheet` 误传**目录**会挂起 | 务必传入**文件路径** |
| `Sample_Project` = "Logs"/"Reports" 无法创建子目录 | 避免使用这两个名称 |
| `--first-tile-only` 不支持 SP flow cells | 用 `--tiles` 替代 |
| `--force` 会追加 FASTQ（多次运行同一输出目录） | 不要对同一目录多次运行 |
| `ExcludeTiles` 排除的 lane 中若存在索引碰撞仍会报错 | 无规避 |
| 样本表验证通过但 demux 失败 | 条码碰撞检测在验证和实际运行中逻辑略有不同 |

---

## 七、与 bcl2fastq2 的差异（legacy stats）

v4.5 与 v4.4 相同，所有差异均属预期且 bcl-convert 计算更准确：

| 项目 | 说明 |
|------|------|
| `AdapterTrimming.txt` | bcl2fastq2 多列额外 cycle（值设为 0） |
| `PercentageOfBases` | bcl2fastq2 把 OverrideCycles 修剪碱基计入总数 |
| `ConversionStats.xml` Raw | Undetermined/all 样本 Raw cluster 非确定性差异 |
| Top Unknown Barcodes | bcl2fastq2 非确定性猜测 |
| `CreateFastqForIndexReads` | bcl2fastq2 有时会翻倍 Raw 计数（bug） |

---

## 八、v4.4 → v4.5 升级要点

| 方面 | v4.4 | v4.5 |
|------|------|------|
| **AutoDetect** | v4.4.7 引入，有 bug | ✅ 修复并增强鲁棒性 |
| **大样本性能** | 一般 | ✅ >1000 样本吞吐量提升 |
| **NoLaneSplitting + Lane** | v4.4.6 支持 | ✅ 完全支持 per-lane 合并输出 |
| **超长索引内存** | 可能溢出 | ✅ 修复 |
| **ORA + NoLaneSplitting** | 不报错 | ✅ 正确报错 |
| **proteomics prep 样本** | v4.4.6 不建议 | v4.5 未提及此限制 |

---

## 九、快速决策表

| 场景 | 推荐设置 |
|------|---------|
| 标准双端测序 demux | `--bcl-input-directory` + `--output-directory` + `--sample-sheet` |
| 不想手动写样本表 | `--auto-detect true`（AutoDetect 模式） |
| 多 lane 合并为一个 FASTQ | `--no-lane-splitting true` |
| 样本跨多个 lane，希望合并输出 | `NoLaneSplitting,true` + Lane 列指定 lanes |
| 不需要 Undetermined FASTQ | `--bcl-only-matched-reads true` |
| 需要 index reads 的 FASTQ | `--create-fastq-for-index-reads true` |
| 大样本数（>1000）运行 | v4.5 默认优化，无需特殊设置 |
| 直接输出 ORA（跳过 FASTQ.GZ） | `--fastq-compression-format dragen --ora-reference <path> --bcl-ora-direct true` |
| 兼容 bcl2fastq2 统计格式 | `--output-legacy-stats true` |
| 混合单/双索引样本在同一 lane | Per-Sample `OverrideCycles`，不启用 `IndependentIndexCollisionCheck` |
| 重新定义读段结构（非标准 RunInfo） | `OverrideReads` 全局设置 |
| 需要 BAM RG 标签传播 | 在 [BCLConvert_Data] 添加 `RGID`、`RGSM`、`RGPU`、`RGPL`、`RGLB`、`RGCN`、`RGPM` 列 |
| 自定义样本管理字段 | `custom_*` 列（v2 样本表自动忽略不报错） |
