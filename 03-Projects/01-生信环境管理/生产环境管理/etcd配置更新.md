# 配置更新

```bash
# 上海/mnt/clustre/users/sanger-dev/bc2/bin/configSet
## 拉取最新的配置
/mnt/clustre/users/sanger-dev/bc2/bin/configSet -s wpm2dev -t save -o wpm2dev.cfg.0722.yaml
## 保存一份，有问题能复原回去
cp wpm2dev.cfg.0722.yaml wpm2dev.cfg.0722.bk.yaml
## 更新
## 更新完后查看wfm日志
/mnt/clustre/users/sanger-dev/bc2/bin/configSet -s wpm2dev -t init -c wpm2dev.cfg.0722.yaml
```