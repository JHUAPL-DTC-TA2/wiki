# Submission Process

Each participant will be provisioned AWS CodeBuild Continuous Integration/Continuous Development (CI/CD) resources to provide automatic feedback regarding the submission compliance with the official evaluation system. This (CI/CD) system will containerize the submitted code using a standard buildspec and simulate a 10-case minature version of the evaluation process to assess the code’s compliance. The estimated time to complete a test is ~10-15 minutes, as CodeBuild needs to queue and provision resources, and build/run containers to complete the test.

To review the system predictions and logs of the CI/CD, the build objects will be placed in your team's scratch bucket under `s3://dtc-scratch-{team_name}/codebuild/`. The CI/CD  will store the CodeBuild logs for each unique build in the `build-logs/` directory, and the evaluation outputs in the `submission_results/` directory (i.e.  the `logs/` and `out/` directories equivalent to those produced by the dtc-server).

Teams can trigger the CodeBuild in **TWO** ways:

- **Compliance Test:** to perform intial compliance testing during the development phase.
  - *Triggers*: pushing updates to a team's CodeCommit branch called `compliance-test`.
  - *Outcomes*: the team's Principal Investigator will recieve an email from AWS's SNS services if the the CI/CD successfully completed.
 
- **Official Event Submissions:** to officially submit for an event evaluation.
  - *Triggers*: pushing updates to a team's CodeCommit with a commit tag called `submission-phase{PHASE_NUMBER}-{EVENT_TYPE}`.
    - `PHASE_NUMBER = {1,2,3}`
    - `EVENT_TYPE = {workshop, challenge}`
  - *Outcomes*: if a build is successful, an email will be sent to the Principal Investigator and a docker image will be stored for review. Your previous image submission will be overwritten.

## Logs
While logs can be configured to save out to teams S3 bucket during development and testing in SageMaker, we will not allow logs to be written to S3 buckets during formal workshop and competition evaluations. Rather, we will copy any logs saved in the client container under `/usr/src/app/logs/` manually to teams s3 buckets following the completion of an event. 
As a result, if you submit code to the CI/CD with logs saving in the client-shell to an s3 bucket, your pipeline will fail. 

At the time of the Event submission deadline, the Challenge administrators will automatically pull the images for evaluation.

More information on the CI/CD can be found in the Data Challenge ICD.
