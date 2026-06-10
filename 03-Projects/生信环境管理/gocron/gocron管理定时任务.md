[github](https://github.com/ouqiang/gocron)
# 部署
## 主节点通过docker部署调度器
```docker
name: gocron
services:
  mysql:
    image: mysql:5.7
    container_name: gocron_mysql
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: gocron
      MYSQL_USER: mjdm
      MYSQL_PASSWORD: 123456
    ports:
      - "33061:3306"
    volumes:
      - /mnt/clustre/users/sanger-dev/sg-users/xuyuan/.docker/data/gocron_mysql:/var/lib/mysql
    restart: always

  gocron:
    image: ouqg/gocron:latest
    container_name: gocron_scheduler
    ports:
      - "5920:5920"
    environment:
      - APP_ENV=prod
      - DB_HOST=mysql
      - DB_PORT=33061
      - DB_USER=mjdm
      - DB_PASSWORD=123456
      - DB_NAME=gocron
    depends_on:
      - mysql
    restart: always

# volumes:
#   mysql_data:
#     driver: local
#     driver_opts:
#       type: none
#       o: bind
#       device: /mnt/clustre/users/sanger-dev/sg-users/xuyuan/.docker/data/gocron_mysql/
```
docker-compose常用命令
```bash
# 1. 停止所有服务（不会删除容器、数据卷，仅停止运行）
docker compose down
# 2. 启动所有服务（后台运行，推荐添加 -d 参数）
docker compose up -d
# 3. 进入容器交互式终端
docker exec -it gocron_scheduler /bin/bash
# 直接输出所有环境变量（键值对格式：KEY=VALUE）
env
```

```bash
# 查看 IP 地址（本地 + 公网）
ip -br addr
hostname -I
hostnamectl
# 安装完后要优化mjdm访问数据库的权限
## 进入MySQL容器
docker exec -it gocron_mysql mysql -uroot -p123456
-- 查看现有 mjdm 用户的授权（确认来源）
SELECT user, host FROM mysql.user WHERE user = 'mjdm';
-- 授权 mjdm 从 Docker 内网（172.0.0.0/8）访问 gocron 数据库，仅赋予必要权限
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER ON gocron.* TO 'mjdm'@'172.0.0.0/8' IDENTIFIED BY '123456';
-- 刷新权限生效
FLUSH PRIVILEGES;
-- 验证授权结果
SHOW GRANTS FOR 'mjdm'@'172.0.0.0/8';
### 权限说明：仅赋予 gocron 运行所需的 DML+DDL 权限，拒绝 GRANT、SUPER 等高危权限；
### 子网适配：若你的 Docker 子网不是 172.0.0.0/8，可通过 docker inspect gocron_mysql | grep Subnet 查看实际子网，替换为 xxx.xxx.0.0/16。

# 服务名映射授权（兼容容器 IP 变化）(若担心 Docker 子网变动，可利用 Compose 服务名的 DNS 解析特性，授权 mjdm 从 gocron_scheduler 容器访问（通过容器名反向解析）：)
-- 先查 gocron 容器的主机名（默认是容器名 gocron_scheduler）
-- 授权 mjdm 仅从 gocron_scheduler 容器访问
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER ON gocron.* TO 'mjdm'@'gocron_scheduler' IDENTIFIED BY '123456';
FLUSH PRIVILEGES;
```
## 各个任务节点部署gocron-node
因为是长时间运行任务,使用systemd服务部署(放置在/etc/systemd/system​文件夹下)
```bash
# 拷贝gocron-node.service
cp /mnt/clustre/users/sanger-dev/sg-users/xuyuan/software/gocron/v1.5.3/gocron-node.service /etc/systemd/system
# 拷贝软件到/opt文件夹下
mkdir -p /opt/gocron
cp /mnt/clustre/users/sanger-dev/sg-users/xuyuan/software/gocron/v1.5.3/gocron-node /opt/gocron
cp /mnt/clustre/users/sanger-dev/sg-users/xuyuan/software/gocron/v1.5.3/gocron /opt/gocron
systemctl daemon-reload
systemctl enable gocron-node
systemctl start gocron-node
systemctl status gocron-node
# 查看systemctl服务日志
journalctl -u gocron-node
```

```conf
[Unit]
Description=gocron-node service
Requires=network.target network-online.target
After=clustre.service
After=munge.service

[Service]
Type=simple
ExecStart=/opt/gocron/gocron-node --allow-root
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

以特定用户身份运行
```conf
[Unit]
Description=gocron-node service
Requires=network.target network-online.target
After=clustre.service
After=munge.service

[Service]
Type=simple
ExecStart=/opt/gocron/gocron-node
User=sanger-dev
Restart=on-failure

[Install]
WantedBy=multi-user.target
```