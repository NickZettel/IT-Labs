Step 1: Log In to Windows Server Core





Start your virtual machine with Windows Server Core.



At the login prompt, enter your Administrator credentials.





Default prompt is a command-line window (cmd.exe).



Note: No GUI is available; all tasks will use the CLI.

Step 2: Launch PowerShell





At the command prompt, type powershell and press Enter.



Verify you’re in PowerShell by checking the prompt (it should start with PS).





PowerShell is the primary tool for managing Server Core.

Step 3: Basic System Information





Run the following command to display system information:

Get-ComputerInfo



Observe key details like the OS version, memory, and system type.



Run this command to check the server’s hostname:

hostname





Record the hostname for future reference.

Step 4: Configure the Computer Name





Check the current computer name:

Get-ComputerInfo -Property CsName



Change the computer name (replace NewServerName with your desired name):

Rename-Computer -NewName "NewServerName"



Restart the server to apply the change:

Restart-Computer



Log back in and verify the new name using hostname.

Step 5: Configure Network Settings





List network adapters:

Get-NetAdapter





Note the Name and InterfaceIndex of your active adapter.



Set a static IP address (replace InterfaceIndex, IPAddress, SubnetMask, and Gateway with appropriate values):

New-NetIPAddress -InterfaceIndex <Index> -IPAddress 192.168.1.100 -PrefixLength 24 -DefaultGateway 192.168.1.1



Configure DNS server:

Set-DnsClientServerAddress -InterfaceIndex <Index> -ServerAddresses ("8.8.8.8", "8.8.4.4")



Verify connectivity:

ping 8.8.8.8

Step 6: Check System Time and Time Zone





Display the current time and time zone:

Get-TimeZone



If needed, set the time zone (e.g., to Pacific Standard Time):

Set-TimeZone -Id "Pacific Standard Time"



Verify the change:

Get-TimeZone

Step 7: Explore PowerShell Help





Access PowerShell’s help system:

Get-Help



Look up details for a specific command, e.g.:

Get-Help Get-ComputerInfo -Full





Explore the output to understand parameters and examples.

Step 8: Log Off





Exit PowerShell back to the command prompt:

exit



Log off the server:

logoff