## What is the role of a domain controller in a network, and why is it important for network authentication (Network+ Objective 1.4)?

The role of a domain controller in a network is to authenticate users and authorize them to access resources. The direct controller holds the Active Directory database, containing accounts, groups, and security policies, and this database is consulted to validate access. Domain controllers often runs a DNS server so clients can connect to the right services. The importance of a DC is that it provides full control over user environments, offers a central login for all users, and enhances security.


## How does managing users and groups in Active Directory differ from local user management?

In local user management, all user information is stored on the individual device, and the user cannot log in to other devices. With Active Directory, user accounts are stored centrally on a domain controller, allowing users to log in from any domain-joined device. Security groups in AD can be used across the entire network, while local groups are limited to a single machine.


## What challenges did you face working with AD DS in Server Core’s command-line environment?

The biggest challenge with this lab was to spinning up a second system to test logging in as Leto. The second VM had firewall issues, and the commands to fix it with PowerShell were fairly lengthy, so it was important to check carefully for any mistakes before submitting.  