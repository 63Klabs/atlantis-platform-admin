# Account-Wide Set-Up Stack Deployment

This guide covers deploying and updating the two account-level CloudFormation stacks that establish shared infrastructure for Atlantis-managed projects. These stacks are deployed once per account (or once per prefix/region) and provide the IAM roles, policies, connections, and configurations consumed by individual project pipelines.

All templates and modules are sourced from S3. You do not need a local clone of the template repository.

## Prerequisites

- AWS CLI v2 installed and configured with appropriate credentials
- Access to the S3 bucket containing Atlantis templates (`s3://63klabs/atlantis/templates/v2/`)
- IAM permissions to create CloudFormation stacks, IAM roles, and IAM policies
- For the project pipeline stack: permissions to create CodeConnections and API Gateway Account resources (if using those features)
- SAM Config repo already set up
  - Prefix, Role Path, Service Role Path, Permissions Boundary information

## S3 Template and Module Locations

All references below use the following `S3ModuleLocation`:

```
63klabs/atlantis
```

63Klabs provides public access the `63klabs` bucket with the `atlantis` namespace to allow organizations to get up and running.

## Using JSON Parameter and Tag Files

Rather than passing parameters inline, store your configuration in JSON files within your management repository. This keeps CLI commands short, makes configuration reviewable in version control, and allows reuse across create and update operations.

It is advisable you get your parameter and tag files together before moving on to the CLI commands.

### Parameter File Format

Create a file named `params-account-wide-infrastructure.json` and fill in each parameter with your own `ParameterValue`:

```json
[
  { "ParameterKey": "OrgPrefix", "ParameterValue": "ACMECO" },
  { "ParameterKey": "RolePath", "ParameterValue": "/sam-app/" },
  { "ParameterKey": "GitHubOrg", "ParameterValue": "" },
  { "ParameterKey": "EnableApiGwCloudWatchLogs", "ParameterValue": "true" },
  { "ParameterKey": "S3ModuleLocation", "ParameterValue": "63klabs/atlantis" }
]
```

> **NOTE:** `OrgPrefix` is used to distinguish account-wide resources created by the Platform team, typically UPPER case with some resemblance of an organization or account name. This will make them stand out in IAM Role/Policy and CloudFormation stack listings. They should NOT be the same as any `Prefix` you will be assigning. They CAN be the same as the `S3BucketNameOrgPrefix` (if using). An upper case `OrgPrefix` is STRONGLY encouraged.

To skip the GitHub connection, set `GitHubOrg` to `""`. To skip API Gateway logging, set `EnableApiGwCloudWatchLogs` to `"false"`. The module URLs are still required even when the features are disabled (CloudFormation fetches the snippets but the resources inside are conditionally created).

Create a file named `params-prefix-based-infrastructure.json` and fill in each parameter with your own `ParameterValue`:

```json
[
  { "ParameterKey": "Prefix", "ParameterValue": "acme" },
  { "ParameterKey": "PrefixUpper", "ParameterValue": "ACME" },
  { "ParameterKey": "S3BucketNameOrgPrefix", "ParameterValue": "" },
  { "ParameterKey": "ServiceRolePath", "ParameterValue": "/sam-svc/" },
  { "ParameterKey": "RolePath", "ParameterValue": "/sam-app/" },
  { "ParameterKey": "PermissionsBoundaryArn", "ParameterValue": "" },
  { "ParameterKey": "GroupNames", "ParameterValue": "" },
  { "ParameterKey": "RoleNames", "ParameterValue": "" },
  { "ParameterKey": "UserNames", "ParameterValue": "" },
  { "ParameterKey": "S3ModuleLocation", "ParameterValue": "63klabs/atlantis" }
]
```

### Tag File Format

Create a file named `tags-account.json` and provide your own tag `Key/Value` pairs:

```json
[
  { "Key": "Department", "Value": "Platform Engineering" },
  { "Key": "CostCenter", "Value": "7G" },
  { "Key": "Owner", "Value": "platform-team" }
]
```

### Suggested Management Repo Structure

```
my-account-management/
├── commands.md
├── tags-account.json
├── account/
│   └── params-account-wide-infrastructure.json
├── acme/
│   └── params-prefix-based-infrastructure.json
├── xlab/
│   ├── params-prefix-based-infrastructure.json
│   └── tags.json # optional xlab specific tagging
└── README.md
```

Each directory corresponds to a prefix and contains the parameter and tag files for that deployment.

---

## Stack 1: Atlantis Account-Wide Infrastructure

