# bcl-convert

---

## 下载(现在需要权限且只提供了rpm包)

https://support.illumina.com/sequencing/sequencing_software/bcl-convert.html

---

## 安装

1. 需要先安装7z
	```bash
	# 下载最新版 7-Zip Linux 二进制包（x86_64 架构）
	wget https://7-zip.org/a/7z2501-linux-x64.tar.xz
	
	# 创建安装目录（如 ~/.local/bin/7z）
	mkdir -p ~/.local/bin/7z
	
	# 解压到目标目录
	tar -xf 7z2501-linux-x64.tar.xz -C ~/.local/bin/7z
	
	# 添加到 PATH（永久生效，需重启终端）
	echo 'export PATH="$HOME/.local/bin/7z:$PATH"' >> ~/.bashrc
	
	```
1. 使用7z解压缩(提取为cpio文件)
	```bash
	# 解压到当前目录
	7zz x 你的文件.rpm
	
	# 解压到指定目录（注意 -o 后无空格）
	7zz x 你的文件.rpm -o/tmp/rpm_extract
	```
2. cpio提取
	```bash
	# 1. 创建一个解压后的存放目录（比如叫 bcl-convert-4.3.13），名字可自定义
	mkdir -p bcl-convert-4.3.13
	
	# 2. 核心解压命令（一键解压，无任何报错，完美适配你的文件）
	cpio -idmv -D bcl-convert-4.3.13 < bcl-convert-4.3.13-2.el8.x86_64.cpio
	```

---
## 打包

```
Bootstrap: docker
From: centos:7

%files
    # 替换yum镜像
    ./CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
    # 将本地的 RPM 文件复制到容器中的 /root 目录
    ./bcl-convert-4.3.13-2.el7.x86_64.rpm /root/bcl-convert-4.3.13-2.el7.x86_64.rpm

%post
    # 安装必要的工具
    yum clean all
    yum makecache
    yum update -y
    yum install -y rpm gdb

    # 安装 bcl-convert RPM 文件
    rpm -i /root/bcl-convert-4.3.13-2.el7.x86_64.rpm
    
%environment
    # 设置环境变量
    export PATH=/usr/local/bin:$PATH

%runscript
    # 默认运行脚本
    echo "This is a Singularity container with bcl-convert installed."
    echo "You can run bcl-convert commands directly."
```

```docker
Bootstrap: docker
From: ubuntu:22.04

%files
    # 将本地编译好的bcl-convert复制到容器中
    /home/ubuntu/app/bioinfo/datasplit_v3/bcl-convert/current/usr/bin/bcl-convert /usr/local/bin/bcl-convert
    # BBTools复制到容器中
    /home/ubuntu/app/bioinfo/datasplit_v3/BBMap/current 

%post
	# 替换 apt 源为阿里云（国内环境加速，可选）
	sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
	sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
	
	# 更新并安装必要工具
	apt-get update	

%environment
    # 设置环境变量
    export PATH=/usr/local/bin:$PATH

%runscript
    # 默认运行脚本
    echo "这是拆分软件集合的singularity容器"
```