# Title
Unauthenticated Jenkins Access Exposing CI/CD Pipelines and Build Information

# Vulnerable Endpoint
`https://34.88.16.212/`

# Description
The target host at `34.88.16.212` is running an Apache Jenkins server (version 2.568.3) that allows unauthenticated access to its web dashboard. The Nuclei scanner flagged this via the `unauthenticated-jenkins` template, and manual verification confirms that any user can view the main dashboard, active projects (specifically Apache Cassandra builds), build histories, and build statuses without needing to log in. Furthermore, the exposed UI displays an accessible "Sign in" button, indicating security is configured but not enforced for read access, and potentially exposing sensitive CI/CD metadata and the `REST API` endpoint to unauthenticated users.

# Steps to Reproduce
1. Open a web browser.
2. Navigate to the target URL: `https://34.88.16.212/`
3. Observe that the Jenkins Dashboard loads successfully without prompting for any credentials.
4. Note the exposed project names (e.g., `cassandra`, `cassandra-5.0`, `converged-core-5`), build numbers, success/failure statuses, and build durations.
5. Verify the Jenkins version (`2.568.3`) and the accessible REST API link at the bottom right of the page.

# Proof of Concept
<img width="1920" height="1035" alt="3" src="https://github.com/user-attachments/assets/f0d8b30c-0173-4b3d-83d4-170b07b063df" />


# Impact
Exposing a Jenkins instance without authentication presents a significant security risk:
- **Information Disclosure:** Attackers can enumerate internal project names, build histories, and CI/CD pipeline structures, gaining valuable reconnaissance for further attacks.
- **API Exploitation:** The exposed `REST API` link may allow unauthenticated users to query detailed build data, console outputs, or environment variables if permissions are not strictly enforced at the API level.
- **Pipeline Tampering (Potential):** Depending on the global security matrix, if write permissions are inadvertently granted to anonymous users, an attacker could potentially trigger builds, modify job configurations, or access the Script Console.
- **Supply Chain Compromise:** The exposed project (Apache Cassandra) is a major open-source database. Leaked build information or unauthorized access to its build server could compromise the integrity of the software supply chain.

# Remediation
1. **Enforce Authentication:** Navigate to **Manage Jenkins** -> **Security** (or **Configure Global Security**) and ensure that "Enable security" is checked.
2. **Configure Authorization:** Set up an authorization matrix (e.g., Matrix-based security or Role-Based Access Control) to ensure the "Anonymous" user has no permissions (or strictly limited read-only permissions if absolutely required, though this is not recommended for production instances).
3. **Restrict Network Access:** Jenkins should never be directly exposed to the public internet. Implement firewall rules, VPNs, or a reverse proxy with IP allowlisting to restrict access to trusted internal networks only.
4. **Disable Script Console (if not needed):** Ensure the `/script` endpoint is secured and only accessible to administrators, as it provides direct remote code execution (RCE) capabilities.
