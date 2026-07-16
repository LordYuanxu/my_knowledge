# API接口文档

> **说明**: 数据合并接口文档  
> **最后更新**: 2026-04-08

---

## 接口概览

| 接口名称 | 请求方式 | 接口地址                      | 说明                                   |
| -------- | -------- | ----------------------------- | -------------------------------------- |
| 任务投递 | POST     | `s/datasplit_v3/data_process` | 启动数据合并流程（所有数据预处理共用） |

---

## 任务投递接口

### 基本信息

| 项目             | 内容                                             |
| ---------------- | ------------------------------------------------ |
| **接口地址**     | `POST s/datasplit_v3/data_process`               |
| **Content-Type** | `application/json`                               |
| **认证方式**     | Client认证                                       |
| **说明**         | 所有数据预处理共用一套接口，通过参数区分具体流程 |

### 请求参数

| 参数              | 类型   | 必填              | 默认值    | 说明                                              |
| ----------------- | ------ | ----------------- | --------- | ------------------------------------------------- |
| `fx_id`           | Int    | 是(和fx_id二选一) | -         | 释放单ID                                          |
| `main_id`         | Int    | 是(和fx_id二选一) | -         | 任务主表id                                        |
| `operation`       | String | 是                | -         | 流程类型，数据合并传 `"merge"`                    |
| `release_address` | string | 否                | ningbo    | 脚本运行地址                                      |
| `cluster`         | string | 否                | None      | 脚本运行集群                                      |
| `project_type`    | string | 否                | datasplit | 指定s3的bucket                                    |
| `test`            | string | 否                | no        | 是否是测试环境(是否使用test_sanger_bioinfo的代码) |
| `is_upload`       | string | 否                | yes       | 是否要把数据上传s3(测试用)                        |
| `is_import`       | string | 否                | yes       | 是否要更新数据库(测试用)                          |
| `store_type`      | string | 否                | s3        | 存储类型                                          |
| `force`           | string | 否                | no        | 强制执行流程(status为end不会执行)                 |


### 请求示例

```json
{
    "fx_id": 680227,
    "operation": "merge"
}
```

### 响应示例

#### 成功响应

```json
{
    "info": "任务投递成功",
    "success": true
}
```

#### 失败响应

```json
{
    "info": "错误信息描述",
    "success": false
}
```

---

## 手动投递方式

### 使用 webapitest.py 脚本

```bash
echo -e "$(\
python2 ~/wpm2/sanger_bioinfo/bin/webapitest.py \
post s/datasplit_v3/data_process \
-c client01 \
-b http://wpm.i-sanger.com \
-n "fx_id;operation;release_address;force" \
-d "680227;merge;ningbo;yes"\
)"
```

### 参数说明

| 参数                                         | 说明             |
| -------------------------------------------- | ---------------- |
| `post`                                       | HTTP方法         |
| `-c client01`                                | 客户端标识       |
| `-b http://wpm2sh.sanger.com`                | 服务器地址       |
| `-n "fx_id;operation;release_address;force"` | 参数名：释放单ID |
| `-d "680227;merge;ningbo;yes"`               | 参数值           |


