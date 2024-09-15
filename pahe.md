
Connect-AzAccount -Identity

# Get Exchange Online credentials
$UserCredential = Get-AutomationPSCredential -Name "Automateac"
Connect-ExchangeOnline -Credential $UserCredential

# Get the current date and time
$currentmonth = Get-Date -Format "yyyy-MM"
$currentTime = Get-Date -Format "HH.mm.ss"

# Define the date range for the message trace (last 10 days)
$startDate = (Get-Date).AddDays(-10)
$endDate = get-date

# Run the message trace
$messages = Get-MessageTrace -StartDate $startDate -EndDate $endDate

# Convert the results to CSV format
$tempPath = [System.IO.Path]::GetTempPath()
$csvPath = "$tempPath\message-trace-results_$currentDate_$currentTime.csv"
$messages | Export-Csv -Path $csvPath -NoTypeInformation

# Azure Storage account details
$storageAccountName = "your storge account name"
$storageAccountKey = "your key"
$containerName = $currentmonth
$blobName = "message-trace-results_$currentDate_$currentTime.csv"

# Create a storage context
$ctx = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

# Check if the container exists, if not, create it
$container = Get-AzStorageContainer -Name $containerName -Context $ctx -ErrorAction SilentlyContinue
if (-not $container) {
    New-AzStorageContainer -Name $containerName -Context $ctx
}

# Upload the CSV file to Azure Blob Storage
Set-AzStorageBlobContent -Container $containerName -File $csvPath -Blob $blobName -Context $ctx

# Clean up
Remove-Item $csvPath

# Disconnect from Exchange Online
Disconnect-ExchangeOnline -Confirm:$false
