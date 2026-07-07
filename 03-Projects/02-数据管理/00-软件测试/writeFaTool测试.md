
## 构建SIF镜像

```bash
cat > /mnt/clustre/users/sanger-dev/sg-users/xuyuan/writeFasta.def << 'EOF'
Bootstrap: docker
From:m.daocloud.io/docker.io/library/ubuntu:20.04

%files
    /mnt/clustre/users/sanger-dev/sg-users/xuyuan/software/writeFaTool_v55.13_Linux_SURFSeqQ/Linux_SURFSeqQ/writeFasta_v55.13 /opt/writeFasta/
    /mnt/clustre/users/sanger-dev/sg-users/xuyuan/software/writeFaTool_v55.13_Linux_SURFSeqQ/Linux_SURFSeqQ/lib/* /opt/writeFasta/lib/

%environment
    export LD_LIBRARY_PATH=/opt/writeFasta/lib:/usr/lib64:/lib64:$LD_LIBRARY_PATH

%runscript
    exec /opt/writeFasta/writeFasta_v55.13 "$@"
EOF

singularity build --fakeroot /mnt/clustre/users/sanger-dev/sg-users/xuyuan/writeFasta.sif /mnt/clustre/users/sanger-dev/sg-users/xuyuan/writeFasta.def
singularity exec /mnt/clustre/users/sanger-dev/sg-users/xuyuan/writeFasta.sif
```


## 使用软件

```bash
singularity \
exec \
/mnt/clustre/users/sanger-dev/sg-users/xuyuan/writeFasta.sif \
/opt/writeFasta/writeFasta_v55.13 \
/mnt/upload/R21016100250027/DNBSEQ-T7+/R21016100250027/workspace/ML150005466 \
/mnt/clustre/users/sanger-dev/sg-users/xuyuan/majorbio_task/datasplit_v3/data_split_new/library_split/软件测试/writeFaTool/ML150005466 \
316 \
150 \
8 \
150 \
8 \
309 \
302 \
60 \
"T7_Run_001" \
0
```