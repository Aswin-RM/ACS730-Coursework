# Lab 3

A real AWS account would use GitHub Actions OIDC instead of stored access keys as OIDC allows GitHub to request short-lived AWS credentials through
an IAM role without keeping long-term keys in GitHub Secrets. 

This course uses session-scoped secrets because the Academy lab environment restricts IAM role
and OIDC setup. The credential's limited lifetime and the permissions of the associated role restrict the damage. Although an attacker could
still use them until they expire or are revoked.

Terraform Version

This lab uses Terraform version 1.10.3

ExpiredToken on deployment: The AWS Academy lab session has ended, and the temporary credentials have expired. Start or resume a lab session, run 
./scripts/refresh-gha-creds.sh Aswin-RM/ACS730-Coursework from the repository root, and rerun the failed GitHub Actions job. No 
repository files need to change.

Input required and not supplied: aws-region: The GitHub repository variable AWS_REGION is missing. This is a configuration issue and not a credential 
problem. Run the credential refresh script or execute gh variable set AWS_REGION --body us-east-1 from the repository root.

 
## Experiments

1. Let the credentials die : 
Prediction: The GitHub Actions deployment workflow will fail when it attempts to authenticate when AWS uses expired temporary credentials.
The expected error is ExpiredToken.

Observation: After the AWS Academy session ended, the deployment failed because the temporary credentials are no longer valid.
After starting a new session and rerunning the refresh script and the job, the deployment succeeded without changing any repository files.

2. Give the apply step a pull request:
Prediction: Changing the apply condition to if: always() will make the apply step eligible to run even when a preceding step or job fails.If the workflow has valid AWS credentials and sufficient permissions, a pull request could potentially trigger Terraform apply and change AWS resources.
Observation: The changed condition made the apply step eligible to run for a pull request. This creates a security risk because changes proposed through a pull request could affect the environment.

