### Step 1: Log In and Launch PowerShell

1. Start your domain controller VM and log in as `rakis\Administrator`.
2. At the command prompt, type `powershell` and press Enter to switch to PowerShell.

### Step 2: Install File Server Role

1. Check if the File Server role is installed (it may be included by default):

   ```
   Get-WindowsFeature -Name FS-FileServer
   ```
   - If not installed (`[ ]`), install it:
     ```
     Install-WindowsFeature -Name FS-FileServer -IncludeManagementTools
     ```
2. Verify installation:

   ```
   Get-WindowsFeature -Name FS-FileServer
   ```
   - Confirm `[X]` under `Installed`.

### Step 3: Create Additional Shares and Folders

1. Create a new directory for a second share:

   ```
   New-Item -Path "C:\DuneData" -ItemType Directory
   ```
2. Create a new SMB share named `DuneShare`:

   ```
   New-SmbShare -Name "DuneShare" -Path "C:\DuneData" -FullAccess "rakis\GodEmperor" -ReadAccess "rakis\Domain Users"
   ```
3. Create subfolders in `C:\SpiceData` (from Lab 3) for granular permissions:

   ```
   New-Item -Path "C:\SpiceData\Public" -ItemType Directory
   New-Item -Path "C:\SpiceData\Restricted" -ItemType Directory
   ```

### Step 4: Configure Advanced NTFS Permissions

1. Set NTFS permissions on `C:\SpiceData\Public` to allow `Domain Users` read/write access:

   ```
   $acl = Get-Acl -Path "C:\SpiceData\Public"
   $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("rakis\Domain.Users", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
   $acl.SetAccessRule($rule)
   Set-Acl -Path "C:\SpiceData\Public" -AclObject $acl
   ```
2. Set NTFS permissions on `C:\SpiceData\Restricted` to allow only `GodEmperor` modify access:

   ```
   $acl = Get-Acl -Path "C:\SpiceData\Restricted"
   $acl.SetAccessRuleProtection($true, $false) # Disable inheritance
   $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("rakis\GodEmperor", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
   $acl.SetAccessRule($rule)
   Set-Acl -Path "C:\SpiceData\Restricted" -AclObject $acl
   ```
3. Verify NTFS permissions:

   ```
   Get-Acl -Path "C:\SpiceData\Public" | Format-List
   Get-Acl -Path "C:\SpiceData\Restricted" | Format-List
   ```

### Step 5: Test Share Access as Letoatre on Server VM

1. Log off and log in as `rakis\letoatre` (password: `P@ssw0rd456`):

   ```
   logoff
   ```
2. In PowerShell, test access to shares:

   ```
   powershell
   Test-Path "\\localhost\SpiceShare\Public"
   Test-Path "\\localhost\SpiceShare\Restricted"
   ```
   - Both should return `True`, as `letoatre` is in `GodEmperor`.
3. Create a test file in both folders:

   ```
   New-Item -Path "\\localhost\SpiceShare\Public\test.txt" -ItemType File
   New-Item -Path "\\localhost\SpiceShare\Restricted\test.txt" -ItemType File
   ```
   - Both should succeed.
4. Log off and log back in as `rakis\Administrator`.

### Step 6: Create a Test User and Test Restricted Access

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
