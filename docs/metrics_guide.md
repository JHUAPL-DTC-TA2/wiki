# Metrics Guide

## Ground Truth CSV

The ground truth CSV contains the relevant ground truth LSIs for each segment of patient data. The CSV has the following fields, indicated by a header row in the first line.

**Table 1: Ground Truth CSV Fields**

| Field	| Definition |
| ----- | ---------- |
|studyid|	Case identifier.|
|segment_num| Segment index within the case.|
|segment_start_time| Start time in seconds for the segment, relative to start of case.|
|segment_stop_time| Stop time in seconds for the segment, relative to start of case.|
|lsi_group|The LSI category.|
|gt| 1 if the LSI occurred during the segment's prediction window, 0 otherwise. |

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
The metrics JSON contains the client model performance metrics calculated for each case: the Mean Squared Correct (MSC).

The metrics JSON has the following format:  
```
{
    "team_name": <string>,
    "event": <string>,
    "evaluation_date": <date>,
    "response_file_name": <string>,
    "overall_msc": <double>,
    "metrics": [
        {
            "studyid": <string>,
            "weighted_msc": <double>,
            "max_msc": <double>,
            "runtime": <double>,
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
|overall_msc| Normalized MSC for entire evaluation.|
|metrics	|List of metrics calculated for each case.|
|studyid|	Case identifier.|
|weighted_msc| MSC calculated for case.|
|max_msc| Maximum MSC score for case.|
|runtime	|Cumulative runtime in seconds at the end of the case.|
|errors	|Errors found by the server during the case.|
|segment_id	|Segment id where error occurred. |
|error	|Error message from the server.|
