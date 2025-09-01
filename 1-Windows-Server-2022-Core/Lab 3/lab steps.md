### Step 1: Log In and Launch PowerShell

1. Start your domain controller VM and log in as `rakis\Administrator`.
2. At the command prompt, type `powershell` and press Enter to switch to PowerShell.

### Step 2: Create an Organizational Unit (OU)

1. Create a new OU named `SpiceUsers` in the `rakis.local` domain:

   ```
   New-ADOrganizationalUnit -Name "SpiceUsers" -Path "DC=rakis,DC=local" -ProtectedFromAccidentalDeletion $true
   ```
2. Verify the OU was created:

   ```
   Get-ADOrganizationalUnit -Filter {Name -eq "SpiceUsers"}
   ```

### Step 3: Move User to the OU

1. Move the `letoatre` user to the `SpiceUsers` OU:

   ```
   Get-ADUser -Identity "letoatre" | Move-ADObject -TargetPath "OU=SpiceUsers,DC=rakis,DC=local"
   ```
2. Confirm the user’s new location:

   ```
   Get-ADUser -Identity "letoatre" -Properties DistinguishedName
   ```
   - Look for `OU=SpiceUsers` in the `DistinguishedName`.

### Step 4: Configure a Password Policy

1. Create a new Password Settings Object (PSO) for stricter password requirements:

   ```
   New-ADFineGrainedPasswordPolicy -Name "SpiceUsersPSO" -Precedence 10 -MinPasswordLength 12 -MaxPasswordAge "30.00:00:00" -PasswordHistoryCount 5 -ComplexityEnabled $true
   ```
2. Apply the PSO to the `SpiceUsers` OU:

   ```
   Add-ADFineGrainedPasswordPolicySubject -Identity "SpiceUsersPSO" -Subjects (Get-ADUser -SearchBase "OU=SpiceUsers,DC=rakis,DC=local" -Filter *)
   ```
3. Verify the PSO:

   ```
   Get-ADFineGrainedPasswordPolicy -Identity "SpiceUsersPSO"
   ```

### Step 5: Create a File Share

1. Create a directory for the file share:

   ```
   New-Item -Path "C:\SpiceData" -ItemType Directory
   ```
2. Create an SMB share named `SpiceShare`:

   ```
   New-SmbShare -Name "SpiceShare" -Path "C:\SpiceData" -FullAccess "rakis\GodEmperor"
   ```
3. Verify the share:

   ```
   Get-SmbShare -Name "SpiceShare"
   ```

### Step 6: Configure NTFS Permissions

1. Set NTFS permissions to allow `GodEmperor` modify access:

   ```
   $acl = Get-Acl -Path "C:\SpiceData"
   $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("rakis\GodEmperor", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
   $acl.SetAccessRule($rule)
   Set-Acl -Path "C:\SpiceData" -AclObject $acl
   ```
2. Verify NTFS permissions:

   ```
   Get-Acl -Path "C:\SpiceData" | Format-List
   ```

### Step 7: Test Permissions as Letoatre (Simulation)

1. Since you’re using a single VM, simulate `letoatre`’s access by checking permissions:

   ```
   Get-SmbShareAccess -Name "SpiceShare"
   ```
   - Confirm `GodEmperor` has `Full` access.
2. Verify the password policy applies to `letoatre`:

   ```
   Get-ADUser -Identity "letoatre" -Properties * | Select-Object -Property msDS-PSOApplied
   ```
   - Look for `SpiceUsersPSO`.

### Step 8: Explore AD PowerShell Commands

1. List all OUs in the domain:

   ```
   Get-ADOrganizationalUnit -Filter *
   ```
2. Explore password policy commands:

   ```
   Get-Help *PasswordPolicy*
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
