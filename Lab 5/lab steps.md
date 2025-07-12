
### Step 1: Log In and Launch PowerShell

1. Start your domain controller VM and log in as `rakis\Administrator`.
2. At the command prompt, type `powershell` and press Enter to switch to PowerShell.

### Step 2: Fix and Complete DNS Permissions for GodEmperor

1. Correctly set permissions for the `GodEmperor` group to manage DNS records in `rakis.local`:

   ```
   $acl = Get-Acl -Path "AD:DC=rakis.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=rakis,DC=local"
   $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("rakis\GodEmperor", "Write", "ContainerInherit,ObjectInherit", "None", "Allow")
   $acl.AddAccessRule($rule)
   Set-Acl -Path "AD:DC=rakis.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=rakis,DC=local" -AclObject $acl
   ```
2. Verify the ACL:

   ```
   Get-Acl -Path "AD:DC=rakis.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=rakis,DC=local" | Format-List
   ```
   - Look for `rakis\GodEmperor` with `Write` permissions.

3. Log off and log in as `rakis\letoatre`:

   ```
   logoff
   ```
4. In PowerShell, test creating a DNS record:

   ```
   powershell
   Add-DnsServerResourceRecordA -ZoneName "rakis.local" -Name "sietch" -IPv4Address "192.168.1.103"
   ```
5. Log off and log back in as `rakis\Administrator`, then verify:

   ```
   Get-DnsServerResourceRecord -ZoneName "rakis.local" -Name "sietch"
   ```

### Step 3: Troubleshoot DNS Resolution

1. Test DNS resolution for `crawlers.spice.rakis.local` and `harvesters.spice.rakis.local`:

   ```
   Resolve-DnsName -Name "crawlers.spice.rakis.local" -Server 192.168.1.100
   Resolve-DnsName -Name "harvesters.spice.rakis.local" -Server 192.168.1.100
   ```
2. Test UDP-specific resolution with `nslookup`:

   ```
   nslookup crawlers.spice.rakis.local 192.168.1.100
   nslookup harvesters.spice.rakis.local 192.168.1.100
   ```
   - If it times out again, enable verbose DNS logging:
     ```
     Set-DnsServerDiagnostics -All $true
     Get-Content -Path "C:\Windows\System32\dns\dns.log" -Tail 10
     ```
   - Look for queries from `127.0.0.1` or errors indicating UDP issues.

3. Log in as `rakis\letoatre` and repeat the `nslookup` tests to confirm consistency.

### Step 4: Create a Group Policy Object

1. Create a GPO to enforce a desktop setting (e.g., disable Control Panel) for the `SpiceUsers` OU:

   ```
   New-GPO -Name "SpiceUsersPolicy" | New-GPLink -Target "OU=SpiceUsers,DC=rakis,DC=local"
   ```
2. Configure the GPO to disable Control Panel:

   ```
   Set-GPRegistryValue -Name "SpiceUsersPolicy" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoControlPanel" -Type DWord -Value 1
   ```
3. Verify the GPO:

   ```
   Get-GPO -Name "SpiceUsersPolicy"
   Get-GPInheritance -Target "OU=SpiceUsers,DC=rakis,DC=local"
   ```

### Step 5: Test GPO as Letoatre

1. Log off and log in as `rakis\letoatre`.
2. Force GPO update:

   ```
   powershell
   gpupdate /force
   ```
3. Verify the GPO effect (since Server Core lacks a GUI, check registry):

   ```
   Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoControlPanel"
   ```
   - Confirm `NoControlPanel` is set to `1`.
4. Log off and log back in as `rakis\Administrator`.

### Step 6: Explore Group Policy Commands

1. List all GPOs:

   ```
   Get-GPO -All
   ```
2. Explore GPO cmdlets:

   ```
   Get-Help *GPO*
   ```

### Step 7: Log Off

1. Exit PowerShell:

   ```
   exit
   ```
2. Log off:

   ```
   logoff
   ```