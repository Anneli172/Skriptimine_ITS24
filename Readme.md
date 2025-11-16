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




# Skript AD ja DNS teenuste paigaldamiseks
# Salvestatakse C:\Skriptid\ADDNS.ps1

# Loo kaust, kui seda pole
New-Item -Path "C:\Skriptid" -ItemType Directory -Force

# Logi skripti käivitamine
Write-Host "Alustan AD ja DNS paigaldust Windows Server 2025 (24H2) jaoks"

# Paigalda AD DS ja DNS serveri rollid
Write-Host "Paigaldan AD-Domain-Services ja DNS rolle..."
Install-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManagementTools -IncludeAllSubFeature

# Kontrolli, kas rollid on paigaldatud
$adStatus = Get-WindowsFeature -Name AD-Domain-Services
$dnsStatus = Get-WindowsFeature -Name DNS
Write-Host "AD-Domain-Services staatus: $($adStatus.InstallState)"
Write-Host "DNS staatus: $($dnsStatus.InstallState)"

# Seadista domeen Tikerber.local
$domain = "Tikerber.local"
$safeModePassword = ConvertTo-SecureString "Passw0rd" -AsPlainText -Force

try {
    Write-Host "Seadistan AD domeeni: $domain"
    Install-ADDSForest `
        -DomainName $domain `
        -InstallDns:$true `
        -SafeModeAdministratorPassword $safeModePassword `
        -NoRebootOnCompletion:$true `
        -Force:$true `
        -ErrorAction Stop

    Write-Host "Active Directory ja DNS paigaldatud edukalt."
}
catch {
    Write-Host "Viga AD paigaldamisel: $_"
    exit
}

# Taaskäivita server
Write-Host "Server taaskäivitub..."
Restart-Computer -Force


Get-WindowsFeature | Where-Object { $_.Name -like "*AD*" -or $_.Name -like "*DNS*" } | Format-Table Name, DisplayName, InstallState



# Skript AD ja DNS teenuste paigaldamiseks
# Salvestatakse C:\Skriptid\ADDNS.ps1

# Loo kaust, kui seda pole
New-Item -Path "C:\Skriptid" -ItemType Directory -Force

# Logi skripti käivitamine
Write-Host "Alustan AD ja DNS paigaldust Windows Server 2025 (24H2) jaoks. $(Get-Date)"

# Kontrolli AD-Domain-Services ja DNS rollide olekut
Write-Host "Kontrollin AD-Domain-Services ja DNS rollide olekut..."
$adFeature = Get-WindowsFeature -Name AD-Domain-Services -ErrorAction SilentlyContinue
$dnsFeature = Get-WindowsFeature -Name DNS -ErrorAction SilentlyContinue

if (-not $adFeature) {
    Write-Host "Viga: AD-Domain-Services rolli ei leitud süsteemist. Kontrollige süsteemi konfiguratsiooni."
    exit
}
if (-not $dnsFeature) {
    Write-Host "Viga: DNS rolli ei leitud süsteemist. Paigaldan DNS rolli..."
    Install-WindowsFeature -Name DNS -IncludeManagementTools -IncludeAllSubFeature
} else {
    Write-Host "DNS rolli staatus: $($dnsFeature.InstallState)"
}

# Paigalda AD-Domain-Services, kui see pole veel paigaldatud
if ($adFeature.InstallState -ne "Installed") {
    Write-Host "Paigaldan AD-Domain-Services rolli..."
    Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools -IncludeAllSubFeature
} else {
    Write-Host "AD-Domain-Services on juba paigaldatud."
}

# Kontrolli paigalduse olekut
$adStatus = Get-WindowsFeature -Name AD-Domain-Services
Write-Host "AD-Domain-Services staatus: $($adStatus.InstallState)"

# Seadista domeen Tikerber.local
$domain = "Tikerber.local"
$safeModePassword = ConvertTo-SecureString "Passw0rd" -AsPlainText -Force

try {
    Write-Host "Seadistan AD domeeni: $domain"
    Install-ADDSForest `
        -DomainName $domain `
        -SafeModeAdministratorPassword $safeModePassword `
        -NoRebootOnCompletion:$true `
        -Force:$true `
        -ErrorAction Stop

    Write-Host "Active Directory domeen seadistatud edukalt."
    if ($dnsFeature.InstallState -eq "Installed") {
        Write-Host "DNS on juba paigaldatud, lisan DNS tsooni käsitsi..."
        Add-DnsServerPrimaryZone -Name $domain -ZoneFile "$domain.dns" -ErrorAction Stop
        Write-Host "DNS tsoon $domain loodud."
    }
}
catch {
    Write-Host "Viga AD või DNS seadistamisel: $_"
    exit
}

# Taaskäivita server
Write-Host "Server taaskäivitub..."
Restart-Computer -Force
