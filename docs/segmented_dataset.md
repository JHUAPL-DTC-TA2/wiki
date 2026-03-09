
# Segmented Datasets

Files for segmented datasets (v3.0+) are nested by partition, by task, and by case with the following general structure:

```
phaseX_v3-0_segmented/
├── train/
    ├── continuous/
        ├── PX_0003/
        ├── PX_0015/
        ├── PX_0230/
        ...
        inventory.csv
    ├── first-look/
        ├── PX_0807/
        ├── PX_0823/
        ├── PX_0830/
        ...
        run1_inventory.csv
        run2_inventory.csv
        run3_inventory.csv
├── val/
    ├── continuous/
        ...
    ├── first-look/
        ...
└── test_release/
    ├── continuous/
        ...
    ├── first-look/
        ...
```

The following sections provide details about the data structure for First Look and Continuous Alert tasks.

## Task 1: First Look Segmented Data

### First Look Inventory Files

There are 3 inventory files included in the First Look data directory for Runs 1-3. Each row of the inventory file contains data for a single case and relative paths to the corresponding files to be included as prediction inputs (see below). For example, the run1_inventory.csv includes paths to the `casualty report` and not the `basic-ehr` data. See the Data Competition Rules for more information on the run types for First Look.

### First Look Case Data

Case data files for First Look are organized in subdirectories by case with *initial* data at the beginning of the case.

Each case directory includes files used across the runs 1-3 with the following data types:

- `<studyid>_*_vs_*.hdf5`: **vital-sign** waveforms and/or trend vitals available at first look.
- `<studyid>_*_basic-ehr_*.json`: **basic EHR** object available at first look.
- `<studyid>_*_lsi-ehr_*.json`: **LSI EHR** records available at first look.
- `<studyid>_*_gt_*.json`: **ground-truth** labels containing LSIs in future time bins.
- `<studyid>_*_casualty-report_*.json`: single **casualty report** record (static patient/case description).

<br>

> **Alignment across files:**
> - All First Look files correspond to the same case at the beginning of pre-hospital vitals. There are no segments.
> - In each run (1-3), a subset of the files will be provided at evaluation time according to the run-specific inventory file.


#### 1) Vital signs HDF5

**Type:** HDF5 file

Top-level structure: 

- `trends/`: lower-frequency numeric arrays (vital "trends") 
- `signal/`: higher-frequency waveform arrays (vital "signals")

Datasets by sensor follow the same format as the unsegmented dataset.

**Note**: Some cases may not contain waveform data.

#### 2) Basic EHR

**Type:** JSON dictionary

Contains structured EHR data available at first look.

Fields may include: 

- Demographics / injury descriptors 
- PTA vitals and GCS components 
- Pupillometry measurements

See the Data Competition ICD Appendix for information on EHR data provided.

#### 3) LSI EHR

**Type:** JSON dictionary

Contains LSI records (from *LSI_table.csv* and *other_lsis.csv*) that occurred at or prior to the first look reference time. If no LSI events occurred before or during first look, this file may not exist.

#### 4) Ground truth

**Type:** JSON dictionary

Contains ground truth LSIs for each prediction time bin into the future of the case after first look.

Fields include:  

- `bin_start_sec`: start time of the first prediction bin
- `num_bins`: the number of prediction bins expected (int)
- `bin_size_sec`: the size of each bin in seconds (int)
- `gt`: list of length `num_bins`, where each item contains list of ground truth LSIs for that bin


#### 5) Casualty report

**Type:** JSON dictionary with key `casualty_report`, which is a list (length 1) describing the casualty.

Some fields may be **NaN**.

---------------------

## Task 2: Continuous Alert Segmented Data 

### Continuous Alert Inventory Files

There is a single inventory files included in the Continuous data directory. Each row of the inventory file contains information for a single case with relative paths to the corresponding files to be included as prediction inputs. Unlike in previous years, individual segments do not appear as separate rows in the inventory; rather, the data files contain all segments by data type (see below for details.

### Continuous Alert Case Data

Case data files for Continuous Alert are organized in subdirectories by case with *time-segmented* data for a single studyid.

Each case directory includes files with concatenated segments for each data type:

- `<studyid>_continuous_metadata.json`: segment-aligned info as **metadata** objects, primarily for book-keeping.
- `<studyid>_continuous_vs.hdf5`: high-frequency **vital-sign** waveforms and lower-frequency trend vitals, stored per segment.
- `<studyid>_continuous_basic-ehr.json`: sparse, segment-aligned **basic EHR** objects (e.g., demographics, PTA vitals, pupillometry).
- `<studyid>_continuous_lsi-ehr.json`: sparse, segment-aligned **LSI EHR** objects.
- `<studyid>_continuous_gt.json`: segment-aligned **ground-truth** labels.
- `<studyid>_continuous_casualty-report.json`: single **casualty report** record (static patient/case description).

<br>

> **Alignment across files:**  
> - The JSON files `*_metadata.json`, `*_basic-ehr.json`, `*_lsi-ehr.json`, and `*_gt.json` are **lists of the same length** within a given case.  
> - **List index `i` refers to the same segment across all of these files**, and corresponds to HDF5 group `segment_{i:03d}` in `*_vs.hdf5`.


#### 1) Metadata

**Type:** JSON list of dictionaries, one per segment.

**Key fields (per segment):**

- `studyid`: case identifier
- `segment_num`: segment number within the case (may have gaps)
- `segment_id`: unique identifier for the segment used during evaluation
- `case_id`, `case_segment_id`: identifiers for the broader case/stream
- `start_time_sec`, `stop_time_sec`: segment bounds in seconds from case start  
  (most segments are 30-second windows; segment 0 is a special “header” segment at time 0)
- `time_since_adm_sec`: Disregard for continuous alert task
- `hosp_adm`: `0/1` Disregard for continuous alert task
- `end_of_case`: `true` on the final segment

**Modality availability flags:**

- `basic-ehr`, `lsi-ehr`, `vs`, `gt`, `casualty-report`: booleans indicating whether that modality has data for this segment.

**Note**: 

- `segment_num` may have have gaps. Use **list index `i`** for guaranteed alignment across files.


#### 2) Vital signs HDF5

**Type:** HDF5 file with one top-level group per segment:

- `segment_000`, `segment_001`, …

Each `segment_XXX` contains two groups:

- `trends/`: lower-frequency numeric arrays (vital “trends”)
- `signal/`: higher-frequency waveform arrays (vital “signals”)

Datasets by sensor within each group follow same format as unsegmented dataset. See unsegmented dataset documentation for details.


#### 3) Basic EHR

**Type:** JSON list, aligned to segments.

This file is **sparse**: most entries are empty objects `{}`. When present, entries may include:

- Demographics / injury descriptors
- PTA vitals and GCS components
- Pupillometry measurements

See the Data Competition ICD Appendix for information on EHR data provided.


#### 4) LSI EHR

**Type:** JSON list, aligned to segments.

This file is **spares** and contains LSI records from `LSI_table.csv` if LSI was received within the time bounds of the segment. 


#### 5) Ground truth

**Type:** JSON list, aligned to segments.

Each entry has:

- `bin_start_sec`: segment-relative start time
- `bin_size_sec`: prediction horizon size in seconds from end of segment
- `num_bins`: N/A for continuous alert
- `gt`: list of LSI events for the segment


#### 6) Casualty report

**Type:** JSON dict with key `casualty_report`, which is a list (length 1) describing the casualty.

As with Basic EHR, some fields may be **NaN**.

