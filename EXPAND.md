## A. Multi-Region Expansion
To expand to a second region, you must define an additional provider instance and explicitly map resources to it. This prevents accidental resource creation in the wrong geographic location.

### 1. provider.tf — Add Region Aliases
Keep your existing provider block. Add the following below it to enable a secondary region.

''hcl
provider "aws" {
  alias  = "secondary"
  region = var.secondary_region

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "RUWANPURAGE PAVITHRA PARAMI RANASINGHE"
    }
  }
}

2. variables.tf — Add Secondary Region Variable
Define the variable that will drive the secondary provider configuration.

```hcl
variable "secondary_region" {
  description = "Secondary AWS region for multi-region deployments"
  type        = string
  default     = null
}

3. Using the Secondary Region (Example)
When expanding, resources are duplicated explicitly by referencing the secondary alias.

```hcl
resource "aws_vpc" "secondary" {
  provider   = aws.secondary
  cidr_block = "10.1.0.0/16"

  tags = {
    Name = "${var.project_name}-${var.environment}-vpc-secondary"
  }
}

4. Backend State (Multi-Region)
To minimize the blast radius, each region must use its own state file path. No shared state.
# Example logic for backend selection

```hcl
key = "production/us-west-2/terraform.tfstate"

B. Multi-Account Expansion
For security isolation, production workloads should live in dedicated AWS accounts accessed via IAM roles.
1. variables.tf — Add Account Role Variable
This allows the same code to target different accounts by passing in a different Role ARN.

```hcl
variable "target_account_role_arn" {
  description = "IAM role ARN for cross-account deployments"
  type        = string
  default     = null
}

2. provider.tf — Add Assume Role Support
Updating the default provider to support cross-account deployment. If target_account_role_arn is null, Terraform behaves exactly as before (local execution).

```hcl
provider "aws" {
  region = var.aws_region

  assume_role {
    role_arn = var.target_account_role_arn
  }

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "RUWANPURAGE PAVITHRA PARAMI RANASINGHE"
    }
  }
}

3. Backend State (Multi-Account)
Each account must have its own bucket and DynamoDB table. No cross-account state sharing is permitted in enterprise environments.

```hcl
bucket         = "company-prod-terraform-state"
dynamodb_table = "terraform-state-locks-prod"
key            = "ecs/terraform.tfstate"

C. Architectural Rationale
 * Explicit Providers: Prevents accidental cross-region drift.
 * Separate State: Prevents blast-radius escalation (one state failure won't kill the global footprint).
 * Assume-Role: Avoids credential sprawl and long-lived IAM keys.
 * No Magic: No hidden modules; fully auditable for SOC2 / ISO / bank reviews.

D. What Is Not Included (On Purpose)
These are considered upgrades to be implemented based on specific business needs, not defaults:
 * No automatic region replication.
 * No implicit account fan-out.
 * No global state files.
Final Positioning Statement
> “This infrastructure is intentionally designed as an upgrade-as-you-scale platform. Startups deploy fast with a single region and account. Enterprises expand without rewrites.”
