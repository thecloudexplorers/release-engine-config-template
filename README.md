# Release Engine Configuration Template

## 🏗️ Repository Overview

This repository serves as the **Configuration Layer Template** in the Release Engine three-tier solution. It provides a simple, user-friendly interface for configuring and deploying workloads while hiding the underlying pipeline complexity.

> **🚨 Important**: This is a **template repository**. Organizations should clone this repository **for each workload** they want to deploy.

## Architecture Context

```text
Configuration Layer (THIS REPO) → Abstraction Layer (Patterns) → Core Layer (Pipelines)
```

### This Repository's Role
- **Workload Configuration**: Simple interface for complex deployment orchestration
- **Environment Management**: Environment-specific variables and parameter files
- **Pipeline Entry Point**: Single `azure-pipelines.yml` that triggers sophisticated deployments
- **Template Foundation**: Starting point for each new workload deployment

## 📁 Repository Structure

### File Organization
```text
├── azure-pipelines.yml          # Main pipeline entry point
├── README.md                    # This documentation
├── AGENTS.md                    # AI assistant instructions
└── _config/                     # Configuration directory
    ├── metadata.yml             # Workload metadata and global settings
    ├── environments/            # Environment-specific variables
    │   ├── vars-development.yml # Development environment settings
    │   ├── vars-test.yml        # Test environment settings
    │   └── vars-production.yml  # Production environment settings
    └── parameters/              # Bicep parameter files
        ├── pattern1.parameters.json
        └── pattern2.parameters.json
```

### Configuration Philosophy
- **Simple Interface**: Hide pipeline complexity behind simple configuration files
- **Environment Promotion**: Automatic promotion from dev → test → production
- **Token Replacement**: Variables automatically replaced in parameter files
- **Single Source**: One repository per workload for complete isolation

## 🚀 Quick Start (Using as Template)

### Step 1: Clone This Template
```bash
# Clone this template repository
git clone https://github.com/thecloudexplorers/release-engine-config-template.git

# Rename for your specific workload
mv release-engine-config-template my-workload-configuration
cd my-workload-configuration
```

### Step 2: Set Up Upstream Tracking
```bash
# Add upstream remote for future updates
git remote add upstream https://github.com/thecloudexplorers/release-engine-config-template.git

# Set new origin (your workload repository)
git remote set-url origin https://github.com/myorg/my-workload-configuration.git

# Verify configuration
git remote -v
```

### Step 3: Configure Your Workload
1. **Select Pattern**: Choose appropriate pattern from your workload pattern repository
2. **Update Pipeline**: Modify `azure-pipelines.yml` with your pattern and repositories
3. **Configure Metadata**: Set workload information in `_config/metadata.yml`
4. **Set Environment Variables**: Configure `_config/environments/vars-*.yml` files
5. **Create Parameter Files**: Add pattern-specific parameter files

## ⚙️ Configuration Guide

### Pipeline Configuration (`azure-pipelines.yml`)

#### Template Structure
```yaml
name: $(Build.DefinitionName)_$(Date:yyyyMMdd)$(Rev:.r)

resources:
  repositories:
    - repository: release-engine-core # DO NOT CHANGE ALIAS
      type: github
      name: thecloudexplorers/release-engine-core
      endpoint: <your-github-endpoint>
      ref: refs/heads/main

    - repository: workload # DO NOT CHANGE ALIAS
      type: github
      name: <your-org>/release-engine-<your-org>-workload-patterns
      endpoint: <your-github-endpoint>
      ref: refs/heads/main

trigger:
  - main
  - users/*
  - develop

pool: <your-agent-pool>

extends:
  template: /patterns/<pattern_name>/workload.yml@workload
  parameters:
    deploymentSettings:
      configurationFilePath: /_config
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
- **Repository References**: Update to point to your organization's repositories
- **GitHub Endpoints**: Use your organization's GitHub service connections
- **Agent Pool**: Specify your Azure DevOps agent pool
- **Pattern Selection**: Choose appropriate pattern from your workload repository

### Metadata Configuration (`_config/metadata.yml`)

#### Example Configuration
```yaml
variables:
  # Landing Zone Information
  landingZoneName: "applications"
  landingZoneAbbreviation: "app"
  
  # Workload Information
  applicationName: "my-web-application"
  applicationAbbreviation: "mwa"
  workload: "webapp_with_database" # Must match pattern name
  workload_location: "westeurope"
  
  # Azure Environment
  subscriptionId: "12345678-1234-1234-1234-123456789abc"
  
  # Ownership (Optional but Recommended)
  primary_owner: "devops-team@myorg.com"
  cost_center: "IT-APPS-001"
  environment_type: "non-prod"
```

### Environment Variables (`_config/environments/vars-{environment}.yml`)

#### Development Environment Example (`vars-development.yml`)
```yaml
variables:
  # Environment Identification
  environmentAbbreviation: dev
  environment_stage_abbreviation: dev
  
  # Location Settings
  workloadLocation: westeurope
  
  # Development-Appropriate Sizing
  appServicePlanSku: B1          # Basic tier for development
  sqlDatabaseSku: Basic          # Minimal database for dev
  storageAccountSku: Standard_LRS # Local redundancy sufficient
  
  # Feature Flags
  enableBackup: false            # No backup needed in dev
  enableMonitoring: true         # Keep monitoring for debugging
  enableDiagnostics: true        # Enable for troubleshooting
