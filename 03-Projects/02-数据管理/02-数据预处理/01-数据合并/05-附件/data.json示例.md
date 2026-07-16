# data.json 示例

> **说明**: 数据合并工作流运行时生成的 data.json 结构示例  
> **最后更新**: 2026-04-08

---

## 完整示例

```json
{
    "id": "DataProcessMerge_SP2026040800278_694371_20260408_110531",
    "type": "workflow",
    "WPM": true,
    "wfm_port": "7321",
    "add_time": "2026/04/08 11:05:31",
    "start_time": "",
    "stage_id": "0",
    "interaction": true,
    "name": "datasplit_v3.data_process_new.data_process_merge.data_process_merge",
    "client": "client01",
    "CLUSTER": "isanger1",
    "IMPORT_REPORT_DATA": true,
    "member_id": "",
    "member_type": 0,
    "cmd_id": 0,
    "output": "s3nb3://dataprocess/2026/DataProcessMerge/ningbo/20260408/DataProcessMerge_SP2026040800278_694371_20260408_110531/",
    "UPDATE_STATUS_API": "datasplit.update_status",
    "update_info": "{\"69d5c5fb97b24a264c0e5108\": \"data_process_merge\"}",
    "endpoint": "",
    "to_file": [
        "datasplit_v3.data_process_new.data_process_merge.data_process_merge.export_data_process_merge_sequence_list(sequence_dir)",
        "datasplit_v3.data_process_new.data_process_merge.data_process_merge.export_data_process_merge_info(data_process_merge_info)"
    ],
    "options": {
        "data_process_merge_info": "{\"main_id\": \"69d5c5fb97b24a264c0e5108\", \"project_type\": \"datasplit\"}",
        "is_import": 1,
        "is_upload": 1,
        "main_id": "69d5c5fb97b24a264c0e5108",
        "release_address": "ningbo",
        "sequence_dir": "{\"main_id\": \"69d5c5fb97b24a264c0e5108\", \"project_type\": \"datasplit\"}",
        "testmod": "no",
        "update_info": "{\"69d5c5fb97b24a264c0e5108\": \"data_process_merge\"}"
    },
    "work_dir": "/mnt/ilustre/isanger1_workspaceDatasplitV3/20260408/DataProcessMerge_DataProcessMerge_SP2026040800278_694371_20260408_110531",
    "main_id": 1165056844852363285,
    "IMPORT_REPORT_AFTER_END": false,
    "DBVersion": 0,
    "ntm_port": "7322",
    "wf_type": "report",
    "creator": "webSubmit",
    "user": "isanger1",
    "userid": "",
    "py2dir": "/mnt/ilustre/users/isanger1/app/program/Python",
    "py3dir": "/mnt/ilustre/users/isanger1/app/program/Python35",
    "pytmp": "/var/pytmp/isanger1/python2.7",
    "biocluster": "/mnt/ilustre/users/isanger1/wpm2/sanger_bioinfo",
    "pytmpbiocluster": "/var/pytmp/isanger1/sanger_bioinfo",
    "workspace": "/mnt/ilustre/isanger1_workspaceDatasplitV3",
    "softwaredir": "/mnt/ilustre/users/isanger1/app",
    "scriptdir": "/mnt/ilustre/users/isanger1/wpm2/sanger_bioinfo/scripts",
    "packagedir": "/mnt/ilustre/users/isanger1/wpm2/sanger_bioinfo/src/mbio/packages",
    "platform": "SLURM",
    "defautqueue": "ISANGER",
    "lustrename": "/mnt/ilustre",
    "wpmport": "7335",
    "workspacebyclass": true,
    "rootworkspace": "/mnt/ilustre/isanger1_workspaceDatasplitV3",
    "testbiocluster": "/mnt/ilustre/users/isanger1/wpm2/test_sanger_bioinfo",
    "pytmptestbiocluster": "/var/pytmp/isanger1/test_sanger_bioinfo",
    "computer_cluster": 2
}
```