This stack creates account-wide, ABAC-scoped managed policies and shared resources (GitHub connections, API Gateway logging) for project pipelines. Deploy one stack per account (not per prefix).

### Stack Name Convention

```
<OrgPrefix>-Atlantis-Account-Wide-Infra
```

Example: `ACMECO-Atlantis-Account-Wide-Infra`

### Create Stack

```bash
aws cloudformation create-stack \
  --stack-name ACMECO-Atlantis-Account-Wide-Infra \
  --template-url https://s3.amazonaws.com/63klabs/atlantis/templates/v2/account/account-wide-infrastructure.yml \
  --parameters file://account/params-account-wide-infrastructure.json \
  --tags file://tags-account.json \
  --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
```

### Update Stack

```bash
aws cloudformation update-stack \
  --stack-name ACMECO-Atlantis-Account-Wide-Infra \
  --template-url https://s3.amazonaws.com/63klabs/atlantis/templates/v2/account/account-wide-infrastructure.yml \
  --parameters file://account/params-account-wide-infrastructure.json \
  --tags file://tags-account.json \
  --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
```

To update only specific parameters while keeping others unchanged, use `UsePreviousValue`:

```json
[
  { "ParameterKey": "OrgPrefix", "UsePreviousValue": true },
  { "ParameterKey": "RolePath", "ParameterValue": "/sam-app/" },
  { "ParameterKey": "GitHubOrg", "UsePreviousValue": true },
  { "ParameterKey": "EnableApiGwCloudWatchLogs", "UsePreviousValue": true },
  { "ParameterKey": "S3ModuleLocation", "UsePreviousValue": true }
]
```

### What Gets Created

| Resource | Condition | Description |
|----------|-----------|-------------|
| CodeBuild CRUD Managed Policy | Always | ABAC-scoped policy for CodeBuild and /aws/codebuild/ logs |
| Cognito CRUD Managed Policy | Always | ABAC-scoped policy for Cognito User Pool management |
| GitHub Connection | GitHubOrg is not empty | CodeConnections connection to GitHub (requires manual completion) |
| API Gateway CloudWatch Role | EnableApiGwCloudWatchLogs is true | IAM role + API Gateway Account config for CloudWatch logging |

### Post-Deployment: GitHub Connection

If you provided a `GitHubOrg`, the connection will be in a **PENDING** state. To activate it:

1. Open the CodeConnections console URL from the stack outputs
2. Find your connection and click "Update pending connection"
3. Authorize AWS to access your GitHub organization
4. The user completing this step must have admin permissions in the GitHub organization

### Key Outputs

| Output | Export Name Pattern | Condition |
|--------|-------------------|-----------|
| CodeBuild CRUD Policy ARN | `<ORG>-ProjectPipeline-CodeBuildCrud-Arn` | Always |
| Cognito CRUD Policy ARN | `<ORG>-ProjectPipeline-CognitoCrud-Arn` | Always |
| GitHub Connection ARN | `<ORG>-GitHub-Connection-Arn` | GitHubOrg provided |
| API GW CloudWatch Role ARN | `<ORG>-ApiGateway-CloudWatch-Role-Arn` | Logging enabled |

---

## Stack 2: Prefix-Based Infrastructure

This stack creates prefix-based resources and scoped IAM service roles and managed policies for Pipeline, Storage, and Network management. Deploy one stack per prefix per account.

This can be used as a more comprehensive method compared to configuring and deploying each service-role independently using `config.py/deploy.py service-role ${prefix} ${pipeline|storage|network}`

> NOTE: If you use this method of deployment you DO NOT need to configure and deploy individual service roles for the prefix.

### Stack Name Convention

```
<PrefixUpper>-Atlantis-Prefix-Based-Infra
```

Examples:
- `ACME-Atlantis-Prefix-Based-Infra`
- `XLAB-Atlantis-Prefix-Based-Infra`

### Create Stack

```bash
aws cloudformation create-stack \
  --stack-name ACME-Atlantis-Prefix-Based-Infra \
  --template-url https://s3.amazonaws.com/63klabs/atlantis/templates/v2/account/prefix-based-infrastructure.yml \
  --parameters file://acme/params-prefix-based-infrastructure.json \
  --tags file://tags-account.json \
  --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
```

### Update Stack

```bash
aws cloudformation update-stack \
  --stack-name ACME-Atlantis-Prefix-Based-Infra \
  --template-url https://s3.amazonaws.com/63klabs/atlantis/templates/v2/account/prefix-based-infrastructure.yml \
  --parameters file://acme/params-prefix-based-infrastructure.json \
  --tags file://tags-account.json \
  --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
```

