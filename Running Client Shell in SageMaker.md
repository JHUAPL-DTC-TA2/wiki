# Client Container Shell v2.0

## Overview
This is the README for the [Client Container Shell](https://us-east-1.console.aws.amazon.com/codesuite/codecommit/repositories/client-shell/browse?region=us-east-1) and supporting materials for teams participating in the DARPA Triage Challenge Data Competition. The Client Container Shell can be used to prepare submissions for the workshop and challenge events in accordance with the Data Competition ICD (available at https://triagechallenge.darpa.mil).

### Minimum Requirements
- Python 3.10 or newer
- Docker (See Configuring Docker)
- Model that implements methods in provided `DTC_BaseModel` base class: `predict()`, `acknowledge()`, `cleanup()`, and `timed_out()` (See example in *template_model.py* )

### Quick Start
1. Clone client-shell repository from CodeCommit   
  `git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/client-shell`  
2. Configure model according to Client Shell (See [Configuring your Model](#configuring-your-model))
3. Download and start RabbitMQ Server (See [Starting the RabbitMQ server](#starting-the-rabbitmq-server))
4. Run the client using one of two options: 
   * Run as Docker container (See [Running the Client with Docker](#running-the-client-with-docker))
   * Run locally within AWS WorkSpace (See [Running the Client locally](#running-the-client-locally))
5. Test connection between client and server using messaging stub (See [Passing Messages to the Client](#passing-messages-to-the-client)).
6. Evaluate client using `dtc-evaluator` and run metrics on output (See [Evaluating Your Model in SageMaker](#evaluating-your-submission-in-sagemaker)).

## Configuring your Model
Your model must inherit from `dtc_messaging.model.DTC_BaseModel` in order to interface with the evaluator. The model class must override five methods: `predict()`, `acknowledge()`, `cleanup()`, `timed_out()`, `error()`.  Each method above is a callback in response to receipt of one of the message types (MessageType) listed below:

- `PREDICT_MESSAGE`: Provides new patient data and requests LSI prediction by calling `predict()` in the model.
- `ACKNOWLEDGE_MESSAGE`: Acknowledges receipt of prediction by calling `acknowledge()` in the model.
- `CLEANUP_MESSAGE`: Indicates when the processing for the current patient case is complete by calling `cleanup()` in the model.
- `TIMED_OUT_MESSAGE`: Indicates the model has timed out during the current prediction request by calling `timed_out()` in the model.
- `ERROR_MESSAGE`: Contains error message from the evaluation server.

In addition, there is a `CONNECTION_MESSAGE` that establishes initial connection between server and model, which is handled by the model base class. As a simple example implementation of the methods listed above, please see the provided *template_model.py*.

## Starting the RabbitMQ server
To start a RabbitMQ server with the management plugin enabled, run the following commands in your terminal:

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

`docker run -d --network sagemaker --rm  552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-rabbitmq:latest`

RabbitMQ will automatically reserve and map ports `15672` and `5672` on the host to ports `15672` and `5672` in the server container, respectively. These ports are used by the RabbitMQ server.

## Running the Client with Docker

### Building with Docker Image
To containerize your model, start by authenticating to be able to pull the `dtc-base-image`

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

Build your docker image with the following command:

`docker build --network sagemaker -t dtc-<TEAM_NAME>:<TAG> .`

This command builds the Docker based on the Dockerfile provided. By default, the client-shell Dockerfile builds itself off of  `dtc-base-image:latest` which uses GPU. See the below **Base Images** section, to substitute for a cpu-only base image.
Both images include the `dtc_messaging` python package used to interface with the official evaluation server, and `awscli` to access available AWS resources provisioned to your team.


#### Base Images
There are two base images available for your use in the AWS ECR:
- dtc-base-image:latest 
- dtc-base-image-cpu:latest

`dtc-base-image:latest` is configured for teams whose models require GPU and is built off of `nvidia/cuda:12.3.2-cudnn9-devel-ubuntu22.04`, while `dtc-base-image-cpu:latest` holds a lighter-weight framework for those models that only run on CPU, and it is built off of `ubuntu:22.04`.

You should always use images tagged `latest`, however a full history of dtc-base-image versions is available on the ECR. As of date, the following base images exists:

| Image Name         | Tags           | Release    |
|:--------           | :----:         | :-------:  |
| dtc-base-image     | latest, v1-1   | May 2024   |
| dtc-base-image-cpu | latest, v1-0   | June 2024  |
| dtc-base-image     | v1-0           | March 2024 | 


Further, the [dtc-base-image repository](https://us-east-1.console.aws.amazon.com/codesuite/codecommit/repositories/dtc-base-image/browse?region=us-east-1) holds the code used to generate these images.
The `main` branch contains dtc-base-image source code and the `cpu-only` branch  contains dtc-base-image-cpu source code.

### Running the Docker Container
After building the image, run the application in a Docker container with the necessary environment variables:

`docker run --network sagemaker -it --rm dtc-<TEAM_NAME>:<TAG>`

This command runs your application in a Docker container, connecting it to an existing RabbitMQ server. The container will be removed automatically after the application exits.

## Running the Client locally

To run the Client outside docker, you must first install the `dtc_messaging` package and dependencies in your local enviroment. Clone the [dtc-base-image repo](https://git-codecommit.us-east-1.amazonaws.com/v1/repos/dtc-base-image) and run the following within the cloned repo: `python3.10 -m pip install .`. 

The following command will run the client locally with an existing RabbitMQ server:
`python run_client.py --host localhost --queue rpc_queue`


## Passing messages to the Client

The `send_message.py` script in the `stubs/` directory can be used to test receipt of sample messages of each MessageType with a running client container. You may run the command:

`python send_message.py --queue rpc_queue -m {CONNECTION_MESSAGE|PREDICT_MESSAGE|ACKNOWLEDGE_MESSAGE|ERROR_MESSAGE|TIMED_OUT_MESSAGE|CLEANUP_MESSAGE}`

to pass a sample message to the client, where a single message type is selected. This should print out the client response message's channel, method, properties, and body data. For example, using the provided `ExampleModel` class in `template_model.py` as the model, running `python send_message.py -m CONNECTION_MESSAGE` would print this response:

```
RESPONSE:
CHANNEL: <BlockingChannel impl=<Channel number=1 OPEN conn=<SelectConnection OPEN transport=<pika.adapters.utils.io_services_utils._AsyncPlaintextTransport object at 0x109060690> params=<ConnectionParameters host=localhost port=5672 virtual_host=/ ssl=False>>>>
METHOD: <Basic.Deliver(['consumer_tag=ctag1.8851c9e07ab74dfa9a4d64efc4e4df26', 'delivery_tag=1', 'exchange=', 'redelivered=False', 'routing_key=amq.gen-PyPl17EcuEBP5s3LDzgwKA'])>
PROPERTIES: <BasicProperties(['correlation_id=9c13597d-7aa1-45f6-9e1e-a6734c826328', 'type=CONNECTION_MESSAGE'])>
BODY: b'{"response": {"response": "connected"}}'
```

## Uploading docker image to AWS ECR (Elastic Container Registry)
Use the following steps to authenticate and push an image to your team ECR. Note that for submission, this is done automatically within the CodeBuild CI/CD build process for successful builds.

Start by retrieving an authentication token and authenticate your Docker client to your registry.

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

Tag your image so you can push the image to this repository:

`docker tag dtc-<TEAM_NAME>:<TAG> 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-<TEAM_NAME>:<TAG>`

Run the following command to push this image to your newly created AWS repository:

`docker push 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-<TEAM_NAME>:<TAG>`


## Configuring Docker

All SageMaker app types (JupyterLab, CodeEditor, Studio Classic) support Docker API access via a proxy docker engine. To access docker in your SageMaker instance, **restart** or **create** an instance using the "install-docker-cobalt-{app-type}-v2" lifecycle policy.

> Note: An existing Code Editor or JupyterLab instance will not have Docker installed until you fully stop the instance and re-run with the lifecycle policy. Restarting the instance will not delete any data in the `/home/sagemaker-user/` directory, but will delete data in other directories. 

To check Docker installed correctly, run `docker version` on a system terminal to output API and engine details.


## Configuring Docker Images to Access S3 Buckets Using AWS Credentials

By default, your credentials will not be passed from SageMaker to your Docker images. As a result, your Docker images is restricted to access your provisioned AWS services. If your submission requires accessing data from your team’s S3 bucket, you may transfer your SageMaker credentials to your Docker image by passing your AWS credentials (i.e., `USER_AWS_ACCESS_KEY`, `USER_AWS_SECRET_ACCESS_KEY`) to the Docker image. These access keys should have been emailed to you for your SageMaker account in an email that looks similar to [this](https://github.com/JHUAPL-DTC-TA2/wiki/blob/main/DTC%20Participant%20AWS%20User%20Guide.md#connecting-to-your-teams-sagemaker-studio). Follow these steps:

1. In your SageMaker terminal, set the environmental variables for `KEY` and `SECRET KEY` with your `USER_AWS_ACCESS_KEY` and `USER_AWS_SECRET_ACCESS_KEY`, respectively.
    
   ```
   export KEY=<YOUR_AWS_ACCESS_KEY>
   export SECRET_KEY=<YOUR_AWS_SECRET_KEY>
   ```
    To ensure persistence of these variables, it is recommended you add these two lines to your `~/.bashrc` file.

2. Modify your `Dockerfile` to transfer the target file: Add the following command to your Dockerfile to transfer the target file from S3 to your desired directory within the Docker container. Replace `<TEAM_NAME>`, `<TARGET_FILE>`, and `<TARGET_DIRECTORY>` with your team name, the file you want to transfer, and the target directory inside the Docker container, respectively:
    ```dockerfile
    RUN aws s3 cp s3://dtc-scratch-<TEAM_NAME>/<TARGET_FILE> /<TARGET_DIRECTORY>
    ```

3. In the `Dockerfile` script, you will find the following lines:
    ```dockerfile
    ARG KEY
    ARG SECRET_KEY
    ENV AWS_ACCESS_KEY_ID=$KEY
    ENV AWS_SECRET_ACCESS_KEY=$SECRET_KEY
    ```

   These specify your credentials, as build arguments, to be passed into the Dockerfile. These will be used to during Docker build phase to access your S3 bucket. When you are ready to build your `Dockerfile`, run the command:
    ```docker build --network sagemaker --build-arg KEY=$KEY --build-arg SECRET_KEY=$SECRET_KEY -t {image_name}:{image_tag} . ```
    Note: The CI/CD is expecting these build arguments, so your Dockerfile submission must provide them. Refer to the ICD and [these instructions](https://github.com/JHUAPL-DTC-TA2/wiki/blob/main/Submission%20Process.md) for more information.
    **Warning:** For security reasons, do not push your credentials to CodeCommit.
   



## Evaluating Your Submission in SageMaker
In order to run the evaluator, you must first authenticate your session to the AWS ECR. To authenticate, run `aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`
Before running the evaluator, ensure that your client container is running (see [Running the Client with Docker](#running-the-client-with-docker) or [Running the Client locally](#running-the-client-locally)).

Additionaly, you need to set two environment variables `KEY` and `SECRET_KEY`. These can be set by running the following the SageMaker terminal:
```
export KEY=<your key>
export SECRET_KEY=<your key>
``` 
Place these in your `~/.bashrc` to avoid having to set for every session. These are the keys used for your SageMaker login [here](https://github.com/JHUAPL-DTC-TA2/wiki/blob/main/DTC%20Participant%20AWS%20User%20Guide.md#connecting-to-your-teams-sagemaker-studio).

The evaluator can then be run using a convenience script in the client-shell repository: `eval/run_server.sh`

### run_server.sh 
The `run_server.sh` script located at `eval/run_server.sh` will pull the latest `dtc-evaluator` from AWS ECR and run it.
```
bash ./run_server.sh --output-dir [OUTPUT_DIR] --inventory-file [INVENTORY_FILE] --dataset-dir [DATASET_DIR] [--include-basic-ehr] [--include-expanded-ehr]
```

The following arguments are used to specify the data and evaluation configuration:

- `--output-dir` or `-o` specifies where model predictions and logs will be stored. This path can point to a location that is either local to SageMaker or your team's S3-scratch bucket.
- `--inventory-file` or `-i` specifies the list of data segments to be used by the evaluator. This must be from a phase 2 dataset (*phase1_v2+* or *phase2_v1+*). The inventory file can exist in an S3 bucket or locally. An example inventory file can be found in *client_shell/eval*.
- `--dataset-dir` or `-d` specifies the path to the segmented dataset. This can be a local path or an S3 path. Note that this dataset must correspond to the inventory file provided above.
- `--include-basic-ehr` is an optional flag that includes basic EHR data in the run for each case.
- `--include-expanded-ehr` is an optional flag that includes expanded EHR data (as well as basic EHR data) in the run for each case.

Example: 
```
./run_server.sh --output-dir outputs --inventory-file inventory_phase1_v2-0_val_mini.csv --dataset-dir s3://dtc-training-data/phase1/phase1_v2-0_segmented/val --include-basic-ehr
```
This command will store evaluation outputs in the `./outputs` directory, using the `./inventory_phase1_v2-0_val_mini.csv` as the inventory file and the `s3://dtc-training-data/phase1_v2-0_segmented/val` as the source dataset with basic EHR data included for each case. Example evaluation output and logs can be found in `eval/example_output/out` and `eval/example_output/logs`, respectively.

  
### run_metrics.sh

The `run_metrics.sh` script located at `eval/run_metrics.sh` will compute performance metrics on the evaluation output from `run_server.sh`.

The script saves off three files within OUTPUT_DIR/metrics:

1. A **ground truth** JSON file containg ground truth for all segments listed in the inventory file.
2. A **responses** JSON file containing the model's responses to all segments from the evaluation.
3. A **metrics** JSON containing the Mean Squared Correct (MSC) metrics for each case. 

See this [Metrics Guide](Metrics%20Guide.md) for more details on the contents of these files. 

To compute metrics for an evaluation run, first install the requirements located in `eval/requirements.txt`:

```
pip install -r eval/requirements.txt --timeout 1000
```

After installing the requirements, run the metrics script from within the `eval/` directory:

```
bash ./run_metrics.sh --output-dir [OUTPUT_DIR] --inventory-file [INVENTORY_FILE] --dataset-dir [DATASET_DIR] [--allow-incomplete]
```

The `output-dir`, `inventory-file`, and `dataset-dir` should match the inputs used for `run_server.sh`.

If you want to run metrics on an incomplete run, you may include the optional `allow-incomplete` flag. Otherwise, the script will check to ensure all segments in the inventory file were run and throw an error if responses are missing.

The python scripts used to generate the ground truth, response, and metrics JSONs are located in `eval/src`, but these scripts should not be altered to ensure consistent metrics with the competition.

Example output of the metrics can be found in `eval/example_output/metrics`.

## Release Notes
### v2.0
- updated metrics scripts for phase 2
- addition of new CLI args to run_server.sh (--include-basic-ehr, --include-expanded-ehr) with minor refactoring
- moved eval/ and stubs/ to top-level directory
- added example output from evaluator for phase 2
- updated Dockerfile entrypoint so it includes run_client.py for easier override of args

### v1.3
- Added ENV passable variables to evaluator and client-shell for AWS keys

### v1.2
- Added support for cpu-only base image and updated base-image tags to reflect versions.
- Added a tools directory in the client-shell with scripts to support running evaluation and metrics.
- Added instructions to the wiki for running the evaluator in SageMaker.
  
### v1.1
- Moved dtc_messaging module to dtc-base-image repo.
  - Fixed bug in message_handler CLEANUP_MESSAGE enum.
  - Removed top-level “response” key from message dicts.
  - Adds serialization of Numpy arrays as lists.
  - Renamed Messenger.py to Client.py.
  - Added purge of RabbitMQ queue during initialization of client.
- Uses new Docker base image, dtc-base-image:v1-1.
  - Includes AWS CLI for copying files from S3 in Docker image.
  - Creates logs directory, which will be mounted and saved off during evaluation.
  - Installs latest version of dtc_messaging module.
- Added comments to Dockerfile with example setup of AWS credentials and S3 copy.
- Added .gitignore to prevent AWS credentials from being checked into repo.
- Updated send_message.py stub to match message formats used in evaluation.
- Updated template_model.py to access all possible fields expected in received messages.
- Updated README with instructions for copying from S3 during Docker build.

### v1.0

New Features:

- RabbitMQ Client: Establishes a connection to a RabbitMQ server, sets up a queue for RPC requests, and listens for incoming requests.
- Model Class: Abstract base for creating prediction models with methods for predictions, acknowledgments, end-of-case, and error scenarios. You will extend this class to implement your custom models for the evaluation. You merely need to implement four functions.
- MessageType Enum: Categorizes communication with predefined message types including connection, prediction, acknowledgment, and error handling signals.
- Message Handlers: Includes an abstract base class and specific implementations for handling various message types, ensuring appropriate communication with the evaluator.
- Factory Pattern for Message Handlers: Simplifies the creation of message handlers based on the message type, supporting scalable and modular development.

---
(c) 2025 The Johns Hopkins University Applied Physics Laboratory LLC
