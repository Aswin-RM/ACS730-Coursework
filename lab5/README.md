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


#Experiment 1: Delete the depends_on.

- Prediction: Removing the explicit depends_on argument should not change the number of resources in the Terraform plan because Terraform can identify dependencies through resource references.

- Conclusion: Terraform's dependency graph allows the EC2 instance to start before the route table association is ready. This cause problems when the instance tries to access the internet during startup. LocalStack does not fully reproduce real AWS networking behaviour, so this issue is not visible during local testing. Keeping depends_on helps ensure the network association is completed before the instance is created.

#Experiment 2:  Break only the formatting. 

- Prediction: Adding three spaces before a closing brace in network.tf should cause the terraform fmt -check -recursive step to fail. The LocalStack job should not run because it depends on the formatting and validation job.

- Conclusion: Formatting checks help catch configuration issues early in the CI pipeline. Running terraform fmt -recursive restores the correct formatting. This prevents unnecessary LocalStack testing when the formatting gate has already failed.
