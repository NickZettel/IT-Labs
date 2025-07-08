

### Step 1: Log In and Launch PowerShell

1. Start your first VM (the domain controller) and log in as `rakis\Administrator`.
2. At the command prompt, type `powershell` and press Enter to switch to PowerShell.

### Step 2: Install the DHCP Server Role

1. Check available server roles:

   ```
   Get-WindowsFeature | Where-Object {$_.Name -like "DHCP*"}
   ```
   - Look for `DHCP` in the output.
2. Install the DHCP server role with management tools:

   ```
   Install-WindowsFeature -Name DHCP -IncludeManagementTools
   ```
3. Verify the installation:

   ```
   Get-WindowsFeature -Name DHCP
   ```
   - Ensure `[X]` appears under `Installed`.

### Step 3: Authorize the DHCP Server in Active Directory

1. Authorize the DHCP server in the `rakis.local` domain:

   ```
   Add-DhcpServerInDC -DnsName $env:COMPUTERNAME.rakis.local -IPAddress 192.168.1.100
   ```
   - Replace `192.168.1.100` with your server’s static IP from Lab 1.
2. Verify authorization:

   ```
   Get-DhcpServerInDC
   ```
   - Confirm your server appears in the list.

### Step 4: Create a DHCP Scope

1. Create a DHCP scope for IP address assignment (e.g., 192.168.1.101–192.168.1.200):

   ```
   Add-DhcpServerv4Scope -Name "RakisScope" -StartRange 192.168.1.101 -EndRange 192.168.1.200 -SubnetMask 255.255.255.0 -State Active
   ```
2. Configure the default gateway (replace `192.168.1.1` if different):

   ```
   Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -Router 192.168.1.1
   ```
3. Set DNS server (use your domain controller’s IP, e.g., 192.168.1.100):

   ```
   Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -DnsServer 192.168.1.100 -DnsDomain rakis.local
   ```
4. Verify the scope:

   ```
   Get-DhcpServerv4Scope
   ```

### Step 5: Configure the Second VM for DHCP

1. Start your second VM and log in as `rakis\letoatre`.
2. Switch to PowerShell:

   ```
   powershell
   ```
3. Configure the network adapter to obtain an IP address via DHCP:

   ```
   Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Enabled
   ```
   - Replace `Ethernet` with your adapter name (check with `Get-NetAdapter`).
4. Release and renew the IP address:

   ```
   ipconfig /release
   ipconfig /renew
   ```
5. Verify the assigned IP address:

   ```
   Get-NetIPAddress -InterfaceAlias "Ethernet"
   ```
   - Confirm the IP is within the scope (e.g., 192.168.1.101–192.168.1.200).

### Step 6: Test DHCP Functionality

1. On the second VM, test connectivity to the domain controller:

   ```
   ping 192.168.1.100
   ```
2. Verify DNS resolution:

   ```
   nslookup rakis.local
   ```
   - Ensure it resolves to 192.168.1.100.
3. On the first VM (domain controller), check leased IP addresses:

   ```
   Get-DhcpServerv4Lease -ScopeId 192.168.1.0
   ```
   - Look for the second VM’s assigned IP and MAC address.

### Step 7: Assign DHCP Permissions to GodEmperor Group

1. On the first VM, grant the `GodEmperor` group permissions to manage DHCP:

   ```
   Add-DhcpServerSecurityGroup -ComputerName $env:COMPUTERNAME.rakis.local
   Add-ADGroupMember -Identity "DhcpAdmins" -Members "GodEmperor"
   ```
2. Log in to the second VM as `rakis\letoatre` and test DHCP management:

   ```
   Get-DhcpServerv4Scope -ComputerName <DC-Name>.rakis.local
   ```
   - Replace `<DC-Name>` with your domain controller’s hostname.

### Step 8: Log Off

1. On both VMs, exit PowerShell:

   ```
   exit
   ```
2. Log off:

   ```
   logoff
   ```
