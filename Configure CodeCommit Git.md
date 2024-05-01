# Accessing CodeCommit Credentials and Configuring Git for Your Team

This guide outlines the steps for team members to obtain AWS CodeCommit credentials and set up Git in your SageMaker Studio for interacting with your team's CodeCommit repository. If you require further assistance or experience any difficulties during this process, do not hesitate to contact us at APL-DTC-Help@jhuapl.edu for support.

## Obtaining Git CodeCommit Credentials

The AWS Secrets Manager securely stores your team's CodeCommit credentials. To access your credentials, you can use the AWS Command Line Interface (CLI) from your SageMaker (or Workspace) terminal. It's important to note that access is restricted to your own team's credentials, ensuring each team member can only retrieve their respective credentials. However, this setup also ensures that every team member has the capability to securely access their team's CodeCommit credentials.

To retrieve your CodeCommit credentials, replace {team_tag} with your actual team name and execute the following command:

```bash
aws secretsmanager get-secret-value --secret-id {team_tag}/codecommit
```

Executing this command will produce a JSON object containing your credentials. To locate your username and password, search for the `SecretString` field within the JSON output.

Should you encounter the error message No such file or directory: 'less', this issue can be resolved by installing the less package. Run the following command to install less:

```bash
sudo apt-get install less
```

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

All teams are provided one repository. Contact us if you'd like to set up more repositories for your team. You should be able to access your team's repository outside AWS. You can use the same CodeCommit credentials.

### Your Team's Repository Name
Your CodeCommit repository is set up under the name `{team_tag}`, where `{team_tag}` should be replaced with your team tag.

You can clone it with the following command: 

```bash
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/{team_tag}
```

When you first push to or pull from your CodeCommit repository, Git will prompt you for your username and password. Enter the credentials obtained from AWS Secrets Manager.

Note: For CodeEditor, you will be prompted to use your provided Git username and password at the top of the browser.
![image](https://github.com/JHUAPL-DTC-TA2/wiki/images/codecommit-1.png)


