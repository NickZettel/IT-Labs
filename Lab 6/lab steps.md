### Step 1: Log In and Launch PowerShell

1. Start your domain controller VM and log in as `rakis\Administrator`.
2. At the command prompt, type `powershell` and press Enter to switch to PowerShell.

### Step 2: Fix and Complete DNS Permissions for GodEmperor

1. Ensure the Active Directory module is loaded:

   ```
   Import-Module ActiveDirectory
   Get-Module -ListAvailable -Name ActiveDirectory
   ```
2. Initialize the AD drive:

   ```
   Remove-PSDrive -Name AD -Force -ErrorAction SilentlyContinue
   New-PSDrive -Name AD -PSProvider ActiveDirectory -Root "" -Scope Global
   ```
3. Set permissions for `GodEmperor` to manage DNS records in `rakis.local`:

   ```
   $acl = Get-Acl -Path "AD:DC=rakis.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=rakis,DC=local"
   $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("rakis\GodEmperor", "Write", "ContainerInherit,ObjectInherit", "None", "Allow")
   $acl.AddAccessRule($rule)
   Set-Acl -Path "AD:DC=rakis.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=rakis,DC=local" -AclObject $acl
   ```
4. Verify the ACL:

   ```
   Get-Acl -Path "AD:DC=rakis.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=rakis,DC=local" | Format-List
   ```
   - Look for `rakis\GodEmperor` with `Write` permissions.

5. Log off and log in as `rakis\letoatre` (password: `P@ssw0rd456`):

   ```
   logoff
   ```
6. In PowerShell, test creating a DNS record:

   ```
   powershell
   Add-DnsServerResourceRecordA -ZoneName "rakis.local" -Name "sietch" -IPv4Address "192.168.1.103"
   ```
7. Log off and log back in as `rakis\Administrator`, then verify:

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
3. If `nslookup` times out, enable DNS debug logging:

   ```
   Set-DnsServerDiagnostics -All $true
   Get-Content -Path "C:\Windows\System32\dns\dns.log" -Tail 20
   ```
   - Look for queries from `127.0.0.1` or errors indicating UDP issues.
4. Log in as `rakis\letoatre` and repeat the `nslookup` tests to confirm consistency.

### Step 4: Create a Group Policy Object

1. Create a GPO to enforce a security setting (e.g., enforce password-protected screensaver) for the `SpiceUsers` OU:

   ```
   New-GPO -Name "SpiceUsersPolicy" | New-GPLink -Target "OU=SpiceUsers,DC=rakis,DC=local"
   ```
2. Configure the GPO to enable a password-protected screensaver (registry-based for Server Core):

   ```
   Set-GPRegistryValue -Name "SpiceUsersPolicy" -Key "HKCU\Software\Policies\Microsoft\Windows\Control Panel\Desktop" -ValueName "ScreenSaveActive" -Type DWord -Value 1
   Set-GPRegistryValue -Name "SpiceUsersPolicy" -Key "HKCU\Software\Policies\Microsoft\Windows\Control Panel\Desktop" -ValueName "ScreenSaverIsSecure" -Type DWord -Value 1
   Set-GPRegistryValue -Name "SpiceUsersPolicy" -Key "HKCU\Software\Policies\Microsoft\Windows\Control Panel\Desktop" -ValueName "ScreenSaveTimeOut" -Type DWord -Value 900
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
3. Verify the GPO settings (check registry, as Server Core lacks a GUI):

   ```
   Get-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\Control Panel\Desktop" -Name "ScreenSaveActive","ScreenSaverIsSecure","ScreenSaveTimeOut"
   ```
   - Confirm `ScreenSaveActive=1`, `ScreenSaverIsSecure=1`, `ScreenSaveTimeOut=900`.
4. Log off and log back in as `rakis\Administrator`.

### Step 6: Bonus – DHCP Troubleshooting Prep

1. Verify DHCP service status on the server:

   ```
   Get-Service -Name DHCPServer
   Restart-Service -Name DHCPServer
   ```
2. Check DHCP scope and leases:

   ```
   Get-DhcpServerv4Scope
   Get-DhcpServerv4Lease -ScopeId 192.168.1.0
   ```
3. Enable DHCP audit logging:

   ```
   Set-DhcpServerAuditLog -Enable $true
   Get-Content -Path "C:\Windows\System32\dhcp\DhcpSrvLog-*.log" -Tail 20
   ```
   - Look for DHCP Discover/Request packets from the client VM’s MAC address.

### Step 7: Explore Group Policy Commands

1. List all GPOs:

   ```
   Get-GPO -All
   ```
2. Explore GPO cmdlets:

   ```
   Get-Help *GPO*
   ```

### Step 8: Log Off

1. Exit PowerShell:

   ```
   exit
   ```
2. Log off:

   ```
   logoff
   ```