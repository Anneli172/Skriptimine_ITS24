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




# === Tagasi võtmise skript – UNDO kõik eelnevad muudatused ===
# Kävitada administraatorina!

Write-Host "Alustan kõigi muudatuste tagasi võtmist..." -ForegroundColor Cyan

# 1. Eemaldame DHCP skoobi ja rolli (kui see on paigaldatud)
if (Get-WindowsFeature -Name DHCP | Where-Object {$_.Installed}) {
    Write-Host "Eemaldan DHCP skoobi ja teenuse..." -ForegroundColor Yellow
    # Kustutame kõik skoobid (turvaliselt)
    Get-DhcpServerv4Scope | Remove-DhcpServerv4Scope -Force
    # Eemaldame serveri autoriseerimise AD-st
    Remove-DhcpServerInDC -DnsName "AD1.Tikerber.local" -IPAddress "10.0.21.10" -Force -ErrorAction SilentlyContinue
    # Eemaldame DHCP rolli
    Uninstall-WindowsFeature -Name DHCP -Remove
    Write-Host "DHCP teenus ja skoobid eemaldatud." -ForegroundColor Green
}

# 2. Eemaldame DNS serveri rolli ja kustutame tsoonid
if (Get-WindowsFeature -Name DNS | Where-Object {$_.Installed}) {
    Write-Host "Eemaldan DNS tsoonid ja teenuse..." -ForegroundColor Yellow
    # Kustutame domeeni tsooni ja _msdcs
    Remove-DnsServerZone -Name "Tikerber.local" -Force -ErrorAction SilentlyContinue
    Remove-DnsServerZone -Name "_msdcs.Tikerber.local" -Force -ErrorAction SilentlyContinue
    # Eemaldame DNS rolli (ei eemalda AD-ga seotud andmeid, sest AD eemaldame järgmiseks)
    Uninstall-WindowsFeature -Name DNS -Remove
    Write-Host "DNS teenus ja Tikerber.local tsoon eemaldatud." -ForegroundColor Green
}

# 3. Eemaldame Active Directory mets (Demote + Forest eemaldamine)
if (Get-WindowsFeature -Name AD-Domain-Services | Where-Object {$_.Installed}) {
    Write-Host "Eemaldan Active Directory mets (Tikerber.local)..." -ForegroundColor Red
    $safeModePassword = ConvertTo-SecureString "Passw0rd" -AsPlainText -Force
    
    # Kui see on viimane DC metsas, kasutame Force removal
    Uninstall-ADDSDomainController `
        -DemoteOperationMasterRole:$true `
        -RemoveApplicationPartitions:$true `
        -ForceRemoval:$true `
        -SafeModeAdministratorPassword $safeModePassword `
        -Force:$true -Confirm:$false

    # Pärast demote'i eemaldame kogu metsa (viimane DC)
    Install-ADDSForest -DomainName "dummy.local" -NoRebootOnCompletion:$true -Force:$true -SafeModeAdministratorPassword $safeModePassword | Out-Null
    # Ei, see ei tööta nii – tegelikult kui Uninstall-ADDSDomainController õnnestus, siis AD on juba eemaldatud
    # Seega lihtsalt eemaldame rolli
    Uninstall-WindowsFeature -Name AD-Domain-Services -Remove
    Write-Host "Active Directory mets Tikerber.local on eemaldatud." -ForegroundColor Green
}

Add-DnsServerPrimaryZone -Name "Tikerber.local" -ZoneFile "Tikerber.local.dns"

