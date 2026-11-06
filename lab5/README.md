# Lab 5

Instructions for this section will be provided in class and on Blackboard when we reach it.

Put your work for Lab 5 in this folder.

## Why for_each here and count there

The public subnets use for_each because each subnet has a stable key.
This makes the configuration easier to maintain. Adding a subnet with
a new key creates only that subnet. Removing a subnet destroys only
the subnet associated with that key.

Using count for subnets would assign numeric indexes such as [0], [1],
and [2]. Removing a subnet in the middle could shift the indexes and
cause Terraform to replace resources unnecessarily.

The optional S3 artifact bucket uses count because it is a single
optional resource. It is either created once or not created at all.

In short: use count when the question is how many, and for_each when
the question is which ones.

## Why not plan and apply against real AWS on every pull request

Running Terraform against real AWS for every pull request can create
unnecessary costs. Multiple students running repeated checks could
consume the fixed $50 lab budget.

Real AWS credentials would also be required in GitHub Actions. AWS
Academy credentials expire, which can cause unreliable CI checks.
Using long-lived credentials in GitHub would introduce security risks.

LocalStack allows the workflow to test Terraform configuration
without provisioning paid AWS resources. It makes the checks faster
and avoids unnecessary AWS API calls.

However, LocalStack does not prove that every configuration will work
in real AWS. For example, it does not fully verify IAM permissions or
whether EC2 user data successfully installs and starts Nginx.

## Experiments

### Experiment 1: Disable the optional artifact bucket

**Prediction:** Setting `create_artifact_bucket=false` would prevent Terraform
from creating the optional S3 bucket. The plan would contain 9 resources
instead of 10.

**Action:** Ran
`terraform plan -var-file=localstack.tfvars -var='create_artifact_bucket=false'`.

**Result:** Terraform reported 9 to add, 0 to change, and 0 to destroy.

**Explanation:** The count condition on the S3 bucket evaluated to zero,
so Terraform excluded that resource from the plan. The other 9 resources
remained in the configuration.

### Experiment 2: Change the EC2 instance type

**Prediction:** Changing the instance type to `t3.small` would select the
alternative allowed EC2 size.

**Action:** Ran
`terraform plan -var-file=localstack.tfvars -var='instance_type=t3.small'`.

**Result:** Terraform reported 10 to add, 0 to change, and 0 to destroy.

**Explanation:** The plan included all 10 resources because the previous
infrastructure had already been destroyed. The alternative instance type
was accepted by the variable validation. This was a plan-only experiment;
no resources were created.
