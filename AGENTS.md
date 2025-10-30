# AI Agent Instructions for Release Engine Configuration Repository

## 🏗️ Repository Overview

This repository serves as the **Configuration Layer (Facade)** in the Release Engine three-tier solution. It provides a simple, user-friendly interface for configuring and deploying workloads while hiding the underlying pipeline complexity.

## Repository Role in Release Engine Solution

### Three-Tier Architecture
```text
Configuration Layer (THIS REPO) → Abstraction Layer (Patterns) → Core Layer (Pipelines)
```

### This Repository's Purpose
- **Template Repository**: Organizations clone this repo for each workload they want to deploy
- **Simple Configuration Interface**: Provides easy-to-use configuration files
- **Environment Management**: Manages environment-specific variables and parameters
- **Pipeline Entry Point**: Single `azure-pipelines.yml` file that triggers complex orchestration

## 📁 Repository Structure

### Standard Structure
```text
├── azure-pipelines.yml          # Main pipeline entry point
├── README.md                    # Workload documentation
└── _config/
    ├── metadata.yml             # Workload metadata and global settings
    ├── environments/            # Environment-specific variables
    │   ├── vars-development.yml # Development environment
    │   ├── vars-test.yml        # Test environment  
    │   └── vars-production.yml  # Production environment
    └── parameters/              # Bicep parameter files
        ├── pattern1.parameters.json
        └── pattern2.parameters.json
```

### File Purposes

#### `azure-pipelines.yml`
- **Single entry point** for the entire deployment pipeline
- **References pattern templates** from workload pattern repository
- **Defines environments** and their dependencies
- **Triggers pipeline orchestration** through Release Engine core

#### `_config/metadata.yml`
- **Global workload settings** that apply across all environments
- **Naming conventions** and organizational standards
- **Subscription and tenant information**
- **Workload classification** and ownership details

#### `_config/environments/vars-{environment}.yml`
- **Environment-specific variables** (dev, test, prod)
- **SKU selections** appropriate for each environment
- **Environment naming** and location settings
- **Service connection overrides**

#### `_config/parameters/*.parameters.json`
- **Bicep parameter files** for infrastructure templates
- **Environment-agnostic parameters** that get tokenized
- **Resource-specific configurations**
- **Feature flags and optional components**

## 🔧 Configuration Guidelines

### Pipeline Configuration (`azure-pipelines.yml`)

#### Template Structure
```yaml
name: $(Build.DefinitionName)_$(Date:yyyyMMdd)$(Rev:.r)

resources:
  repositories:
    - repository: release-engine-core # ALIAS - DO NOT CHANGE
      type: github
      name: thecloudexplorers/release-engine-core
      endpoint: thecloudexplorers
      ref: refs/heads/main # Use specific branch/tag for stability

    - repository: workload # ALIAS - DO NOT CHANGE  
      type: github
      name: <your-org>/release-engine-<your-org>-workload-patterns
      endpoint: <your-github-endpoint>
      ref: refs/heads/main # Use specific branch/tag for stability

trigger:
  - main
  - users/*
  - develop # Add branches as needed

pool: <your-agent-pool> # Replace with your Azure DevOps agent pool

extends:
  template: /patterns/<pattern_name>/workload.yml@workload
  parameters:
    deploymentSettings:
      configurationFilePath: /_config
      # Optional: Override parameter file name
      iacParameterFileName: /parameters/<specific_parameters>.json
      
      environments:
        - name: development
          dependsOn: build
          displayName: Dev
          primaryEnvironment: true
        - name: test
          dependsOn: development
          displayName: Test
        - name: production
          dependsOn: test
          displayName: Prod
```

#### Critical Configuration Points

1. **Repository References**
   - **DO NOT change repository aliases** (`release-engine-core`, `workload`)
   - **Update repository names** to point to your organization's forks
   - **Pin to specific branches/tags** for production stability
   - **Use appropriate GitHub service connections**

2. **Pattern Selection**
   - **Choose appropriate pattern** from your workload pattern repository
   - **Template path format**: `/patterns/<pattern_name>/workload.yml@workload`
   - **Verify pattern exists** in the referenced workload repository

