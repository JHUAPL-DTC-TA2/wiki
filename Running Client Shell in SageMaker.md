# Client Container Shell v1.1

## Overview
This is the README for the [Client Container Shell](https://us-east-1.console.aws.amazon.com/codesuite/codecommit/repositories/client-shell/browse?region=us-east-1) and supporting materials for teams participating in the DARPA Triage Challenge Data Competition. The Client Container Shell can be used to prepare submissions for the Phase 1 workshop and challenge events in accordance with the Data Competition ICD (available at https://triagechallenge.darpa.mil).

This initial release provides all necessary resources to prepare submissions for the Phase 1 workshop. Additional testing functionality will be provided in future releases.

### Minimum Requirements
- Python 3.10 or newer
- Docker (See Configuring Docker)
- Model that implements methods in provided `DTC_BaseModel` base class: `predict()`, `acknowledge()`, `cleanup()`, and `timed_out()` (See example in *template_model.py* )

### Quick Start
1. Clone client-shell repository from CodeCommit: [repo link](https://us-east-1.console.aws.amazon.com/codesuite/codecommit/repositories/client-shell/browse?region=us-east-1)
2. Configure model according to Client Shell (See **Configuring your Model**)
3. Download and start RabbitMQ Server (See **Starting the RabbitMQ server**)
4. Run the client using one of two options: 
   * Run as Docker container (See **Running the Client with Docker**)
   * Run locally within AWS Workspace (See **Running the Client locally**)
5. Test connection between client and server using messaging stub (See **Passing Messages to the Client**).
6. Test with dtc-evaluator image and run metrics (See **Evaluating Your Model in SageMaker**).


### Message Types and Handlers
The client supports a fixed set of message types (MessageType) with corresponding callbacks:

- `CONNECTION_MESSAGE`: Establishes initial connection between server and model.
- `PREDICT_MESSAGE`: Provides new patient data and requests LSI prediction by calling `predict()` in the model.
- `ACKNOWLEDGE_MESSAGE`: Acknowledges receipt of prediction by calling `acknowledge()` in the model.
- `CLEANUP_MESSAGE`: Indicates when the processing for the current patient case is complete by calling `cleanup()` in the model.
- `TIMED_OUT_MESSAGE`: Indicates the model has timed out during the current prediction request by calling `timed_out()` in the model.
- `ERROR_MESSAGE`: Contains error message from the evaluation server.

Each message type is associated with a specific handler method that calls a corresponding function in the model class or client.

## Configuring your Model
Your model must inherit from `dtc_messaging.model.DTC_BaseModel` in order to interface with the evaluator. This model class requires implementation of four class methods: `predict()`, `acknowledge()`, `cleanup()`, and `timed_out()`. For a simple example implementation, please see the provided `template_model.py`.

## Starting the RabbitMQ server
To start a RabbitMQ server with the management plugin enabled, run the following commands in your terminal:

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

`docker run --network sagemaker --rm  552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-rabbitmq:latest`

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

To run the Client outside docker install the package `pika` using pip (do this once):
`python -m pip install pika -e .`

The following command will run the client locally with an existing RabbitMQ server:
`python run_client.py --host localhost --queue rpc_queue`


## Passing messages to the Client

The `send_message.py` script in the `stub/` directory will send sample messages of each MessageType to a running client container. You may run the command:

`python send_message.py --queue rpc_queue -m {CONNECTION_MESSAGE|PREDICT_MESSAGE|ACKNOWLEDGE_MESSAGE| ... }`

to pass a sample message to the client, where a single message type is selected. This should print out the client response message's channel, method, properties, and body data. For example, using the provided `ExampleModel` class in `template_model.py` as the model, running `python send_message.py -m CONNECTION_MESSAGE` would print this response:

```
RESPONSE:
CHANNEL: <BlockingChannel impl=<Channel number=1 OPEN conn=<SelectConnection OPEN transport=<pika.adapters.utils.io_services_utils._AsyncPlaintextTransport object at 0x109060690> params=<ConnectionParameters host=localhost port=5672 virtual_host=/ ssl=False>>>>
METHOD: <Basic.Deliver(['consumer_tag=ctag1.8851c9e07ab74dfa9a4d64efc4e4df26', 'delivery_tag=1', 'exchange=', 'redelivered=False', 'routing_key=amq.gen-PyPl17EcuEBP5s3LDzgwKA'])>
PROPERTIES: <BasicProperties(['correlation_id=9c13597d-7aa1-45f6-9e1e-a6734c826328', 'type=CONNECTION_MESSAGE'])>
BODY: b'{"response": {"response": "connected"}}'
```

Two example segment files have been included to test messaging with a `PREDICT_MESSAGE`. These segment files were generated from a full case in the training dataset using the included script `tools/segment_case.py`.

## Uploading image to AWS ECR (Elastic Container Registry)
Use the following steps to authenticate and push an image to your team ECR.

Start by retrieving an authentication token and authenticate your Docker client to your registry.

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

Tag your image so you can push the image to this repository:

`docker tag --network sagemaker dtc-<TEAM_NAME>:<TAG> 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-<TEAM_NAME>:<TAG>`

Run the following command to push this image to your newly created AWS repository:

`docker push --network sagemaker 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-<TEAM_NAME>:<TAG>`


## Configuring Docker

All SageMaker app types (JupyterLab, CodeEditor, Studio Classic) support Docker API access via a proxy docker engine. To access docker in your SageMaker instance, **restart** or **create** an instance using the "install-docker-{app-type}-v4" lifecycle policy.

> Note: An existing Code Editor or JupyterLab instance will not have Docker installed until you fully stop the instance and re-run with the lifecycle policy. Restarting the instance will not delete any data in the `/home/sagemaker-user/` directory, but will delete data in other directories. 

To check Docker installed correctly, run `docker version` on a system terminal to output API and engine details.


## Configuring Docker Images to Access S3 Buckets Using AWS Credentials

If your submission requires accessing data from your team’s S3 bucket, you must configure your Docker images to use your team’s AWS credentials (i.e., `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`). By default, your credentials will not be passed from SageMaker to your Docker images. As a result, your Docker images won't be able to access AWS services like your SageMaker terminal does. To enable this, you need to transfer your SageMaker credentials to your Docker image. Follow these steps:

1. Configure your SageMaker with your AWS credentials: In a SageMaker terminal, run `aws configure` and input your AWS credentials. To find your AWS credentials, open a Workspace terminal and run `cat ~/.aws/credentials`. The `aws configure` command will generate files that store your credentials in your SageMaker home directory (i.e., `/home/sagemaker-user/.aws`).

2. Move the AWS credentials into your project directory: Run `mv ~/.aws <PROJECT_DIRECTORY>`. Replace `<PROJECT_DIRECTORY>` with the path to your project directory. This command will move the `.aws` directory containing your credentials to your project directory, allowing Docker to access these credentials during the build process. Finally, run `source ~/.bashrc` (or restart your terminal session) to refresh your default permissions (allowing access to your team's S3 buckets, ECR, and other provisioned AWS resources).

3. Use the Dockerfile in `client-shell` to build your Docker image: Navigate to the `client-shell` directory. The Dockerfile in this directory contains commands to set up your Docker image. (See updated `Dockerfile`)

4. Modify your Dockerfile to transfer the target file: Add the following command to your Dockerfile to transfer the target file from S3 to your desired directory within the Docker container. Replace `<TEAM_NAME>`, `<TARGET_FILE>`, and `<TARGET_DIRECTORY>` with your team name, the file you want to transfer, and the target directory inside the Docker container, respectively:
```dockerfile
RUN aws s3 cp s3://dtc-scratch-<TEAM_NAME>/<TARGET_FILE> /<TARGET_DIRECTORY>
```

**Note:** During the evaluation phase, evaluators will use their own credentials to access your team's S3 buckets.

**Warning:** For security reasons, do not push your credentials to CodeCommit. We have updated the `.gitignore` file to exclude all files stored in the `.aws` directory. Ensure your `.gitignore` file includes the following entry:
```
.aws
```
## Evaluating Your Model in SageMaker
In order to run the evaluator you must be authenticated to the AWS ECR. To authenticate, run `aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`
Before running the evaluator, ensure your client container is up and running (see [Running the Client with Docker](#running-the-client-with-docker) or [Running the Client locally](#running-the-client-locally) ). @suscenm1 

The evaluator can be run using a single script in the client_shell repository under `tools/eval/run_server.sh`

### run_server.sh 
The `run_server.sh` script will pull the latest `dtc-evaluator` from the ECR and run it. All you need to do is specify the following command line arguments.
- `--output-dir` or `-o` specifies where model predictions and logs will be stored. This path can point to a location that is either local to SageMaker or your team's S3-scratch bucket.
- `--inventory-file` or `-i` specifies the list of data segments to be used by the evaluator. The inventory file can exist in an S3 bucket or locally. An example inventory file can be found in client_shell/tools/eval.
- `--dataset-dir` or `-d` specifies the path to the segmented dataset. This can be a local path or an S3 path. 

 Example usage: 
- `./run_server.sh --output-dir /path/to/output --inventory-file inventory_phase1_v1-2_val.csv --dataset-dir s3://dtc-training-data/phase1_v1-2_segmented/val`
  
### run_metrics.sh

The `run_metrics.sh` script located at `tools/eval/run_metrics.sh` will compute performance metrics for an evaluation conducted using `run_server.sh`.

The script saves off three files within OUTPUT_DIR/metrics:

1. A **ground truth** JSON file containing all segments listed in the `inventory.csv`.
2. A **responses** JSON file containing the model's responses to all segments from the evaluation.
3. A **metrics** JSON containing the Jaccard Index and Prediction Lead-Time for each case. 

See this [Metrics Guide](Metrics%20Guide.md) for more details on the contents of these files. 

To compute metrics for an evaluation run, first install the requirements located in `tools/eval/requirements.txt`:

```
pip install -r requirements.txt --timeout 1000
```

After installing the requirements, run the metrics script:

```
bash ./run_metrics.sh --output-dir [OUTPUT_DIR] --inventory-file [INVENTORY_FILE] --dataset-dir [DATASET_DIR] [--allow-incomplete]
```

The `output-dir`, `inventory-file`, and `dataset-dir` should match the inputs used for `run_server.sh`.

If you want to run metrics on an incomplete run, you may include the optional `allow-incomplete` flag. Otherwise, the script will check to ensure all segments in the inventory file were run and throw an error if responses are missing.

The python scripts used to generate the ground truth, response, and metrics JSONs are located in `tools/eval/src`, but should not be altered.  

## Release Notes

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
- Updated README with instructinons for copying from S3 during Docker build.

### v1.0

New Features:

- RabbitMQ Client: Establishes a connection to a RabbitMQ server, sets up a queue for RPC requests, and listens for incoming requests.
- Model Class: Abstract base for creating prediction models with methods for predictions, acknowledgments, end-of-case, and error scenarios. You will extend this class to implement your custom models for the evaluation. You merely need to implement four functions.
- MessageType Enum: Categorizes communication with predefined message types including connection, prediction, acknowledgment, and error handling signals.
- Message Handlers: Includes an abstract base class and specific implementations for handling various message types, ensuring appropriate communication with the evaluator.
- Factory Pattern for Message Handlers: Simplifies the creation of message handlers based on the message type, supporting scalable and modular development.

---
(c) 2024 The Johns Hopkins University Applied Physics Laboratory LLC
