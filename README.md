%md
# End-to-end Databricks project

## Overview
Building Lakehouse using Databricks, Azure storage as a source, and Power BI for visualization. 

## Requirements
1. Azure subscription
1. Databricks (or Databricks community edition)
1. Power BI

## Project Tasks
1. Create Azure Blob Storage and Azure Data Lake Storage Gen2
1. Upload dataset (Superstore)
1. Create Databricks service
1. Connect Databricks to the Storage using Access Key
1. Connect Databricks to the Storage using Service Principal
1. Create Key Vault secret and Databricks Scope
1. Connect Unity Catalog
1. Create Delta Tables
1. Connect Power BI to Databricks

### Task 1: Create Storage
1. Navigate to the Azure Portal.
1. Choose Storage services.
1. Create Blob Storage.
1. Create Data Lake Storage Gen2 (need to check `Enable hierarchical namespace`).

### Task 2: Upload dataset
1. In the Blob Storage create container `superstore`.
1. In the Data Lake Storage create container `datasets`. In the Data Lake Storage you can create folders inside container. Create folder `superstore`.
1. Upload dataset (3 CSV files) to the container:
    - Blob Storage -> superstore container -> CSV files
    - Data Lake Storage -> datasets container -> superstore -> CSV files

### Task 3: Create Databricks service
1. Navigate to the Azure Portal.
1. Choose Databricks service.
1. Create New service in the same region as Storage accounts.

### Task 4: Connect Databricks to the Storage using Access Key
1. Launch Databricks.
1. Create new Notebook.
1. Create new Compute.
1. Run the following code `Mount blob` in the Notebook and check running command `display(dbutils.fs.ls("/mnt/superstore"))
1. Unmount using command `dbutils.fs.unmount("/mnt/superstore")`. If your cluster was terminated, mount point will be automatically unmount.

### Task 5: Connect Databricks to the Storage using Service Principal
1. Create a Service Principal:
    - In the Azure portal, go to the `Microsoft Entra ID service`.
    - Under `Manage`, click `App Registrations`.
    - Click `+ New` registration. Enter a name for the application and click `Register`.
    - Click `Certificates & Secrets`.
    - Click `+ New` client secret.
    - Add a description for the secret and click Add.
    - Copy and save the `value` for the new secret.
    - In the application registration overview, copy and save the `Application (client) ID` and `Directory (tenant) ID`.
1. Assign role:
    - In the Azure portal, go to the `Storage accounts service`.
    - Select an Azure storage account to use with this application registration.
    - Click `Access Control (IAM)`.
    - Click `+ Add` and select `Add role assignment` from the dropdown menu.
    - Set the Select field to the Microsoft Entra ID application name and set Role to `Storage Blob Data Contributor` and click `Next`.
    - Under `Assign access to`, select `User, group or Service Principal`.
    - Click `+Select Members`, select your service principal, and click `Review and Assign`.
1. Run the following code `Mount Gen2` in the Notebook and check running command `display(dbutils.fs.ls("/mnt/datasets/superstore/raw"))

### Task 6: Create Key Vault secret and Databricks Scope
1. Navigate to the Azure Portal.
1. Choose `Key Vault` and create a new one.
1. Select `Access control (IAM)` and add a new role `Key Vault Administrator` for yourself.
1. Select `Secrets` tab under `Objects`.
1. Create new secret for `client_id`, `client_secret`, `tenant_id`.
1. Keep names of the new secrets.
1. Select `Access configuration` tab under `Settings`.
1. Set `Permission model` to `Vault access policy`.
1. Select the `Networking` tab under `Settings`.
1. In `Firewalls and virtual networks` set Allow access from: to Allow public access from specific virtual networks and IP addresses. Under `Exception`, check Allow trusted Microsoft services to bypass this firewall.
1. Launch Databricks Workspace.
1. Go to `https://<databricks-instance>#secrets/createScope`.
1. Enter the name of the secret scope, choose Manage Principal, Enter the DNS Name and Resource ID (Vault URI and Resource ID from the Properties tab under Settings).
1. Click the Create button.
1. Run a code to mount Storage Gen2 using secret scope.

### Task 7: Connect Unity Catalog
1. Do not use DBFS with Unity Catalog external locations
    > Unity Catalog secures access to data in external locations by using full cloud URI paths to identify grants on managed object storage directories. DBFS mounts use an entirely different data access model that bypasses Unity Catalog entirely. Databricks recommends that you do not reuse cloud object storage volumes between DBFS mounts and UC external volumes, including when sharing data across workspaces or accounts.
1. We need to create Access connector to the Storage. Navigate to the Azure Portal. In the Azure portal, go to the `Access Connector for Azure Databricks`.
1. Create new Access connector in the same region where your Storage and Databricks are located.
1. Assign role:
    - In the Azure portal, go to the `Storage accounts service`.
    - Select an Azure storage account to use with this application registration.
    - Click `Access Control (IAM)`.
    - Click `+ Add` and select `Add role assignment` from the dropdown menu.
    - Set the Select field to the Microsoft Entra ID application name and set Role to `Storage Blob Data Contributor` and click `Next`.
    - Under `Assign access to`, select `Managed Identity`.
    - Click `+Select Members`, in the drop down menu `Managed identity` select `Access Connector for Azure Databricks`, find your connector, click `Select`, and click `Review and Assign`.
1. Create a new storage credential:
    - In the `Databricks Workspace` navigate to `Catalog`.
    - Click `+` and select `Add a storage credential`.
    - Credential Type is `Azure Managed Identity`.
    - Storage credential name whatever you want.
    - Access connector ID is a `Recource ID` of our `Access Connector for Azure Databricks`.
    - Click `Create`.
1. Create a new external location:
    - In the `Catalog` click `+` and select `Add an external location`.
    - External location name whatever you want.
    - Storage credential select from the previous step.
    - URL is a path like `abfss://<your-container>@<your-storage>.dfs.core.windows.net/`.
    - Click `Create`.
    - Click `Test connection`.
1. You can check that everything is ok running this code in the Notebook (put actual container and storage):
    ```
    display(dbutils.fs.ls("abfss://<your-container>@<your-storage>.dfs.core.windows.net/"))
    ```
1. Create Catalog:
    - In the `Catalog` click `+` and select `Add a catalog`.
    - Catalog name whatever you want.
    - Type `Standard`.
    - In the Storage location choose external location from the previous step.
    - Click `Create`.
    > This external location will be default location for this Catalog. If you create a managed table (without specifying location), then the table will be created in this external location. You might choose default external location for all catalogs in the current Metastore in the Azure Databricks account console.
1. If you don't have any permissions to create a new catalog, then turn on admin account. Proceed here https://accounts.azuredatabricks.net/login
1. If you have an error to login, you need to navigate to `Microsoft Entra ID` and in the `Users` tab create a new user with `Global Admin` role.
1. Try to log in using new User and Password from the previous step.
1. Navigate to `User management` and edit current users. You can set up admin account.
1. In the Azure Databricks account console you can set up deafult path for the Metastore in the `Catalog` tab.
1. Create Schema.

### Task 8: Create Delta Tables
1. Read the CSV into a DataFrame and write it out in Delta format to a specified location.
1. Create a Delta table using the Delta files at the specified location.

### Task 9: Connect Power BI to Databricks
1. Navigate to `Partner Connect` and choose Power BI. Then connection file will be created.
1. Create access token, copy and save it.
1. Open connection file. It will start Power BI with connection setup. Use token connection type.