3. **Environment Configuration**
   - **Primary Environment**: First environment after build (usually development)
   - **Dependencies**: Each environment depends on the previous one
   - **Display Names**: User-friendly names for pipeline UI
   - **Environment Names**: Must match variable file names in `_config/environments/`

### Metadata Configuration (`_config/metadata.yml`)

#### Template Structure
```yaml
variables:
  # Landing Zone Information
  landingZoneName: "<workload_category>" # e.g., "platform", "applications", "data"
  landingZoneAbbreviation: "<3_char_code>" # e.g., "plt", "app", "dat"
  
  # Application/Workload Information  
  applicationName: "<descriptive_workload_name>"
  applicationAbbreviation: "<short_code>" # 2-4 characters
  workload: "<pattern_name>" # Must match pattern name in workload repository
  workload_location: "<azure_region>" # e.g., "westeurope", "eastus"
  
  # Azure Environment
  subscriptionId: "<azure_subscription_id>"
  
  # Organization Information (Optional)
  ado_org_name: "<azure_devops_org>"
  tenant_name: "<azure_tenant_domain>"
  
  # Ownership and Governance (Optional)
  primary_owner: "<email@domain.com>"
  secondary_owner: "<email@domain.com>"
  cost_center: "<cost_center_code>"
  environment_type: "<prod|non-prod>"
```

### Environment Variables (`_config/environments/vars-{environment}.yml`)

#### Standard Variables
```yaml
variables:
  # Environment Identification
  environmentAbbreviation: <env> # dev, tst, prd
  environment_stage_abbreviation: <env> # Usually same as above
  
  # Location and Regional Settings
  workloadLocation: <azure_region> # Can differ from metadata for multi-region
  
  # Environment-Specific Sizing
  # Examples - adjust based on your pattern needs:
  appServicePlanSku: <sku> # B1 for dev, S1 for test, P1V2 for prod
  sqlDatabaseSku: <sku> # Basic for dev, S0 for test, S2 for prod
  storageAccountSku: <sku> # Standard_LRS for dev/test, Standard_GRS for prod
  
  # Feature Flags (Optional)
  enableBackup: <true|false>
  enableMonitoring: <true|false>
  enableDiagnostics: <true|false>
  
  # Environment-Specific Service Connections (Optional)
  serviceConnection: <environment_specific_connection>
```

### Parameter Files (`_config/parameters/*.parameters.json`)

#### Structure and Tokenization
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "resourceLocation": {
      "value": "#{workloadLocation}#"
    },
    "applicationName": {
      "value": "#{applicationName}#"
    },
    "environmentAbbreviation": {
      "value": "#{environmentAbbreviation}#"
    },
    "storageAccountSku": {
      "value": "#{storageAccountSku}#"
    },
    "tags": {
      "value": {
        "Environment": "#{environmentAbbreviation}#",
        "Owner": "#{primary_owner}#",
        "CostCenter": "#{cost_center}#",
        "Application": "#{applicationName}#"
      }
    }
  }
}
```

#### Token Replacement Rules
- **Token Format**: `#{variable_name}#`
- **Variable Sources**: 
  1. Environment variables (`vars-{environment}.yml`)
  2. Metadata variables (`metadata.yml`)
  3. Built-in pipeline variables
- **Case Sensitivity**: Variable names are case-sensitive
- **Validation**: Pipeline will fail if tokens are not replaced

## 🛠️ Repository Setup and Customization

### Creating New Configuration Repository

#### Step 1: Clone Template
```bash
# Clone this template repository
git clone https://github.com/thecloudexplorers/release-engine-config-template.git

# Rename directory for your workload
mv release-engine-config-template release-engine-<workload-name>-configuration
cd release-engine-<workload-name>-configuration
```

#### Step 2: Configure Upstream
```bash
# Add upstream for future updates
git remote add upstream https://github.com/thecloudexplorers/release-engine-config-template.git

# Update origin to your repository
git remote set-url origin https://github.com/<your-org>/release-engine-<workload-name>-configuration.git
```

#### Step 3: Customize Configuration

