> L03-01

Now that we have our Azure resources and HR data, it's time to deploy our HR web application.

## Step 1 - Register the Contoso HR app in Azure AD

```powershell
# Create an app registration for the Contoso HR app in Azure AD
$appName = "Contoso HR App"

$clientId = az ad app create --display-name $appName --query appId --output tsv
$clientSecret = az ad app credential reset --id $clientId --query password --output tsv
```

## Step 2 - Update the application configuration

Using **Windows Terminal** let's update the ASP.NET Configuration (stored in appsettings.json) with our values:

```Powershell
# Move to the Contoso HR web app root directory
cd ".\Code\Lab 3 - Web App VM\contoso-web-app-asp-net\ContosoHR"

# Replace the SQL Server connection settings
$sqlConnectionString = "Data Source = $sqlServerName.database.windows.net; Initial Catalog = ContosoHR; Column Encryption Setting = Enabled;Attestation Protocol = AAS; Enclave Attestation Url = $attestUrl/attest/SgxEnclave; User Id = $sqlAdmin; Password = $sqlPassword"

dotnet user-secrets set "ConnectionStrings:ContosoHRDatabase" $sqlConnectionString

# Add the client credentials associated with the app registration to app settings
dotnet user-secrets set "KeyVault:clientId" $clientId
dotnet user-secrets set "KeyVault:secret" $clientSecret

# Allow the registered app to use keys in the managed HSM
az keyvault role assignment create `
  --hsm-name $hsmName `
  --role "Managed HSM Crypto Service Encryption User" `
  --scope "/" `
  --assignee $clientId
```

## Step 3 - Publish the ASP.NET application

Now we need to build and publish the app for deployment.

```Powershell
dotnet publish
```

> 💡 The published application will be in the "./bin/Release/net5.0/publish" folder

> ⚠️ Update the appsettings.json file with the **Connection String**, **Client ID** and **Client Secret** values above and save the file.

## Step 4 - Prepare the VM

Connect to the Confidential VM using Remote Desktop.

```powershell
# Connect to the VM with Remote Desktop using Bastion
az network bastion rdp --name $bastionName --resource-group $rgName --target-resource-id $vmId
```

Authenticate using the VM administrator credentials from [L01-06 Confidential VM](../Lab%201%20-%20Azure%20Resources/L01-06-CreateConfidentialVm.md).

```
Username = celAdmin
Password = nooneWillEverGuessThis0ne!+$
```

### 4.1 Install .NET 5.0 Hosting Bundle
IIS (Internet Information Services) is already installed on the VM but we need to install the .NET 5.0 Hosting Bundle to host ASP.NET 5.0 applications.

1. In the VM, download the bundle from here:
    ```
    https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/runtime-aspnetcore-5.0.17-windows-hosting-bundle-installer
    ```
2. Run the installer and choose **Repair**

The web server is now ready to host ASP.NET apps.

### 4.2 Copy the Contoros HR app to the VM

1. Copy the files in the _published_ folder on your PC to the ***wwwroot*** folder on the VM.

    ![Screenshot of the Contoso HR app](../images/L03-VM-wwwroot.png)

### 4.3 Configure the web server

2. Open IIS Manager and select the default Application Pool.

    ![Screenshot of the Contoso HR app](../images/L03-VM-IIS.png)

3. Click **Basic Settings** under _Edit Application Pool_ on the right.
4. Change the _.NET CLR version_ dropdown to **No Managed Code** and click **OK**.
    
    ![Screenshot of the Contoso HR app](../images/L03-VM-AppPool.png)

5. Click **Recycle** under _Application Pool Tasks_ on the right.

## Step 5 - Test the website

Open a browser and navigate to "http://your-vm-public-ip" you should now see the Contoso HR app with employee data from the Azure SQL database.

![Screenshot of the Contoso HR app](../images/L03-ContosoHr-Screenshot.png)

---

At this point we have the app running in Azure but our database admin can still see sensitive employee information. Next, let's protect this information: [L04 01 Read Data](./../Lab%204%20-%20Protect%20HR%20Data/L04-01-ReadHrData.md)