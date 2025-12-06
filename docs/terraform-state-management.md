# Terraform State Management

This document explains how we manage Terraform state for the Multiplication Trainer infrastructure.

## Remote State with S3 and DynamoDB

We use AWS S3 to store our Terraform state files and DynamoDB for state locking to enable safe team collaboration.

### Key Components

1. **S3 Bucket**
   - Stores the actual state files
   - Versioning enabled to maintain history
   - Server-side encryption for security
   - Unique per AWS account

2. **DynamoDB Table**
   - Used for state locking
   - Prevents concurrent modifications
   - Ensures state consistency

### File Structure

- `backend.tf`: Contains the backend configuration
- `.gitignore`: Excludes local state files from version control

## Setup Instructions

### Prerequisites

- AWS CLI configured with appropriate permissions
- Terraform installed

### Initial Setup

1. **Update the backend configuration**
   - Open `backend.tf`
   - Update the `bucket` name if needed (must be globally unique)
   - Ensure the `region` matches your AWS region

2. **Initialize Terraform**
   ```bash
   terraform init
   ```

3. **Create S3 Bucket and DynamoDB Table**
   - If the S3 bucket and DynamoDB table don't exist:
     1. Uncomment the resource blocks in `backend.tf`
     2. Run:
        ```bash
        terraform apply -target=aws_s3_bucket.terraform_state \
                       -target=aws_s3_bucket_versioning.terraform_state \
                       -target=aws_s3_bucket_server_side_encryption_configuration.terraform_state \
                       -target=aws_dynamodb_table.terraform_locks
        ```
     3. After creation, re-comment the resources in `backend.tf`

4. **Migrate Existing State (if any)**
   ```bash
   terraform init -migrate-state
   ```
   - Type `yes` when prompted to confirm the migration

5. **Verify the Setup**
   ```bash
   terraform state list
   ```
   This should show your existing resources without any errors.

## Security Considerations

- The S3 bucket has server-side encryption enabled
- IAM policies should be configured to restrict access
- State files contain sensitive information and should never be committed to version control

## Best Practices

1. **Never commit `.tfstate` files**
   - These are excluded in `.gitignore`
   - State files may contain sensitive information

2. **Use workspaces for environments**
   - Different workspaces for dev, staging, prod
   - Each workspace maintains its own state

3. **Regularly backup state**
   - S3 versioning provides point-in-time recovery
   - Consider additional backup strategies for critical infrastructure

## Troubleshooting

### Common Issues

- **Access Denied**: Ensure your AWS credentials have the necessary permissions
- **State Locked**: If a state operation fails, check the DynamoDB table for stale locks
- **State Drift**: Use `terraform refresh` to sync state with actual infrastructure

### Recovery

If you need to recover from a corrupted state:
1. Check S3 version history for the state file
2. Download a previous version
3. Use `terraform state push` to restore

## Team Collaboration

- All team members should use the same backend configuration
- Always run `terraform init` after pulling changes that modify the backend
- Use `terraform state` commands to inspect and modify state when needed
