# Common Issues and Troubleshooting

Below we describe common issues and troubleshooting steps for AWS WorkSpaces and SageMaker Studio. Please contact the JHU APL team (APL-DTC-Help@jhuapl.edu) if you have any questions or need further troubleshooting.  

## WorkSpace

1. **"Password Expired"**  
Passwords can be reset via the "Forgot Password" link on the WorkSpaces client. Depending on your client version and machine OS, you may need to fully log-out to see the "Forgot Password" link near the bottom of the view. Refer to [this forum post](https://forum.triagechallenge.darpa.mil/t/workspaces-password-reset/407) for more instructions.  
In some cases, browser cache can affect the reset password screen. If you encounter an error in reseting your password after clicking "Forgot Password", try using another browser or an incognito/private browser window that does not use the pre-existing browser cache.  


2. **"Unable to connect to Network"**  
Connection issues to WorkSpaces are typically a result of an external firewall or network restriction blocking the WorkSpaces port. [This AWS document](https://docs.aws.amazon.com/workspaces/latest/adminguide/workspaces-port-requirements.html) describes the ports that need to be available for the WorkSpaces to properly stream to the client.  
If the issues still persists, follow the logging steps described [here](https://docs.aws.amazon.com/whitepapers/latest/best-practices-deploying-amazon-workspaces/collecting-a-workspaces-support-log-bundle-for-debugging.html) to collect low-level system logs. Send these logs to the APL team and we will help debug and troubleshoot. 

3. **Sudo Access and Installing Applications**  
Participants are unable to perform "sudo" commands on their WorkSpace, making it difficult or impossible to install new libraries or programs via `apt-get` or other standard package managers. While we don't recommend using the WorkSpace instance for any development or model training, we are happy to modify your team's base image with any additional tools or software that you'd like to have, just email the JHU APL team. Keep in mind that the WorkSpace is mainly there to provide way to stream SageMaker securely, and offers little in terms of compute resources (4 Cores, 16 GB RAM, no GPU). 

## SageMaker
1. **Managing Environment**  
Refer to this [link](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-lab-use-manage.html) for best-practices and tutorials on how to manage conda environments within SageMaker.  
When executing code in a notebook or script, SageMaker will prompt you to start an "instance". We recommend you pick an instance that matches the requirements for the code to run successfully. (i.e. pick a high RAM instance if your code needs a lot of memory, or a GPU instance if your code needs CUDA.) Refer to [SageMaker Instances](sagemaker_instances.md) for a list of instance types and their specifications.  
Make sure to shutdown instances when you are done using them via the instance management panel in the SageMaker UI. 

2. **Broken Packages/Images**  
Sometimes the default AWS images provided in SageMaker are updated and package versions are changed or broken. This [link](https://docs.aws.amazon.com/sagemaker/latest/dg/notebooks-available-images.html) contains the list of all AWS pre-built images.  
If the pre-built images are not working for your particular use-case, e-mail us to set up a custom image built to your specifications via Dockerfile. 

