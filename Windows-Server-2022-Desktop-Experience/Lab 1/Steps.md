### Step 1: Prepare the Server

Log in to your Windows Server 2022 as Administrator.
Open Server Manager (it should launch automatically; if not, search for it in the Start menu).
Update the server: In Server Manager, go to Dashboard > Quick Start > Configure this local server > Click Check for updates and install any available updates. Restart if prompted.
Set a static IP address (best practice for DCs to avoid DHCP issues):

Right-click the Start button > Network Connections.
Right-click your network adapter > Properties.
Select Internet Protocol Version 4 (TCP/IPv4) > Properties.
Set IP address (e.g., 192.168.1.10), Subnet mask (255.255.255.0), Default gateway (your router IP), and Preferred DNS server (use 127.0.0.1 for now, as the DC will host DNS later).
Click OK to apply.


Rename the server for clarity (e.g., DC01):

In Server Manager, click Local Server > Click the computer name > Change System Properties > Change > Enter new name > OK > Restart.



### Step 2: Install Active Directory Domain Services Role

In Server Manager, click Manage > Add Roles and Features.
In the wizard:

Before you begin: Next.
Installation Type: Role-based or feature-based installation > Next.
Server Selection: Select your local server > Next.
Server Roles: Check Active Directory Domain Services (this will prompt to add required features like .NET Framework; confirm).
Features: No additional changes needed > Next.
AD DS: Next.
Confirmation: Check Restart the destination server automatically if required > Install.


Wait for installation to complete (may take a few minutes). Do not close the wizard.

### Step 3: Promote the Server to a Domain Controller

After installation, in Server Manager, you'll see a notification flag (yellow exclamation). Click it > Promote this server to a domain controller.
In the Active Directory Domain Services Configuration Wizard:

Deployment Configuration: Select Add a new forest > Enter Root domain name (e.g., contoso.local) > Next.

Note: Use .local for labs to avoid conflicts with real domains.


Domain Controller Options:

Forest/Domain functional level: Leave as default (Windows Server 2016 or higher).
Check Domain Name System (DNS) server and Global Catalog (GC).
Enter a Directory Services Restore Mode (DSRM) password (remember this for recovery).
Next.


DNS Options: Ignore any delegation warnings (common in labs) > Next.
Additional Options: Verify NetBIOS name (e.g., CONTOSO) > Next.
Paths: Leave defaults (databases/logs on C:) > Next.
Review Options: Review and note any scripts > Next.
Prerequisites Check: Resolve any errors (warnings are usually OK) > Install.


The server will restart automatically upon completion.

### Step 4: Basic Post-Installation Configuration

After reboot, log in as the domain administrator (e.g., CONTOSO\Administrator).
Open Active Directory Users and Computers (search in Start menu or run dsa.msc).
Explore the default structure: Domain > Builtin, Computers, Domain Controllers, Users, etc.
