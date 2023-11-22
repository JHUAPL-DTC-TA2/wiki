This user guide provides step-by-step to configure your AWS Workspace and AWS SageMaker environment as well as troubleshooting notes for common issues. Contact the JHU APL team if you have any questions. 

## Table of Contents

- Tutorials
	- Configuring your WorkSpace
	- Connecting to your team's SageMaker Studio
	- Creating an instance in SageMaker Studio
	- Loading data from S3
- Common Issues

## Tutorials 
### Configuring your WorkSpace

AWS WorkSpace is a Desktop-As-A-Service platform that streams a virtualized desktop environment to your local machine. In this challenge, we are using WorkSpaces to provide participants a pre-configured, secure environment to access DTC data and develop their models in. 

Currently, all participants will use the same WorkSpace bundle: Ubuntu 20.04 Power Bundle (4 vCPU, 16 GiB RAM) with the following pre-installed libraries and programs: FireFox, Python 3.9, VSCode, and MySQL WorkBench. We may change the default bundle or provide alternative bundles in the future. 

Once you are registered for the TA2 challenge, you should receive two emails contain information about the WorkSpace and SageMaker domain. The WorkSpace email will look similar to the one show below and contains the information you need to configure your WorkSpace.

>Dear Amazon WorkSpaces User,
	A new Amazon Workspace has been provided for you. Follow the steps below to quickly get up and running with your WorkSpace:
	Complete your user profile and download a WorkSpaces client using the following link: **Link Here**
	Launch the client and enter the following registration code: **Registration Code here**
	Log in with your user name, and enter a new password. Your user name is **Username here**
	you may download clients for additional devices at https://clients.amazonworkspaces.com/
	If you have any Issues connecting to your WorkSpace, please contact your administrator.
	Sincerely,
	Amazon WorkSpaces

Follow the steps below to finish setting up your WorkSpace.
1. Click the link for your user profile and create a password for your account. Account passwords can be reset by an administrator or through the client.

!(Workspace 1)[images/workspace-1.png]

2. If you haven't already, download and install the WorkSpaces client at https://clients.amazonworkspaces.com/. The WorkSpaces client is compatible with most operating systems.
3. Open the WorkSpace client and input your registration code in the in the input box.

![[Screenshot 2023-11-07 at 1.27.31 AM.png]]4. Log-in using the username from the WorkSpaces email and the password you created in Step 1. 

!(Workspace 2)[images/workspace-2.png]

5. Once you successfully log-in, the client will expand to fill your screen and load your WorkSpace. On its first load, this may take a couple minutes to fully initialize. If you have a multi-monitor or high-resolution screen, the WorkSpace desktop may seem zoomed-out or small. Closing the client and re-logging in sometimes helps with this issue. If the issue persists, disable High-DPI mode in Settings -> Display Settings. 
!(Workspace 3)[images/workspace-3.png]

That's it! A couple of additional points:
- The default WorkSpace supports "Paste-Only" clipboard redirect. This means you can copy and paste things from your local workstation to the WorkSpace, but not the other way around.
- Your WorkSpace user will not have access to local administrator by default, but the user account is able to install user level libraries and packages. 
- Pre-installed applications such as FireFox, VSCode, and MySQL WorkBench can be found in the Applications library, which can be opened by clicking "Show Applications".
- WorkSpaces will shut down after an hour of inactivity. If this is too short or getting the way, let us know. 

### Connecting to your team's SageMaker Studio

SageMaker Studio is a web-based IDE, similar to JupyterLab, that lets you run code on AWS instances. Each team is provided a studio domain, and individual users are provided user profiles. Each user profile has an *Elastic File Store* within their profile to store files. When creating a kernel or running arbitrary code, SageMaker studio will prompt you to create an instance from an image. Instances define the hardware specifications and images define the software specifications. SageMaker studio uses AWS IAM for authentication, but to ease this process, we created a set-up script that will create a log-in application in your workspace to make accessing SageMaker studio as easy as possible. 

Once you are registered for the TA2 challenge, you should receive two emails contain information about the WorkSpace and SageMaker domain. The SageMaker domain email will look similar to the one show below and contains the information you need to connect to SageMaker. **You will need to first set up your WorkSpace before proceeding to connect to SageMaker.**

>Dear User,
>Below is your SageMaker Studio log-in details. Please contact APL-DTC-Help@jhuapl.edu if you have any questions.
>
>AWS Access Key: **Access Key here**
>AWS Secret Key: **Secret Key here**
>Studio Domain ID: **Domain ID here**
>User Profile: **User Profile Name here**

Follow the steps below to configure your WorkSpace to connect to your team's SageMaker Studio. You only need to configure your SageMaker Studio once. After initial configuration, you can use the generated launch script to reconnect to your SageMaker studio. 

1. Connect to your Ubuntu WorkSpace using the AWS WorkSpace client.
2. Open a terminal window and run the following command to configure your AWS credentials. These credentials are shared by your team and allow you to create pre-signed URLs to your team's SageMaker domain.

	`aws configure`

	The CLI will then prompt you to input your AWS Access key and secret key, which can be copied directly from the set-up email. Set the default region to `us-east-1` and leave the default output format empty. 

3. Each WorkSpace comes pre-loaded with a set-up script named `dtc-setup.sh`. This script creates a launch script, `open_presigned_domain_url.sh`, that when ran will generate an authenticated URL and open it on the default browser. To use the script, run it directly in a terminal and replace the arguments with your own values for domain ID and user profile name from the set-up email. 

	`dtc-setup.sh {DOMAIN_ID} {USER_PROFILE_NAME}`

!(SageMaker 1)[images/sagemaker-1.png]
You'll know the script ran successfully when there is a new file on your desktop named `open_presigned_domain_url.sh`. 

4. Launch the script directly from the Desktop by right-clicking it and clicking "Run as Program". You can also run it from the terminal. 

!(SageMaker 2)[images/sagemaker-2.png]

Running the launch script should open a new tab or window on your browser and load SageMaker Studio default page. 

!(SageMaker 3)[images/sagemaker-3.png]




