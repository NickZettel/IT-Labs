
### Step 1: Log In and Launch PowerShell

1. Start your domain controller VM and log in as `rakis\Administrator`.
2. At the command prompt, type `powershell` and press Enter to switch to PowerShell.

### Step 2: Verify Existing DNS Role

1. Since Lab 2 included the `-InstallDns` parameter when promoting the domain controller, check if the DNS server role is already installed:

   ```
   Get-WindowsFeature -Name DNS
   ```
   - If `[X]` appears under `Installed`, skip to Step 3.
2. If not installed, install the DNS server role with management tools:

   ```
   Install-WindowsFeature -Name DNS -IncludeManagementTools
   ```
3. Verify the DNS service is running:

   ```
   Get-Service -Name DNS
   ```
   - If stopped, start it: `Start-Service -Name DNS`.

### Step 3: Verify the Active Directory-Integrated DNS Zone

1. Check for the existing `rakis.local` DNS zone (created automatically during AD DS promotion):

   ```
   Get-DnsServerZone -Name "rakis.local"
   ```
   - Confirm it exists and is `AD-Integrated`.
2. List DNS records in the zone:

   ```
   Get-DnsServerResourceRecord -ZoneName "rakis.local"
   ```
   - Note the default A, NS, and SOA records.

### Step 4: Create a New Forward Lookup Zone

1. Create a new forward lookup zone for a subdomain (e.g., `spice.rakis.local`):

   ```
   Add-DnsServerPrimaryZone -Name "spice.rakis.local" -ZoneFile "spice.rakis.local.dns"
   ```
2. Verify the zone:

   ```
   Get-DnsServerZone -Name "spice.rakis.local"
   ```

### Step 5: Add DNS Records

1. Add an A record for a fictional server in the new zone (e.g., `arrakis.spice.rakis.local`):

   ```
   Add-DnsServerResourceRecordA -ZoneName "spice.rakis.local" -Name "arrakis" -IPv4Address "192.168.1.101"
   ```
2. Add a CNAME record aliasing `dune` to `arrakis.spice.rakis.local`:

   ```
   Add-DnsServerResourceRecordCName -ZoneName "spice.rakis.local" -Name "dune" -HostNameAlias "arrakis.spice.rakis.local"
   ```
3. Verify the records:

   ```
   Get-DnsServerResourceRecord -ZoneName "spice.rakis.local"
   ```

### Step 6: Test DNS Resolution as Administrator

1. Test name resolution for the new records:

   ```
   Resolve-DnsName -Name "arrakis.spice.rakis.local"
   Resolve-DnsName -Name "dune.spice.rakis.local"
   ```
   - Confirm both resolve to `192.168.1.101`.

### Step 7: Test DNS Resolution as Letoatre

1. Log off the server:

   ```
   logoff
   ```
2. Log in as `rakis\letoatre` (using the password set in Lab 2, e.g., `P@ssw0rd456`).
3. Launch PowerShell:

   ```
   powershell
   ```
4. Test DNS resolution:

   ```
   nslookup arrakis.spice.rakis.local
   nslookup dune.spice.rakis.local
   ```
   - Verify both resolve to `192.168.1.101`.
5. Log off and log back in as `rakis\Administrator`.

### Step 8: Assign DNS Permissions to GodEmperor Group

1. Grant the `GodEmperor` group permissions to manage DNS:

   ```
   Add-DnsServerResourceRecord -ZoneName "rakis.local" -A -Name "test" -IPv4Address "192.168.1.102" -AllowUpdateAny
   $acl = Get-DnsServerZone -Name "rakis.local" | Get-Acl
   $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("rakis\GodEmperor", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
   $acl.AddAccessRule($rule)
   Set-Acl -Path "AD:DC=rakis.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=rakis,DC=local" -AclObject $acl
   ```
2. Log off, log back in as `rakis\letoatre`, and test adding a DNS record:

   ```
   powershell
   Add-DnsServerResourceRecordA -ZoneName "rakis.local" -Name "fremen" -IPv4Address "192.168.1.103"
   ```
3. Log off and log back in as `rakis\Administrator`, then verify:

   ```
   Get-DnsServerResourceRecord -ZoneName "rakis.local" -Name "fremen"
   ```

### Step 9: Explore DNS PowerShell Commands

1. List all DNS zones:

   ```
   Get-DnsServerZone
   ```
2. Explore DNS cmdlets:

   ```
   Get-Help *DnsServer*
   ```

### Step 10: Log Off

1. Exit PowerShell:

   ```
   exit
   ```
2. Log off:

   ```
   logoff
   ```