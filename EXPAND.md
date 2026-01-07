# ===============================================================
# Multi-Region and Multi-Account Expansion Instructions
# Additive Changes Only — No Refactor or Rewrite
# ===============================================================

# ===============================================================
# A. Multi-Region Expansion
# ===============================================================
# Files Impacted:
#   - provider.tf
#   - backend.tf
#   - terraform.tfvars (optional)

# ---------------------------------------------------------------
# 1. provider.tf — Add Region Aliases
# Keep your existing provider block. Add the following below it.

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

# ---------------------------------------------------------------
# 2. variables.tf — Add Secondary Region Variable

variable "secondary_region" {
  description = "Secondary AWS region for multi-region deployments"
  type        = string
  default     = null
}

# ---------------------------------------------------------------
# 3. Using the Secondary Region (Example)
# When expanding, resources are duplicated explicitly.

resource "aws_vpc" "secondary" {
  provider   = aws.secondary
  cidr_block = "10.1.0.0/16"

  tags = {
    Name = "${var.project_name}-${var.environment}-vpc-secondary"
  }
}

# ---------------------------------------------------------------
# 4. Backend State (Multi-Region)
# Each region must use its own state file.

# Example:
key = "production/us-west-2/terraform.tfstate"
# No shared state. No risk.

# ===============================================================
# B. Multi-Account Expansion
# ===============================================================
# Files Impacted:
#   - provider.tf
#   - backend.tf
#   - variables.tf

# ---------------------------------------------------------------
# 1. variables.tf — Add Account Role Variable

variable "target_account_role_arn" {
  description = "IAM role ARN for cross-account deployments"
  type        = string
  default     = null
}

# ---------------------------------------------------------------
# 2. provider.tf — Add Assume Role Support

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
# If target_account_role_arn is null, Terraform behaves exactly as before.

# ---------------------------------------------------------------
# 3. Backend State (Multi-Account)
# Each account must have its own bucket and DynamoDB table.

bucket         = "company-prod-terraform-state"
dynamodb_table = "terraform-state-locks-prod"
key            = "ecs/terraform.tfstate"
# No cross-account state sharing. Non-negotiable in enterprise.

# ===============================================================
# C. Architectural Rationale
# ===============================================================
# - Explicit providers prevent accidental cross-region drift
# - Separate state prevents blast-radius escalation
# - Assume-role avoids credential sprawl
# - No magic modules that hide complexity
# - Fully auditable for SOC2 / ISO / bank reviews
# This is exactly how enterprise VARs expect IaC to behave.

# ===============================================================
# D. What Is Not Included by Default (On Purpose)
# ===============================================================
# - No automatic region replication
# - No implicit account fan-out
# - No global state files
# - No opinionated enterprise guardrails
# These are upgrades, not defaults.

# ===============================================================
# Final Positioning Statement
# ===============================================================
# “This infrastructure is intentionally designed as an upgrade-as-you-scale platform.
# Startups deploy fast with a single region and account.
# Enterprises expand without rewrites.”
