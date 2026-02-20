- 로컬에서 도커 이미지 빌드하면 CPU/메모리 부하가 심하게 걸려서(특히 멀티 아키텍처) 작성해둔 AWS ECR에 이미지를 빌드해서 푸시하는 Github Actions 워크플로우임
- 퍼블릭 레포지토리 한정 Actions 러너 무제한 무료이니 후배님들 Fork 하거나 Clone 하셔서 알아서 잘 쓰시면 됩니당~~

---

# GitHub Actions OIDC with AWS ECR

## AWS OIDC Provider

![image](images/image.png)

- Provider URL: `https://token.actions.githubusercontent.com`
- Audience: `sts.amazonaws.com`

![image](images/image-1.png)

## IAM Role (ECR)

- Role Name: `GitHubActionsECRRole`

### A) Web Identity

![image](images/image-2.png)

### B) Custom Trust Policy

- Role Type: `Custom trust policy`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:<ORG>/<REPO>:ref:refs/heads/main"
                }
            }
        }
    ]
}
```

- `REPO` should be replaced with `*`(wildcard) if you want to allow all Github repositories.

## IAM Policy (ECR)

- Policy Name: `GitHubActionsECRPolicy`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        { "Sid": "ECRAuth", "Effect": "Allow", "Action": ["ecr:GetAuthorizationToken"], "Resource": "*" },
        {
            "Sid": "ECRPushPull",
            "Effect": "Allow",
            "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:BatchGetImage",
                "ecr:CompleteLayerUpload",
                "ecr:DescribeRepositories",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
            ],
            "Resource": "arn:aws:ecr:<AWS_REGION>:<ACCOUNT_ID>:repository/<ECR_REPOSITORY>"
        }
    ]
}
```

- `ECR_REPOSITORY` should be replaced with `*`(wildcard) if you want to allow all ECR repositories.

## IAM Role Policy Attachment

![image](images/image-3.png)

## Github Repository Secrets

- `AWS_REGION`: `ap-northeast-2`
- `AWS_ACCOUNT_ID`: `<ACCOUNT_ID>`
- `AWS_ROLE_NAME`: `GitHubActionsECRRole`

# How to Use

![images](images/image-4.png)

![image](images/image-5.png)
