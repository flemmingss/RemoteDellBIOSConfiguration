
<#
.Synopsis
   Remote Dell BIOS Configuration
.DESCRIPTION
   A PowerShell module to view and change BIOS settings on remote Dell computers
.EXAMPLE
  Get-DellBIOSConfigurationAttributes -ComputerName <name of target computer>
  Returns categories and attributes availabe in the BIOS witch current and possible values
.EXAMPLE
  Set-DellBIOSConfigurationAttribute -ComputerName <name of target computer> -BIOSPassword <BIOS password> -Category <BIOS Category (PSChildName)> -Attribute <BIOS Attribute> -Value <New value for BIOS Attribute>
  Sets a new value for a attribute in the BIOS.
.NOTES
  Author: Flemming Sørvollen Skaret
  https://github.com/flemmingss/
  https://flemmingss.com/
#>

# Av FLESKA

function Set-DellBIOSProviderPaths ($ComputerName)
{
    $ErrorActionPreference = "Stop"
    $script:module_source = "\\domain\share\powershell_modules\DellCommand PowerShellProvider\DellBIOSProvider" # Where the DellCommand PowerShellProvider is located
    $script:module_destination = "\\" + $ComputerName + "\c$\Program Files\WindowsPowerShell\Modules\DellBIOSProvider"
}


function Copy-DellBIOSProvider
{
    $ErrorActionPreference = "Stop"
    try
    {

        if (Test-Path -Path $module_destination)
        {
            Write-Host "The DellBIOSProvider module already exists on $ComputerName, module uploading not required" -ForegroundColor DarkYellow
            $script:module_copied = $false
        }
        else
        {
            Write-Host "The DellBIOSProvider module don't exist on $ComputerName, copying module from source" -ForegroundColor DarkYellow
            Copy-Item -Path $module_source -Destination $module_destination -Recurse
            $script:module_copied = $true
        }
    }
    catch
    {
        Write-Host "Error: $Error.Exception" -ForegroundColor Red
    }
}


function Remove-DellBIOSProvider
{
    $ErrorActionPreference = "Stop"
    try
    {
        if ($module_copied)
        {
            Write-Host "Deleting module from $ComputerName (Cleanup)" -ForegroundColor Yellow
            Remove-Item -Path $module_destination -Recurse
        }
    }
    catch
    {
        Write-Host "Error: $Error.Exception" -ForegroundColor Red
    }
}


function Set-DellBIOSConfigurationAttribute 
{
param (
	[Parameter(Mandatory=$true,Position=0)]
	[string]$ComputerName,
	[Parameter(Mandatory=$true,Position=1)]
	[string]$BIOSPassword,
	[Parameter(Mandatory=$true,Position=2)]
	[string]$Category,
	[Parameter(Mandatory=$true,Position=3)]
	[string]$Attribute,
	[Parameter(Mandatory=$true,Position=4)]
	[string]$Value
	)

    $ErrorActionPreference = "Stop"
    Set-DellBIOSProviderPaths -ComputerName $ComputerName
    Write-Host "Module source: $module_source" -ForegroundColor DarkYellow
    Write-Host "Module Destionation: $module_destination" -ForegroundColor DarkYellow
    Copy-DellBIOSProvider

    if (Test-Path -Path $module_destination)
    {
        Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            
            $ErrorActionPreference = "Stop"
            try
            {
                Write-Host "Executing commands on target: $Using:ComputerName" -ForegroundColor DarkYellow
                Write-Host "Importing module DellBIOSProvider to session" -ForegroundColor DarkYellow
                Import-Module -Name "DellBIOSProvider"
                Write-Host "Old value for $Using:Category\$Using:Attribute (check):" (Get-Item -Path DellSMBios:\$Using:Category\$Using:Attribute).CurrentValue -ForegroundColor Yellow
                Write-Host "New value for $Using:Category\$Using:Attribute (set): $Using:Value" -ForegroundColor Yellow
                Set-Item -Path DellSMBios:\$Using:Category\$Using:Attribute -Value "$Using:Value" -Password $Using:BIOSPassword
                Write-Host "New value for $Using:Category\$Using:Attribute (verify):" (Get-Item -Path DellSMBios:\$Using:Category\$Using:Attribute).CurrentValue -ForegroundColor Yellow
                Write-Host "Script executed successfully on target: $Using:ComputerName" -ForegroundColor Green
            }
            catch
            {
                Write-Host "Error: $Error.Exception" -ForegroundColor Red
            }
            finally
            {
                Remove-Module -Name "DellBIOSProvider"
            }
        }

    }
    else
    {
    Write-Host "Error: DellBIOSProvider not in $module_destination" -ForegroundColor Red
    }
    Remove-DellBIOSProvider
}


function Get-DellBIOSConfigurationAttributes
{
param (
	[Parameter(Mandatory=$true,Position=0)]
	[string]$ComputerName
	)

    $ErrorActionPreference = "Stop"
    Set-DellBIOSProviderPaths -ComputerName $ComputerName
    Write-Host "Module source: $module_source" -ForegroundColor DarkYellow
    Write-Host "Module Destionation: $module_destination" -ForegroundColor DarkYellow
    Copy-DellBIOSProvider

    if (Test-Path -Path $module_destination)
    {
        Invoke-Command -ComputerName $ComputerName -ScriptBlock {

            $ErrorActionPreference = "Stop"
            try
            {
                Write-Host "Executing commands on target: $Using:ComputerName" -ForegroundColor DarkYellow
                Write-Host "Importing module DellBIOSProvider to session" -ForegroundColor DarkYellow
                Import-Module -Name "DellBIOSProvider"
                $DellSMBios_Root = Get-ChildItem DellSMBios:\ | Where-Object { $_.description -like "Displays the attributes to configure*" } #| Select-Object Category | ForEach-Object { Write-host $_.Category }
                $DellSMBios_Attributes = foreach ($_ in $DellSMBios_Root)
                {
                    $Category = $_.Category
                    Get-ChildItem DellSMBios:\$Category
                }
                $DellSMBios_Attributes | Select-Object PSChildName, Attribute, CurrentValue, PossibleValues | Format-table
                Write-Host "(PSChildName corresponds to Category while using the Set-DellBIOSConfigurationAttribute function)"
            }
            catch
            {
                Write-Host "Error: $Error.Exception" -ForegroundColor Red
            }
            finally
            {
                Remove-Module -Name "DellBIOSProvider"
            }
        }

    }
    else
    {
    Write-Host "Error: DellBIOSProvider not in $module_destination" -ForegroundColor Red
    }
    Remove-DellBIOSProvider
}
