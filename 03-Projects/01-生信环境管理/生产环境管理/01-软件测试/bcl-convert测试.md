# bcl-convert测试(4.4.6)

**测试任务:** CF9-20260603PE300-P2-N1
**bcl路径:** /mnt/upload/nextseq/VH01218/260605_VH01218_400_AAJ3FTMM5
**测试路径:** /mnt/data/bcl_data/260605_VH01218_400_AAJ3FTMM5

**测试任务:** CF4-20260603PE300-P2-N1
**bcl路径**: /mnt/upload/nextseq/VH01218/260603_VH01218_399_AAJ3GNHM5
**测试路径:** /mnt/data/bcl_data/260603_VH01218_399_AAJ3GNHM5

```bash
# 拷贝测试文件
rsync -avh --progress --rsh="ssh -F ${XDG_CONFIG_HOME}/.ssh/config" clustre:/mnt/upload/nextseq/VH01218/260605_VH01218_400_AAJ3FTMM5 /mnt/data/bcl_data
rsync -avh --progress --rsh="ssh -F ${XDG_CONFIG_HOME}/.ssh/config" clustre:/mnt/upload/nextseq/VH01218/260603_VH01218_399_AAJ3GNHM5 /mnt/data/bcl_data

# bcl-convert
## 机械硬盘
/usr/bin/time -f "最大使用内存=%Mkb\nCPU使用率=%P\n程序耗时=%es" \
bcl-convert \
--bcl-input-directory /mnt/data/bcl_data/260605_VH01218_400_AAJ3FTMM5 \
--output-directory /mnt/data/result/CF9-20260603PE300-P2-N1 \
--sample-sheet /home/ubuntu/majorbio_task/datasplit_v3/data_split/library_split/CF9-20260603PE300-P2-N1/260603PE300-P2-N1.library_sheet.csv \
--force \
--bcl-num-conversion-threads 4 \
--bcl-num-compression-threads 16 \
--bcl-num-decompression-threads 8 \
--bcl-num-parallel-tiles 2 \
--fastq-gzip-compression-level 6 \
--strict-mode true \
--bcl-enable-tile-metrics true \
--bcl-only-matched-reads true \
--bcl-sampleproject-subdirectories true \
--sample-name-column-enabled true \
--run-info /home/ubuntu/majorbio_task/datasplit_v3/data_split/library_split/CF9-20260603PE300-P2-N1/260603PE300-P2-N1.modified.RunInfo.xml
## 第一次运行
最大使用内存=28179512kb
CPU使用率=1246%
程序耗时=161.49s
## 第二次运行
最大使用内存=28269004kb
CPU使用率=1252% 
程序耗时=160.75s
## 第三次运行
最大使用内存=28392616kb                                                                                                                                                                                     
CPU使用率=1190%                                                                                                                                                                                            
程序耗时=175.43s

## 固态硬盘
/usr/bin/time -f "最大使用内存=%Mkb\nCPU使用率=%P\n程序耗时=%es" \
bcl-convert \
--bcl-input-directory /mnt/data/bcl_data/260605_VH01218_400_AAJ3FTMM5 \
--output-directory /mnt/disk/result/CF9-20260603PE300-P2-N1 \
--sample-sheet /home/ubuntu/majorbio_task/datasplit_v3/data_split/library_split/CF9-20260603PE300-P2-N1/260603PE300-P2-N1.library_sheet.csv \
--force \
--bcl-num-conversion-threads 4 \
--bcl-num-compression-threads 16 \
--bcl-num-decompression-threads 8 \
--bcl-num-parallel-tiles 2 \
--fastq-gzip-compression-level 6 \
--strict-mode true \
--bcl-enable-tile-metrics true \
--bcl-only-matched-reads true \
--bcl-sampleproject-subdirectories true \
--sample-name-column-enabled true \
--run-info /home/ubuntu/majorbio_task/datasplit_v3/data_split/library_split/CF9-20260603PE300-P2-N1/260603PE300-P2-N1.modified.RunInfo.xml
## 第一次运行 
最大使用内存=28012148kb
CPU使用率=1145%
程序耗时=186.62s
## 第二次运行
最大使用内存=28156204kb
CPU使用率=1189%
程序耗时=176.38s
```

# 特殊情况测试
## 1. CreateFastqForIndexReads为1测试

修改sample_sheet.csv中的信息

文件名后缀为
>R1_001.fastq.gz
>R2_001.fastq.gz
>I1_001.fastq.gz
>I2_001.fastq.gz 
## 2.拆分不出数据

空fastq.gz文件会生成