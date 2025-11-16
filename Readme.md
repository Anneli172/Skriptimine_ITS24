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
