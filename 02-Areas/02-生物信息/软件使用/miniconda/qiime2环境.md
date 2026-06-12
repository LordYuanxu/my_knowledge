# qiime2
## 创建conda环境
```bash
# 创建qiime2环境
$HOME/app/bioinfo/datasplit_v3/miniconda3/bin/conda env create \
-n qiime2-amplicon-2024.10 \
--file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml
# 激活环境，命令行前缀会显示环境名
conda activate qiime2-2023.9
# 验证安装成功
qiime --help  # 若显示帮助信息，说明安装成功
# conda activate qiime2-amplicon-2024.10
# 需要安装 BLAST+ 到 QIIME 2 环境
```

## qiime2简单使用
```bash
# fasta转qza
$HOME/app/bioinfo/datasplit_v3/miniconda3/envs/qiime2-amplicon-2024.10/bin/qiime \
tools \ 
import \
--type 'FeatureData[Sequence]' \
--input-path {fasta} \
--output-path {qza}

# 获取consensus方法的blastdb
## 需要安装 BLAST+ 到 QIIME 2 环境(不用重复)
$HOME/app/bioinfo/datasplit_v3/miniconda3/envs/qiime2-amplicon-2024.10/bin/qiime \
feature-classifier \
$HOME/app/bioinfo/datasplit_v3/blast/v2.16.0/bin/makeblastdb \
--i-sequences $HOME/app/bioinfo/datasplit_v3/database/taxon_db/qiime2_qza/NT_Taxon_v2024_16s_bacteria.qza \
--o-database $HOME/app/bioinfo/datasplit_v3/database/taxon_db/qiime2_qza/ref_blastdb.qza

# 创建缓存
$HOME/app/bioinfo/datasplit_v3/miniconda3/envs/qiime2-amplicon-2024.10/bin/qiime \
tools \
cache-create \
--cache $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/tmp/

# qiime2
/usr/bin/time -f "最大使用内存=%Mkb\nCPU使用率=%P\n程序耗时=%es" \
$HOME/app/bioinfo/datasplit_v3/miniconda3/envs/qiime2-amplicon-2024.10/bin/qiime \
feature-classifier \
classify-consensus-blast \
--i-query $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/S22504260958-EJD1101473.qza \
--i-blastdb $HOME/app/bioinfo/datasplit_v3/database/taxon_db/qiime2_qza/NT_Taxon_v2024_16s_bacteria.ref_blastdb.qza \
--i-reference-taxonomy $HOME/app/bioinfo/datasplit_v3/database/taxon_db/qiime2_qza/NT_Taxon_v2024_16s_bacteria_tax.qza \
--p-maxaccepts 5 \
--p-perc-identity 0.8 \
--p-query-cov 0.8 \
--o-classification $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/taxonomy.qza \
--p-num-threads 16 \
--o-search-results  $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/tmp_blast.txt.qza \
--use-cache $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/tmp

# blast_qza_to_txt
$HOME/app/bioinfo/datasplit_v3/miniconda3/envs/qiime2-amplicon-2024.10/bin/qiime \
tools \
export \
--input-path $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/tmp_blast.txt.qza \
--output-path $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/tmp_blast

# qza_to_txt
$HOME/app/bioinfo/datasplit_v3/miniconda3/envs/qiime2-amplicon-2024.10/bin/qiime \
tools \
export \
--input-path $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/taxonomy.qza \
--output-path $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/packages/taxonomy
```