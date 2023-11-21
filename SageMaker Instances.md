We are going to increase service quotas for the following 12 instance types:

**Quota Design: Max 3 instances per team**

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|Row|Instance Type|Specs  <br>(nCPU, nGB RAM, nGPU, GPU Memory)|Purpose|Quota|Price per Hour ($)|
|1|ml.t3.medium|2, 4, 0, 0|General Purpose|3 per team|0.05|
|2|ml.m5.large|2, 8, 0, 0|General Purpose|3 per team|0.115|
|3|ml.m5.2xlarge|8, 32, 0, 0|General Purpose|3 per team|0.461|
|4|ml.c5.xlarge|4,8,0,0|Compute Optimized|3 per team|0.204|
|5|ml.c5.2xlarge|8, 16, 0, 0|Compute Optimized|3 per team|0.408|
|6|ml.c5.9xlarge|36, 72, 0, 0|Compute Optimized|1 per team|1.836|
|7|ml.r5.8xlarge|32, 256, 0, 0|Memory Optimized|1 per team|2.419|
|8|ml.r5.16xlarge|64, 512, 0, 0|Memory Optimized|1 per team|4.838|
|9|ml.p3.2xlarge|8, 61, 1, 16|GPU - General|1 per team|3.825|
|10|ml.p3.8xlarge|32, 244, 4, 64|GPU - General|1 per team|14.688|
|11|ml.g4dn.4xlarge|16,64,1,16|GPU - General|2 per team|1.505|
|12|ml.g4dn.8xlarge|32,128,1,16|GPU - Training|1 per team|2.72|
|13|ml.g5.4xlarge|16, 64, 1, 24|GPU - Inference|1 per team|2.03|

