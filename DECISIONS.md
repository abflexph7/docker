What we chose: aws-actions/configure-aws-credentials with role-to-assume

What the alternative was: storing AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY as GitHub secrets

Why we chose this: access keys are long-lived credentials — if the repo is compromised, an attacker has permanent AWS access. An OIDC token is valid for the duration of one pipeline run only. After the run, the token is useless.

Trade-off: requires a one-time IAM OIDC provider and role setup.

What we chose: docker tag myapp:$SHA where SHA is the 7-character commit hash

What the alternative was: docker push myapp:latest on every build

Why we chose this: :latest is ambiguous — you cannot tell which code is running. With abc1234 you can run git show abc1234 and see exactly what code produced that image. This is essential for rollbacks and incident investigations.

Trade-off: ECR accumulates images over time; add a lifecycle policy to expire images older than 30 days.

What we chose: exit-code: "1" on the Trivy scan step

What the alternative was: scan but continue (exit-code: "0"), or not scan at all

Why we chose this: a vulnerable image that reaches ECR can reach production. Failing the pipeline forces the issue to be fixed before deployment. The cost is a slightly longer build cycle.

Trade-off: a critical CVE in a dependency can block deployments until patched. Mitigation: use --ignore-unfixed to skip CVEs with no available fix.