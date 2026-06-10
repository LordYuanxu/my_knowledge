# cli
```bash
scoop bucket add scoop-yakitrak https://github.com/yakitrak/scoop-yakitrak.git
scoop install obsidian-cli

# 安装完成之后set
obsidian-cli set-default "[vault-name]"
```

# 插件
-  templater(模版引擎)
```
<% tp.date.now("YYYY-MM-DD") %> // 自动插入当前日期
<% tp.file.title %> // 自动取笔记标题
```
