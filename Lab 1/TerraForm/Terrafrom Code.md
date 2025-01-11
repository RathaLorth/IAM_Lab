```

terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0" # Specifies the Azure AD provider and its version
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0" # Specifies the Azure Resource Manager provider and its version
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0" # Specifies the Random provider for generating secrets
    }
  }

  backend "local" {
    path = "terraform.tfstate" # Stores the Terraform state locally
  }

  required_version = ">= 1.3.0" # Ensures Terraform version compatibility
}

# Configures the Azure AD provider, which is used to manage Azure Active Directory resources
provider "azuread" {
  # Configuration options if needed
}

provider "azurerm" {
  features {} # Enables all necessary features for the Azure Resource Manager
  subscription_id = var.subscription_id # Uses the provided subscription ID
}

# Data Source to get current tenant ID
data "azurerm_client_config" "current" {}

# Declares variables for resource configurations
variable "resource_group_name" {
  description = "Name of the Azure Resource Group" # Description of the variable
  type        = string
  default     = "Cyber-Corp-Test-ResourceGroup" # Default value for the resource group name
}

variable "location" {
  description = "Azure region" # Location of the resource group
  type        = string
  default     = "East US" # Default Azure region
}

variable "environment" {
  description = "Deployment environment" # Environment tag for the deployment
  type        = string
  default     = "Test" # Default value indicating this is a test environment
}

variable "project_name" {
  description = "Project name" # Description of the project
  type        = string
  default     = "Cyber Corp Lab" # Default project name
}

variable "config_file" {
  description = "Path to the configuration JSON file" # Path to the configuration file
  type        = string
  default     = "config.json"

  validation {
    condition     = can(jsondecode(file(var.config_file))) # Validates that the config file is valid JSON
    error_message = "Configuration file missing or invalid. Please check config.json." # Error message for invalid config
  }
}

# Reads and processes configuration data from a JSON file
locals {
  config = try(jsondecode(file(var.config_file)), {})

  users           = lookup(local.config, "users", [])
  groups          = lookup(local.config, "groups", {})
  memberships     = flatten([
    for group, members in lookup(local.config, "memberships", {}) : [
      for member in members : {
        group  = group,
        member = member
      }
    ]
  ])
  role_assignments = lookup(local.config, "role_assignments", [])
}

# Creates an Azure Resource Group
resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location

  tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}

# Generates secure passwords for users
resource "random_password" "user_password" {
  for_each = { for user in local.users : user.email => user } # Creates a password for each user
  length   = 16 # Specifies the password length
  special  = true # Includes special characters in the password
}

# Creates Azure AD users from the configuration
resource "azuread_user" "users" {
  for_each = { for user in local.users : user.email => user } # Creates a resource for each user

  user_principal_name   = each.value.email # Sets the user's email as the principal name
  display_name          = each.value.name # User's display name
  mail_nickname         = lower(replace(each.value.name, " ", "")) # Generates a mail nickname
  mail                  = each.value.email # User's email address
  password              = random_password.user_password[each.key].result # Assigns the generated password
  force_password_change = true # Requires the user to change their password on first login
  department            = lookup(each.value, "department", null) # Optional department field
  job_title             = lookup(each.value, "jobTitle", null) # Optional job title field
}

# Creates Azure AD groups from the configuration
resource "azuread_group" "groups" {
  for_each = local.groups # Creates a group resource for each group in the config

  display_name       = each.value.displayName # Group's display name
  mail_nickname      = lower(replace(each.value.displayName, " ", "")) # Generates a mail nickname
  security_enabled   = lookup(each.value, "security_enabled", true) # Enables security features
  assignable_to_role = each.value.assignable_to_role # Allows role assignments based on config
}

# Maps users to groups based on the configuration
resource "azuread_group_member" "group_members" {
  for_each = { for pair in local.memberships : "${pair.group}-${pair.member}" => pair } # Maps users to groups

  group_object_id  = azuread_group.groups[each.value.group].id # Group's object ID
  member_object_id = azuread_user.users[each.value.member].id # User's object ID
}

# Assigns roles to groups
resource "azuread_directory_role_assignment" "role_assignments" {
  for_each = { for assignment in local.role_assignments : "${assignment.group}-${assignment.role_id}" => assignment } # Assigns roles to groups

  role_id             = each.value.role_id # Role ID being assigned
  principal_object_id = azuread_group.groups[each.value.group].id # Object ID of the group receiving the role
}

# Outputs the resource group name
output "resource_group_name" {
  description = "The name of the Azure Resource Group." # Outputs the resource group name
  value       = azurerm_resource_group.rg.name # Value of the resource group name
}

# Outputs a list of user principal names
output "user_principal_names" {
  description = "List of user principal names." # Outputs a list of user principal names
  value       = [for user in azuread_user.users : user.user_principal_name] # Generates the list from user resources
}

# Outputs group names mapped to their object IDs
output "group_object_ids" {
  description = "Mapping of group names to their object IDs." # Outputs group names mapped to their IDs
  value       = { for group_name, group in azuread_group.groups : group_name => group.id } # Creates the mapping
}

# Outputs group memberships
output "group_memberships" {
  description = "Mapping of groups to their members." # Outputs group memberships
  value       = {
    for group, members in local.memberships : group => [for member in members : member] # Maps group memberships
  }
}

# Outputs role assignments
output "role_assignments" {
  description = "Mapping of role IDs to principal object IDs." # Outputs role assignments
  value       = {
    for role_id in distinct([for ra in azuread_directory_role_assignment.role_assignments : ra.role_id]) :
    role_id => [
      for ra in azuread_directory_role_assignment.role_assignments :
      ra.principal_object_id if ra.role_id == role_id # Maps roles to object IDs
    ]
  }
}

# Outputs the subscription ID
output "subscription_id" {
  description = "Azure subscription ID." # Outputs the subscription ID
  value       = var.subscription_id # Value of the subscription ID
}
