# 文库拆分流程

```bash
# 先 Dry Run 测试
nextflow run /home/ubuntu/majorbio_task/nextflow/main.nf \
  --json_input /home/ubuntu/majorbio_task/nextflow/library_split.json \
  --output_dir /mnt/data/result/datasplit \
  --dry_run
```


---