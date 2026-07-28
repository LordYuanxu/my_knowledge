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