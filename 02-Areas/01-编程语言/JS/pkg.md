
# pkg

---

## Usage

```powershell
# 1. 设置自定义缓存目录（~ 在 PowerShell 中解析为 $HOME）
$CACHE = Join-Path $HOME ".pkg-cache/custom"

# 2. 创建目录（-Force 相当于 mkdir -p，目录已存在也不报错）
New-Item -ItemType Directory -Path $CACHE -Force | Out-Null

# 3. 设置环境变量（当前进程生效）
$env:PKG_CACHE_PATH = $CACHE
$env:PKG_IGNORE_TAG = "true"

# 4. 下载预编译二进制（文件名必须带 built- 前缀）
$binaryUrl = "https://github.com/yao-pkg/pkg-binaries/releases/download/node16/built-v16.16.0-linux-armv7"
$outFile = Join-Path $CACHE "built-v16.16.0-linux-armv7"

Invoke-WebRequest -Uri $binaryUrl -OutFile $outFile

# 5. 编译应用
npx pkg -t node16-linux-armv7 .
```