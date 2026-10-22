# Title
CVE-2023-48795: SSH Terrapin Prefix Truncation Vulnerability

# Vulnerable Endpoint
`176.9.70.54:22` (SSH)
# Description
The SSH service is vulnerable to the Terrapin attack (CVE-2023-48795). This protocol-level flaw allows an active man-in-the-middle (MitM) attacker to silently truncate messages during the handshake by manipulating sequence numbers. The server lacks support for the "strict key exchange" extension and/or advertises vulnerable cipher/MAC combinations (`chacha20-poly1305@openssh.com` or CBC mode with Encrypt-then-MAC).

# Steps to Reproduce

1. Run a Nuclei scan against the target:
```
nuclei -u 176.9.70.54 -id CVE-2023-48795
```
2. Alternatively, use Nmap to check advertised algorithms for vulnerable ciphers:
```
nmap -p 22 --script ssh2-enum-algos 176.9.70.54
```

# Proof of Concept
<img width="1890" height="217" alt="14" src="https://github.com/user-attachments/assets/592b2318-b512-4336-b006-4b2b27c4f1dd" />


# Impact
An attacker with a MitM position can downgrade security features, bypass integrity checks by dropping encrypted messages, and potentially force the use of weaker authentication algorithms without detection.
# Remediation
1. **Upgrade SSH:** Update the SSH software to a version that supports strict key exchange (e.g., OpenSSH 9.6+). _Note: Both client and server must support it._
2. **Workaround:** If upgrading is not immediately possible, disable vulnerable algorithms in `/etc/ssh/sshd_config` by removing `chacha20-poly1305` and any `*-etm@openssh.com` MACs. Restart the SSH service afterward.
