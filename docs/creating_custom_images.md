# Creating custom Docker Images to use in SageMaker 

This guide walks you through the process of pushing a custom Docker image to an Amazon Elastic Container Registry (ECR) repository named dtc-{team-prefix} and subsequently using this image in AWS SageMaker Studio for your data science and machine learning projects.

**Prerequisites**
- AWS CLI installed and configured on your machine.
- Docker installed on your machine.
- Team IAM Credentials (AWS Access keys)
- Knowledge of your team's prefix to replace {team-prefix} in the repository name.

## Finding your team's repository

Each team has a default repository created for them under the name `dtc-{team-prefix}`. To check the status of your repository, you can use the AWS CLI.

```bash
aws ecr describe-repositories --repository-name dtc-<team-name>
```

Only your team and DTC Admins have the permissions to view your ECR repository. Contact admins if you would like additional repositories. 

## Authenticate Docker of Your ECR Repository 

Before pushing images to ECR, you must authenticate Docker to the ECR registry. Run the following command directly to authenticate your terminal to push to your ECR repository. 

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 552707247569.dkr.ecr.us-east-1.amazonaws.com
```

## Pushing your Docker Image

Navigate to the directory containing your Dockerfile and run the following command, replacing {image-name} with your desired image name. If not specified, it will default to `latest`.

```bash
docker build --network sagemaker -t dtc-{team-prefix}:{image-name} .
```

Tag the image to the ECR repository URL. 

```bash
docker tag dtc-{team-prefix}:{image-name} 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-{team-prefix}:{image-name}
```

Finally, push the image to your ECR repository.

```bash
docker push 552707247569.dkr.ecr.us-east-1.amazonaws.com/dtc-{team-prefix}:{image-name}
```

## Attaching your image to SageMaker Studio

To attach the image to your SageMaker studio domain, follow the CLI-specific instructions provided in this [AWS reference document](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi-create.html). 

Once you reach the final step of creating the App Image Config, send an email to APL-DTC-Help@jhuapl.edu with your image name, app config name and team name for an admin to add it to your domain. 

> Note: You do not need to provide an execution role ARN when creating the image or image configurations through AWS. The execution role should be an optional argument. 