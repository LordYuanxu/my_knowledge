# 接口文档

> **说明**: 三代菌鉴接口文档  
> **最后更新**: 2026-04-07

---

## 接口概览

| 接口名称 | 请求方式 | 接口地址                                     | 说明             |
| -------- | -------- | -------------------------------------------- | ---------------- |
| 任务投递 | POST     | `s/datasplit_v3/strain_identification_third` | 启动三代菌鉴流程 |

---

## 接口投递参数

### 基本信息

| 项目             | 内容                                              |
| ---------------- | ------------------------------------------------- |
| **接口地址**     | `POST s/datasplit_v3/strain_identification_third` |
| **Content-Type** | `application/json`                                |
| **认证方式**     | Client认证                                        |

### 请求参数

| 参数              | 类型   | 必填 | 默认值    | 说明                                              |
| ----------------- | ------ | ---- | --------- | ------------------------------------------------- |
| **`main_id`**     | string | 是   | -         | MongoDB中 strain_identification_third 表的主键id  |
| `release_address` | string | 否   | ningbo    | 脚本运行地址                                      |
| `cluster`         | string | 否   | None      | 脚本运行集群                                      |
| `project_type`    | string | 否   | datasplit | 指定s3的bucket                                    |
| `test`            | string | 否   | no        | 是否是测试环境(是否使用test_sanger_bioinfo的代码) |


### 请求示例

```json
{
    "main_id": "69cdbf908e08c37e72effc57"
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
post s/datasplit_v3/strain_identification_third \
-c client01 \
-b http://wpm2sh.sanger.com \
-n "main_id" \
-d "69cdbf908e08c37e72effc57"\
)"
```

### 参数说明

| 参数                                         | 说明                  |
| -------------------------------------------- | --------------------- |
| `post`                                       | HTTP方法              |
| `s/datasplit_v3/strain_identification_third` | 接口路径              |
| `-c client01`                                | 客户端标识            |
| `-b http://wpm2sh.sanger.com`                | 服务器地址            |
| `-n "main_id"`                               | 参数名                |
| `-d "69cdbf908e08c37e72effc57"`              | 参数值（MongoDB主键） |
