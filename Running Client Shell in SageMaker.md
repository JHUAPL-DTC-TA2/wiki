# Client Container Shell v1.0

UNDER CONSTRUCTION!

## Overview
This repository contains the Client Container Shell which will be part of the evaluation system participants of the Data Challenge will be modifying. This Client Shell will be able to communicate with the official evaluation system.


Any modifications will be announce in the DTC Forum.


### Minimum Requirements
- Python 3.6 or newer
- pika library for RabbitMQ communication
- Access to a running RabbitMQ server (See Managing the RabbitMQ server)
- Ensure Docker is installed and running on your machine. If you need to install Docker, follow the instructions on the Docker official website. (Running the Client Container with Docker)
- A prediction model that supports the operations defined in the MessageType enumeration

### Quick Start
1. Clone repository client-shell repository from CodeCommit.
2. Start RabbitMQ Server (See Starting the RabbitMQ server).
3. Run the container client using one of two options: \
    a. Locally (See Running the Client locally)\
    b. Docker container (Running the Client Container with Docker)
4. Test connection using the messaging stubs (`client/stubs/send_message.py`)


### Message Types and Handlers
The client supports a fixed number of message types (MessageType) and corresponding handlers:

- `CONNECTION_MESSAGE`: Handles initial connection messages.
- `PREDICT_MESSAGE`: Processes prediction requests using the model's predict method.
- `ACKNOWLEDGE_MESSAGE`: Handles acknowledgments from the model.
- `CLEANUP_MESSAGE`: Invoked when processing for a case (or patient) is completed.
- `TIMED_OUT_MESSAGE`: Handles timeout notifications.
- `ERROR_MESSAGE`: Processes error messages.

Each message type is associated with a specific handler method that calls a function in your model class.

### Configuring your model
Your model must inherit from `dtc_client.client.DTC_BaseModel` in order to interface with the evaluator. This model class is also required to define 4 class methods: `predict()`, `acknowledge()`, `cleanup()`, `timed_out()`, and `error()`.

```python
from dtc_messaging.model import DTC_BaseModel


class ExampleModel(DTC_BaseModel):
    """
    Example model which inherits from BaseModel. Modify this model to implement your prediction logic.
    This model demonstrates returning static responses for demonstration purposes.
    """
    
    def __init__(self):
        super().__init__()

    def predict(self, data):
        """
        Generate predictions for the given data and format the results.
        
        :param data: Input data for prediction.
        :return: A dictionary containing the segment ID and the response.
        """
        # Include all the computations needed to produce a prediction
        ...

    def acknowledge(self, data):
        """
        Acknowledge receipt of data.
        
        :param data: Acknowledgement data.
        :return: An acknowledgment receipt.
        """
        # Include all the computations needed to process a reciept acknowledgement
        ...

    def cleanup(self, data):
        """
        Handle the clean up during the end-of-case.
        
        :param data: Data related to the end of the case.
        :return: A confirmation receipt.
        """
        # Include all the computations needed to process after an end-of-case
        ...

    def timed_out(self, data):
        """
        Handle the clean up during the end-of-case.
        
        :param data: Data related to the end of the case.
        :return: A confirmation receipt.
        """
        # Include all the computations needed to process after an end-of-case
        ...

    def error(self, data):
        """
        Handle errors.
        
        :param data: Error data.
        :return: An error receipt.
        """
        # Include all the computations needed to process after an end-of-case
        ...
```


## Starting the RabbitMQ server
To start a RabbitMQ server with the management plugin enabled, run the following command in your terminal:

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com`

`docker pull 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-rabbitmq:latest`

`docker run --network sagemaker --rm -p 15672:15672 -p 5672:5672 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-rabbitmq:latest`

This command maps port `15672` and `5672` on the host to port `15672` and `5672` in the container, respectively. These ports are used by the RabbitMQ management UI.


## Running the Client Container with Docker

### Building with Docker Image
To containerize the Client Container, start by building a Docker image with the following command:

`docker build --network sagemaker -t 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-base-image-client:latest .`

This command builds the Docker image named client based on the Dockerfile in the current directory.

### Running the Docker Container
After building the image, run the application in a Docker container with the necessary environment variables:

`docker run --network sagemaker -it --rm -e QUEUE_NAME='rpc_queue' -e AMQP_HOST='host.docker.internal' 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-<TEAM_NAME>-client:<TAG>`

This command runs your application in a Docker container, connecting it to a running RabbitMQ server with the specified `QUEUE_NAME` and `AMQP_HOST` environment variables. The container will be removed automatically after the application exits.







## Running the Client locally

### Starting the Client
To run the Client outside docker install the package using pip:

`python -m pip install pika -e .`

Then run this:
`python run_client.py --host localhost --queue rpc_queue`


### Passing messages to the Client

See scripts in the `stub/` directory to send messages to the client container. You may run the command:

`python send_message.py -m {CONNECTION_MESSAGE|PREDICT_MESSAGE|ACKNOWLEDGE_MESSAGE| ... }`

to pass a sample prediction message to the client. This should print out the message's channel, method, properties, and body. For example, running `python send_message.py -m CONNECTION_MESSAGE` would print this response:

```
RESPONSE:
CHANNEL: <BlockingChannel impl=<Channel number=1 OPEN conn=<SelectConnection OPEN transport=<pika.adapters.utils.io_services_utils._AsyncPlaintextTransport object at 0x109060690> params=<ConnectionParameters host=localhost port=5672 virtual_host=/ ssl=False>>>>
METHOD: <Basic.Deliver(['consumer_tag=ctag1.8851c9e07ab74dfa9a4d64efc4e4df26', 'delivery_tag=1', 'exchange=', 'redelivered=False', 'routing_key=amq.gen-PyPl17EcuEBP5s3LDzgwKA'])>
PROPERTIES: <BasicProperties(['correlation_id=9c13597d-7aa1-45f6-9e1e-a6734c826328', 'type=CONNECTION_MESSAGE'])>
BODY: b'{"response": {"response": "connected"}}'
```

# Client Shell Features
- RabbitMQ Client: Establishes a connection to a RabbitMQ server, sets up a queue for RPC requests, and listens for incoming requests.
- Model Class: Abstract base for creating prediction models with methods for predictions, acknowledgments, end-of-case, and error scenarios. You will extend this class to implement your custom models for the evaluation. You merely need to modify functions...
- MessageType Enum: Categorizes communication with predefined message types including connection, prediction, acknowledgment, and error handling signals.
- Message Handlers: Includes an abstract base class and specific implementations for handling various message types, ensuring appropriate communication with the evaluator.
- Factory Pattern for Message Handlers: Simplifies the creation of message handlers based on the message type, supporting scalable and modular development.