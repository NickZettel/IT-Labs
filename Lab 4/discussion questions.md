
## How does DNS support Active Directory and network services?
Dns resolves names within the network to IP addresses. Resources within a network (like a printer) are usually referenced to with their DNS name. Active Didectory also relies on DNS records to help clients find domain controllers. 

## What is the difference between an A record and a CNAME record?
An A record maps a domain name to an IP address while a CNAME record is an alias that can be used to point to that name.

## How did logging in as `letoatre` to test DNS permissions differ from using 
`Administrator`?
Logging in as an admin can cause some commands to succeed due to elevated local permissions as opposed to actually working as expected. A different user(letoatre) than admin ensures that rather than the dns server calling itsef, it can get called by a different IP address, ensuring a more realistic test.