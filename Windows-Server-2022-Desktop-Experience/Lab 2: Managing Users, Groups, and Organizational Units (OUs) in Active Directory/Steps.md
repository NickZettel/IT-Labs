### Step 1: Create Organizational Units (OUs)

Open Active Directory Users and Computers (dsa.msc).

Right-click contoso.local > New > Organizational Unit.

Name: "Departments", check "Protect container from accidental deletion" > OK.

Inside Departments, create sub-OUs: IT, HR, Sales (protect each).

Verify: Expand contoso.local > Departments to see IT, HR, Sales.



### Step 2: Create User Accounts

In ADUC, right-click IT OU > New > User.

Add user: Full name "John Doe", logon name "jdoe", password "P@ssw0rd123", check "User must change password at next logon".

Repeat for Jane Smith (jsmith, HR OU), Bob Johnson (bjohnson, Sales OU).

In PowerShell (as Administrator):

New-ADUser -Name "Alice Brown" -GivenName "Alice" -Surname "Brown" -SamAccountName "abrown" -UserPrincipalName "abrown@contoso.local" -Path "OU=IT,OU=Departments,DC=contoso,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true

### Step 3: Create and Manage Groups

In ADUC, right-click Departments OU > New > Group.

Create: IT_Admins (Global, Security).

Repeat: HR_Staff (HR OU), Sales_Team (Sales OU).

Add members: John Doe and Alice Brown to IT_Admins, Jane Smith to HR_Staff, Bob Johnson to Sales_Team.

In PowerShell:

Add-ADGroupMember -Identity "IT_Admins" -Members "abrown"

### Step 4: Verify and Test

In ADUC, confirm OUs contain correct users and groups.

In PowerShell:

Get-ADUser -Filter * -SearchBase "OU=Departments,DC=contoso,DC=local" | Select Name, DistinguishedName
Get-ADGroupMember -Identity "IT_Admins"

Check Event Viewer > Windows Logs > Security for AD events.

(Optional) Test user logon on a domain-joined client.
