# Lab 6: Configuration Management and Golden AMIs

## 1. Overview

This lab demonstrates automated EC2 configuration using Terraform, Ansible, AWS Systems Manager (SSM), and Packer.

The infrastructure contains two Amazon Linux 2023 web server instances. Terraform provisions the infrastructure, Ansible configures the servers, and Packer creates a golden AMI with the application already installed and configured.

GitHub Actions automates the Terraform and Ansible workflow.

## 2. Infrastructure and Configuration

Terraform provisions the EC2 instances and their security group.

Ansible connects to the instances through AWS Systems Manager Session Manager. This avoids the need to manage long-lived SSH private keys or expose SSH access to the internet.

The dynamic AWS EC2 inventory discovers instances using their Lab and Role tags. The Ansible playbook applies the webserver role to the discovered instances.

The role:
- Installs the required packages.
- Creates an unprivileged service account.
- Creates the application directory.
- Deploys the web page from a Jinja2 template.
- Installs and enables the systemd service.
- Starts the web application.

The SSM connection requires the SSM agent, suitable IAM permissions, network connectivity to Systems Manager, and an S3 bucket for Ansible's SSM file transfers.

The S3 bucket used in this lab is `acs730-lab6-ssm-983639779985` in the AWS account's `us-east-1` region.

## 3. Idempotency

Ansible is designed to be idempotent. Running the playbook repeatedly should not make unnecessary changes when the target system already matches the desired configuration.

The second Ansible run reported:

- `changed=0`
- `failed=0`
- `unreachable=0`

This confirms that the configuration was already in the desired state and that the second run did not need to make changes.

## 4. Boot-to-Ready Comparison

I measured the time from launching an EC2 instance until its web page became available.

| Configuration method | Boot-to-ready time |
|---|---:|
| Ansible configuration after boot | 111 seconds |
| Packer golden AMI | 27 seconds |
| Time saved with golden AMI | 84 seconds |

The golden AMI reduced boot-to-ready time by approximately 76%.

The Ansible approach installs and configures the application after the instance starts. The golden AMI already contains the configured application and service, so it can serve the web page sooner.

I verified that the golden AMI's main page displayed the expected ACS730 Lab 6 content. The Packer-baked `/baked.html` page also returned HTTP 200.

Evidence:
- `evidence/boot-to-ready-ansible.txt`
- `evidence/boot-to-ready-golden.txt`
- `evidence/packer-build.txt`

## 5. Packer Golden AMI

Packer creates an Amazon Machine Image with the web application and its configuration already installed.

The AMI created for this lab was:

`ami-0301c975c4cf96509`

The Packer build uses Ansible to configure the image and then copies an additional HTML page into the image.

When an EC2 instance launches from this AMI, it does not need to repeat the full Ansible configuration process.

## 6. Experiments and Observations

### Experiment 1: Ansible idempotency

**Prediction:** Running the same Ansible playbook a second time should report no changes if the server already matches the desired configuration.

**Result:** The second run reported `changed=0`, `failed=0`, and `unreachable=0`.

**Explanation:** Ansible's declarative modules check the current system state before making changes. Since the packages, account, files, and service were already configured correctly, Ansible did not need to modify them.

### Experiment 2: Ansible configuration versus a golden AMI

**Prediction:** An instance launched from a preconfigured golden AMI should become ready faster than an instance that requires Ansible configuration after boot.

**Result:** The Ansible-configured instance took 111 seconds, while the golden AMI instance took 27 seconds.

**Explanation:** The golden AMI moves the configuration work to image-building time. The application is already installed when the instance launches, reducing the time required to serve the web page.

## 7. Cost Considerations

A golden AMI reduces configuration time during instance launches, but it does not eliminate costs.

The AMI is backed by EBS snapshots. Deregistering an AMI does not automatically delete its associated snapshots. Those snapshots can continue to incur storage charges.

Other resources that can incur costs include:
- Running EC2 instances.
- EBS volumes.
- EBS snapshots.
- S3 storage used for Ansible SSM transfers.

The golden AMI approach shifts some work and cost to image creation and storage rather than removing all costs.

## 8. Risks and Limitations

### Ansible configuration

- Configuration depends on the instance being reachable through SSM.
- Ansible must run successfully after the instance starts.
- Configuration changes can increase deployment time.
- Incorrect playbook changes can affect multiple instances.

### Packer golden AMI

- The image can become outdated when application code or configuration changes.
- Images must be rebuilt and tested after updates.
- Old AMIs and snapshots can accumulate storage costs.
- A faulty image can reproduce the same problem across every instance launched from it.

## 9. Recommendation

For environments that frequently launch identical web servers and require fast startup, I would recommend a tested Packer golden AMI. For environments where configuration changes frequently or servers need ongoing configuration management, Ansible is more flexible. The two tools can also be used together, with Ansible configuring the image during the Packer build.

## 10. GitHub Actions Automation

The `lab6-configure.yml` workflow automates the Terraform and Ansible process.

It:
- Checks out the repository.
- Installs Ansible and the required AWS collections.
- Installs the SSM Session Manager plugin.
- Configures AWS credentials.
- Initializes Terraform.
- Imports the existing Lab 6 resources into the workflow's Terraform state.
- Runs Terraform plan and apply.
- Uses Ansible to configure the EC2 instances through SSM.

The workflow completed successfully on GitHub Actions.

## 11. Evidence

The `evidence` directory contains the results collected during the lab, including:

- Ansible idempotency output.
- Ansible boot-to-ready measurement.
- Golden AMI boot-to-ready measurement.
- Packer build output.
- Golden AMI screenshot.
- Teardown evidence.

