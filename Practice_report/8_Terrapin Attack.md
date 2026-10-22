# Title
Terrapin Attack (CVE-2023-48795) - SSH Prefix Truncation Vulnerability

# Vulnerable Endpoint
`159.69.219.59:22` (SSH)

# Description
The target SSH service is running OpenSSH version 7.6p1, which is vulnerable to the Terrapin attack (CVE-2023-48795). This is a protocol-level flaw in the SSH transport layer. An active Man-in-the-Middle (MitM) attacker can exploit this vulnerability to silently intercept and modify the initial handshake, specifically by truncating an arbitrary number of packets from the beginning of the encrypted session. This allows the attacker to downgrade the connection's security features (such as disabling keystroke timing obfuscation) without the client or server detecting the tampering, effectively breaking the integrity of the SSH session.

# Steps to Reproduce
1. Ensure Go is installed and download the dedicated Terrapin scanner:
```
go install github.com/RUB-NDS/Terrapin-Scanner@latest
```
2. Run the scanner against the target's SSH port:

```
~/go/bin/Terrapin-Scanner --connect 159.69.219.59:22
```
3. Review the output. The scanner checks if the server supports the vulnerable `ChaCha20-Poly1305` cipher and if it lacks the `strict KEX` mitigation extension.

# Proof of Concept
Scanner Output
```
================================================================================
==================================== Report ====================================
================================================================================

Remote Banner: SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.5

ChaCha20-Poly1305 support:   true
CBC-EtM support:             false

Strict key exchange support: false

The scanned peer is VULNERABLE to Terrapin.
```

<img width="1930" height="754" alt="9" src="https://github.com/user-attachments/assets/1bdb74f8-d046-4723-9a2d-e34a6c76500e" />


# Impact
- **Integrity Bypass:** The core cryptographic guarantee of SSH (that data cannot be tampered with in transit) is broken. An attacker can silently alter the negotiation phase.
- **Security Feature Downgrade:** The attacker can force the connection to drop modern security extensions (like `SSH_MSG_EXT_INFO`), potentially forcing the use of weaker authentication methods or disabling keystroke timing obfuscation.
- **Requires MitM Position:** The attacker must be positioned on the network path between the client and the server (e.g., on a shared Wi-Fi network, compromised router, or via ARP spoofing) to execute the attack.

# Remediation
1. **Upgrade OpenSSH:** Update the OpenSSH server to version **9.6 or later**. OpenSSH 9.6 introduced the "strict KEX" mode, which automatically resets sequence numbers during the handshake, completely mitigating the Terrapin attack.
2. **Disable Vulnerable Ciphers (Temporary Mitigation):** If an immediate upgrade is not possible, edit the `/etc/ssh/sshd_config` file and explicitly disable the vulnerable ciphers by removing `chacha20-poly1305@openssh.com` from the `Ciphers` list. (Note: AES-GCM ciphers are not affected).
3. **Patch Client Side:** Ensure that SSH clients connecting to this server are also updated to version 9.6+ to negotiate the strict KEX mode properly.
