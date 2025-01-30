


#### Setup ip for an interface
```cmd
netsh interface ipv4 set address name="eth2" static <IP | 192.168.100.5> <mask |255.255.255.0> <gateway |192.168.100.2> 
```

### Set Static dns ip address
```cmd
netsh interface ipv4 set dns name="Ethernet 2" static <dnsIP | 192.168.100.40>
```


### Get computer name
```cmd
wmic computersystem get name
```


### Change computer name
```cmd
WMIC ComputerSystem where Name="<COMPUTERNAME>" call Rename Name="<NewName>"
```

## Install AD

### AD requirements

```powershell
get-windowsfeature
install-windowsfeature AD-Domain-Services
Import-Module ADDSDeployment
```

### Create Domain and add computer 

```powershell
# Create domain
Install-ADDSForest -DomainName "<name>"
# Get AD computers
Get-ADComputer -Filter * | Select-Object Name

# Add computer to AD
Add-Computer -DomainName <Domain01>
```

## Create / Delete rules

```powershell
# ALLOW HTTP
netsh advfirewall firewall add rule name="Allow_HTTP" dir=in action=allow protocol=TCP localport=80
# ALLOW MESSENGER
netsh advfirewall firewall add rule name="Allow_Messenger" dir=in action=allow program="C:\Program Files\Messenger\msmsgs.exe" enable=yes
# Delete rules
netsh advfirewall firewall delete rule name="Allow_Messenger"
# show rules
netsh advfirewall firewall show rule name=all # | findstr /i "Messenger"
```

## Ressources

[Windows netsh modif](https://www.howtogeek.com/103190/change-your-ip-address-from-the-command-prompt/)
[Install AD with powershell](https://www.microsoft.com/en-gb/industry/blog/technetuk/2016/06/08/setting-up-active-directory-via-powershell/)