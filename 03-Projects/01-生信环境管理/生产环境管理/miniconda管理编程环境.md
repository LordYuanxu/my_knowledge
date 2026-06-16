# 注意事项
## 1.如何登录宁波旧集群
### winscp
1. windows安装GOW(Gnu On Windows)，其中包含PuTTYgen工具
2. winscp中生成ppk格式的密钥对
3. 保存之后将公钥写入远程服务器

### ssh
~/sg-users/xuyuan/isanger.id_rsa
~/sg-users/xuyuan/sanger.id_rsa
运行
```
ssh -i ~/sg-users/xuyuan/isanger.id_rsa isanger@10.2.3.172
ssh -i ~/sg-users/xuyuan/isanger.id_rsa sanger@10.2.3.172
```

# miniconda管理一些环境
**software_dir:** /mnt/lustre/users/sanger/app
**miniconda3安装路径:** /mnt/lustre/users/sanger/app/bioinfo/datasplit_v3/miniconda3
## 安装miniconda
```bash
# 下载安装包
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
## 如果是centos7(cat /etc/os-release获取),那么glibc是固定死的，下载兼容库
### 版本 1：Miniconda3-py39_4.12.0 （推荐，2022 年稳定版，生信环境最适配，无任何坑）
wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-x86_64.sh
### 版本 2：Miniconda3-py38_4.11.0 （备选，同样兼容，稳定无坑）
wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.11.0-Linux-x86_64.sh

# 执行安装脚本
## 协议
## 更换默认的安装路径
## 初始化Conda
bash Miniconda3-latest-Linux-x86_64.sh
conda config --set auto_activate_base false

# 配置国内镜像源(可以用chsrc先测速)
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/
conda config --set show_channel_urls yes
## 请向 /mnt/clustre/users/sanger-dev/.condarc 中手动添加:
cat > ~/.condarc << 'EOF'
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/r
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.ustc.edu.cn/anaconda/cloud
  pytorch: https://mirrors.ustc.edu.cn/anaconda/cloud
EOF

# 第一步：清理conda的旧缓存（关键！避免缓存残留导致配置不生效）
conda clean -i -y
# 第二步：刷新conda的频道索引（让conda读取新的channels配置）
conda update -n base conda -y --all

# 安装mamba插件
conda install -y -n base mamba -c conda-forge
```
## python环境
```bash
# 查看所有可安装的 Python 版本（含不同构建版本）
conda search python
## 简化输出（只显示版本号，过滤冗余信息）
conda search python | grep -E "^python "  # Linux/Mac
conda search python | findstr "^python "   # Windows
## 查看某个版本的详细信息
conda search python=3.9.19 -i

# 查看已安装的python环境(本地库)
## 查看所有 conda 环境及对应的 Python 版本
conda env list  # 先看环境列表
conda run -n py39_env python --version  # 查看 py39_env 环境的 Python 版本
## 批量查看所有环境的 Python 版本（Linux/Mac）
for env in $(conda env list | grep -v '^#' | awk '{print $1}'); do
  echo "环境 $env 的 Python 版本："
  conda run -n $env python --version 2>/dev/null || echo "无 Python"
done

# 安装，最新稳定版本3.14.2
conda create -n py_env python=3.14.2 -y
```

## R环境
```bash
# 创建R环境
conda create -n r r-base=4.4.0 -y

# 激活和使用
## 验证
# 1. 查看R版本（最核心验证）
R --version
# 2. 查看R安装路径是否正确
R -e 'Sys.getenv("R_HOME")'
# 3. 查看conda已创建的所有环境（确认环境存在）
conda info --envs

# 安装R包
## 通过 conda 安装（推荐，二进制包，速度快、兼容性好）
conda install -y -n r r-ggplot2 r-vegan r-phyloseq r-dada2 r-taxize
## 在 R 交互界面安装，如Bioconductor相关的包
if (!require("BiocManager", quietly = TRUE)) install.packages("BiocManager", repos="https://mirrors.tuna.tsinghua.edu.cn/CRAN/")
BiocManager::install(c("dada2", "phyloseq", "Biostrings"))

```
## Perl环境
```bash
# 安装perl环境
## 搜索 conda-forge 频道下的 perl 包，列出所有可用版本(查看你的系统能安装的 Perl 版本，避免选不存在的版本)
conda search perl -c conda-forge --full-name
## 1. 创建名为perl_env的环境，指定Perl版本（推荐5.34）
conda create -n perl_env perl=5.34 -c conda-forge -y
### 安装 5.32.1（100% 兼容 glibc 2.17） 
conda create -n perl_env perl=5.32.1 -c conda-forge -y
## 2. 激活这个Perl环境（关键！激活后所有perl命令都用这个环境的）
conda activate perl_env
## 3. 在 conda 的 Perl 环境中安装 JSON 模块，先安装cpanm（如果没装）
conda install -c conda-forge perl-app-cpanminus -y
cpanm JSON

# 寻找模块信息
## 验证JSON模块有没有加载
perl -e 'use JSON; print "JSON module installed successfully!\n"'
## 如果模块已安装且能加载，直接输出路径
perl -e 'use JSON; print $INC{"JSON.pm"}, "\n";'
## 验证JSON模块安装在哪里
perl -e 'print "Search paths:\n"; print join("\n", @INC), "\n";'
## 配置PERL5LIB环境变量（让Perl优先搜索这个目录，一般是perl编译时指定）
export PERL5LIB=/usr/local/share/perl/5.34.0:$PERL5LIB
echo 'export PERL5LIB="$HOME/perl5/lib/perl5:$PERL5LIB"' >> ~/.bashrc
echo 'export PERL_MM_OPT="INSTALL_BASE=$HOME/perl5"' >> ~/.bashrc  # 告诉CPAN安装到用户目录
echo 'export PERL_MB_OPT="--install_base $HOME/perl5"' >> ~/.bashrc
```