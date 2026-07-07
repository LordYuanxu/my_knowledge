## 测试

```bash
# fastqc进行质控
/usr/bin/time -f "最大使用内存=%Mkb\nCPU使用率=%P\n程序耗时=%es" \
~/app/bioinfo/datasplit_v3/FastQC/current/fastqc \
-o /mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/datasplit_v3/data_split_new/library_split/软件测试/FastQC/CF16-20260615X25BX1A/a30b85418e5053e3933d6d3--L2MLE2809113 \
-t 4 \
--memory 1024 \
/mnt/dlustre/users/sanger/wpm2/workspace/20260617/LibrarySplit_CF16-20260615X25BX1A_20260617_124031/BclToFastqParallel/BclToFastq/BclConvert/bcl_result/Fastq/6a30b85418e5053e3933d6d3--L2MLE2809113/L2MLE2809113_S1_L001_R1_001.fastq.gz \
/mnt/dlustre/users/sanger/wpm2/workspace/20260617/LibrarySplit_CF16-20260615X25BX1A_20260617_124031/BclToFastqParallel/BclToFastq/BclConvert/bcl_result/Fastq/6a30b85418e5053e3933d6d3--L2MLE2809113/L2MLE2809113_S1_L001_R2_001.fastq.gz

# 运行multiqc进行结果整合
multiqc .

```

## 测试小结

1. 输出文件夹不会新建
2. 文件名称`.fastq.gz`变为`_fastqc.html`和`_fastqc.zip`
3. 重运行不需要删除原来的结果,会覆盖