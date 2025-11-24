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
 



PS C:\Users\Administrator> Install-ADDSForest -DomainName "tikerber.local" -DomainNetbiosName "TIKERBER" -SafeModeAdministratorPassword (ConvertTo-SecureString "Passw0rd" -AsPlainText -Force) -InstallDns -Force
Install-ADDSForest : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'Doma
inNetbiosName' was not recognized.
At line:1 char:1
+ Install-ADDSForest -DomainName "tikerber.local" -DomainNetbiosName "T ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Install-ADDSForest], TestFailedException
    + FullyQualifiedErrorId : Test.VerifyDcPromoCore.DCPromo.General.77,Microsoft.DirectoryServices.Deployment.PowerSh
   ell.Commands.InstallADDSForestCommand

Message
-------
Verification of prerequisites for Domain Controller promotion failed. The specified argument 'DomainNetbiosName' was...


PS C:\Users\Administrator> (Get-Command Install-ADDSForest).Parameters["DomainNetbiosName"]


Name            : DomainNetbiosName
ParameterType   : System.String
ParameterSets   : {[ADDSForest, System.Management.Automation.ParameterSetMetadata]}
IsDynamic       : False
Aliases         : {}
Attributes      : {ADDSForest}
SwitchParameter : False



 Install-ADDSForest -DomainName "tikerber.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "Passw0rd" -AsPlainText -Force) -InstallDns -Force
Install-ADDSForest : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'Inst
allDNS' was not recognized.
At line:1 char:1
+ Install-ADDSForest -DomainName "tikerber.local" -SafeModeAdministrato ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Install-ADDSForest], TestFailedException
    + FullyQualifiedErrorId : Test.VerifyDcPromoCore.DCPromo.General.77,Microsoft.DirectoryServices.Deployment.PowerSh
   ell.Commands.InstallADDSForestCommand

Message
-------
Verification of prerequisites for Domain Controller promotion failed. The specified argument 'InstallDNS' was not re...




PS C:\Users\Administrator> Install-ADDSForest -DomainName "tikerber.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "Passw0rd" -AsPlainText -Force) -Force
Install-ADDSForest : Verification of prerequisites for Domain Controller promotion failed. The specified argument 'NewD
omain' was not recognized.
At line:1 char:1
+ Install-ADDSForest -DomainName "tikerber.local" -SafeModeAdministrato ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Install-ADDSForest], TestFailedException
    + FullyQualifiedErrorId : Test.VerifyDcPromoCore.DCPromo.General.77,Microsoft.DirectoryServices.Deployment.PowerSh
   ell.Commands.InstallADDSForestCommand

Message
-------
Verification of prerequisites for Domain Controller promotion failed. The specified argument 'NewDomain' was not rec...




Get-WindowsFeature AD-Domain-Services

Display Name                                            Name                       Install State
------------                                            ----                       -------------
[X] Active Directory Domain Services                    AD-Domain-Services             Installed


PS C:\Users\Administrator> Uninstall-WindowsFeature AD-Domain-Services -IncludeManagementTools

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
False   Maybe          Failed         {}
Uninstall-WindowsFeature : A prerequisite check for the AD-Domain-Services feature failed.
1. The Active Directory domain controller needs to be demoted before the AD DS role can be removed.
At line:1 char:1
+ Uninstall-WindowsFeature AD-Domain-Services -IncludeManagementTools
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (Active Directory Domain Services:ServerComponentWrapper) [Uninstall-W
   indowsFeature], Exception
    + FullyQualifiedErrorId : Alteration_PrerequisiteCheck_Failed,Microsoft.Windows.ServerManager.Commands.RemoveWindo
   wsFeatureCommand


