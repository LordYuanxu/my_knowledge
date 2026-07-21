# S3数据迁移方案

> **适用场景**：不同S3集群间大规模数据迁移  
> **数据规模**：约 800TB  
> **核心要求**：保留对象元数据、简单验证数据完整性、支持断点重试 


## 一.方案设计

### 1.1 背景与目标

- **源路径1**: s3nb3/dataprocess/2026/DataProcessSpring 511T 1051648个对象
- **源路径2**: s3nb4/dataprocess/2026/DataProcessSpring 307T 469441个对象
- **目标路径**: s3nb2/dataprocess/2026/DataProcessSpring
- **mongo库更新**: data_process_spring_detail的path.raw_spring_path和path.raw75_spring_path(mirna产品)

**核心目标**：
- 数据完整性：对象数量、总大小、ETag/MD5 100% 一致
- 元数据保留：Content-Type、自定义元数据（x-amz-meta-）、对象标签
- 可中断恢复：网络抖动或进程崩溃后，已完成的文件无需重传
- 业务影响最小化：支持按业务线/时间前缀分批迁移，增量同步兜底

### 1.2调研相关软件

- rclone
```bash
# 配置源和目标
rclone config

# 运行
rclone copy src_s3:bucket-name/folder/path dst_s3:bucket-name/folder/path \
  --transfers 64 \ # 并发传输数,根据网络调优（可提高到 64-128）
  --checkers 128 \ # 并发检查数,大目录下加快文件对比速度
  --s3-copy-cutoff 5G \ # 大于 5G 走 multipart copy，减少大文件失败重传
  --s3-upload-cutoff 5G \ # 分片上传阈值
  --s3-chunk-size 128M \ # 默认 5M 太小，128M 减少分片数量和 API 调用
  --metadata \ # 保留元数据, 保留 Content-Type、x-amz-meta-* 等
  --checksum \ # MD5 校验, 确保数据一致性（依赖 ETag）
  --fast-list \ # MinIO 支持，减少 API 调用次数，加速目录遍历
  --max-backlog 200000 \ # 待处理队列, 防止 800TB 海量文件导致队列溢出阻塞
  -P # 实时进度, 显示传输速度、剩余时间、已完成数量
  --log-file # 输出日志,后台运行时可排查问题
```

**注意事项:**
1. 因为源头和目标是不同的endpoint,数据必须经过本地机器中转,传输速度受限于执行命令这台机器的网卡带宽
2. rclone 采用的是**流式传输**, 数据从源端读取后，直接在内存中缓冲（默认约 128KB-几MB），然后实时推送到目标端。整个过程中**不会生成完整的本地临时文件**。
##  二.测试

### 测试1
 源文件夹: s3nb4://dataprocess/2026/DataProcessSpring/ningbo/20260519/DataProcessSpring_6a0c19a1e7909c7c525af202_20260519_160944/(6.1G 32objects)
 目标文件夹: s3nb2://dataprocess/2026/DataProcessSpring/ningbo/20260519/DataProcessSpring_6a0c19a1e7909c7c525af202_20260519_160944/
```bash
rclone copy \
s3nb4:dataprocess/2026/DataProcessSpring/ningbo/20260519/DataProcessSpring_6a0c19a1e7909c7c525af202_20260519_160944/ \
s3nb2:dataprocess/2026/DataProcessSpring/ningbo/20260519/DataProcessSpring_6a0c19a1e7909c7c525af202_20260519_160944/ \
--transfers 64 \
--checkers 128 \
--s3-copy-cutoff 64M \
--s3-upload-cutoff 64M \
--s3-chunk-size 64M \
--metadata \
--checksum \
--fast-list \
--max-backlog 200000 \
-P \
--log-file rclone_copy.log \
--log-level INFO
```
![[rclone运行结果.png]]
![[转移前s3元数据.png]]
![[转移后s3元数据.png]]
元数据完整copy过去

## 测试2
宁波新节点都是**25Gbps**网卡
compute-5之前的都是10Gbps网卡

## 三.预估

1. 如果在宁波运行，转移速度可以达到1GiB/s, **800T大概需要耗时12天**

## 四.实际测试

希望能够往新的bucket中传输

```bash
# 新建bucket
mc mb s3nb2/archive

# MJ-M-20221124105
rclone copy \
s3nb3:dataprocess/2026/DataProcessSpring/ningbo/20260209/DataProcessSpring_69894db0add9b72cff7fb562_20260209_110001/ \
s3nb2:archive/2026/DataProcessSpring/ningbo/20260209/DataProcessSpring_69894db0add9b72cff7fb562_20260209_110001/ \
--transfers 64 \
--checkers 128 \
--s3-copy-cutoff 64M \
--s3-upload-cutoff 64M \
--s3-chunk-size 64M \
--metadata \
--checksum \
--fast-list \
--max-backlog 200000 \
-P \
--log-file rclone_copy.log \
--log-level INFO
```

遇到的问题
1. 原来项目的文件大小没有一个汇总的字段(后面可以加上)
2. 验证: 如果相同的分片处理，可以根据ETag是否一致进行检查
3. data_process_spring主表加几个字段表示转移状态
	- transfer_status: "end"
	- transfer_path: "s3nb2://archive/2026/DataProcessSpring/ningbo/20260209/DataProcessSpring_69894db0add9b72cff7fb562_20260209_110001/"
	- transfer_ts: "2026-07-20 16:01:34"