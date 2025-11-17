See repo on Skriptimise tunni jaoks
# Skript võrgu seadistamiseks ja arvuti nime muutmiseks
# Salvestatakse C:\Skriptid\Configure_Network_and_ComputerName.ps1

# Loo kaust, kui seda pole
New-Item -Path "C:\Skriptid" -ItemType Directory -Force

# Võrguadapteri nimi (eeldan, et adapter on 'Ethernet')
$adapter = Get-NetAdapter | Where-Object { $_.Status -eq "Up" } | Select-Object -First 1

# Seadista staatiline IP-aadress
New-NetIPAddress -InterfaceAlias $adapter.Name -IPAddress 10.0.21.10 -PrefixLength 24 -DefaultGateway 10.0.21.1

# Seadista DNS serverid
Set-DnsClientServerAddress -InterfaceAlias $adapter.Name -ServerAddresses ("127.0.0.1", "1.1.1.1")

# Muuda arvuti nimi AD1-ks ja taaskäivita
Rename-Computer -NewName "AD1" -Force
Write-Host "Arvuti nimi muudetud AD1-ks. Server taaskäivitub."
Restart-Computer -Force

Skript 2

# Skript AD ja DNS teenuste paigaldamiseks
# Salvestatakse C:\Skriptid\Install_AD_and_DNS.ps1



# Paigalda AD DS ja DNS serveri rollid
Install-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManagementTools

# Seadista domeen Tikerber.local
$domain = "Tikerber.local"
$safeModePassword = ConvertTo-SecureString "Passw0rd" -AsPlainText -Force

Install-ADDSForest `
    -DomainName $domain `
    -InstallDns:$true `
    -SafeModeAdministratorPassword $safeModePassword `
    -Force:$true

Write-Host "Active Directory ja DNS paigaldatud. Server taaskäivitub."


Skript 3

# Skript DHCP serveri paigaldamiseks ja seadistamiseks
# Salvestatakse C:\Skriptid\Install_DHCP.ps1

# Loo kaust, kui seda pole
New-Item -Path "C:\Skriptid" -ItemType Directory -Force

# Paigalda DHCP serveri roll
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Teavita DHCP serverit AD-s (vajalik autoriseerimiseks)
Add-DhcpServerInDC -DnsName "AD1.Tikerber.local" -IPAddress "10.0.21.10"