To update only specific parameters while keeping others unchanged, use `UsePreviousValue`:

```json
[
  { "ParameterKey": "Prefix", "UsePreviousValue": true },
  { "ParameterKey": "PrefixUpper", "UsePreviousValue": true },
  { "ParameterKey": "S3BucketNameOrgPrefix", "UsePreviousValue": true },
  { "ParameterKey": "ServiceRolePath", "UsePreviousValue": true },
  { "ParameterKey": "RolePath", "UsePreviousValue": true },
  { "ParameterKey": "PermissionsBoundaryArn", "ParameterValue": "arn:aws:iam::123456789012:policy/MyBoundary" },
  { "ParameterKey": "GroupNames", "UsePreviousValue": true },
  { "ParameterKey": "RoleNames", "UsePreviousValue": true },
  { "ParameterKey": "UserNames", "UsePreviousValue": true },
  { "ParameterKey": "S3ModuleLocation", "UsePreviousValue": true }
]
```

### What Gets Created

| Resource | Description |
|----------|-------------|
| Pipeline Management Service Role | CloudFormation service role for deploying pipeline stacks |
| Pipeline Management Managed Policy | PassRole policy for the pipeline service role |
| Storage Management Service Role | CloudFormation service role for deploying storage stacks |
| Storage Management Managed Policy | PassRole policy for the storage service role |
| Network CloudFront Management Policy | Managed policy for CloudFront, OAC, cache policies, API GW domains |
| Network Route53 Management Policy | Managed policy for Route53 DNS record management |
| Network CloudFront Management Role | Developer service role (CloudFront only, no Route53) |
| Network CloudFront Management Managed Policy | PassRole policy for the CloudFront-only role |
| Network Full Management Role | Full access service role (CloudFront + Route53) |
| Network Full Management Managed Policy | PassRole policy for the full network role |

### Key Outputs

| Output | Export Name Pattern |
|--------|-------------------|
| Pipeline Service Role ARN | `<PREFIX>-CloudFormation-Pipeline-Mgmt-Service-Role-Arn` |
| Storage Service Role ARN | `<PREFIX>-CloudFormation-Storage-Mgmt-Service-Role-Arn` |
| Network CloudFront Role ARN | `<PREFIX>-CloudFormation-Network-CloudFront-Mgmt-Service-Role-Arn` |
| Network Full Role ARN | `<PREFIX>-CloudFormation-Network-Full-Mgmt-Service-Role-Arn` |

---

## Monitoring Stack Operations

### Wait for Completion

```bash
aws cloudformation wait stack-create-complete \
  --stack-name ACME-Atlantis-Prefix-Based-Infra
```

### Check Stack Status

```bash
aws cloudformation describe-stacks \
  --stack-name ACME-Atlantis-Prefix-Based-Infra \
  --query "Stacks[0].StackStatus" \
  --output text
```

### View Stack Outputs

```bash
aws cloudformation describe-stacks \
  --stack-name ACME-Atlantis-Prefix-Based-Infra \
  --query "Stacks[0].Outputs" \
  --output table
```

### View Stack Events (for troubleshooting)

```bash
aws cloudformation describe-stack-events \
  --stack-name ACME-Atlantis-Prefix-Based-Infra \
  --query "StackEvents[?ResourceStatus=='CREATE_FAILED' || ResourceStatus=='UPDATE_FAILED']" \
  --output table
```

---

## Deployment Order

When setting up a new account:

1. Deploy **Account-Wide Infrastructure** stack first (one per account)
1. Deploy **Prefix-Based Infrastructure** stack second (one per prefix)

When updating, either stack can be updated independently. The Account-Wide Infrastructure stack does not depend on any Prefix-Based Infrastructure stack or vice versa.

---

## Deleting Stacks

Before deleting, verify no other stacks reference the exported values:

```bash
aws cloudformation list-imports \
  --export-name ACME-CloudFormation-Pipeline-Mgmt-Service-Role-Arn
```

There may be multiple exported values per stack. Check each one.

If no imports are returned, the stack can be safely deleted:

```bash
aws cloudformation delete-stack \
  --stack-name ACME-Atlantis-Prefix-Based-Infra
```

> **Note:** The GitHub connection resource has `DeletionPolicy: Retain` and will not be deleted when the project pipeline stack is deleted. You must manually delete it from the CodeConnections console if needed.
