```powershell
$base = (Get-ADDomain).DistinguishedName
$lab  = "OU=Lab,$base"
$upn  = (Get-ADDomain).DNSRoot
$pw   = Read-Host -AsSecureString "Password for users"

$users = @(
  @{N="Kono IT";         S="kono.it";         D="IT";         G="GG_IT"},
  @{N="Sheva HR";        S="sheva.hr";        D="HR";         G="GG_HR"},
  @{N="Carol Sales";     S="carol.sales";     D="Sales";      G="GG_Sales"},
  @{N="Bob Accountant";  S="bob.accountant";  D="Accountant"; G="GG_Accountant"}
)

foreach ($u in $users) {
  New-ADUser -Name $u.N -SamAccountName $u.S -UserPrincipalName "$($u.S)@$upn" -Path "OU=$($u.D),OU=Users,$lab" -AccountPassword $pw -Enabled $true
  Add-ADGroupMember -Identity $u.G -Members $u.S
}

New-ADUser -Name "Admin Taras" -SamAccountName "adm.taras" -UserPrincipalName "adm.taras@$upn" -Path "OU=IT,OU=Users,$lab" -AccountPassword $pw -Enabled $true
Add-ADGroupMember -Identity "Domain Admins" -Members "adm.taras"
```
