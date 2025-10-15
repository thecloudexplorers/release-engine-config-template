# Instructions for Configuring the Storage Account Workload

To ensure the `azure-pipelines.yml` file and `_config` structure in this repository are correctly configured for the storage account workload, follow these steps:

## Azure Pipelines Configuration

1. **Reference Example Configuration**:
   - Use the `azure-pipelines.yml` file from the `set-example_workload-configuration` repository as a reference.
   - Path: `C:\repos\01-tcs\01-tcsnlps\ssc-set\set-example_workload-configuration\azure-pipelines.yml`

2. **Key Elements to Include**:
   - **Name**: Ensure the pipeline name follows the format `$(Build.DefinitionName)_$(Date:yyyyMMdd)$(Rev:.r)`.
   - **Resources**:
     - Add repositories for `release-engine` and `release-engine-example-workload-pattern` with the correct aliases and tags.
   - **Trigger**:
     - Use `users/*` as the trigger.
   - **Pool**:
     - Specify `MSHosted-WorkloadsPool` as the pool.
   - **Extends**:
     - Reference the appropriate workload template (e.g., `/patterns/storage_account/workload.yml@workload`).
     - Include parameters for `platformWorkloadSettings` such as `configurationFilePath`, `iacParameterFileName`, and `environments`.

3. **Validation**:
   - After making changes, validate the pipeline configuration using Azure DevOps tools.
   - Ensure the pipeline runs successfully in both `production` and `development` environments.

4. **Version Control**:
   - Commit and push changes to the repository.
   - Use meaningful commit messages to describe the updates.

## `_config` Structure

1. **Metadata File**:
   - Update `_config/pipelines/metadata.yml` with the following details:

     ```yaml
     variables:
       landingZoneType: platform
       designArea: storage
       workload: storage_account
       workloadLocation: westeurope

       ado_org_name: "thecloudexplorers"
       tenant_name: "thecloudexplorers"

       landing_zone_name: "Storage"
       landing_zone_description: "Storage Account Workload"
       landing_zone_name_abbreviation: "stg"

       managementGroupId: "mg30010"

       solution_name: "Storage Account Workload"
       solution_description: "This workload deploys and manages storage accounts."
       solution_name_abbreviation: "stg"
       criticality: "Medium"

       workload_instance: "002"

       primary_owner: "owner@thecloudexplorers.com"
       secondary_owner: "secondary@thecloudexplorers.com"
       tertiary_owner: "tertiary@thecloudexplorers.com"
     ```

2. **Environment-Specific Variables**:
   - Update `_config/pipelines/vars-development.yml` with:

     ```yaml
     variables:
       environmentAbbreviation: dev
       workloadLocation: westeurope
       environment_stage_abbreviation: dev
       storageAccountName: "devstorageaccount"
       resourceGroupName: "dev-storage-rg"
     ```

   - Update `_config/pipelines/vars-production.yml` with:

     ```yaml
     variables:
       environmentAbbreviation: prd
       workloadLocation: westeurope
       environment_stage_abbreviation: prd
       storageAccountName: "prdstorageaccount"
       resourceGroupName: "prd-storage-rg"
     ```

By following these steps, you can ensure the repository is correctly configured for the storage account workload.
