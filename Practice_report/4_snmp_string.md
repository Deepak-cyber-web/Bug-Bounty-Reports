# Security Finding Report: SNMP Default Community String

## 1. Overview

During a security assessment of the target host `13.60.237.224`, the Nuclei scanner, using the template `snmpv1-community-detect-string`, detected that the SNMP service (UDP 161) on this host is using the default read-only community string `public`. An attacker can read system information without authentication, constituting an information disclosure risk.

## 2. Finding Details

| Item | Value |
|---|---|
| Target IP | 13.60.237.224 |
| Service / Port | SNMP / UDP 161 |
| Protocol Version | SNMP v1/v2c |
| Detection Template | `snmpv1-community-detect-string` (javascript) |
| Severity (Scanner) | High |
| Recommended Severity (Assessment) | Medium |
| Matched Community String | `public` |
| Returned Data | FQDN=`13.60.237.224`, Hostname=`13.60.237.224:161`, community_string=`public` |

## 3. Root Cause and Risk Analysis

In SNMP v1/v2c, the community string is the sole authentication credential and is transmitted in plaintext. Using the default value `public` (read-only) or `private` (read-write) is a typical insecure configuration weakness (CWE-200 Information Exposure / CWE-1188 Insecure Default Configuration).

**Potential impact:**

- Reading system description, OS version, running processes (including command-line arguments), network interfaces, and routing tables — intelligence useful for follow-on attacks.
- If a writable community string (e.g., `private`) exists, device configuration could be modified.
- Violates security baseline requirements and may be flagged as non-compliant in audits.

## 4. Evidence and Reproduction

**Scan command:**

```bash
nuclei -u 13.60.237.224 -tags default-login,crushftp,vuln
```

**Manual verification command (authorized environments only):**

```
snmpwalk -v2c -c public 13.60.237.224 1.3.6.1.2.1.1
```
If OID values such as `sysDescr` or `sysName` are returned, the `public` string is confirmed valid.

# 5. Scope and Limitations

- This finding proves the SNMP service is anonymously readable; it **does not** equate to exploitation of a specific CVE for privilege escalation or code execution.
- Actual harm depends on the sensitivity of exposed information and the host's role in the network (core device vs. edge host, internet-facing or not).
- If UDP 161 is restricted to the internal network with existing access controls, the actual exposure is correspondingly reduced.

# 6. Remediation Recommendations

1. **Change community strings**: Replace `public`/`private` with high-entropy random strings and rotate them periodically.
2. **Upgrade the protocol**: Prefer SNMP v3 with authentication (authNoPriv/authPriv) and encryption (priv) to avoid plaintext credentials.
3. **Access control**: Restrict UDP 161 to management subnets via ACLs or firewalls; disable SNMP entirely if not needed.
4. **Least privilege**: For read-only scenarios, configure a read-only view and strictly limit accessible MIB subtrees.
5. **Baseline enforcement**: Include default community string checks in configuration baselines and re-scan periodically.

# 7. Conclusion
The finding is **valid**: the target host's SNMP service uses the default read-only community string `public`, posing an information disclosure risk. Recommend tracking remediation at Medium severity and re-testing after the fix to confirm `snmpwalk` no longer returns data.
