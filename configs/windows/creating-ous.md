
```powershell
$base = (Get-ADDomain).DistinguishedName
New-ADOrganizationalUnit -Name "Lab" -Path $base
$lab = "OU=Lab,$base"
"Users","Groups","Workstations","Servers" | ForEach-Object { New-ADOrganizationalUnit -Name $_ -Path $lab }
"IT","HR","Sales","Accountant" | ForEach-Object { New-ADOrganizationalUnit -Name $_ -Path "OU=Users,$lab" }
"GG_IT","GG_HR","GG_Sales","GG_Accountant" | ForEach-Object {
  New-ADGroup -Name $_ -GroupScope Global -GroupCategory Security -Path "OU=Groups,$lab" }
```