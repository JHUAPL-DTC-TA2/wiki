# Using the S3 Bucket "dtc-import" for Data Transfer

This guide will go over how to use the S3 bucket "dtc-import" for transferring data into the DTC Virtual Private Cloud (VPC). This document is intended for DTC participants who need to upload data securely and efficiently to our AWS environment. The `dtc-import` bucket has been set up with specific prefixes for each team. 

## Prerequisites

- AWS CLI installed on your machine. [Download AWS CLI](https://aws.amazon.com/cli/)
- IAM Credentials for your team.
- Knowledge of your specific team's prefix within the `dtc-import` bucket.

## Finding Your Team's Prefix

Each team has been assigned a unique prefix within the `dtc-import` bucket. Your team's prefix is structured as follows: `team-tag/`. Replace `team-tag` with your actual tag. Your team tag is typically the first part of your WorkSpace or SageMaker username, before the hyphen. For example, if your workspace username is `apl-smithj`, your team tag is `apl`.   

## Uploading Files to Your Team's Prefix

To upload files from your external network to your team's prefix in the `dtc-import` bucket, follow these steps:

1. **Open your terminal or command prompt.**

2. **Configure the AWS CLI with your credentials.**
   Run `aws configure` and follow the prompts to input your AWS Access Key ID, Secret Access Key, region, and output format. Region should be set to `us-east-1` and output format could be left blank.

3. **Use the AWS CLI to upload files.**
   To upload a file to the S3 bucket, use the following command, replacing `path/to/your/file` with the path to the file you wish to upload and `team-tag` with your actual team tag.

    ```bash
    aws s3 cp path/to/your/file s3://dtc-import/{team-tag}/
    ```

For example, if your team tag is `jhu` and you're uploading `data.csv` from your desktop, the command would be:

```bash
aws s3 cp ~/Desktop/data.csv s3://dtc-import/jhu/data.csv 
```

## Downloading Files from Your Team's Prefix
To download files from your team's prefix to a machine inside your WorkSpace or SageMaker Studio environment, use the following AWS CLI command. WorkSpace and SageMaker should already have awscli pre-installed. 

```bash
aws s3 cp s3://dtc-import/{team-tag}/filename /local/path 
```

Replace /local/path with the path where you want to download the file on your local machine, team-tag with your team's tag, and filename with the name of the file you're downloading. **Only members of your team are able to copy data from your team prefix.** 

Additionally, if you want to list the contents of your prefix, you can use the following command. (could also be ran outside of the VPC). 

```bash
aws s3 ls s3://dtc-import/{team-tag}/
```

## Best Practices
- Organize your data: Use sub-prefixes under your team's prefix to organize data further, e.g., team-tag/project-name/.
- Clean up: Regularly review and delete old or unnecessary files to keep your team's prefix organized and minimize storage costs.
