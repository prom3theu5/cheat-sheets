# Create switch and setup Nat

```powershell
	$env:NAT_NAME=""
	$env:SWITCH_NAME=""
	
	Remove-NetNat -Name "$env:NAT_NAME" -Confirm:$false -ErrorAction "SilentlyContinue"
    
    Remove-VMSwitch -SwitchName "$env:SWITCH_NAME" -Force -ErrorAction "SilentlyContinue"

    New-VMSwitch -SwitchName "$env:SWITCH_NAME" -SwitchType Internal
    
    $ifIndex = Get-NetAdapter -Name "vEthernet ($env:SWITCH_NAME)" | Select-Object -ExpandProperty "ifIndex"
    
    New-NetIPAddress 172.41.0.1 -PrefixLength 24 -InterfaceIndex $ifIndex
    
    New-NetNat -Name $env:NAT_NAME -InternalIPInterfaceAddressPrefix 172.41.0.0/24
```