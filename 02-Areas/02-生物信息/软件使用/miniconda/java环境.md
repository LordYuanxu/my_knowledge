# java
## 创建java环境

```bash
# 创建qiime2环境
$HOME/app/bioinfo/datasplit_v3/miniconda3/bin/conda create \
-n java_env \
-c conda-forge \
openjdk=17
# 激活环境，命令行前缀会显示环境名
conda activate java_env
# 验证安装成功
java -version
```

