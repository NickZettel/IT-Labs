
### Step 1: Log In to Windows Server Core

1. Start your virtual machine with Windows Server Core.
2. At the login prompt, enter your Administrator credentials.
   - Default prompt is a command-line window (cmd.exe).
   - Note: No GUI is available; all tasks will use the CLI.

### Step 2: Launch PowerShell

1. At the command prompt, type `powershell` and press Enter.
2. Verify you’re in PowerShell by checking the prompt (it should start with `PS`).
   - PowerShell is the primary tool for managing Server Core.

### Step 3: Basic System Information

1. Run the following command to display system information:

   ```
   Get-ComputerInfo
   ```
2. Observe key details like the OS version, memory, and system type.
3. Run this command to check the server’s hostname:

   ```
   hostname
   ```
   - Record the hostname for future reference.

### Step 4: Configure the Computer Name

1. Check the current computer name:

   ```
   Get-ComputerInfo -Property CsName
   ```
2. Change the computer name (replace `NewServerName` with your desired name):

   ```
   Rename-Computer -NewName "NewServerName"
   ```
3. Restart the server to apply the change:

   ```
   Restart-Computer
   ```
4. Log back in and verify the new name using `hostname`.

### Step 5: Configure Network Settings

1. List network adapters:

   ```
   Get-NetAdapter
   ```
   - Note the `Name` and `InterfaceIndex` of your active adapter.
2. Set a static IP address (replace `InterfaceIndex`, `IPAddress`, `SubnetMask`, and `Gateway` with appropriate values):

   ```
   New-NetIPAddress -InterfaceIndex <Index> -IPAddress 192.168.1.100 -PrefixLength 24 -DefaultGateway 192.168.1.1
   ```
3. Configure DNS server:

   ```
   Set-DnsClientServerAddress -InterfaceIndex <Index> -ServerAddresses ("8.8.8.8", "8.8.4.4")
   ```
4. Verify connectivity:

   ```
   ping 8.8.8.8
   ```

### Step 6: Check System Time and Time Zone

1. Display the current time and time zone:

   ```
   Get-TimeZone
   ```
2. If needed, set the time zone (e.g., to Pacific Standard Time):

   ```
   Set-TimeZone -Id "Pacific Standard Time"
   ```
3. Verify the change:

   ```
   Get-TimeZone
   ```

### Step 7: Explore PowerShell Help

1. Access PowerShell’s help system:

   ```
   Get-Help
   ```
2. Look up details for a specific command, e.g.:

   ```
   Get-Help Get-ComputerInfo -Full
   ```
   - Explore the output to understand parameters and examples.

### Step 8: Log Off

1. Exit PowerShell back to the command prompt:

   ```
   exit
   ```
2. Log off the server:

logoff