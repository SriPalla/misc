To read all certificates from an Azure Key Vault using its resource ID with PowerShell, you can use the Azure PowerShell module. Below are the steps and a script that demonstrates how to accomplish this:

### Prerequisites

1. **Azure PowerShell Module**: Make sure you have the Azure PowerShell module installed. You can install it using the following command:

    ```powershell
    Install-Module -Name Az -AllowClobber -Scope CurrentUser
    ```

2. **Authentication**: Ensure you are authenticated to your Azure account. You can authenticate using the following command:

    ```powershell
    Connect-AzAccount
    ```

### Script to Read All Certificates

Here is a PowerShell script that retrieves all certificates from an Azure Key Vault using its resource ID:

```powershell
# Define the resource ID of the Key Vault
$resourceId = "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<key-vault-name>"

# Parse the resource ID to get the Key Vault name and resource group
$resourceIdParts = $resourceId -split "/"
$keyVaultName = $resourceIdParts[8]
$resourceGroupName = $resourceIdParts[4]

# Get the Key Vault
$keyVault = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName

if ($null -eq $keyVault) {
    Write-Error "Key Vault not found."
    exit
}

# Retrieve all certificates from the Key Vault
$certificates = Get-AzKeyVaultCertificate -VaultName $keyVaultName

# Display the certificates
foreach ($cert in $certificates) {
    Write-Output "Certificate Name: $($cert.Name)"
    Write-Output "Thumbprint: $($cert.Certificate.Thumbprint)"
    Write-Output "Issuer: $($cert.Certificate.IssuerName.Name)"
    Write-Output "Not Before: $($cert.Certificate.NotBefore)"
    Write-Output "Expires: $($cert.Certificate.NotAfter)"
    Write-Output "----------------------------------------"
}
```

### Explanation of the Script

1. **Resource ID**: The script starts by defining the resource ID of the Key Vault. Replace `<subscription-id>`, `<resource-group-name>`, and `<key-vault-name>` with your actual values.

2. **Parse the Resource ID**: The resource ID is parsed to extract the Key Vault name and resource group name.

3. **Get the Key Vault**: The script uses `Get-AzKeyVault` to get the Key Vault based on the extracted name and resource group.

4. **Retrieve Certificates**: The script retrieves all certificates in the Key Vault using `Get-AzKeyVaultCertificate`.

5. **Display Certificates**: Finally, the script iterates over the retrieved certificates and displays information about each certificate, including the name, thumbprint, issuer, and validity period.

### Running the Script

1. Save the script to a file, for example, `Get-KeyVaultCertificates.ps1`.
2. Open a PowerShell window and navigate to the directory where the script is saved.
3. Run the script:

    ```powershell
    .\Get-KeyVaultCertificates.ps1
    ```

This script will output the details of all certificates stored in the specified Azure Key Vault.
