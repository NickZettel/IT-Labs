
1. Create a new user `paulatre` outside the `GodEmperor` group:

   ```
   New-ADUser -Name "Paul Atreides" -SamAccountName "paulatre" -UserPrincipalName "paulatre@rakis.local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd789" -AsPlainText -Force) -Enabled $true -Path "OU=SpiceUsers,DC=rakis,DC=local"
   ```
2. Log off and log in as `rakis\paulatre`:

   ```
   logoff
   ```
3. Test access to shares:

   ```
   powershell
   Test-Path "\\localhost\SpiceShare\Public"
   Test-Path "\\localhost\SpiceShare\Restricted"
   ```
   - `Public` should return `True`, `Restricted` should return `False` or fail.
4. Try creating a file in `Public`:

   ```
   New-Item -Path "\\localhost\SpiceShare\Public\paul.txt" -ItemType File
   ```
   - Should succeed.
5. Try creating a file in `Restricted`:

   ```
   New-Item -Path "\\localhost\SpiceShare\Restricted\paul.txt" -ItemType File
   ```
   - Should fail due to permissions.
6. Log off and log back in as `rakis\Administrator`.

### Step 7: Optional – Test Share Access from Client VM

1. Start the client VM and log in as `rakis\letoatre`.
2. Verify the client’s DHCP-assigned IP:

   ```
   Get-NetIPAddress -InterfaceAlias "Ethernet"
   ```
   - Should be in `192.168.1.101–192.168.1.200`.
3. Test share access:

   ```
   Test-Path "\\192.168.1.100\SpiceShare\Public"
   Test-Path "\\192.168.1.100\SpiceShare\Restricted"
   New-Item -Path "\\192.168.1.100\SpiceShare\Public\client.txt" -ItemType File
   New-Item -Path "\\192.168.1.100\SpiceShare\Restricted\client.txt" -ItemType File
   ```
   - Both should succeed for `letoatre`.
4. Log off and log in as `rakis\paulatre` on the client VM, then repeat the tests:

   ```
   Test-Path "\\192.168.1.100\SpiceShare\Public"
   Test-Path "\\192.168.1.100\SpiceShare\Restricted"
   New-Item -Path "\\192.168.1.100\SpiceShare\Public\paulclient.txt" -ItemType File
   New-Item -Path "\\192.168.1.100\SpiceShare\Restricted\paulclient.txt" -ItemType File
   ```
   - `Public` should succeed, `Restricted` should fail.

### Step 8: Explore File Server Commands

1. List all SMB shares:

   ```
   Get-SmbShare
   ```
2. Explore SMB cmdlets:

   ```
   Get-Help *Smb*
   ```

### Step 9: Log Off

1. Exit PowerShell:

   ```
   exit
   ```
2. Log off:

   ```
   logoff
   ```