# Accessing CodeCommit Credentials and Configuring Git for Your Team

This guide provides instructions for team members on how to access AWS CodeCommit credentials and configure Git to work with CodeCommit repositories. Credentials may be obtained directly from your team PI via email or through AWS Secrets Manager under the `{team_tag}/codecommit` secret. Should you need assistance or encounter any issues, please reach out to APL-DTC-Help@jhuapl.edu.

## Obtaining Git CodeCommit Credentials

There are two ways to obtain your team's CodeCommit credentials.

1. From Your Team's PI  
Your team PI will provide you with CodeCommit credentials directly. These credentials include a username and a password that you will use to access your team's CodeCommit repository. Keep this information secure and do not share it with unauthorized individuals.

2. From AWS Secrets Manager  
Git credentials for each team are also stored in AWS Secrets Manager, you can retrieve them using the AWS CLI. You can only retrieve your own team's secrets. Replace {team_tag} with your actual team name to execute the following command:  
```bash
aws secretsmanager get-secret-value --secret-id {team_tag}/codecommit
```
This command will return a JSON object containing your credentials. Look for the `SecretString` field to find your username and password.

## Configuring Git

We recommend configuring your Git name, email and cache settings before cloning. 

First, configure Git with your name and email. Replace Your Name and your_email@example.com with your actual name and email:

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

To avoid entering your credentials every time you interact with your CodeCommit repository, you can use Git's credential caching feature. First, ensure you have the AWS CLI installed and configured on your machine.

Then, execute the following command to cache your credentials for a specified time (in seconds). For example, to cache credentials for 2 hours (7200 seconds):

```
git config --global credential.helper 'cache --timeout=7200'
```

## Repository Access

All teams are provided one repository. Contact us if you'd like to set up more repositories for your team. 

### Your Team's Repository Name
Your CodeCommit repository is set up under the name `{team_tag}`, where `{team_tag}` should be replaced with your team tag.

You can clone it with the following command: 

```bash
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/{team_tag}
```

When you first push to or pull from your CodeCommit repository, Git will prompt you for your username and password. Enter the credentials provided by your team PI or obtained from AWS Secrets Manager.

