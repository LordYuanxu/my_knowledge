# 使用

## 预览文件元数据

```bash
/mnt/clustre/users/sanger-dev/app/bioinfo/datasplit_v3/spring2/spring2-linux-x86_64.AppImage \
--preview \
-i /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/output/6a66e4342302c5a8669157e8--L1HIJ1400454--Ph2nd50_3.R1.raw.spring

/mnt/clustre/users/sanger-dev/app/bioinfo/datasplit_v3/spring2/spring2-linux-x86_64.AppImage \
--preview \
--audit \
-i /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/output/6a66e4342302c5a8669157e8--L1HIJ1400454--Ph2nd50_3.R1.raw.spring
```

```plain
SPRING2 Archive Metadata Preview:
--------------------------------
Archive Version:   legacy spring
Original Inputs:   Unavailable in legacy spring archives
Assay Type:        Unavailable in legacy spring archives
Mode:              Single-end
Total Read Records:    32312462
Max Read Length:   151 (using short-read encoder)
Preserve Order:    Yes
Preserve IDs:      Yes
Preserve Quality:  Yes
Quality Mode:      Lossless
Compression Level: 6
Use CRLF:          No
```

## 解压缩

```bash
# 解压缩旧版本为fastq文件
/mnt/clustre/users/sanger-dev/app/bioinfo/datasplit_v3/spring2/spring2-linux-x86_64.AppImage \
-d \
-i /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/output/6a66e4342302c5a8669157e8--L1HIJ1400454--Ph2nd50_3.R1.raw.spring \
-o /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/output/6a66e4342302c5a8669157e8--L1HIJ1400454--Ph2nd50_3.R1.raw.fastq

# 解压缩旧版本为fastq.gz文件
/mnt/clustre/users/sanger-dev/app/bioinfo/datasplit_v3/spring2/spring2-linux-x86_64.AppImage \
-d \
-i /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/output/6a66e4342302c5a8669157e8--L1HIJ1400454--Ph2nd50_3.R1.raw.spring \
-o /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/output/6a66e4342302c5a8669157e8--L1HIJ1400454--Ph2nd50_3.R1.raw.fastq.gz
```

## 压缩

```bash
# 压缩gz
/mnt/clustre/users/sanger-dev/app/bioinfo/datasplit_v3/spring2/spring2-linux-x86_64.AppImage \
-c \
--note "sample=test author=yuan" \
-R1 /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/restore.fastq.gz \
-o /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/restore.fastq.spring2
# 解压缩gz
/mnt/clustre/users/sanger-dev/app/bioinfo/datasplit_v3/spring2/spring2-linux-x86_64.AppImage \
-d \
-i /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/restore.fastq.spring2 \
-o /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/restore.spring2.fastq.fastq.gz
# note加元数据
/mnt/clustre/users/sanger-dev/app/bioinfo/datasplit_v3/spring2/spring2-linux-x86_64.AppImage \
-c \
-R1 /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/restore.fastq.gz \
-o /mnt/clustre/users/sanger-dev/wpm2/workspace/20260727/DataProcessSpring_DataProcessSpringTest_6a66e2462302c5a8669157e2_20260727_124710/SpringParallel/Spring/Spring/restore.fastq.spring2
```