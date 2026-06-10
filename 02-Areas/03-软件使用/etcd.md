# 优雅格式化输出
```
# 基础版（直接格式化所有 Value）
etcdctl get /wpm2dev/ftm/CF6-20260303X25BX4A_20260305_141148 --prefix | \
awk 'NR % 2 == 0' | jq .
# 进阶版（Key + 格式化 Value 成对显示）
etcdctl get /wpm2dev/ftm/CF6-20260303X25BX4A_20260305_141148 --prefix | \
while read -r key; do
  if [[ -n "$key" ]]; then
    read -r value
    echo -e "\033[32mKey: $key\033[0m"  # 绿色高亮Key
    echo "$value" | jq .
    echo "----------------------------------------"  # 分隔线
  fi
done
```