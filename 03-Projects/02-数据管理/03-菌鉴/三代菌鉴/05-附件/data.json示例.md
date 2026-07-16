# data.json 示例

> **说明**: 三代菌鉴工作流运行时生成的 data.json 结构示例  
> **最后更新**: 2026-04-07

---

## 完整示例

```json
{
    "id": "ThirdStrainIdentification_69cdbf908e08c37e72effc57_20260402_090835",
    "type": "workflow",
    "WPM": true,
    "wfm_port": "7321",
    "add_time": "2026/04/02 09:08:35",
    "start_time": "",
    "stage_id": "0",
    "interaction": true,
    "name": "datasplit_v3.strain_identification.strain_identification_third",
    "client": "client01",
    "CLUSTER": "clustre",
    "IMPORT_REPORT_DATA": true,
    "member_id": "",
    "member_type": 0,
    "cmd_id": 0,
    "output": "s3nb4://datasplit/2026/ThirdStrainIdentification/ningbo/20260402/ThirdStrainIdentification_69cdbf908e08c37e72effc57_20260402_090835",
    "UPDATE_STATUS_API": "datasplit.update_status",
    "update_info": "{\"69cdbf908e08c37e72effc57\": \"strain_identification_third\"}",
    "endpoint": "",
    "to_file": [
        "datasplit_v3.strain_identification.strain_identification_third.export_strain_identification_third_info(sample_info_json)"
    ],
    "options": {
        "main_id": "69cdbf908e08c37e72effc57",
        "sample_info_json": "{\"main_id\": \"69cdbf908e08c37e72effc57\", \"project_type\": \"datasplit\"}",
        "testmod": "no",
        "update_info": "{\"69cdbf908e08c37e72effc57\": \"strain_identification_third\"}"
    },
    "work_dir": "/mnt/clustre/users/sanger-dev/wpm2/workspace/20260402/StrainIdentificationThird_ThirdStrainIdentification_69cdbf908e08c37e72effc57_20260402_090835",
    "main_id": 16859268,
    "IMPORT_REPORT_AFTER_END": false,
    "DBVersion": 0,
    "ntm_port": "7322",
    "wf_type": "report",
    "creator": "webSubmit",
    "user": "sanger-dev",
    "userid": "",
    "py2dir": "/mnt/clustre/users/sanger-dev/app/program/Python",
    "py3dir": "/mnt/clustre/users/sanger-dev/app/program/Python-3.11.1",
    "biocluster": "/mnt/clustre/users/sanger-dev/wpm2/sanger_bioinfo",
    "workspace": "/mnt/clustre/users/sanger-dev/wpm2/workspace",
    "softwaredir": "/mnt/clustre/users/sanger-dev/app",
    "scriptdir": "/mnt/clustre/users/sanger-dev/wpm2/sanger_bioinfo/scripts",
    "packagedir": "/mnt/clustre/users/sanger-dev/wpm2/sanger_bioinfo/src/mbio/packages",
    "platform": "SLURM",
    "defautqueue": "chaifen",
    "lustrename": "/mnt/clustre",
    "wpmport": "7335",
    "rootworkspace": "/mnt/clustre/users/sanger-dev/wpm2/workspace",
    "testbiocluster": "/mnt/clustre/users/sanger-dev/wpm2/test_sanger_bioinfo"
}
```
