# AI4NG IAM Roles

Two-stage IAM role deployment to avoid circular dependencies.

## Deployment Order

### 1. Deploy CI/CD Roles FIRST
```bash
aws cloudformation create-stack \
  --stack-name AI4NG-CICD-Roles \
  --template-body file://ci-cd-roles.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=ProjectName,ParameterValue=AI4NG
```

### 2. Deploy Application Roles SECOND
```bash
aws cloudformation create-stack \
  --stack-name AI4NG-App-Roles \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=ProjectName,ParameterValue=AI4NG
```

## Why Two Templates?

- **ci-cd-roles.yaml**: CodePipeline, CodeBuild, CloudFormation execution roles
- **template.yaml**: Lambda, ECS, Step Functions, Cognito roles

This prevents circular dependencies where pipelines need roles to deploy the resources that create those same roles.

## Usage in Pipelines

```yaml
# Use CI/CD roles in pipeline templates
RoleArn: 
  Fn::ImportValue: AI4NG-CodePipelineServiceRole-Arn
```