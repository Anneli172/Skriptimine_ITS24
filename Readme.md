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
