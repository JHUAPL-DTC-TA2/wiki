# Recommended Instance Types for SageMaker Studio Apps  

Applies for CodeEditor and JupyterLab instances.    

**Note**: Instance and cost information are subject to change.   
See the **"Details Link"** column for the most up-to-date instance details, including cost.
  
| Row | Instance Type  | CPU | RAM (GiB) | GPU | GPU Memory (GiB) | GPU Type       | Purpose          | Quota       | Price per Hour ($) | Details Link                                              |
|-----|----------------|-----|-----------|-----|------------------|----------------|------------------|-------------|-------------------|----------------------------------------------------------|
| 1   | ml.t3.medium   | 2   | 4         | 0   | 0                | None           | General Purpose  | 15 per team  | 0.05              | [Details](https://instances.vantage.sh/aws/ec2/t3.medium) |
| 2   | ml.m5.large    | 2   | 8         | 0   | 0                | None           | General Purpose  | 15 per team  | 0.115             | [Details](https://instances.vantage.sh/aws/ec2/m5.large)  |
| 3   | ml.m5.2xlarge  | 8   | 32        | 0   | 0                | None           | General Purpose  | 15 per team  | 0.461             | [Details](https://instances.vantage.sh/aws/ec2/m5.2xlarge)|
| 4   | ml.c5.xlarge   | 4   | 8         | 0   | 0                | None           | Compute Optimized| 15 per team  | 0.204             | [Details](https://instances.vantage.sh/aws/ec2/c5.xlarge) |
| 5   | ml.c5.2xlarge  | 8   | 16        | 0   | 0                | None           | Compute Optimized| 15 per team  | 0.408             | [Details](https://instances.vantage.sh/aws/ec2/c5.2xlarge)|
| 6   | ml.c5.9xlarge  | 36  | 72        | 0   | 0                | None           | Compute Optimized| 6 per team  | 1.836             | [Details](https://instances.vantage.sh/aws/ec2/c5.9xlarge)|
| 7   | ml.r5.8xlarge  | 32  | 256       | 0   | 0                | None           | Memory Optimized | 6 per team  | 2.419             | [Details](https://instances.vantage.sh/aws/ec2/r5.8xlarge)|
| 8   | ml.r5.16xlarge | 64  | 512       | 0   | 0                | None           | Memory Optimized | 6 per team  | 4.838             | [Details](https://instances.vantage.sh/aws/ec2/r5.16xlarge)|
| 9   | ml.p3.2xlarge  | 8   | 61        | 1   | 16               | Tesla V100     | GPU - General    | 6 per team  | 3.825             | [Details](https://instances.vantage.sh/aws/ec2/p3.2xlarge) |
| 10  | ml.p3.8xlarge  | 32  | 244       | 4   | 64               | Tesla V100     | GPU - General    | 6 per team  | 14.688            | [Details](https://instances.vantage.sh/aws/ec2/p3.8xlarge)|
| 11  | ml.g4dn.4xlarge| 16  | 64        | 1   | 16               | NVIDIA T4      | GPU - General    | 12 per team  | 1.505             | [Details](https://instances.vantage.sh/aws/ec2/g4dn.4xlarge)|
| 12  | ml.g4dn.8xlarge| 32  | 128       | 1   | 16               | NVIDIA T4      | GPU - Training   | 6 per team  | 2.72              | [Details](https://instances.vantage.sh/aws/ec2/g4dn.8xlarge)|
| 13  | ml.g5.4xlarge  | 16  | 64        | 1   | 24               | AMD Radeon Pro | GPU - Inference  | 1 per team  | 2.03              | [Details](https://instances.vantage.sh/aws/ec2/g5.4xlarge) |
