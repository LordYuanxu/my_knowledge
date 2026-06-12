[官网地址](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)

# 下载

## linux集群

```bash
# 下载（v0.12.1 是当前较新版本）
wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.12.1.zip
# 解压
unzip fastqc_v0.12.1.zip
# 给执行权限
chmod +x FastQC/fastqc
# 加入 PATH（临时）
export PATH=$PWD/FastQC:$PATH
# 验证
fastqc --version
```