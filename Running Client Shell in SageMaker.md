# Client Container Shell v1.0

## Overview
This repository contains the Client Shell and supporting materials for teams participating in the DARPA Triage Challenge Data Competition. This Client Shell can be used to prepare submissions for the Phase 1 workshop and challenge events in accordance with the Data Competition ICD (available at https://triagechallenge.darpa.mil).

This initial release provides all necessary resources to prepare submissions for the Phase 1 workshop. Additional testing functionality will be provided in future releases.

### Minimum Requirements
- Python 3.6 or newer
- Docker
- Model that implements methods in provided `DTC_BaseModel` base class: `predict()`, `acknowledge()`, `cleanup()`, and `timed_out()` (See example in *template_model.py* )

### Quick Start
1. Clone client-shell repository from CodeCommit: [repo link](https://us-east-1.console.aws.amazon.com/codesuite/codecommit/repositories/client-shell).
2. Configure model according to Client Shell (See [Configuring your Model](https://github.com/JHUAPL-DTC-TA2/wiki/blob/client-shell-instructions/Running%20Client%20Shell%20in%20SageMaker.md#configuring-your-model)).
3. Download and start RabbitMQ Server (See [Starting the RabbitMQ server](https://github.com/JHUAPL-DTC-TA2/wiki/blob/client-shell-instructions/Running%20Client%20Shell%20in%20SageMaker.md#starting-the-rabbitmq-server)).
4. Run the client using one of two options: 
   * Run as Docker container (See [Running the Client with Docker](https://github.com/JHUAPL-DTC-TA2/wiki/blob/client-shell-instructions/Running%20Client%20Shell%20in%20SageMaker.md#running-the-client-with-docker))
   * Run locally within AWS Workspace (See [Running the Client locally](https://github.com/JHUAPL-DTC-TA2/wiki/blob/client-shell-instructions/Running%20Client%20Shell%20in%20SageMaker.md#running-the-client-locally))
5. Test connection between client and server using messaging stub (See [Passing Messages to the Client](https://github.com/JHUAPL-DTC-TA2/wiki/blob/client-shell-instructions/Running%20Client%20Shell%20in%20SageMaker.md#passing-messages-to-the-client)).


### Message Types and Handlers
The client supports a fixed set of message types (MessageType) with corresponding callbacks:

- `CONNECTION_MESSAGE`: Establishes initial connection between server and model.
- `PREDICT_MESSAGE`: Provides new patient data and requests LSI prediction by calling `predict()` in the model.
- `ACKNOWLEDGE_MESSAGE`: Acknowledges receipt of prediction by calling `acknowledge()` in the model.
- `CLEANUP_MESSAGE`: Indicates when the processing for the current patient case is complete by calling `cleanup()` in the model.
- `TIMED_OUT_MESSAGE`: Indicates the model has timed out during the current prediction request by calling `timed_out()` in the model.
- `ERROR_MESSAGE`: Contains error message from the evaluation server.

Each message type is associated with a specific handler method that calls a corresponding function in the model class or client.

### Configuring your Model
Your model must inherit from `dtc_messaging.model.DTC_BaseModel` in order to interface with the evaluator. This model class requires implementation of four class methods: `predict()`, `acknowledge()`, `cleanup()`, and `timed_out()`. For a simple example implementation, please see the provided `template_model.py`.

## Starting the RabbitMQ server
To start a RabbitMQ server with the management plugin enabled, run the following commands in your terminal:

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

`docker pull 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-rabbitmq:latest`

`docker run  --rm  552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-rabbitmq:latest`

RabbitMQ will automatically reserve and map ports `15672` and `5672` on the host to ports `15672` and `5672` in the server container, respectively. These ports are used by the RabbitMQ server.

## Running the Client with Docker

### Building with Docker Image
To containerize your model, start by authenticating to be able to pull the `dtc-base-image:latest`:

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

Build your docker image with the following command:

`docker build -t dtc-<TEAM_NAME>:<TAG> .`

This command builds the Docker based on the Dockerfile provided. This uses the standard image called `dtc-base-image:latest` which is built on top of the `nvidia/cuda:12.3.2-cudnn9-devel-ubuntu22.04` image.

### Running the Docker Container
After building the image, run the application in a Docker container with the necessary environment variables:

`docker run -it --rm dtc-<TEAM_NAME>:<TAG>`

This command runs your application in a Docker container, connecting it to an existing RabbitMQ server. The container will be removed automatically after the application exits.

## Running the Client locally

To run the Client outside docker install the package `pika` using pip (do this once):
`python -m pip install pika -e .`

The following command will run the client locally with an existing RabbitMQ server:
`python run_client.py --host localhost --queue rpc_queue`


## Passing messages to the Client

The `send_message.py` script in the `stub/` directory will send sample messages of each MessageType to a running client container. You may run the command:

`python send_message.py -m {CONNECTION_MESSAGE|PREDICT_MESSAGE|ACKNOWLEDGE_MESSAGE| ... }`

to pass a sample message to the client, where a single message type is selected. This should print out the client response message's channel, method, properties, and body data. For example, using the provided `ExampleModel` class in `template_model.py` as the model, running `python send_message.py -m CONNECTION_MESSAGE` would print this response:

```
RESPONSE:
CHANNEL: <BlockingChannel impl=<Channel number=1 OPEN conn=<SelectConnection OPEN transport=<pika.adapters.utils.io_services_utils._AsyncPlaintextTransport object at 0x109060690> params=<ConnectionParameters host=localhost port=5672 virtual_host=/ ssl=False>>>>
METHOD: <Basic.Deliver(['consumer_tag=ctag1.8851c9e07ab74dfa9a4d64efc4e4df26', 'delivery_tag=1', 'exchange=', 'redelivered=False', 'routing_key=amq.gen-PyPl17EcuEBP5s3LDzgwKA'])>
PROPERTIES: <BasicProperties(['correlation_id=9c13597d-7aa1-45f6-9e1e-a6734c826328', 'type=CONNECTION_MESSAGE'])>
BODY: b'{"response": {"response": "connected"}}'
```

## Upload image to AWS ECR (Elastic Container Registry)
Use the following steps to authenticate and push an image to your team ECR repository.

Start by retrieving an authentication token and authenticate your Docker client to your registry.

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

Tag your image so you can push the image to this repository:

`docker tag dtc-<TEAM_NAME>:<TAG> 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-<TEAM_NAME>:<TAG>`

Run the following command to push this image to your newly created AWS repository:

`docker push 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-<TEAM_NAME>:<TAG>`


## Release Notes

### V1.0
New Features:
- RabbitMQ Client: Establishes a connection to a RabbitMQ server, sets up a queue for RPC requests, and listens for incoming requests.
- Model Class: Abstract base for creating prediction models with methods for predictions, acknowledgments, end-of-case, and error scenarios. You will extend this class to implement your custom models for the evaluation. You merely need to implement four functions.
- MessageType Enum: Categorizes communication with predefined message types including connection, prediction, acknowledgment, and error handling signals.
- Message Handlers: Includes an abstract base class and specific implementations for handling various message types, ensuring appropriate communication with the evaluator.
- Factory Pattern for Message Handlers: Simplifies the creation of message handlers based on the message type, supporting scalable and modular development.

---
# (c) 2024 The Johns Hopkins University Applied Physics Laboratory LLC
