# Metrics Guide
(UNDER CONSTRUCTION FOR PHASE 2)

## Ground Truth JSON

The ground truth JSON contains the relevant ground truth LSIs for each segment of patient data. 

The ground truth JSON has the following format:  
```
{
    "event": <string>,
    "ground_truth": [
        {
            "case_id": <string>,
            "segment_id": <string>,
            "segment_start_time_sec": <double>,
            "segment_stop_time_sec": <double>,
            "start_of_case": <boolean>,
            "at_admission": <boolean>,
            "end_of_case": <boolean>,
            "gt_lsi_list": [
                {
                    "lsi_group": <string>,
                    "lsi_description": <string>,
                    "in_hospital": <int>,
                    "elapsed_from_start": <double>
                },
                ...
            ],
            "ehr_file_name": <string>,
            "vs_file_name": <string>,
            "gt_file_name": <string>
        },
        ...
    ]
}
```

**Table 1: Ground Truth JSON Fields**

| Field	| Definition |
| ----- | ---------- |
|event	|Event name.|
|ground_truth|List of ground truth for all segments.|
|case_id|	Unique case id.|
|segment_id	|Unique segment id. See `inventory.csv` file for ordering.|
|segment_start_time_sec| Start time in seconds for the segment, relative to start of case. |
|segment_stop_time_sec| Stop time in seconds for the segment, relative to start of case. |
|start_of_case| True if the segment is the first segment in the case, false otherwise. |
|at_admission| True if the segment includes the hospital admission time, false otherwise. |
|end_of_case| True if the segment is the last segment in the case, false otherwise. |
|gt_lsi_list| List of ground truth LSIs for segment.|
|lsi_group	|The category of LSI that was performed. |
|lsi_description	|The specific LSI that was performed. |
|in_hospital	|1 if the LSI occurred during hospital admission, 0 otherwise.|
|elapsed_from_start	| LSI time in seconds, relative to start of case.|
|ehr_file_name| Relative path to EHR data for the segment.
|vs_file_name|Relative path to VS data for the segment.
|gt_file_name|Relative path to GT data for the segment.

## Response JSON
The response JSON contains the client model’s prediction responses for each segment of patient data, along with timestamps for when prediction message was sent and received by the evaluation server. 

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
            "segment_id": <string>,
            "model_predictions": [<string>, <string>, ...],
            "cumulative_runtime_sec": <double>,
            "run_id": <GUID>,
            "end_of_case": <bool>,
            "raw_response": [<string>, <string>, ...],
            "error": <string>,
            "lsi_confidence_scores": <dict>,
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
|response	|List of responses for all segments.|
|message_sent_time	|UNIX timestamp in seconds when the server’s prediction message was sent to the client for the segment. |
|message_received_time	|UNIX timestamp in seconds when the client’s prediction message was received by the server for the segment.|
|case_id|	Unique case id.|
|segment_id	|Unique `segment id.` See inventory.csv file for ordering.|
|model_predictions	|List of LSIs predicted for the segment received from the client.|
|cumulative_runtime_sec	|Cumulative runtime over the entire evaluation in seconds for the segment. |
|run_id	|GUID for the evaluation run for the segment.|
|end_of_case	|Boolean value indicating whether the segment is the last segment of the case.|
|raw_response	|If error present, original client response for the segment. |
|error	|Error message from the server, if present.|
|lsi_confidence_scores| Optional dictionary containing confidence scores for each LSI group. |
|embeddings| Optional dictionary containing  embeddings at different levels of internal data analysis.|

## Metrics JSON
The metrics JSON contains the client model’s two performance metrics calculated for each case: the Jaccard Index (a.k.a., IOU) and Prediction Lead-Time (PLT).
$$\text{Jaccard Index}= {|\bigcup\limits_{t=0}^{T} [\hat{Y_t} \cap Y_t]| \over|\bigcup\limits_{t=0}^{T} [\hat{Y_t} \cup Y_t]|}$$
$$PLT=\sum\limits_{\text{Earliest correct prediction for each LSI type}} \text{LSI timestamp} - (\text{Evaluation timepoint})$$
For the workshop evaluation, the prediction lead-time does not include processing delay. 
The metrics JSON has the following format:  
```
{
    "team_name": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "response_file_name": <string>,
    "metrics": [
        {
            "case_id": <string>,
            "jaccard_index": <double>,
            "prediction_lead_time": <double>,
            "correct_predictions": [
                {
                    "lsi_group": <string>,
                    "lead_time": <double>
                },
                ...
            ],
            "incorrect_predictions": {
                "false_positives": [<string>, <string>, ...],
                "misses": [<string>, <string>, ...]
            },
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
|event	|Event name.|
|evaluation_date	|Date of evaluation run.|
|response_file_name|	Response JSON used to generate this Metrics JSON.|
|metrics	|List of metrics calculated for each case.|
|case_id|	Unique case id.|
|jaccard_index	|Jaccard index for the case. |
|prediction_lead_time	|Prediction lead-time for the case, excluding any processing delay.|
|correct_predictions	|List of correctly predicted LSI instances and their individual lead-times, including recurring LSI instances.  |
|lsi_group	|LSI group for correctly predicted LSI instance. |
|lead_time	|Time between the earliest prediction time and actual LSI timestamp for correctly predicted LSI instance.|
|incorrect_predictions	|Incorrect LSI predictions for entire case.|
|false_positivies	|List of predicted LSI groups that did not occur within any prediction window over the case. |
|misses|	List of LSI groups that occurred within at least one prediction window but were never predicted over the case.|
|errors	|Errors found by the server during the case.|
|segment_id	|Segment id where error occurred. |
|error	|Error message from the server.|
