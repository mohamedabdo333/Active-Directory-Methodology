## Overview

`ldapsearch` is a command-line utility used to query LDAP directories. 
In Active Directory environments, it allows direct interaction with the Domain Controller to retrieve directory objects such as users, groups, computers, and domain configuration data.

During penetration testing, ldapsearch is particularly useful when:

- Performing low-noise enumeration
- Anonymous bind is allowed
- Operating from a Linux attack machine
- Manual LDAP filtering is required
- Verifying results obtained from automated tools

Unlike higher-level tools such as PowerView or BloodHound, ldapsearch provides raw LDAP output, giving deeper visibility into object attributes and directory structure.

It is commonly used during the Enumeration phase to identify:

- Domain Base DN
- User accounts
- Group memberships
- Service Principal Names (SPNs)
- Delegation settings
- Password policy attributes
