# Title
LDAP Anonymous Bind Enabled Leading to Information Disclosure

# Vulnerable Endpoint
`80.218.188.207:389` (LDAP)

# Description
The target host `80.218.188.207` (resolving to `80-218-188-207.dclient.hispeed.ch`) is running an OpenLDAP service on port 389 that permits anonymous (unauthenticated) bind requests. By sending a query without credentials, the server responds with a `result: 0 Success` and exposes the RootDSE (root Directory Information Tree). This allows an unauthenticated attacker to extract sensitive infrastructure metadata, including the internal naming context (domain name) and supported authentication mechanisms.

# Steps to Reproduce
1. Ensure `ldap-utils` is installed (`sudo apt install ldap-utils`).
2. Perform an anonymous base query to the LDAP RootDSE:
```
ldapsearch -x -H ldap://80.218.188.207 -s base -b "" "(objectClass=*)"
```
3. Observe that the server returns `result: 0 Success` and the `objectClass: OpenLDAProotDSE`.
4. Alternatively, use Nmap to extract further metadata:
```
nmap -p 389 --script ldap-rootdse 80.218.188.207
```
5. Observe the returned `namingContexts: dc=hoellochforschung,dc=ch` and supported SASL mechanisms.

# Proof of Concept
<img width="1920" height="1026" alt="5" src="https://github.com/user-attachments/assets/03423404-3bfd-4411-bdd1-d9b7fc8b0ac4" />

<img width="1898" height="757" alt="6" src="https://github.com/user-attachments/assets/74921169-871a-45f9-a703-78a15b782127" />

# Impact
- **Internal Infrastructure Disclosure:** The exposure of the `namingContexts` reveals the internal Active Directory/LDAP domain name (`hoellochforschung.ch`), which can be used by attackers for further reconnaissance, phishing, or targeted attacks.
- **Authentication Mechanism Enumeration:** The disclosure of supported SASL mechanisms (NTLM, DIGEST-MD5) aids attackers in understanding the environment and selecting appropriate exploitation techniques.
- **Data Leakage (Potential):** Depending on the LDAP Access Control Lists (ACLs), an attacker may be able to query the `dc=hoellochforschung,dc=ch` base to harvest usernames, email addresses, and group memberships without authentication.

# Remediation
1. **Disable Anonymous Bind:** Modify the OpenLDAP configuration (`slapd.conf` or `cn=config`) to explicitly disallow anonymous binds. Add or modify the following directive:  
    `disallow bind_anon`
2. **Restrict Anonymous Access:** If anonymous access is strictly required for specific applications, configure ACLs to ensure anonymous users can _only_ read the RootDSE and are denied access to the `namingContexts` and any user/group data.
3. **Network Segmentation:** Ensure the LDAP service (port 389 and 636) is not exposed to the public internet. Restrict access to trusted internal networks or via a VPN.
4. **Enforce Encryption:** If credentials are passed to this LDAP server, enforce LDAPS (LDAP over TLS, port 636) or StartTLS to prevent credential sniffing.
