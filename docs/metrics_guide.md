# Metrics Guide

The metrics scripts included in the `client-shell` output the following files:


* **First Look**

    * [Ground Truth CSV](#first-look-ground-truth-csv)

    * [Response JSON](#first-look-response-json)

    * [Metrics JSON](#first-look-metrics-json)


* **Continuous Alert**

    * [Ground Truth CSV](#continuous-alert-ground-truth-csv)

    * [Response JSON](#continuous-alert-response-json)

    * [Metrics JSON](#continuous-alert-metrics-json)


* **Resource Allocation**

    * [Ground Truth CSV](#resource-allocation-ground-truth-csv)

    * [Response JSON](#resource-allocation-response-json)

    * [Metrics JSON](#resource-allocation-metrics-json)

## First Look 

### First Look Ground Truth CSV

The ground truth CSV contains the relevant ground truth LSIs for each segment of patient data. It uses the following file name convention:
`<PHASE>_<EVENT>_first-look-<RUN>_gt_<DATE>.csv`.

The ground truth CSV has the following fields, indicated by a header row in the first line.

**Ground Truth CSV Fields**

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

**Response JSON Fields**

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

**Metrics JSON Fields**

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

**Ground Truth CSV Fields**

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


**Response JSON Fields**

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
|segment_id	| Unique `segment id.` |
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

**Metrics JSON Fields**

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

**Ground Truth CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|case_id| Unique scenario identifier.|
|scenario_id| Integer scenario number.|
|resource_setting| Level of resources available: `high`, `medium`, or `low`.|
|evacuation| Number of patient who can be evacuated within this scenario |
|hospital_bed| Number of hospital beds available within this scenario |
|blood| Number of blood units available within this scenario |
|ventilator| Number of patients who can be provided by ventilator resources within this scenario.| 
|surgery| Number of patients who can be provided with surgery resources within this scenario.|
|studyid| Patient identifier. |
|patient_id| Unique patient id. |
|segment_num| Segment index within the case.|
|segment_id|  Unique segment id. |
|start_time_sec| Start time in seconds for the data segment, relative to start of case.|
|end_time_sec| Stop time in seconds for the data segment, relative to start of case.|
|time_since_adm_sec|Time elapsed from end of data segment to hospital admission.|
|hosp_adm| Time elapsed from start of case to hospital admission.|
|end_of_case| Boolean indicator for data segment at end of case.|
|resources_needed| List of medical resources needed for patient to contribute to lives saved.|
|died_during_care| Boolean indicator whether patient died during hospital care.|
|gt_lsi| List of LSIs received. |
|bin_size_sec | (Ignore) |
|num_bins | (Ignore) |

[(Return to top)](#metrics-guide)

### Resource Allocation Response JSON
The response JSON contains the client model’s prediction responses for each segment of patient data, along with timestamps for when prediction message was sent and received by the evaluation server. It uses the following file name convention:
`<PHASE>_<EVENT>_responses_<TEAM>-ra_<DATE>.json`.

The response JSON has the following format:
```json
{
    "team_name": <str>,
    "event": <str>,
    "evaluation_date": <str>,
    "responses": [
        [
            {
                "message_sent_time": <float>,
                "message_received_time": <float>,
                "case_id": <str>,
                "scenario_id": <int>,
                "cumulative_runtime_sec": <float>,
                "run_id": <str>,
                "raw_response": <Null or dict>,
                "error": <Null or dict>,
                "evacuated_patients": [
                    <str>,
                    ...
                ],
                "resources": [
                    {
                        "patient_id": <str>,
                        "resources": [
                            <str>,
                            ...
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

**Response JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name	|Team name with appended run type shortcut (`ra`).|
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|responses	|List of responses for all segments.|
|message_sent_time	|UNIX timestamp in seconds when the server’s prediction message was sent to the client for the segment. |
|message_received_time	|UNIX timestamp in seconds when the client’s prediction message was received by the server for the segment.|
|case_id| Unique scenario id.|
|scenario_id | Scenario identifier. |
|cumulative_runtime_sec	|Cumulative runtime over the entire evaluation in seconds for the segment. |
|run_id	| GUID for the evaluation run.|
|raw_response	|If error present, original client response for the segment. |
|error	|Error message from the server, if present.|
|evacuated_patients | List of patient ids selected for evacuation. |
|resources | List of resource assignment objects with patient id and list of resources. |

There may be additional fields irrelevant to this task.

[(Return to top)](#metrics-guide)

### Resource Allocation Metrics JSON
The metrics JSON contains the client model performance metrics calculated from all ground truth and all responses. It uses the following file name convention:
`<PHASE>_<EVENT>_metrics_<TEAM>-ra_<DATE>.json`.

The metrics JSON has the following format:
```
{
    "team_name": <str>,
    "run_type": <str>,
    "event": <str>,
    "evaluation_date": <str>,
    "response_file_name": <str>,
    "resource_allocation_score": <float>,
    "scenario_results": [
        {
            "case_id": <str>,
            "scenario_id": <int>,
            "run_id": <str>,
            "E": <int>,
            "S": <int>,
            "saved_count": <int>,
            "eligible_patient_count": <int>,
            "T": <int>,
            "lives_saved_score": <float>,
            "resource_usage": {
                "evacuation": <int>,
                ...
            },
            "resources_remaining": {
                "evacuation": <int>,
                ..
            },
            "cumulative_runtime_sec": <float>,
            "evacuated_saved_patients": [
                <str>,
                ...
            ],
            "assigned_saved_patients": [
                <str>,
                ...
            ]
        },
        ...
    ]
}
```

**Metrics JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name|	Team name.|
|run_type| Run type shortcut `ra`| 
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|response_file_name|	Response JSON used to generate this Metrics JSON.|
|resource_allocation_score| Resource Allocation Score. |
|scenario_results | List of results by scenario within run. |
|case_id | Unique scenario id. |
|scenario_id | Integer scenario identifier. |
|run_id | Unique evaluation run identifier. |
|E | Number of lives saved due to assigned evacuation. |
|saved_count | Total number of lives saved due to evacuation and local resource assignments. |
|S | Equal to `saved_count`.|
|eligible_patient_count | Total lives that could be saved within scenario. |
|T | Equal to `eligible_patient_count`.|
|lives_saves_score | Proporation of lives saved within this scenario. |
|resource_usage | Count by resource type of resources assigned to patients. |
|resources_remaining | Count of unassigned resources available. |
|cumulative_runtime_sec | Total seconds elapsed during evaluation. |
|evacuated_saved_patients | List of patients (by `patient_id`) saved due to evacuation. |
|assigned_saved_patients | List of patients (by `patient_id`) saved due to local resource assignments. |


[(Return to top)](#metrics-guide)