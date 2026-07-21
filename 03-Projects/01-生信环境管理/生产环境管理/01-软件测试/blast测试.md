# 安装

官网: https://blast.ncbi.nlm.nih.gov/doc/blast-help/downloadblastdata.html
形式: 直接解压缩获得二进制执行文件

# 数据库

**nt_16s:** `~/app/bioinfo/datasplit_v3/database/nt_16s`
**nt_ITS**: `~/app/bioinfo/datasplit_v3/database/nt_ITS`

```bash
# 构建索引
~/app/bioinfo/datasplit_v3/blast/current/bin/makeblastdb \
-in ~/app/bioinfo/datasplit_v3/database/nt_ITS/nt_v20260625_ITs.fasta \
-dbtype nucl \
-out ~/app/bioinfo/datasplit_v3/database/nt_ITS/nt_ITs \
-parse_seqids \
-hash_index

# 快速检查索引是否建好 
~/app/bioinfo/datasplit_v3/blast/current/bin/blastdbcmd \
-db ~/app/bioinfo/datasplit_v3/database/nt_ITS/nt_ITs \
-info
```