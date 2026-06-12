# kraken
## 创建kraken环境
```bash
# 创建kraken环境(安装kraken2和bracken)
$HOME/app/bioinfo/datasplit_v3/miniconda3/bin/conda create -n kraken2 -y
$HOME/app/bioinfo/datasplit_v3/miniconda3/bin/conda activate kraken2
$HOME/app/bioinfo/datasplit_v3/miniconda3/bin/conda install -n kraken2 -c bioconda kraken2 -y
$HOME/app/bioinfo/datasplit_v3/miniconda3/bin/conda install -n kraken2 -c bioconda bracken -y
```

## kraken简单使用
```bash
# 构建brakren的索引
$HOME/app/bioinfo/datasplit_v3/miniconda3/envs/kraken2/bin/bracken-build \
-d \ 
-t 8 \
-x $HOME/app/bioinfo/datasplit_v3/miniconda3/envs/kraken2/bin \
-l 100

## 运行kraken
export LD_LIBRARY_PATH=$HOME/app/gcc/7.2.0/lib64:$LD_LIBRARY_PATH
$HOME/app/bioinfo/metaGenomic/kraken2-2.0.8-beta/bin/kraken2 \
--threads 8 \
--db $HOME/app/database/kranken2 \
--report $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473.taxon.xls \
--confidence 0.1 \
--output $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473.mapping.stdout \
$HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473.consensus.fasta

## 运行brakren
$HOME/app/program/Python/bin/python \
$HOME/wpm2/sanger_bioinfo/src/mbio/packages/metagenomic/est_abundance.py \
-i $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473.taxon.xls \
-k $HOME/app/database/kranken2/database100mers.kmer_distrib \
-o $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473_P.xls \
-l P \
-t 0
$HOME/app/program/Python/bin/python \
$HOME/wpm2/sanger_bioinfo/src/mbio/packages/metagenomic/est_abundance.py \
-i $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473.taxon.xls \
-k $HOME/app/database/kranken2/database100mers.kmer_distrib \
-o $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473_G.xls \
-l G \
-t 0
$HOME/app/program/Python/bin/python \
$HOME/wpm2/sanger_bioinfo/src/mbio/packages/metagenomic/est_abundance.py \
-i $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473.taxon.xls \
-k $HOME/app/database/kranken2/database100mers.kmer_distrib \
-o $HOME/sg-users/xuyuan/majorbio_task/datasplit_v3/菌鉴/S22504260958-EJD1101473_S.xls \
-l S \
-t 0
```

## kraken数据库结构
| 文件名                | 作用                                                                              |
| --------------------- | --------------------------------------------------------------------------------- |
| `hash.k2d`            | 核心哈希表索引，存储 k-mer（短序列片段）与物种 TaxID 的映射关系                   |
| `opts.k2d`            | 数据库构建时的参数配置（如 k-mer 长度、哈希表大小等）                             |
| `taxo.k2d`            | 物种分类树（Taxonomy）数据，包含 TaxID 对应的界、门、纲、目、科、属、种等分类信息 |
| `seqid2taxid.map`     | 参考序列 ID 与 TaxID 的映射表（记录每条参考序列属于哪个物种）                     |
| `library.fna`（可选） | 构建数据库所用的原始参考基因组 FASTA 文件（部分场景需要，如更新数据库）           |