# Loo DHCP skoop
Add-DhcpServerv4Scope `
    -Name "KOOL" `
    -StartRange "10.0.21.100" `
    -EndRange "10.0.21.120" `
    -SubnetMask "255.255.255.0" `
    -State Active

# Seadista skoobi sätted (ruuter ja DNS)
Set-DhcpServerv4OptionValue `
    -ScopeId "10.0.21.0" `
    -Router "10.0.21.1" `
    -DnsServer "10.0.21.10" `
    -DnsDomain "Tikerber.local"

Write-Host "DHCP server paigaldatud ja seadistatud."



# === AINULT Active Directory metsa eemaldamine (viimane DC) ===
# Kävitada administraatorina!

Import-Module ADDSDeployment -ErrorAction SilentlyContinue

$safeModePassword = ConvertTo-SecureString "Passw0rd" -AsPlainText -Force

Write-Host "Alustan Active Directory metsa (Tikerber.local) jõuga eemaldamist..." -ForegroundColor Red
Write-Host "See on viimane domeenikontroller – kogu mets kustutatakse!" -ForegroundColor Yellow

Uninstall-ADDSDomainController `
    -DemoteOperationMasterRole:$true `
    -RemoveApplicationPartitions:$true `
    -ForceRemoval:$true `
    -SafeModeAdministratorPassword $safeModePassword `
    -Force:$true `
    -Confirm:$false

# Pärast edukat demote'i eemaldame ka AD DS rolli täielikult
Write-Host "Eemaldan AD-Domain-Services rolli..." -ForegroundColor Yellow
Uninstall-WindowsFeature -Name AD-Domain-Services -Remove

Write-Host "Active Directory mets Tikerber.local on täielikult eemaldatud!" -ForegroundColor Green
Write-Host "Arvuti taaskäivitub 10 sekundi pärast..." -ForegroundColor Cyan

Start-Sleep -Seconds 10
Restart-Computer -Force


¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤¤
# Täiesti töökindel AD paigaldusskript (kopeeri sellisena)

Install-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManagementTools

$pass = ConvertTo-SecureString "Passw0rd" -AsPlainText -Force

$params = @{
    DomainName                    = "tikerber.local"
    DomainNetbiosName             = "TIKERBER"
    SafeModeAdministratorPassword = $pass
    InstallDns                    = $true
    Force                         = $true
}

Install-ADDSForest @params



PS C:\Skriptid> C:\Skriptid\dns.ps1
Install-WindowsFeature : The server could not update the provided feature files in the time allowed.
At C:\Skriptid\dns.ps1:1 char:1
+ Install-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManageme ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (9fe6be53-191b-40f8-910c-e085f60d6786:Guid) [Install-WindowsFeature], Exception
    + FullyQualifiedErrorId : GetAlterationState__CallCycleTimeout,Microsoft.Windows.ServerManager.Commands.AddWindowsFeatureCommand

Success Restart Needed Exit Code      Feature Result                               
------- -------------- ---------      --------------                               
False   No             Failed         {}                                           
Install-ADDSForest : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'DomainNetbiosName' was not recogni
zed.
At C:\Skriptid\dns.ps1:13 char:1
+ Install-ADDSForest @params
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Install-ADDSForest], TestFailedException
    + FullyQualifiedErrorId : Test.VerifyDcPromoCore.DCPromo.General.77,Microsoft.DirectoryServices.Deployment.PowerShell.Commands.InstallADDSForest 
   Command

Message        : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'DomainNetbiosName' was not recognized.
                 
Context        : Test.VerifyDcPromoCore.DCPromo.General.77
RebootRequired : False
Status         : Error




# === AD + DNS paigaldamine ilma ISO-ta (kasutab Microsofti servereid) ===
# Töötab alati, kui masinal on internetiühendus

Write-Host "Paigaldan AD-Domain-Services ja DNS rolli otse Microsofti serveritest..." -ForegroundColor Green

# See on võti – sunnib kasutama Windows Update’i kui allikat ja ootab kauem
Install-WindowsFeature -Name AD-Domain-Services, DNS `
    -IncludeManagementTools `
    -Source wim:C:\Windows\WinSxS\* `
    -Restart:$false `
    -ErrorAction Stop

# Kui ikka aegub, proovib automaatselt Windows Update’ist tõmmata
if (-not (Get-WindowsFeature AD-Domain-Services).Installed) {
    Write-Host "Proovin Windows Update’ist tõmmata (võib võtta 1–3 minutit)..." -ForegroundColor Yellow
    Install-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManagementTools
}

# Veendume, et moodul on laetud
Import-Module ADDSDeployment -Force

$pass = ConvertTo-SecureString "Passw0rd" -AsPlainText -Force

$params = @{
    DomainName                    = "tikerber.local"
    DomainNetbiosName             = "TIKERBER"
    SafeModeAdministratorPassword = $pass
    InstallDns                    = $true
    Force                         = $true
}

Write-Host "Loon domeeni tikerber.local ..." -ForegroundColor Cyan
Install-ADDSForest @params




PS C:\Users\Administrator> #AD paigaldamine DNSiga
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-WindowsFeature DNS -IncludeManagementTools

#Domeeni lillep.local loomine
Install-ADDSForest
    -DomainName "tikerber.local"
    -DomainNetbiosName "TIKERBER"
    -SafeModeAdministratorPassword (ConvertTo-SecureString "Passw0rd" -AsPlainText -Force)
    -InstallDNS
    -Force

Success Restart Needed Exit Code      Feature Result                               
------- -------------- ---------      --------------                               
True    No             NoChangeNeeded {}                                           
True    No             NoChangeNeeded {}                                           
cmdlet Install-ADDSForest at command pipeline position 1
Supply values for the following parameters:
DomainName: tikerber.local
Install-ADDSForest : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'NewDomain' was not recognized.
At line:6 char:1
+ Install-ADDSForest
+ ~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Install-ADDSForest], TestFailedException
    + FullyQualifiedErrorId : Test.VerifyDcPromoCore.DCPromo.General.77,Microsoft.DirectoryServices.Deployment.PowerShell.Commands.InstallADDSForest 
   Command

Message        : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'NewDomain' was not recognized.
                 
Context        : Test.VerifyDcPromoCore.DCPromo.General.77
RebootRequired : False
Status         : Error

-DomainName : The term '-DomainName' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the
 name, or if a path was included, verify that the path is correct and try again.
At line:7 char:5
+     -DomainName "tikerber.local"
+     ~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (-DomainName:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
 
-DomainNetbiosName : The term '-DomainNetbiosName' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the s
pelling of the name, or if a path was included, verify that the path is correct and try again.
At line:8 char:5
+     -DomainNetbiosName "TIKERBER"
+     ~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (-DomainNetbiosName:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
 
-SafeModeAdministratorPassword : The term '-SafeModeAdministratorPassword' is not recognized as the name of a cmdlet, function, script file, or opera
ble program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
At line:9 char:5
+     -SafeModeAdministratorPassword (ConvertTo-SecureString "Passw0rd" ...
+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (-SafeModeAdministratorPassword:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
 
-InstallDNS : The term '-InstallDNS' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the
 name, or if a path was included, verify that the path is correct and try again.
At line:10 char:5
+     -InstallDNS
+     ~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (-InstallDNS:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
 
-Force : The term '-Force' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or 
if a path was included, verify that the path is correct and try again.
At line:11 char:5
+     -Force
+     ~~~~~~
    + CategoryInfo          : ObjectNotFound: (-Force:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
 


