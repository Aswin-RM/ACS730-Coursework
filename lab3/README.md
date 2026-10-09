# Lab 3

A real AWS account would use GitHub Actions OIDC instead of stored access keys as OIDC allows GitHub to request short-lived AWS credentials through
an IAM role without keeping long-term keys in GitHub Secrets. 

This course uses session-scoped secrets because the Academy lab environment restricts IAM role
and OIDC setup. The credential's limited lifetime and the permissions of the associated role restrict the damage. Although an attacker could
still use them until they expire or are revoked.
