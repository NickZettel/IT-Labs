### Step 1: Log In and Launch PowerShell

1. Start your Windows Server Core VM and log in with Administrator credentials.
2. At the command prompt, type `powershell` and press Enter to switch to PowerShell.

### Step 2: Install Active Directory Domain Services Role

1. Check available server roles:

   ```
   Get-WindowsFeature | Where-Object {$_.Name -like "AD*"}
   ```
   - Look for `AD-Domain-Services` in the output.
2. Install the AD DS role:

   ```
   Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
   ```
3. Verify the installation:

   ```
   Get-WindowsFeature -Name AD-Domain-Services
   ```
   - Ensure `[X]` appears under `Installed`.

### Step 3: Promote the Server to a Domain Controller

1. Configure the server as a domain controller for a new domain (replace `yourdomain.local` with your desired domain name):

   ```
   Install-ADDSForest -DomainName "yourdomain.local" -InstallDns -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Force
   ```
   - Note: Replace `P@ssw0rd123` with a secure password for Directory Services Restore Mode.
2. The server will restart automatically after promotion.
3. Log back in using the domain credentials (e.g., `yourdomain\Administrator`).

### Step 4: Verify Active Directory Installation

1. Confirm the server is a domain controller:

   ```
   Get-ADDomainController
   ```
   - Check the output for your server’s hostname and domain.
2. Verify DNS is operational (since DNS was installed with `-InstallDns`):

   ```
   nslookup yourdomain.local
   ```
   - Ensure it resolves to your server’s IP address.

### Step 5: Create a User Account

1. Create a new user account (replace `jdoe` with desired username):

   ```
   New-ADUser -Name "John Doe" -SamAccountName "jdoe" -UserPrincipalName "jdoe@yourdomain.local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd456" -AsPlainText -Force) -Enabled $true -PassThru
   ```
2. Verify the user was created:

   ```
   Get-ADUser -Filter {SamAccountName -eq "jdoe"}
   ```

### Step 6: Create a Security Group and Add the User

1. Create a new security group:

   ```
   New-ADGroup -Name "ITAdmins" -GroupScope Global -GroupCategory Security -PassThru
   ```
2. Add the user to the group:

   ```
   Add-ADGroupMember -Identity "ITAdmins" -Members "jdoe"
   ```
3. Verify group membership:

   ```
   Get-ADGroupMember -Identity "ITAdmins"
   ```

### Step 7: Test User Login (Optional)

1. Log off the server:

   ```
   logoff
   ```
2. Log in as the new user (`yourdomain\jdoe`) to verify the account works.
   - Note: You may need a second system or remote desktop tool, as Server Core lacks a GUI login screen.
3. Log back in as Administrator to continue.

### Step 8: Explore Active Directory PowerShell Commands

1. List all AD users:

   ```
   Get-ADUser -Filter *
   ```
2. List all AD groups:

   ```
   Get-ADGroup -Filter *
   ```
3. Use PowerShell help to explore AD commands:

   ```
   Get-Help *AD* -Category Cmdlet
   ```