1. **Update `azure-pipelines.yml`**
   - Change workload pattern repository reference
   - Select appropriate pattern template
   - Configure environments as needed
   - Set correct agent pool and service connections

2. **Configure `_config/metadata.yml`**
   - Set workload and application names
   - Configure subscription and tenant information
   - Set ownership and governance information

3. **Customize Environment Variables**
   - Update `vars-development.yml`, `vars-test.yml`, `vars-production.yml`
   - Set appropriate SKUs and configurations for each environment
   - Configure feature flags and environment-specific settings

4. **Create Parameter Files**
   - Create parameter files that match your selected pattern
   - Use token replacement for environment-specific values
   - Follow JSON parameter file schema

## 🔄 Template Repository Management

### Using This Repository as Template

#### For Each Workload
1. **Clone Template Repository**
   ```bash
   git clone https://github.com/thecloudexplorers/release-engine-config-template.git
   cd release-engine-config-template
   
   # Rename for your workload
   # Example: release-engine-webapp-frontend-configuration
   ```

2. **Set Up Upstream Remote**
   ```bash
   git remote add upstream https://github.com/thecloudexplorers/release-engine-config-template.git
   git remote -v
   ```

3. **Customize for Workload**
   - Update pipeline configuration with correct pattern reference
   - Modify metadata and environment variables
   - Create appropriate parameter files
   - Update documentation

#### Upstream Synchronization
```bash
# Regular upstream sync (monthly recommended)
git fetch upstream
git checkout -b sync-upstream-$(date +%Y%m%d)
git merge upstream/main
# Resolve conflicts, test changes
git push origin sync-upstream-$(date +%Y%m%d)
# Create pull request to merge into main
```

### Maintenance Best Practices

#### Regular Updates
- **Monthly Sync**: Check for upstream improvements and bug fixes
- **Security Updates**: Apply security patches promptly
- **Documentation Updates**: Keep configuration documentation current
- **Testing**: Validate changes in development environment first

#### Version Control
- **Tag Releases**: Use semantic versioning for stable configurations
- **Branch Strategy**: Use feature branches for configuration changes
- **Rollback Capability**: Maintain ability to rollback to previous versions

## 🧪 Testing and Validation

### Local Testing

#### Configuration Validation
```bash
# Validate pipeline YAML
az pipelines validate --yaml-path azure-pipelines.yml

# Validate parameter files
az deployment group validate \
  --resource-group <test-rg> \
  --template-file <path-to-bicep-template> \
  --parameters @_config/parameters/workload.parameters.json
```

#### Token Replacement Testing
```powershell
# Test token replacement locally
$paramFile = Get-Content "_config/parameters/workload.parameters.json" -Raw
$tokens = [regex]::Matches($paramFile, '#\{([^}]+)\}#')
foreach ($token in $tokens) {
    Write-Host "Found token: $($token.Value)"
}
```

### Pipeline Testing

1. **Development Environment**: Test full pipeline in development first
2. **Multi-Environment**: Test promotion through dev → test → prod
3. **Rollback Testing**: Test failure scenarios and recovery procedures

## 📚 Troubleshooting

### Common Issues

#### Pipeline Not Triggering
- Check branch names in trigger configuration
- Verify service connections have proper permissions
- Validate YAML syntax

#### Token Replacement Failures
- Check variable name matches (case-sensitive)
- Ensure variables are defined in correct files
- Verify token syntax: `#{variable_name}#`

#### Parameter Validation Errors
- Check JSON parameter file schema
- Ensure all required parameters are provided
- Verify parameter value types

## 📞 Support and Resources

### Documentation References
- **Release Engine Architecture**: Core repository documentation
- **Workload Patterns**: Your organization's pattern repository
- **Azure DevOps YAML**: Microsoft documentation

### Getting Help
- **Internal**: Contact DevOps team or Release Engine administrators
- **Community**: GitHub Discussions in Release Engine repositories
- **Issues**: Report bugs through GitHub Issues

---

*This template repository enables simple configuration of complex workload deployments while leveraging the sophisticated Release Engine pipeline orchestration framework.*
