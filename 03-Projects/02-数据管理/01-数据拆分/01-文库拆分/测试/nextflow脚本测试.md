# 信息导出数据库

```bash
/home/ubuntu/app/bioinfo/datasplit_v3/bin/mjdm-cli-linux-x64 \
mongo-export \
library-split \
--split-id 6a680b7afc045e33b5078a63 \
--mongo-env prod \
--output /home/ubuntu/majorbio_task/nextflow/导出数据库/library_split.json \
--format json \
--indent 4

/mnt/clustre/users/sanger-dev/bin/mjdm-cli-linux-x64 \
mongo-export \
library-split \
--split-id 6a680b7afc045e33b5078a63 \
--mongo-env prod \
--output /mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/导出数据库/library_split.json \
--format json \
--indent 4
```

# 指向nextflow脚本

```bash
# 新模式：从数据库导出后直接拆分
nextflow \
run \
/mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/main.nf \
-profile slurm \
-resume \
--split_id 6a671ee3b6b11021674518a6 \
--mongo_env prod \
--output_dir /mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/CF1-20260727PE300-P2-N2/result
# 旧模式仍然可用
nextflow run main.nf -profile slurm --json_input /path/to/library_split.json
```

# 上传s3

```bash
/mnt/clustre/users/sanger-dev/bin/mjdm-cli-linux-x64 \
s3 \
upload \
/mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/CF1-20260727PE300-P2-N2/result/260727PE300-P2-N2/Fastq/6a671eefb6b1102167452036--C260724P_30/C260724P_30_S49_L001_R1_001.fastq.gz \
s3nb4://datasplit/test/6a671eefb6b1102167452036--C260724P_30/C260724P_30_S49_L001_R1_001.fastq.gz

```


# 上海测试

```bash
nextflow \
run \
/mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/main.nf \
-profile slurm \
-resume \
--split_id 6a599b2efa8d9675d619f2e2 \
--mongo_env test \
--output_dir /mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/result

# CF1-20260727PE300-P2-N2
nextflow \
run \
/mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/main.nf \
-profile slurm \
-resume \
--split_id 6a671ee3b6b11021674518a6 \
--mongo_env prod \
--output_dir /mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/nextflow/CF1-20260727PE300-P2-N2/result
```

# 上海模拟天津测试

>拆分任务: CF1-20260613X25BX1B-3
>bcl路径: /mnt/upload/LH00272/2026/20260613_LH00272_0614_B23N7HGLT4
>同步bcl路径: /mnt/clustre/users/sanger-dev/rsync_data/bcl_data/LH00272/2026/20260613_LH00272_0614_B23N7HGLT4
>天津bcl路径: /mnt/data/bcl_data/LH00272/2026/20260613_LH00272_0614_B23N7HGLT4

```bash
# 同步上海数据
rclone \
copy \
-P \
--transfers 16 \
--checkers 32 \
--checksum \
/mnt/upload/LH00272/2026/20260613_LH00272_0614_B23N7HGLT4/ \
/mnt/clustre/users/sanger-dev/rsync_data/bcl_data/LH00272/2026/20260613_LH00272_0614_B23N7HGLT4/ \
--log-file /mnt/clustre/users/sanger-dev/log/rclone/archive_$(date +%Y%m%d).log

# 同步天津数据
# rclone进行远程sftp配置
rclone \
copy \
-P \
--transfers 16 \
--checkers 32 \
--checksum \
/mnt/upload/LH00272/2026/20260613_LH00272_0614_B23N7HGLT4/ \
tianjin_test:/mnt/data/bcl_data/LH00272/2026/20260613_LH00272_0614_B23N7HGLT4/ \
--log-file /mnt/clustre/users/sanger-dev/log/rclone/archive_$(date +%Y%m%d).log
```