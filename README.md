# ecr-public-action
`ecr-public-action` is a Github Action that allows you to docker `build`, `tag` and `publish` to **Amazon ECR Public** in the Github Action workflow.


# sample

```yaml
- name: Build and Push to ECR public
  id: build-and-push
  uses: pahud/ecr-public-action@12e969f
  with:
    tags: |
      public.ecr.aws/d7p2r8s3/${{ steps.repoName.outputs.reponame }}:latest
      public.ecr.aws/d7p2r8s3/${{ steps.repoName.outputs.reponame }}:${{ steps.sha.outputs.sha7 }}
```

# Full Sample

```yaml
ecr_public:
  name: ECR Public
  runs-on: ubuntu-latest
  steps:
    - name: Get repo name
      id: repoName
      run: echo "::set-output name=reponame::$(echo ${{github.repository}} | cut -d '/' -f 2)"
    - name: Get short SHA
      id: sha
      run: echo "::set-output name=sha7::$(echo ${GITHUB_SHA} | cut -c1-7)"
    - name: Checkout
      uses: actions/checkout@v2
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    - name: Build and Push to ECR public
      id: build-and-push
      uses: pahud/ecr-public-action@12e969f
      with:
        context: .devcontainer
        dockerfile: ./.devcontainer/Dockerfile
        tags: |
          public.ecr.aws/d7p2r8s3/${{ steps.repoName.outputs.reponame }}:latest
          public.ecr.aws/d7p2r8s3/${{ steps.repoName.outputs.reponame }}:${{ steps.sha.outputs.sha7 }}
```


# IAM User and Policies

Create an AWS IAM User with `AmazonElasticContainerRegistryPublicPowerUser` managed policy only and configure its `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in the github secrets for this repository. 


## To manage single repository only

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ManageRepositoryContents",
            "Effect": "Allow",
            "Action": [
                "ecr-public:DescribeImageTags",
                "ecr-public:DescribeImages",
                "ecr-public:InitiateLayerUpload",
                "ecr-public:DescribeRepositories",
                "ecr-public:UploadLayerPart",
                "ecr-public:PutImage",
                "ecr-public:CompleteLayerUpload",
                "ecr-public:GetRepositoryPolicy",
                "ecr-public:BatchCheckLayerAvailability"
            ],
            "Resource": "arn:aws:ecr-public::903779448426:repository/my-repo"
        },
        {
            "Sid": "GetAuthorizationToken",
            "Effect": "Allow",
            "Action": [
                "ecr-public:GetAuthorizationToken",
                "sts:GetServiceBearerToken"
            ],
            "Resource": "*"
        }
    ]
}
```

See [Amazon ECR Public managed IAM policies](https://docs.aws.amazon.com/AmazonECR/latest/public/public-ecr-managed-policies.html) and [Amazon ECR identity-based policy examples](https://docs.aws.amazon.com/AmazonECR/latest/public/security_iam_id-based-policy-examples.html) for more details.


