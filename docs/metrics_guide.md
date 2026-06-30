# Metrics Guide

The metrics scripts included in the `client-shell` output the following files:


* **First Look**

    * [Ground Truth CSV](#first-look-ground-truth-csv)

    * [Response JSON](#first-look-response-json)

    * [Metrics JSON](#first-look-metrics-json)


* **Continuous Alert**

    * [Ground Truth CSV](#ground-truth-csv)

    * [Response JSON](#response-json)

    * [Metrics JSON](#metrics-json)


* **Resource Allocation**

    * [Ground Truth CSV](#ground-truth-csv)

    * [Response JSON](#response-json)

    * [Metrics JSON](#metrics-json)

## First Look 

### First Look Ground Truth CSV

The ground truth CSV contains the relevant ground truth LSIs for each segment of patient data. It uses the following file name convention:
`<PHASE>_<EVENT>_first-look-<RUN>_gt_<DATE>.csv`.

The ground truth CSV has the following fields, indicated by a header row in the first line.

**Table 1: Ground Truth CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|studyid|	Case identifier.|
|source| Case dataset source: `umb` or `upitt`.
|segment_num| Segment index within the case.|
|bin_start_time| Start time in seconds for the prediction time bin, relative to start of case.|
|bin_end_time| Stop time in seconds for the prediction time bin, relative to start of case.|
|segment_start_time| Start time in seconds for the data segment, relative to start of case.|
|segment_end_time| Stop time in seconds for the data segment, relative to start of case.|
|lsi_group|Grouped LSI category.|
|lsi_decription|Specific LSI name.|
|lsi_time_from_start|LSI timestamp relative to case start.|
|lsi_time_from_adm|LSI timestamp relative to hospital admission time.|
|lsi_in_hospital|Indicator for whether LSI was received in-hospital (1) or pre-hospital (0)|
|bin_num|Bin number at which LSI was received.|
|gt| Always 1.|


[(Return to top)](#metrics-guide)

### First Look Response JSON
The response JSON contains the client model’s prediction responses for each segment of patient data, along with timestamps for when prediction message was sent and received by the evaluation server. It uses the following file name convention:
`<PHASE>_<EVENT>_responses_<TEAM>-f<RUN>_<DATE>.json`.

The response JSON has the following format:
```
{
    "team_name": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "responses": [
        [
            {
                "message_sent_time": <float>,
                "message_received_time": <float>,
                "case_id": <str>,
                "studyid": <str>,
                "segment_num": <int>,
                "segment_id": <str>,
                "model_predictions": {
                    "airway_invasive": [
                        <float>, <float>, ...
                    ],
                    "blood_products": [
                        <float>, <float>, ...
                    ],
                    "chest_decompression": [
                        <float>, <float>, ...
                    ],
                    "surgical": [
                        <float>, <float>, ...
                    ],
                    "vaso_cardioactive_medications": [
                        <float>, <float>, ...
                    ],
                    "any_lsi": <float>
                },
                "cumulative_runtime_sec": <float>,
                "run_id": <str>,
                "end_of_case": <bool>,
                "raw_response": <null or dict>,
                "error": <null or dict>,
                "embeddings": {
                    ...
                }
            }
        ],
        ...
    ]
}
```

**Table 2: Response JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name	|Team name with appended task type shortcut.|
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|responses	|List of responses for all segments.|
|message_sent_time	|UNIX timestamp in seconds when the server’s prediction message was sent to the client for the segment. |
|message_received_time	|UNIX timestamp in seconds when the client’s prediction message was received by the server for the segment.|
|case_id|	Unique case id.|
|studyid|	Case identifier.|
|segment_num| Segment index within the case.|
|segment_id	|Unique `segment id.` |
|model_predictions	|Dictionary containing LSI confidence scores for each LSI over time.|
|cumulative_runtime_sec	|Cumulative runtime over the entire evaluation in seconds for the segment. |
|run_id	|GUID for the evaluation run for the segment.|
|end_of_case	|Boolean value indicating whether the segment is the last segment of the case.|
|raw_response	|If error present, original client response for the segment. |
|error	|Error message from the server, if present.|
|embeddings| Optional dictionary containing  embeddings at different levels of internal data analysis.|

There may be additional fields irrelevant to this task.

[(Return to top)](#metrics-guide)

### First Look Metrics JSON
The metrics JSON contains the client model performance metrics calculated from all ground truth and all responses. It uses the following file name convention:
`<PHASE>_<EVENT>_metrics_<TEAM>-f<RUN>_<DATE>.json`.

The metrics JSON has the following format:
```
{
    "team_name": <string>,
    "run_type": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "response_file_name": <string>,
    "any_ap": <float>,
    "any_ap_baseline": <float>,
    "existence_score": <float>,
    "temporal_score": <float>,
    "first_look_score": <float>
}
```

**Table 3: Metrics JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name|	Team name.|
|run_type| Task type shortcut: `f1` for First Look Run 1, `f2` for First Look Run 2, `f3` for First Look Run 3.| 
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|response_file_name|	Response JSON used to generate this Metrics JSON.|
|any_ap| Average Precision for "any LSI" prediction.|
|any_ap_baseline	| Baseline Average Precision for "any LSI" prediction calculated from prevalence of cases that receive at least one LSI.|
|existence_score	| Existence component of First Look Score.|
|temporal_score	| Temporal component of First Look Score. |
|first_look_score	| First Look Score. |

[(Return to top)](#metrics-guide)

## Continuous Alert

### Continuous Alert Ground Truth CSV

The ground truth CSV contains the relevant ground truth LSIs for each segment of patient data. It uses the following file name convention:
`<PHASE>_<EVENT>_continuous-alert_gt_<DATE>.csv`.

The ground truth CSV has the following fields, indicated by a header row in the first line.

**Table 1: Ground Truth CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|studyid|	Case identifier.|
|source| Case dataset source: `umb` or `upitt`.
|segment_num| Segment index within the case.|
|bin_start_time| Start time in seconds for the prediction time bin, relative to start of case.|
|bin_end_time| Stop time in seconds for the prediction time bin, relative to start of case.|
|segment_start_time| Start time in seconds for the data segment, relative to start of case.|
|segment_end_time| Stop time in seconds for the data segment, relative to start of case.|
|lsi_group|Grouped LSI category.|
|lsi_decription|Specific LSI name.|
|lsi_time_from_start|LSI timestamp relative to case start.|
|lsi_time_from_adm|LSI timestamp relative to hospital admission time.|
|lsi_in_hospital|Indicator for whether LSI was received in-hospital (1) or pre-hospital (0)|
|bin_num|Bin number at which LSI was received.|
|gt| Always 1.|

[(Return to top)](#metrics-guide)

### Continuous Alert Response JSON
The response JSON contains the client model’s prediction responses for each segment of patient data, along with timestamps for when prediction message was sent and received by the evaluation server. It uses the following file name convention:
`<PHASE>_<EVENT>_responses_<TEAM>-ca_<DATE>.json`.

The response JSON has the following format:
```json
{
    "team_name": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "responses": [
        [
            {
                "message_sent_time": <float>,
                "message_received_time": <float>,
                "case_id": <str>,
                "studyid": <str>,
                "segment_num": <int>,
                "segment_id": <str>,
                "model_predictions": {
                    "airway_invasive": <float>,
                    "blood_products": <float>,
                    "chest_decompression": <float>,
                    "surgical": <float>,
                    "vaso_cardioactive_medications": <float>
                },
                "cumulative_runtime_sec": <float>,
                "run_id": <str>,
                "end_of_case": <bool>,
                "raw_response": <None or dict>,
                "error": <None or dict>,
                "embeddings": {
                    ...
                }
            },
            ...
        ],
        ...
    ]
}
```


**Table 2: Response JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name	|Team name with appended task type shortcut.|
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|responses	|List of responses for all segments.|
|message_sent_time	|UNIX timestamp in seconds when the server’s prediction message was sent to the client for the segment. |
|message_received_time	|UNIX timestamp in seconds when the client’s prediction message was received by the server for the segment.|
|case_id|	Unique case id.|
|studyid|	Case identifier.|
|segment_num| Segment index within the case.|
|segment_id	|Unique `segment id.` |
|model_predictions	|Dictionary containing LSI confidence scores for each LSI.|
|cumulative_runtime_sec	|Cumulative runtime over the entire evaluation in seconds for the segment. |
|run_id	|GUID for the evaluation run for the segment.|
|end_of_case	|Boolean value indicating whether the segment is the last segment of the case.|
|raw_response	|If error present, original client response for the segment. |
|error	|Error message from the server, if present.|
|embeddings| Optional dictionary containing embeddings at different levels of internal data analysis.|


There may be additional fields irrelevant to this task.

[(Return to top)](#metrics-guide)

### Continuous Alert Metrics JSON
The metrics JSON contains the client model performance metrics calculated from all ground truth and all responses. It uses the following file name convention:
`<PHASE>_<EVENT>_metrics_<TEAM>-ca_<DATE>.json`.

The metrics JSON has the following format:
```
{
    "team_name": <string>,
    "run_type": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "response_file_name": <string>,
    "avg_precision": <float>,
    "continuous_alert_score": <float>
}
```

**Table 3: Metrics JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name|	Team name.|
|run_type| Task type shortcut: `ca`.| 
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|response_file_name|	Response JSON used to generate this Metrics JSON.|
|avg_precision| Simple Average Precision calculated over all predictions (not used for score).|
|continuous_alert_score | Continuous Alert Score. |

[(Return to top)](#metrics-guide)

## Resource Allocation

### Resource Allocation Ground Truth CSV

The ground truth CSV contains the relevant ground truth resources needed for each segment of patient data. It uses the following file name convention:
`<PHASE>_<EVENT>_resource-allocation_gt_<DATE>.csv`.

The ground truth CSV has the following fields, indicated by a header row in the first line.

case_id
scenario_id
resource_setting
evacuation
hospital_bed
blood
ventilator
surgery
studyid
patient_id
segment_num
segment_id
start_time_sec
stop_time_sec
time_since_adm_sec
hosp_adm
end_of_case
resources_needed
died_during_care
gt_lsi
bin_start_sec
bin_size_sec
num_bins

**Table 1: Ground Truth CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|studyid|	Case identifier.|
|source| Case dataset source: `umb` or `upitt`.
|segment_num| Segment index within the case.|
|bin_start_time| Start time in seconds for the prediction time bin, relative to start of case.|
|bin_end_time| Stop time in seconds for the prediction time bin, relative to start of case.|
|segment_start_time| Start time in seconds for the data segment, relative to start of case.|
|segment_end_time| Stop time in seconds for the data segment, relative to start of case.|
|lsi_group|Grouped LSI category.|
|lsi_decription|Specific LSI name.|
|lsi_time_from_start|LSI timestamp relative to case start.|
|lsi_time_from_adm|LSI timestamp relative to hospital admission time.|
|lsi_in_hospital|Indicator for whether LSI was received in-hospital (1) or pre-hospital (0)|
|bin_num|Bin number at which LSI was received.|
|gt| Always 1.|

[(Return to top)](#metrics-guide)

### Resource Allocation Response JSON
The response JSON contains the client model’s prediction responses for each segment of patient data, along with timestamps for when prediction message was sent and received by the evaluation server. It uses the following file name convention:
`<PHASE>_<EVENT>_responses_<TEAM>-ra_<DATE>.json`.

The response JSON has the following format:
```
{
    "team_name": "disr-ra",
    "event": "workshop_p2",
    "evaluation_date": "2026-06-20",
    "responses": [
        [
            {
                "message_sent_time": 1781918920.003388,
                "message_received_time": 1781918922.316723,
                "case_id": "1quec6ie",
                "studyid": null,
                "scenario_id": 8,
                "segment_num": 0,
                "segment_id": null,
                "model_predictions": null,
                "cumulative_runtime_sec": 992.2616789340973,
                "run_id": "ra2830f2-1078-479c-ad16-06d26be3270c",
                "end_of_case": true,
                "raw_response": null,
                "error": "",
                "embeddings": null,
                "evacuated_patients": [
                    "ulbcap9l",
                    "4xt8p526",
                    "1jlykkax",
                    "6oi5mtk3",
                    "syfi9wyv",
                    "odrh2xms",
                    "itjz6ejz",
                    "erdqykh2",
                    "fs6frl0p"
                ],
                "resources": [
                    {
                        "patient_id": "evf8d4jy",
                        "resources": [
                            "hospital_bed"
                        ]
                    },
                    ...
                ]
            },
            ...
        ]
    ]
}
```

**Table 2: Response JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name	|Team name with appended run type shortcut.|
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|responses	|List of responses for all segments.|
|message_sent_time	|UNIX timestamp in seconds when the server’s prediction message was sent to the client for the segment. |
|message_received_time	|UNIX timestamp in seconds when the client’s prediction message was received by the server for the segment.|
|case_id|	Unique case id.|
|studyid|	Case identifier.|
|segment_num| Segment index within the case.|
|segment_id	|Unique `segment id.` |
|model_predictions	|Dictionary containing LSI confidence scores for each LSI over time.|
|cumulative_runtime_sec	|Cumulative runtime over the entire evaluation in seconds for the segment. |
|run_id	|GUID for the evaluation run for the segment.|
|end_of_case	|Boolean value indicating whether the segment is the last segment of the case.|
|raw_response	|If error present, original client response for the segment. |
|error	|Error message from the server, if present.|
|embeddings| Optional dictionary containing  embeddings at different levels of internal data analysis.|

[(Return to top)](#metrics-guide)

### Resource Allocation Metrics JSON
The metrics JSON contains the client model performance metrics calculated from all ground truth and all responses. It uses the following file name convention:
`<PHASE>_<EVENT>_metrics_<TEAM>-ra_<DATE>.json`.

The metrics JSON has the following format:
```
{
    "team_name": "disr",
    "run_type": "ra",
    "event": "workshop_p2",
    "evaluation_date": "2026-06-20",
    "response_file_name": "/usr/src/app/metrics/phase3_workshop_p2_responses_disr-ra_20260620.json",
    "resource_allocation_score": 0.5849136451310364,
    "scenario_results": [
        {
            "case_id": "1quec6ie",
            "scenario_id": 8.0,
            "run_id": "ra2830f2-1078-479c-ad16-06d26be3270c",
            "E": 4,
            "S": 9,
            "saved_count": 13,
            "eligible_patient_count": 18,
            "T": 18,
            "lives_saved_score": 0.7222222222222222,
            "resource_usage": {
                "evacuation": 9,
                "hospital_bed": 53,
                "blood": 74,
                "ventilator": 17,
                "surgery": 10
            },
            "resources_remaining": {
                "evacuation": 0,
                "hospital_bed": 0,
                "blood": 0,
                "ventilator": 0,
                "surgery": 0
            },
            "cumulative_runtime_sec": 992.2616789340973,
            "evacuated_saved_patients": [
                "1jlykkax",
                "4xt8p526",
                "6oi5mtk3",
                "odrh2xms"
            ],
            "assigned_saved_patients": [
                "0zmxsdve",
                "5gu9o3oy",
                "7c2be8qy",
                "cltourrc",
                "dzoqxavt",
                "jvutvx9j",
                "orzyjn4h",
                "ss9dbva5",
                "zowjq7p6"
            ]
        },
        ...
    ]
}
```

**Table 3: Metrics JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name|	Team name.|
|run_type| Run type shortcut: `f1` for First Look Run 1, `f2` for First Look Run 2, `f3` for First Look Run 3.| 
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|response_file_name|	Response JSON used to generate this Metrics JSON.|
|any_ap| Average Precision for "any LSI" prediction.|
|any_ap_baseline	| Baseline Average Precision for "any LSI" prediction calculated from prevalence of cases that receive at least one LSI.|
|existence_score	| Existence component of First Look Score.|
|temporal_score	| Temporal component of First Look Score. |
|first_look_score	| First Look Score. |

[(Return to top)](#metrics-guide)