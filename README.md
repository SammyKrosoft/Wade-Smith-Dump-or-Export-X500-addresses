# Wade-Smith-Dump-or-Export-X500-addresses
PowerShell for Wade Smith: dump or export X500 addresses for all users. Users who have multiple X500 addresses will have multiple entries in the export (1 per X500 address).

First connect to your Exchange Online environment:

```PowerShell
$UPN = "MyAdminUser@contoso.ca"
Connect-ExchangeOnline -UserPrincipalName $UPN
```

```PowerShell
$Collection = @()
Get-Mailbox | Select-Object PrimarySMTPAddress,@{Name="x500 Email Address";Expression={$_.EmailAddresses |Where-Object {$_ -match "x500:*"}}} | Foreach {
                    $UserPrimarySMTPAddress = $_.PrimarySMTPAddress
                    $ObjectX500Addresses = $_."x500 Email Address"
                    $ObjectX500Addresses.count
                    if ($ObjectX500Addresses.count -gt 1){
                       Foreach ($X500address in $ObjectX500Addresses){ 
                            write-host $X500address -ForegroundColor Green
                            $Collection += [pscustomobject]@{PrimarySMTPAddress = $UserPrimarySMTPAddress; 'x500 Email Address' = $X500address}
                            }
                    } Elseif($ObjectX500Addresses.count -eq 1) {
                       $Collection += [pscustomobject]@{PrimarySMTPAddress = $UserPrimarySMTPAddress; 'x500 Email Address' = $ObjectX500Addresses}
                    }
                    
} # <= end of Foreach loop after the pipe on Line 14

$Collection | ft
```