```

#### Production Environment Example (`vars-production.yml`)
```yaml
variables:
  # Environment Identification
  environmentAbbreviation: prd
  environment_stage_abbreviation: prd
  
  # Location Settings
  workloadLocation: westeurope
  
  # Production-Grade Sizing
  appServicePlanSku: P2V3         # Premium tier for production
  sqlDatabaseSku: S2              # Standard tier with good performance
  storageAccountSku: Standard_GRS # Geo-redundant storage
  
  # Feature Flags
  enableBackup: true              # Backup critical for production
  enableMonitoring: true          # Full monitoring in production
  enableDiagnostics: true         # Full diagnostics enabled
```

### Parameter Files (`_config/parameters/*.parameters.json`)

#### Token Replacement System
Parameter files use token replacement for environment-specific values:

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
    "appServicePlanSku": {
      "value": "#{appServicePlanSku}#"
    },
    "enableBackup": {
      "value": "#{enableBackup}#"
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
- **Format**: `#{variable_name}#` (case-sensitive)
- **Sources**: Environment variables, metadata variables, built-in pipeline variables
- **Validation**: Pipeline fails if tokens aren't replaced

## 🔄 Template Maintenance

### Keeping Up-to-Date with Upstream

#### Regular Synchronization (Recommended: Monthly)
```bash
# Fetch upstream improvements
git fetch upstream

# Create integration branch
git checkout -b upstream-sync-$(date +%Y%m%d)

# Merge upstream changes
git merge upstream/main

# Resolve conflicts (preserve your workload configuration)
# Test changes in development environment
# Push and create PR to main
```

#### What to Sync
- ✅ **Configuration Improvements**: Better configuration patterns and examples
- ✅ **Bug Fixes**: Fixes to pipeline configuration or token replacement
- ✅ **Documentation Updates**: Improved setup guides and troubleshooting
- ✅ **New Features**: Additional configuration options and capabilities

#### What to Preserve
- 🔒 **Workload Configuration**: Your specific workload metadata and settings
- 🔒 **Environment Variables**: Your environment-specific configurations
- 🔒 **Parameter Files**: Your pattern-specific parameter configurations
- 🔒 **Pipeline Customizations**: Your organization-specific pipeline modifications

## 🎯 Workload Types and Examples

### Web Application Workloads
Perfect for web applications with databases:
- **Frontend Applications**: React, Angular, Vue.js applications
- **Backend APIs**: REST APIs, GraphQL services
- **Full-Stack Applications**: Applications with frontend and backend components

### Function App Workloads
Ideal for serverless computing scenarios:
- **Event Processing**: Process events from queues, topics, or storage
- **API Backends**: Lightweight API implementations
- **Integration Solutions**: Connect different systems and services

### Container Workloads
Suitable for containerized applications:
- **Microservices**: Individual services in a larger system
- **Legacy Applications**: Containerized legacy applications
- **Multi-Container Applications**: Applications requiring multiple containers

### Data Platform Workloads
For data processing and analytics:
- **Data Warehouses**: Analytics and reporting solutions
- **ETL Pipelines**: Data transformation and movement
- **Streaming Analytics**: Real-time data processing

## 🧪 Testing and Validation

### Local Testing

#### Configuration Validation
```bash
# Validate pipeline YAML syntax
az pipelines validate --yaml-path azure-pipelines.yml --organization https://dev.azure.com/myorg --project myproject

# Validate JSON parameter files
Get-Content "_config/parameters/workload.parameters.json" | Test-Json
```

#### Token Replacement Testing
```powershell
# Check for unreplaced tokens
$paramFiles = Get-ChildItem "_config/parameters/*.json"
foreach ($file in $paramFiles) {
    $content = Get-Content $file.FullName -Raw
    $tokens = [regex]::Matches($content, '#\{([^}]+)\}#')
    if ($tokens.Count -gt 0) {
        Write-Host "Tokens found in $($file.Name):" -ForegroundColor Yellow
        $tokens | ForEach-Object { Write-Host "  $($_.Value)" }
    }
}
```

### Pipeline Testing Strategy

#### Development Environment First
1. **Deploy to Development**: Test full pipeline in development environment
2. **Validate Resources**: Ensure all expected resources are created
3. **Test Functionality**: Verify deployed workload functions correctly
4. **Check Outputs**: Validate all expected outputs are generated

#### Multi-Environment Testing
1. **Environment Promotion**: Test dev → test → prod promotion
2. **Configuration Differences**: Verify environment-specific configurations work
3. **Rollback Testing**: Test failure scenarios and recovery procedures

## 🛠️ Customization Examples

### Adding Custom Environments

#### Adding Staging Environment
1. **Create Variable File**: `_config/environments/vars-staging.yml`
2. **Update Pipeline**: Add staging environment to pipeline configuration
3. **Set Dependencies**: Configure staging to depend on test environment

```yaml
# In azure-pipelines.yml
environments:
  - name: development
    dependsOn: build
    displayName: Dev
    primaryEnvironment: true
  - name: test
    dependsOn: development
    displayName: Test
  - name: staging
    dependsOn: test
    displayName: Staging
  - name: production
    dependsOn: staging
    displayName: Prod
```

### Multi-Pattern Workloads

#### Using Multiple Patterns
Some workloads may require multiple patterns:
```yaml
# Override parameter file for specific pattern
extends:
  template: /patterns/multi_stage_pattern/workload.yml@workload
  parameters:
    deploymentSettings:
      configurationFilePath: /_config
      # Use specific parameter file for dependent stages
      iacParameterFileName: /parameters/multi_stage.dependent2.parameters.json
      environments: [...]
```

### Organization-Specific Customizations

#### Custom Naming Conventions
Modify metadata.yml to match your organization:
```yaml
variables:
  # Use your organization's naming standards
  resourcePrefix: "myorg"
  namingConvention: "resource-{prefix}-{workload}-{environment}"
  
  # Organization-specific tags
  companyName: "My Organization"
  department: "Engineering"
  businessUnit: "Digital Products"
```

## 📚 Troubleshooting Guide

### Common Issues

#### Pipeline Not Triggering
**Problem**: Pipeline doesn't start when pushing to main branch
**Solutions**:
- ✅ Check trigger configuration in `azure-pipelines.yml`
- ✅ Verify branch protection rules
- ✅ Ensure service connections have proper permissions

#### Token Replacement Failures
**Problem**: Tokens like `#{variableName}#` not being replaced
**Solutions**:
- ✅ Check variable name spelling (case-sensitive)
- ✅ Ensure variable exists in environment or metadata files
- ✅ Verify token syntax: `#{variableName}#`

#### Parameter Validation Errors
**Problem**: Bicep deployment fails with parameter errors
**Solutions**:
- ✅ Check JSON parameter file syntax
- ✅ Ensure all required parameters are provided
- ✅ Verify parameter value types match Bicep template expectations

#### Resource Deployment Failures
**Problem**: Azure resource deployment fails
**Solutions**:
- ✅ Check Azure permissions for service principal
- ✅ Verify resource quotas and limits
- ✅ Check resource naming conflicts
- ✅ Review deployment logs for specific errors

### Debugging Commands

#### Local Debugging
```powershell
# Check configuration file syntax
Test-Json -Path "_config/metadata.yml"

# List all configuration files
Get-ChildItem -Recurse "_config/" -Include "*.yml", "*.json"

# Validate parameter files
$paramFiles = Get-ChildItem "_config/parameters/*.json"
foreach ($file in $paramFiles) {
    Write-Host "Validating $($file.Name)..."
    try {
        Get-Content $file.FullName | ConvertFrom-Json | Out-Null
        Write-Host "✅ Valid JSON" -ForegroundColor Green
    } catch {
        Write-Host "❌ Invalid JSON: $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

## 📞 Support and Resources

### Getting Help
- **Comprehensive Guide**: Detailed instructions in [AGENTS.md](./AGENTS.md)
- **Community Support**: GitHub Discussions for questions and best practices
- **Issue Reporting**: GitHub Issues for bugs and feature requests
- **Documentation**: Complete examples and templates

### Related Repositories
- **[Release Engine Core](https://github.com/thecloudexplorers/release-engine-core)**: Core pipeline orchestration framework
- **[Workload Patterns](https://github.com/thecloudexplorers/release-engine-example-workload-pattern)**: Pattern template repository
- **[Architecture Documentation](https://github.com/thecloudexplorers/release-engine-core/blob/main/docs/Release-Engine-Solution-Architecture.md)**

### External Resources
- **[Azure DevOps YAML Schema](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema)**
- **[Azure Bicep Documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/)**
- **[JSON Parameter Files](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/parameter-files)**

## 🎉 Success Stories

### Example Workloads
This template has been successfully used for:

- **E-commerce Platforms**: Multi-tier web applications with databases and caching
- **API Services**: RESTful services with authentication and monitoring
- **Data Analytics**: Data processing pipelines with storage and compute
- **Microservices**: Container-based distributed applications
- **Legacy Modernization**: Lift-and-shift applications with cloud-native enhancements

### Best Practices from the Field
- 📈 **Start Simple**: Begin with basic patterns, add complexity gradually
- 🔄 **Iterate Frequently**: Deploy often to catch issues early
- 📊 **Monitor Everything**: Use comprehensive monitoring from day one
- 🛡️ **Security First**: Implement security controls from the beginning
- 📝 **Document Decisions**: Keep clear documentation of configuration choices

---

*This template repository enables simple configuration of sophisticated workload deployments. Clone once per workload, configure for your needs, and enjoy automated multi-environment deployments with enterprise-grade pipeline orchestration.*