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
Install-WindowsFeature : The server could not update the provided feature files in the time allowed.                                      At C:\Skriptid\ADDS_install.ps1:2 char:1                                                                                                  + Install-WindowsFeature AD-Domain-Services, DNS -IncludeManagementTool ...                                                               + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~                                                                       + CategoryInfo          : OperationTimeout: (4c331522-3ab9-4b71-97eb-5afe4385f820:Guid) [Install-WindowsFeature], Exception               + FullyQualifiedErrorId : GetAlterationState__CallCycleTimeout,Microsoft.Windows.ServerManager.Commands.AddWindowsFeatureCommand                                                                                                                                                Success Restart Needed Exit Code      Feature Result                                                                                      ------- -------------- ---------      --------------                                                                                      False   No             Failed         {}                                                                                                  Start-Service : Cannot find any service with service name 'DNS'.                                                                          At C:\Skriptid\ADDS_install.ps1:3 char:1                                                                                                  + Start-Service -Name DNS                                                                                                                 + ~~~~~~~~~~~~~~~~~~~~~~~                                                                                                                     + CategoryInfo          : ObjectNotFound: (DNS:String) [Start-Service], ServiceCommandException                                           + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.StartServiceCommand                                                                                                                                                                                                                                                                                                                    cmdlet Install-ADDSForest at command pipeline position 1                                                                                  Supply values for the following parameters:                                                                                               DomainName: tikerber.local                                                                                                                SafeModeAdministratorPassword: ********                                                                                                   Confirm SafeModeAdministratorPassword: ********                                                                                                                                                                                                                                     The target server will be configured as a domain controller and restarted when this operation is complete.                                Do you want to continue with this operation?                                                                                              [Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): A                                                 Install-ADDSForest : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'NewDomain' was not rec ognized.                                                                                                                                  At C:\Skriptid\ADDS_install.ps1:8 char:1                                                                                                  + Install-ADDSForest                                                                                                                      + ~~~~~~~~~~~~~~~~~~                                                                                                                          + CategoryInfo          : NotSpecified: (:) [Install-ADDSForest], TestFailedException                                                     + FullyQualifiedErrorId : Test.VerifyDcPromoCore.DCPromo.General.77,Microsoft.DirectoryServices.Deployment.PowerShell.Commands.Insta     llADDSForestCommand                                                                                                                                                                                                                                                              Message        : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'NewDomain' was not recogni                  zed.                                                                                                                                                                                                                                                               Context        : Test.VerifyDcPromoCore.DCPromo.General.77                                                                                RebootRequired : False                                                                                                                    Status         : Error                                                                                                                                                                                                                                                              -DomainName : The term '-DomainName' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spe lling of the name, or if a path was included, verify that the path is correct and try again.                                              At C:\Skriptid\ADDS_install.ps1:9 char:5                                                                                                  +     -DomainName "tikerber.local"                                                                                                        +     ~~~~~~~~~~~                                                                                                                             + CategoryInfo          : ObjectNotFound: (-DomainName:String) [], CommandNotFoundException                                               + FullyQualifiedErrorId : CommandNotFoundException                                                                                                                                                                                                                              -DomainNetbiosName : The term '-DomainNetbiosName' is not recognized as the name of a cmdlet, function, script file, or operable program.  Check the spelling of the name, or if a path was included, verify that the path is correct and try again.                                At C:\Skriptid\ADDS_install.ps1:10 char:5                                                                                                 +     -DomainNetbiosName "TIKERBER"                                                                                                       +     ~~~~~~~~~~~~~~~~~~                                                                                                                      + CategoryInfo          : ObjectNotFound: (-DomainNetbiosName:String) [], CommandNotFoundException                                        + FullyQualifiedErrorId : CommandNotFoundException                                                                                    
-SafeModeAdministratorPassword : The term '-SafeModeAdministratorPassword' is not recognized as the name of a cmdlet, function, script fi
le, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
At C:\Skriptid\ADDS_install.ps1:11 char:5
+     -SafeModeAdministratorPassword (ConvertTo-SecureString "Passw0rd" ...
+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (-SafeModeAdministratorPassword:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException

-InstallDns : The term '-InstallDns' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spe
lling of the name, or if a path was included, verify that the path is correct and try again.
At C:\Skriptid\ADDS_install.ps1:12 char:5
+     -InstallDns
+     ~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (-InstallDns:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException

-Force : The term '-Force' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of t
he name, or if a path was included, verify that the path is correct and try again.
At C:\Skriptid\ADDS_install.ps1:13 char:5
+     -Force
+     ~~~~~~
    + CategoryInfo          : ObjectNotFound: (-Force:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
