# Using OpenCode with Claude Opus 4.8

OpenCode is a terminal-based coding assistant that can use the DTC-provided Amazon Bedrock inference profile. It uses the AWS identity already available in your environment, so you do not need to create an API key or Bedrock bearer token.

!!! important
    Each team can run inference only through its assigned application inference profile. The only model available through OpenCode at this time is **Claude Opus 4.8**. Other Bedrock models and other teams' inference profiles are not available.

## Get your team's inference profile ARN

The application inference profile ARN is team-specific and will be sent to each team's principal investigator (PI) by email. Contact your team PI to obtain the ARN before beginning setup.

The ARN has the following format:

```text
arn:aws:bedrock:us-east-1:<AWS_ACCOUNT_ID>:application-inference-profile/<PROFILE_ID>
```

Use the complete ARN provided by your PI. Do not substitute a foundation model ID, a system inference profile, or another team's ARN.

## Verify your AWS identity

Open a terminal in your environment and run:

```bash
aws sts get-caller-identity
```

In a SageMaker Studio JupyterLab or Code Editor space, OpenCode automatically uses the space's temporary AWS role credentials. Do not run `aws configure` inside the Studio space and do not generate an API token.

In an AWS WorkSpace, first complete the AWS CLI setup in [Connecting to your team's SageMaker Studio](index.md#connecting-to-your-teams-sagemaker-studio). OpenCode uses the same credentials stored for the AWS CLI.

If `aws sts get-caller-identity` fails, resolve the AWS login or credential problem before installing OpenCode.

## Install OpenCode

Install the latest OpenCode release:

```bash
curl -fsSL https://opencode.ai/install | bash
```

Alternatively, if Node.js and npm are installed:

```bash
npm install -g opencode-ai@latest
```

Open a new terminal if requested by the installer, then verify the installation:

```bash
opencode --version
```

## Configure the DTC model

Create the global OpenCode configuration directory:

```bash
mkdir -p ~/.config/opencode
```

Create `~/.config/opencode/opencode.json` with the following contents. Replace `<TEAM_INFERENCE_PROFILE_ARN>` with the complete ARN provided by your team PI.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "amazon-bedrock/anthropic-claude-opus-4.8-dtc",
  "provider": {
    "amazon-bedrock": {
      "options": {
        "region": "us-east-1"
      },
      "models": {
        "anthropic-claude-opus-4.8-dtc": {
          "name": "DTC Claude Opus 4.8",
          "id": "<TEAM_INFERENCE_PROFILE_ARN>"
        }
      }
    }
  }
}
```

Keep the provider, region, model key, and model name exactly as shown. Only replace the inference profile ARN.

You can instead place the same configuration in an `opencode.json` file at the root of a specific project. A project configuration applies only when OpenCode is run in that project.

## Start OpenCode

Change to your project directory and launch OpenCode:

```bash
cd /path/to/your/project
opencode
```

Do not run OpenCode's `/connect` command for Amazon Bedrock. Authentication is supplied by your SageMaker role or configured AWS CLI credentials, and the model is already selected by `opencode.json`.

## Troubleshooting

### The model identifier is invalid

Confirm that you copied the complete application inference profile ARN from your PI, including the `application-inference-profile/` portion, and that the configured region is `us-east-1`.

You can ask AWS to resolve the profile directly:

```bash
aws bedrock get-inference-profile \
  --region us-east-1 \
  --inference-profile-identifier '<TEAM_INFERENCE_PROFILE_ARN>'
```

An `AccessDeniedException` from this diagnostic command can also mean your role is not permitted to inspect the profile. Contact your team PI if the ARN is correct but OpenCode still cannot invoke it.

### Access denied when invoking the model

Make sure you are using your own team's inference profile. IAM policies restrict each team to its assigned profile. Contact your team PI if the correct profile still returns an access-denied error.

### AWS Marketplace subscription error

If the error mentions `aws-marketplace:ViewSubscriptions` or `aws-marketplace:Subscribe`, the Claude model subscription has not been fully enabled for the team's AWS account. Team members should not add Marketplace permissions or select a different model. Send the error to your team PI so the account administrator can complete model activation.

### OpenCode uses unexpected credentials

A Bedrock bearer token takes precedence over the normal AWS credential chain. Check whether one is set:

```bash
printenv AWS_BEARER_TOKEN_BEDROCK
```

If the command prints a value, remove it for the current terminal and delete the corresponding export from your shell startup file:

```bash
unset AWS_BEARER_TOKEN_BEDROCK
```

Also remove any previously saved Amazon Bedrock credential from OpenCode authentication. OpenCode should use only the AWS identity reported by `aws sts get-caller-identity`.

### An older OpenCode version ignores the profile ARN

Update OpenCode and check the installed version:

```bash
npm install -g opencode-ai@latest
opencode --version
```

## Security considerations

OpenCode can read and edit project files and run terminal commands as your current user. In SageMaker Studio, AWS commands run with the permissions of the space's execution role. Review commands before approving them, do not place AWS credentials in prompts or project files, and do not commit `opencode.json` if it contains team-specific information that should remain outside the repository.
