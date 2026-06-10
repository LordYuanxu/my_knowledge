# koroFileHeader

---

## 修改配置文件信息

```json
{
  // 本项目专用的文件头部注释
  "fileheader.customMade": {
    "Author": "张三",
    "Date": "Do not edit",
    "LastEditors": "Do not edit",
    "LastEditTime": "Do not edit",
    "FilePath": "Do not edit",
    "Description": "",
    "Project": "MyProject"
  },
  
  // 函数注释模板
  "fileheader.cursorMode": {
    "description": "",
    "param": "",
    "return": ""
  },
  
  // 本项目特有的自动添加规则
  "fileheader.configObj": {
    "autoAdd": true,
    "autoAlready": true,
    "createFileTime": true,
    "dateFormat": "YYYY-MM-DD HH:mm:ss",
    // 只在这些文件夹内自动添加注释（相对项目根目录）
    "folderWhitelist": [
      "/src",
      "/lib",
      "/packages"
    ],
    // 禁止在这些文件夹内添加（优先级高于白名单）
    "folderBlacklist": [
      "/node_modules",
      "/dist",
      "/.vscode",
      "/test/fixtures"
    ]
  }
}
```

