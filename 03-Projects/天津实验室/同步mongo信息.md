# 同步一个月内的正式信息

## 要同步的表

- sg_split(拆分任务主表)
- sg_split_library(拆分任务文库信息表) 
- sg_split_specimen(拆分任务样本信息表)
- sg_split_raw_qc(样本raw数据信息统计表)
- sg_split_clean_qc(样本clean数据信息统计表)
- sg_split_library_qc(多样性文库信息统计表)
- sg_split_summary(拆分总体统计表)
- sg_split_lane_summary(拆分lane的统计信息)
- sg_split_lane_summary_detail(拆分文库的详细统计信息)
- sg_split_unknow_barcode(拆分unknow_barcode信息)
- sg_board(拆分版信息)
- sg_board_lane(拆分lane信息)
- sg_board_library(拆分文库信息)
- sg_board_specimen(拆分样本信息)

## 同步方法

```bash
# mongodump + mongorestore(mongodb-database-tools)
# 导出数据库
URI="mongodb://datasplit:m329ak8k39fm@10.2.9.11,10.2.9.12,10.2.9.13/datasplit?authMechanism=SCRAM-SHA-1"
OUTDIR=~/rsync_data/mongo_data
MONGODUMP=~/app/bioinfo/datasplit_v3/mongodb-database-tools/current/bin/mongodump
# 要导出的 collection 列表
COLLECTIONS="sg_split sg_split_library sg_split_specimen sg_split_raw_qc sg_split_clean_qc sg_split_library_qc sg_split_summary sg_split_lane_summary sg_split_lane_summary_detail sg_split_unknow_barcode sg_board sg_board_lane sg_board_library sg_board_specimen"

for coll in $COLLECTIONS; do
    echo "正在导出: $coll"
    $MONGODUMP --uri "$URI" --collection "$coll" --out "$OUTDIR"
done
echo "全部导出完成"

# 导入数据库
URI="mongodb://datasplit:m329ak8k39fm@10.8.0.84,10.8.0.177,10.8.0.62/datasplit?authMechanism=SCRAM-SHA-1"
INDIR=~/rsync_data/mongo_data/datasplit
MONGORESTORE=~/app/bioinfo/datasplit_v3/mongodb-database-tools/current/bin/mongorestore

for coll in $COLLECTIONS; do
    echo "正在导入: $coll"
    $MONGORESTORE \
    --uri "$URI" \
    --collection "$coll" \
    --numInsertionWorkersPerCollection 8 \
    "$INDIR/${coll}.bson"
done
```

