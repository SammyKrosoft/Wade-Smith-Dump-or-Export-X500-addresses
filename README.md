# Wade-Smith-Dump-or-Export-X500-addresses
PowerShell for Wade Smith: dump or export X500 addresses for all users. Users who have multiple X500 addresses will have multiple entries in the export (1 per X500 address).

First connect to your Exchange Online environment:

```PowerShell
$UPN = "MyAdminUser@contoso.ca"
Connect-ExchangeOnline -UserPrincipalName $UPN
```

Then:
- initiate a collection variable,
- get all mailboxes,
- for each mailbox and for each X500 address on that mailbox, create a PSCustomObject entry with its Primary SMTP Address, and each X500 address, and store it in the collection variable
- at the end, either dump that variable to display the table (```$Collection | ft```) or export it in a CSV file

## Script getting all mailboxes (no CSV input file)

```PowerShell
# THIS SCRIPT GETS ALL THE MAILBOXES OF THE TENANT TO GET THEIR X500 ADDRESS

$Collection = @()
Get-Mailbox | Select-Object PrimarySMTPAddress,@{Name="x500 Email Address";Expression={$_.EmailAddresses |Where-Object {$_ -match "x500:*"}}} | Foreach {
                    $UserPrimarySMTPAddress = $_.PrimarySMTPAddress
                    $ObjectX500Addresses = $_."x500 Email Address"
                    if ($ObjectX500Addresses.count -gt 1){
                       Foreach ($X500address in $ObjectX500Addresses){ 
                            $Collection += [pscustomobject]@{PrimarySMTPAddress = $UserPrimarySMTPAddress; 'x500 Email Address' = $X500address}
                            }
                    } Elseif($ObjectX500Addresses.count -eq 1) {
                       $Collection += [pscustomobject]@{PrimarySMTPAddress = $UserPrimarySMTPAddress; 'x500 Email Address' = $ObjectX500Addresses}
                    }
                    
} # <= end of Foreach loop after the pipe on Line 14

# If you want to directly dump the results:
$Collection | ft

# If you want to store it in a CSV file:
$Collection | Export-CSV -NoTypeInfo c:\temp\X500AddressesExport.csv
```

## Script getting mailboxes from a CSV input file

If you want to use a CSV file as an input, you can enclose the "Get-Mailbox" part inside a ```Foreach ($Item in Import-CSV $CSVInputFile) { ... }``` loop, below is a sample one (tested):

> NOTE1: the CSV input file must have a column or header named PrimarySMTPAddress for the below script to work

> NOTE2: update/customize the Input CSV file path from the $CSVInputFile variable to match your own input file

```PowerShell
# THIS SCRIPT GETS THE MAILBOXES IN THE SAMPLE INPUTCSVFILE.CSV, WHICH PATH IS STORED IN THE $CSVInputFile VARIABLE.
# NOTE: THE CSV INPUT FILE MUST HAVE A HEADAER CALLED PrimarySMTPAddress (case insensitive) FOR THE SCRIPT TO GET THE PRIMARY SMTP VALUE.

$Collection = @()

# UPDATE THIS VARIABLE TO YOUR INPUT CSV FILE
$CSVInputFile = "c:\temp\InputCSVFile.csv"

Foreach ($Item in Import-CSV $CSVInputFile) {

        Get-Mailbox $Item.PrimarySMTPAddress | Select-Object PrimarySMTPAddress,@{Name="x500 Email Address";Expression={$_.EmailAddresses |Where-Object {$_ -match "x500:*"}}} | Foreach {
                            $UserPrimarySMTPAddress = $_.PrimarySMTPAddress
                            # $ObjectX500Addresses = $_."x500 Email Address"
                            if ($ObjectX500Addresses.count -gt 1){
                               Foreach ($X500address in $ObjectX500Addresses){ 
                                    # write-host $X500address -ForegroundColor Green
                                    $Collection += [pscustomobject]@{PrimarySMTPAddress = $UserPrimarySMTPAddress; 'x500 Email Address' = $X500address}
                                    }
                            } Elseif($ObjectX500Addresses.count -eq 1) {
                               $Collection += [pscustomobject]@{PrimarySMTPAddress = $UserPrimarySMTPAddress; 'x500 Email Address' = $ObjectX500Addresses}
                            }
                    
        } # <= end of Foreach loop after the pipe on Line 14


}

# If you want to directly dump the results:
$Collection | ft

# If you want to store it in a CSV file:
$Collection | Export-CSV -NoTypeInfo c:\temp\X500AddressesExport$(get-date -F "ddMMyyyy_hh-mm-ss").csv
```


Here is a sample output, from the exported CSV opened in Excel:

<img width="654" alt="image" src="https://github.com/SammyKrosoft/Wade-Smith-Dump-or-Export-X500-addresses/assets/33433229/7185b289-f059-46d5-979b-115da64ff23b" width = 100%>
