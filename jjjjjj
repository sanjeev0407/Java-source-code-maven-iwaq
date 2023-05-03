# Setting Octopus URL and API Key variables
$OctopusUrl = "http://octopus/api" # Octopus URL
$octopusApiKey = "$apikey"

# Authenticating to the API using the API Key
$header = @{ "X-Octopus-ApiKey" = $octopusApiKey }

$daysToFilter = 30
$startDate = (Get-Date).AddDays(-$daysToFilter).ToString("yyyy-MM-ddTHH:mm:ss.fffZ")

# Get list of Spaces
$spaces = (Invoke-RestMethod -Method Get -Uri "$OctopusUrl/spaces/all" -Headers $header)

# Loop through all spaces and targets
foreach ($space in $spaces) {
    $spaceid = $space.id
    $spacename = $space.name
    write-host "Fetching the Deployment Targets in $($spaceid) ($($spacename))"

    $machines = (Invoke-RestMethod -Method Get -Uri "$OctopusUrl/$($spaceid)/machines?skip=0&take=100000" -Headers $header)
    $items = $machines.items

    # Loop through all Deployment Targets to check their Health Status
    foreach ($item in $items) {
        $machineid = $item.id
        $machinename = $item.name
        write-host "Checking Health Status for $($machineid) ($($machinename))"

        # Checking the Health Status of the current Deployment Target and outputting information
        $healthStatus = $item.healthStatus
        $ip = Test-Connection $machinename -Count 1 -ErrorAction SilentlyContinue | Select-Object -ExpandProperty IPV4Address

        # Create a custom object to output to CSV
        $reportObject = [PSCustomObject] @{
            SpaceId = $spaceid
            SpaceName = $spacename
            TargetId = $machineid
            TargetName = $machinename
            HealthStatus = $healthStatus
            IPAddress = $ip.IPAddressToString
        }

        # Output the custom object to CSV file
        $reportObject | Export-Csv -Path "targets-report.csv" -Append -NoTypeInformation
    }
}

# Display a message once the script has completed
Write-Host "Targets report generated successfully"

# Get a list of all spaces
$spaces = Invoke-RestMethod -Uri "$octopusUrl/spaces?skip=0" -Headers $header

# Create an empty array to store the results
$results = @()

foreach ($space in $spaces.Items) {
    # Get the dashboard information for the space
    $dashboardInformation = Invoke-RestMethod -Uri "$octopusUrl/$($space.Id)/dashboard?highestLatestVersionPerProjectAndEnvironment=true" -Headers $header

    foreach ($environmentToUse in $dashboardInformation.Environments) {
        # Get the failed deployments to the environment within the past $daysToFilter days
        $failedDeployments = @($dashboardInformation.Items | Where-Object { $_.EnvironmentId -eq $environmentToUse.Id -and $_.State -eq "Failed" -and $_.Created -ge $startDate })
        
		# Add the results to the array
        foreach ($deployment in $failedDeployments) {
            $project = $dashboardInformation.Projects | Where-Object { $_.Id -eq $deployment.ProjectId }
            $tenantName = $null

            if ($null -ne $deployment.TenantId)
            {        
                $tenant = $dashboardInformation.Tenants | Where-Object { $_.Id -eq $deployment.TenantId }
                $tenantName = $tenant.Name
            }

            $result = [PSCustomObject] @{
                Space = $space.Name
                Environment = $environmentToUse.Name
                Project = $project.Name
                ReleaseVersion = $deployment.ReleaseVersion
                State = $deployment.State
                Created = $deployment.Created
            }

            $results += $result
        }
    }
}

# Export the results to a CSV file
$results | Export-Csv -Path "failed_deployments.csv" -NoTypeInformation

# Display a message once the script has completed
Write-Host "failed_deployments report generated successfully"