PS C:\Users\Kasutaja> ssh -v kasutaja@10.0.21.100
OpenSSH_for_Windows_9.5p2, LibreSSL 3.8.2
debug1: Connecting to 10.0.21.100 [10.0.21.100] port 22.
debug1: Connection established.
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_rsa type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_rsa-cert type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ecdsa type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ecdsa-cert type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ecdsa_sk type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ecdsa_sk-cert type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ed25519 type 3
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ed25519-cert type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ed25519_sk type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_ed25519_sk-cert type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_xmss type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_xmss-cert type -1
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_dsa type 3
debug1: identity file C:\\Users\\Kasutaja/.ssh/id_dsa-cert type -1
debug1: Local version string SSH-2.0-OpenSSH_for_Windows_9.5
debug1: Remote protocol version 2.0, remote software version OpenSSH_9.6p1 Ubuntu-3ubuntu13.11
debug1: compat_banner: match: OpenSSH_9.6p1 Ubuntu-3ubuntu13.11 pat OpenSSH* compat 0x04000000
debug1: Authenticating to 10.0.21.100:22 as 'kasutaja'
debug1: load_hostkeys: fopen C:\\Users\\Kasutaja/.ssh/known_hosts2: No such file or directory
debug1: load_hostkeys: fopen __PROGRAMDATA__\\ssh/ssh_known_hosts: No such file or directory
debug1: load_hostkeys: fopen __PROGRAMDATA__\\ssh/ssh_known_hosts2: No such file or directory
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: algorithm: curve25519-sha256
debug1: kex: host key algorithm: ssh-ed25519
debug1: kex: server->client cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: kex: client->server cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: SSH2_MSG_KEX_ECDH_REPLY received
debug1: Server host key: ssh-ed25519 SHA256:m4vSTyqTqgt+Y9xizepnO6eYzzF9wqzhrUAXpO9+TLg
debug1: load_hostkeys: fopen C:\\Users\\Kasutaja/.ssh/known_hosts2: No such file or directory
debug1: load_hostkeys: fopen __PROGRAMDATA__\\ssh/ssh_known_hosts: No such file or directory
debug1: load_hostkeys: fopen __PROGRAMDATA__\\ssh/ssh_known_hosts2: No such file or directory
debug1: Host '10.0.21.100' is known and matches the ED25519 host key.
debug1: Found key in C:\\Users\\Kasutaja/.ssh/known_hosts:5
debug1: ssh_packet_send2_wrapped: resetting send seqnr 3
debug1: rekey out after 134217728 blocks
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: ssh_packet_read_poll2: resetting read seqnr 3
debug1: SSH2_MSG_NEWKEYS received
debug1: rekey in after 134217728 blocks
debug1: get_agent_identities: agent returned 1 keys
debug1: Will attempt key: C:\\Users\\Kasutaja/.ssh/id_ed25519 ED25519 SHA256:DdCeEAiP83+YY+Tkp9loREBMloVqOdzSyWqZYCI9Luo agent
debug1: Will attempt key: C:\\Users\\Kasutaja/.ssh/id_rsa
debug1: Will attempt key: C:\\Users\\Kasutaja/.ssh/id_ecdsa
debug1: Will attempt key: C:\\Users\\Kasutaja/.ssh/id_ecdsa_sk
debug1: Will attempt key: C:\\Users\\Kasutaja/.ssh/id_ed25519_sk
debug1: Will attempt key: C:\\Users\\Kasutaja/.ssh/id_xmss
debug1: Will attempt key: C:\\Users\\Kasutaja/.ssh/id_dsa ED25519 SHA256:DdCeEAiP83+YY+Tkp9loREBMloVqOdzSyWqZYCI9Luo
debug1: SSH2_MSG_EXT_INFO received
debug1: kex_input_ext_info: server-sig-algs=<ssh-ed25519,ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,sk-ssh-ed25519@openssh.com,sk-ecdsa-sha2-nistp256@openssh.com,rsa-sha2-512,rsa-sha2-256>
debug1: kex_ext_info_check_ver: publickey-hostbound@openssh.com=<0>
debug1: kex_ext_info_check_ver: ping@openssh.com=<0>
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey,password
debug1: Next authentication method: publickey
debug1: Offering public key: C:\\Users\\Kasutaja/.ssh/id_ed25519 ED25519 SHA256:DdCeEAiP83+YY+Tkp9loREBMloVqOdzSyWqZYCI9Luo agent
debug1: Authentications that can continue: publickey,password
debug1: Trying private key: C:\\Users\\Kasutaja/.ssh/id_rsa
debug1: Trying private key: C:\\Users\\Kasutaja/.ssh/id_ecdsa
debug1: Trying private key: C:\\Users\\Kasutaja/.ssh/id_ecdsa_sk
debug1: Trying private key: C:\\Users\\Kasutaja/.ssh/id_ed25519_sk
debug1: Trying private key: C:\\Users\\Kasutaja/.ssh/id_xmss
debug1: Offering public key: C:\\Users\\Kasutaja/.ssh/id_dsa ED25519 SHA256:DdCeEAiP83+YY+Tkp9loREBMloVqOdzSyWqZYCI9Luo
debug1: Authentications that can continue: publickey,password
debug1: Next authentication method: password


$ mkdir -p ~/.docker/cli-plugins/
$ curl -SL https://github.com/docker/compose-cli/releases/download/v2.0.0-rc.1/docker-compose-linux-amd64 -o ~/.docker/cli-plugins/docker-compose
$ chmod +x ~/.docker/cli-plugins/docker-compose
