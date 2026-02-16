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
## Syntax
#### Authenticated 
```
TheFl4m3$ ldapsearch -x -H ldap://<DC_IP> -D "<USER>@<DOMAIN>" -w '<PASSWORD>' -b "<BASE_DN>"  "<LDAP_FILTER>"  [ATTRIBUTES]

example
ldapsearch -x -H ldap://$DC -D "user@example.local" -w 'PASSWORD' -b "$BASE" "(objectClass=user)" sAMAccountName
```
#### unauthenticated 
```
TheFl4m3@htb[/htb]$ ldapsearch -H ldap://10.129.1.207 -x -b "dc=inlanefreight,dc=local"
```
## Common Enumeration Queries
The following queries represent common LDAP filters used during Active Directory enumeration.
#### Get Base DN
```
ldapsearch -x -H ldap://DC_IP -s base namingContexts
```
Retrieve the Base DN to determine the domain naming context.
#### Users
Filter -> `(&(objectCategory=person)(objectClass=user))`
```
ldapsearch -x -H ldap://10.129.18.168 -b "dc=inlanefreight,dc=local" "(&(objectClass=user)(cn=kevin*))"

User member of
ldapsearch -x -H ldap://$DC -b "$BASE" "(&(objectCategory=person)(objectClass=user)(sAMAccountName=<USER>))" memberOf
```
Used to enumerate specific user accounts or retrieve all domain users.
#### Groups
```
ldapsearch -x -H ldap://10.129.18.168 -b "dc=inlanefreight,dc=local" "(&(objectClass=group)(cn=Protected Users))" member

or

ldapsearch -x -H ldap://10.129.18.168 -b "dc=inlanefreight,dc=local" "(&(objectClass=group)(cn=Protected Users))" 

ldapsearch -x -H ldap://$DC -b "$BASE" "(&(objectClass=group)(cn=Domain Admins))" member

ldapsearch -x -H ldap://$DC -b "$BASE" "(&(objectClass=group)(cn=<GROUP_NAME>))" member
```
Used to enumerate group objects and inspect group membership.
#### Privileged Groups
Filter -> `(&(objectClass=group)(cn=Protected Users))`

```
ldapsearch -x -H ldap://DC_IP -b "DC=example,DC=local" "(&(objectClass=group)(cn=Protected Users))" member
```
#### SPNs
```
(&(objectClass=user)(servicePrincipalName=*))

count users:

ldapsearch -x -H ldap://$DC -b "$BASE" "(servicePrincipalName=*)" sAMAccountName | grep -c "^sAMAccountName:"

see users:

ldapsearch -x -H ldap://$DC -b "$BASE" "(servicePrincipalName=*)" sAMAccountName servicePrincipalName
```
Used to identify accounts with Service Principal Names (SPNs), commonly targeted in Kerberoasting attacks.
#### Delegation
##### Detect Users with Unconstrained Delegation Enabled
```
ldapsearch -x -H ldap://DC_IP -b "DC=example,DC=local" \
"(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=524288))" sAMAccountName
```
###### Breakdown
- `-x` → Simple authentication (no SASL)
- `-H ldap://DC_IP` → Specifies the Domain Controller
- `-b "DC=example,DC=local"` → Defines the Base DN
- `objectClass=user` → Filters user objects
- `userAccountControl:1.2.840.113556.1.4.803:=524288`  
    Uses a bitwise filter to identify accounts with the `TRUSTED_FOR_DELEGATION` flag enabled
- `sAMAccountName` → Returns only the username attribute

This query identifies user accounts configured with **Unconstrained Delegation** (`TRUSTED_FOR_DELEGATION` flag).

Accounts with this configuration can cache Kerberos tickets from users who authenticate to them. If exploited, this may allow attackers to extract Ticket Granting Tickets (TGTs) and perform lateral movement or privilege escalation.
#### Account Flags
Active Directory stores multiple account configuration flags inside the `userAccountControl` attribute. Bitwise LDAP filters can be used to identify accounts with specific security-relevant properties.
##### Accounts Allowing Reversible Encryption
`(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=128))`

Identifies accounts configured with **reversible password encryption** enabled.  
This setting weakens password storage security and may expose plaintext-equivalent credentials.
##### Disabled Users
`(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))`

Identifies disabled user accounts.  
While typically not directly exploitable, disabled accounts may still retain group memberships and historical privileges useful for reconnaissance.
#### Password Policy
```
ldapsearch -x -H ldap://$DC -b "$BASE" "(objectClass=domain)" minPwdLength lockoutThreshold maxPwdAge
```
Used to review domain password policy settings prior to password spraying attempts.
## Limitations

- Output can be verbose and requires manual parsing
- May require authenticated bind depending on domain configuration
- Less convenient compared to higher-level enumeration frameworks

# Role in Active Directory Methodology

ldapsearch is primarily used during the Enumeration phase to manually query LDAP objects and validate findings from automated tools.

Typical workflow:

1. Identify Base DN
2. Enumerate users and groups
3. Identify privileged accounts
4. Extract SPNs
5. Check delegation settings
6. Review password policy

This structured enumeration enables informed decision-making before moving to credential attacks, privilege escalation, or lateral movement.
