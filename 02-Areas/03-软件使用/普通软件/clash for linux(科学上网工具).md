# 命令行安装

## 安装

```bash
# 创建并进入程序目录
mkdir ~/.config/
mkdir ~/.config/mihomo/
cd ~/.config/mihomo/

# 下载clash(https://v2free.net/ssr-download/clash-linux.tar.gz)
wget --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
"https://v2free.net/ssr-download/clash-linux.tar.gz"
## 解压缩
tar xvf clash-linux.tar.gz
## 授权可执行权限
chmod +x clash-linux

# 下载clash配置文件
## 用wget下载clash配置文件（重复执行就是更新订阅更新节点），替换默认的配置文件。当然，你也可以用浏览器打开订阅链接，下载后拷贝或移动到~/.config/mihomo/目录替换覆盖config.yaml文件。
wget -U "Mozilla/6.0" -O ~/.config/mihomo/config.yaml "https://v1.v2ai.top/link/mZjzOIujWBlc0cG7m4ZLaU82bX0lP4waBc33?clash=2"
```

## 使用

```bash
# 命令行设置代理
export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"
# 取消代理
unset http_proxy
unset https_proxy
```

---

# web管理

## 安装

```bash
# 下载(https://v2free.net/ssr-download/clash-dashboard.tar.gz)
## 下载并移动 clash-dashboard.tar.gz 到 ~/.config/mihomo/ 目录
tar zxvf clash-dashboard.tar.gz

# 修改配置
## 进入clash配置文件目录
cd ~/.config/mihomo
## 编辑clash的配置文件
vim config.yaml 
# 在配置文件中修改或增加以下内容
external-controller: 127.0.0.1:9090 # 如果你不是从本机访问，需要从其它机器访问这个Clash Dashboard ,则改为：0.0.0.0:9090
external-ui: /home/你的用户名/.config/mihomo/clash-dashboard # clash-dashboard的路径,注意，你的用户名替换成实际linux用户名
secret: 'PaaRwW3B1Kj9' # PaaRwW3B1Kj9 是登录web管理界面的密码，请自行设置你自己的,不要照抄教程中的密码
# 重启clash
```

---

# 长运行

## 创建systemd服务文件

```bash
mkdir -p ~/.config/systemd/user/
cat > ~/.config/systemd/user/clash.service << 'EOF'
[Unit]
Description=Clash for Linux
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/home/%u/.config/mihomo/clash-linux -f /home/%u/.config/mihomo/config.yaml
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
EOF
```

## 启动并设为开机自启

```bash
# 重载配置
systemctl --user daemon-reload
# 启动
systemctl --user start clash
# 开机自启
systemctl --user enable clash
# 查看状态
systemctl --user status clash
systemctl --user stop clash      # 停止
systemctl --user restart clash   # 重启
systemctl --user disable clash   # 取消开机自启
```

## 验证

```bash
# 查看端口是否在监听（默认 7890）
ss -tlnp | grep 7890
# 测试代理
curl -x http://127.0.0.1:7890 https://www.google.com
# 查看日志
journalctl --user -u clash -f   # 实时看日志
```