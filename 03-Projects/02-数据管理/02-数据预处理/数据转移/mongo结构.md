# 数据库设计

---

## data_process_transfer(主表)

| 字段名          | 数据类型       | 示范                                   | 描述                   | 来源       | 备注                                              |
| ------------ | ---------- | ------------------------------------ | -------------------- | -------- | ----------------------------------------------- |
| `_id`        | ObjectId对象 | ObjectId("6656970d22d978455f5900b2") | MongoDB自动生成的唯一ID     | mongo库生成 |                                                 |
| `unique_id`  | String     | ZX-M-230625042                       | 系统插入的id              | 系统插入     |                                                 |
| `status`     | String     | no                                   | 状态,默认为no(未开始)        | 运行更新     | 主要分成下面几种no(未开始),start(进行中),failed(失败),end(成功结束) |
| `desc`       | String     | Job has been finished                | 一些描述信息,默认为空          | 运行更新     | 如果失败，会更新失败原因                                    |
| `created_ts` | String     | 2024-05-29 10:46:39                  | 文档创建时间               | 系统插入     |                                                 |
| `start_ts`   | String     | 2024-05-29 10:46:39                  | 工作流开始时间              | 运行更新     | 格式YYY-MM-DD hh-mm-ss                            |
| `task_id`    | String     | -                                    | 工作流workflow_id(便于定位) | 运行更新     |                                                 |
| `end_time`   | String     | 2024-05-29 10:48:18                  | 工作流结束时间              | 运行更新     |                                                 |

---

## data_process_transfer_detail(样本表)

| 字段名             | 数据类型       | 示范                                   | 描述               | 来源       | 备注       |
| :-------------- | :--------- | :----------------------------------- | :--------------- | :------- | :------- |
| `_id`           | ObjectId对象 | ObjectId("6523bd42e3016430060548e2") | MongoDB自动生成的唯一ID | mongo库生成 |          |
| `main_id`       | ObjectId对象 | ObjectId("6656970d22d978455f5900b2") | 主表id             | 系统插入     | 为了和主表对应上 |
| `sample_name`   | String     | SCLC0085                             | 样本名              | 系统插入     |          |
| `data_center`   | String     | ningbo/tianjin                       | 长存地              | 系统插入     |          |
| **`path_info`** | Array      |                                      | 存放文件信息           | 运行插入     | 一个文件一个对象 |

### path_info列表中每个对象结构

| 字段名               | 数据类型   | 示范                                                                              | 描述          | 备注    |
| ----------------- | ------ | ------------------------------------------------------------------------------- | ----------- | ----- |
| **`path`**        | String | s3nb4://datasplit/2026/scseqpng/MJ20250303009/DXB260305001/QC/SCLC0085_Control1 | s3长存的路径     |       |
| `work_path`       | String | /mnt/upload/picture/QC/SCLC0085_Control1                                        | 本地路径        | 可能不存在 |
| `md5`             | String | 214b1b571773873d94c021e37e9d6825                                                | 文件的md5值     |       |
| `size`            | String | 860160                                                                          | 文件的byte大小   |       |
| **`path_cache`**  | String |                                                                                 | 宁波的缓存路径     |       |
| `is_delete_cache` | int    | 0/1                                                                             | 宁波的缓存路径是否删除 |       |
| type              | string |                                                                                 | 文件类型        |       |
