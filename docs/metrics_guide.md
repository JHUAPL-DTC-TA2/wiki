# Metrics Guide

The metrics scripts included in the `client-shell` output the following files:

* [Ground Truth CSV](#ground-truth-csv)

* [Response JSON](#response-json)

* [Metrics JSON](#metrics-json)

* [Detailed Metrics CSV](#detailed-metrics-csv)

* [Threshold Metrics CSV](#threshold-metrics-csv)

## Ground Truth CSV

The ground truth CSV contains the relevant ground truth LSIs for each segment of patient data. It uses the following file name convention:
`<PHASE>_<EVENT>_gt_<DATE>.csv`.

The ground truth CSV has the following fields, indicated by a header row in the first line.

**Table 1: Ground Truth CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|studyid|	Case identifier.|
|source| Case dataset source: `umb` or `upitt`.
|segment_num| Segment index within the case.|
|segment_start_time| Start time in seconds for the segment, relative to start of case.|
|segment_end_time| Stop time in seconds for the segment, relative to start of case.|
|lsi_group|The LSI category.|
|max_lsi_time| Time of last LSI for given LSI group, relative to start of case.|
|gt| 1 if the LSI occurred during the segment's prediction window, 0 otherwise. |
|time_since_adm_sec| Time elapsed from hospital admission to `segment_stop_time` |

## Response JSON
The response JSON contains the client model’s prediction responses for each segment of patient data, along with timestamps for when prediction message was sent and received by the evaluation server. It uses the following file name convention:
`<PHASE>_<EVENT>_responses_<TEAM>-<RUNTYPE>_<DATE>.json`.

The response JSON has the following format:
```
{
    "team_name": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "responses": [
        {
            "message_sent_time": <double>,
            "message_received_time": <double>,
            "case_id": <string>,
            "studyid": <string>,
            "segment_num": <double>,
            "segment_id": <string>,
            "model_predictions": <dict>,
            "cumulative_runtime_sec": <double>,
            "run_id": <GUID>,
            "end_of_case": <bool>,
            "raw_response": [<string>, <string>, ...],
            "error": <string>,
            "embeddings": <dict>
        },
        ...
    ]
}
```
**Table 2: Response JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|team_name	|Team name.|
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|responses	|List of responses for all segments.|
|message_sent_time	|UNIX timestamp in seconds when the server’s prediction message was sent to the client for the segment. |
|message_received_time	|UNIX timestamp in seconds when the client’s prediction message was received by the server for the segment.|
|case_id|	Unique case id.|
|studyid|	Case identifier.|
|segment_num| Segment index within the case.|
|segment_id	|Unique `segment id.` See inventory.csv file for ordering.|
|model_predictions	|Dictionary containing LSI confidence scores for each LSI group received from the client.|
|cumulative_runtime_sec	|Cumulative runtime over the entire evaluation in seconds for the segment. |
|run_id	|GUID for the evaluation run for the segment.|
|end_of_case	|Boolean value indicating whether the segment is the last segment of the case.|
|raw_response	|If error present, original client response for the segment. |
|error	|Error message from the server, if present.|
|embeddings| Optional dictionary containing  embeddings at different levels of internal data analysis.|

## Metrics JSON
The metrics JSON contains the client model performance metrics calculated for each case: the Mean Squared Correct (MSC). It uses the following file name convention:
`<PHASE>_<EVENT>_metrics_<TEAM>-<RUNTYPE>_<DATE>.json`.

The metrics JSON has the following format:
```
{
    "team_name": <string>,
    "run_type": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "response_file_name": <string>,
    "event_normalized_msc": <double>,
    "run_normalized_msc": <double>,
    "avg_specificity_sensitivity_benchmark": {
        "benchmark": <dict>,
        "surpassed": <bool>
    },
    "segment_weights": <dict>,
    "sensitivity_specificity_by_LSI": [
        {
            "lsi_group": <string>,
            "threshold": <double>,
            "sensitivity": <double>,
            "specificity": <double>,
            "balanced_accuracy": <double>
        },
        ...
    ],
    "metrics_by_case": [
        {
            "studyid": <string>,
            "case_normalized_msc": <double>,
            "runtime": <double>,
            "source": <string>
            "errors": [
                {
                    "segment_id": <string>,
                    "error": <string>
                },
                ...
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
|run_type| Run type: `B` for Basic EHR, `E` for Expanded EHR, `V` for Vitals Only.| 
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|response_file_name|	Response JSON used to generate this Metrics JSON.|
|event_normalized_msc| Normalized MSC for entire evaluation.|
|run_normalized_msc| Normalized MSC for single run.|
|avg_specificity_sensitivity_benchmark| Benchmark criteria and passing result.|
|benchmark| Dictionary of LSI group and minimum avg sensitivity/specificity score.|
|surpassed| True if run passed all LSI benchmarks, False otherwise.|
|segment_weights| Dictionary of weights by source, LSI group, and ground truth value.|
|sensitivity_specificity_by_LSI| List of specificty and sensitivity with highest balanced accuracy per LSI group.|
|lsi_group| The LSI category.|
|threshold| Threshold for max balanced accuracy.|
|sensitivity| Sensitivity at threshold.|
|specificity| Specificity at threshold.|
|balanced_accuracy| Average of specificity and sensitivity.|
|metrics_by_case	|List of metrics calculated for each case.|
|studyid|	Case identifier.|
|case_normalized_msc| Normalized MSC calculated for case.|
|runtime	|Cumulative runtime in seconds at the end of the case.|
|errors	|Errors found by the server during the case.|
|segment_id	|Segment id where error occurred. |
|error	|Error message from the server.|

## Detailed Metrics CSV
The detailed metrics CSV contains the client model performance metrics calculated for each segment and LSI group. It uses the following file name convention:
`<PHASE>_<EVENT>_detailed-metrics_<TEAM>-<RUNTYPE>_<DATE>.csv`.

**Table 4: Detailed Metrics CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|studyid|	Case identifier.|
|source| Case dataset source: `umb` or `upitt`.
|segment_num| Segment index within the case.|
|segment_start_time| Start time in seconds for the segment, relative to start of case.|
|segment_end_time| Stop time in seconds for the segment, relative to start of case.|
|lsi_group|The LSI category.|
|max_lsi_time| Time of last LSI for given LSI group, relative to start of case.|
|gt| 1 if the LSI occurrs within the segment's prediction window, 0 otherwise. |
|time_since_adm_sec| Time elapsed from hospital admission to `segment_stop_time` |
|ignore_segment| 0 if segment prediction contributes to MSC, 1 if ignored due to minimum prediction horizon or minimum lead time. |	
|segment_weight| Unnormalized weight based on inverse frequency of segments with same source and LSI ground truth, 0 if this prediction is within minimum lead time or outside of prediction horizon. |
| case_normalized_segment_weight|`segment_weight` divided by sum of `segment_weight`s over case.|
| run_normalized_segment_weight	| `segment_weight` divided by sum of `segment_weight`s over run. |
| event_normalized_segment_weight | `run_normalized_segment_weight` multiplied by run weight. |
| model_prediction| LSI confidence score for segment. |
| cumulative_runtime_sec | Cumulative runtime in seconds at the end of the segment. |
| error	| Errors found by the server during the segment.|
|unweighted_msc| Raw Mean Squared Correct (MSC) value for segment and LSI group. |
|case_normalized_msc|  `unweighted_msc` *  `case_normalized_segment_weight`|
|run_normalized_msc| `unweighted_msc` *  `run_normalized_segment_weight` |
|event_normalized_msc| `unweighted_msc` *  `event_normalized_segment_weight` |

## Threshold Metrics CSV
The threshold metrics CSV contains the client model binary classification metrics calculated at different thresholds for each LSI group. It uses the following file name convention:
`<PHASE>_<EVENT>_threshold-metrics_<TEAM>-<RUNTYPE>_<DATE>.csv`.

**Table 4: Threshold Metrics CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|lsi_group|	The LSI category.|
| threshold         | Cut-off value to classify a prediction as positive or negative |
| sensitivity       | True positive rate; proportion of actual positives correctly identified |
| specificity       | True negative rate; proportion of actual negatives correctly identified |
| PPV               | Positive Predictive Value; proportion of predicted positives that are correct |
| NPV               | Negative Predictive Value; proportion of predicted negatives that are correct |
| TP                | True Positives; segments correctly predicted as positive       |
| TN                | True Negatives; segments correctly predicted as negative       |
| FP                | False Positives; segments incorrectly predicted as positive    |
| FN                | False Negatives; segments incorrectly predicted as negative    |
| balanced_accuracy | Average of sensitivity and specificity                      |
| F1                | Harmonic mean of precision (PPV) and sensitivity             |


