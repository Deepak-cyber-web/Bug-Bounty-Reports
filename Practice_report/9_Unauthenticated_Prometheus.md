# Title
**Unauthenticated Prometheus Metrics Endpoint Exposure on `138.201.90.157`**

# Vulnerable Endpoint
|Endpoint|Vulnerability|
|---|---|
|`https://138.201.90.157/metrics`|Unauthenticated Information Disclosure (Medium)|

**Target Host:** `138.201.90.157`  
**Detection Tool:** Nuclei (template: `prometheus-metrics`)

# Description
During a security assessment of the host `138.201.90.157`, an exposed Prometheus metrics endpoint was identified at the `/metrics` path.

The endpoint is publicly accessible over HTTPS without requiring any form of authentication. It returns raw telemetry data in Prometheus exposition format. This data is typically meant for internal monitoring systems and can expose sensitive operational details about the underlying application and infrastructure to unauthenticated attackers.

Based on the output, the application appears to be written in Go (evidenced by `go_*` metrics) and utilizes an access control system (evidenced by `access_evaluation_duration_*` and `access_permissions_duration_*` metrics).

# Steps to Reproduce

1. Open a web browser or a command-line tool (like `curl`).
2. Navigate to the following URL:
```
https://138.201.90.157/metrics
```
3. Observe that the server responds with an HTTP `200 OK` status and returns a large plaintext document containing system and application metrics without prompting for credentials.
```
curl -i https://138.201.90.157/metrics
```

# Proof of Concept
<img width="1920" height="1042" alt="12" src="https://github.com/user-attachments/assets/70df16d1-c4b1-4150-874d-765a510fad45" />


# Impact
While metrics endpoints do not typically allow direct system compromise, exposing them publicly poses a **Medium** risk due to information disclosure:

1. **Technology Fingerprinting:** The metrics reveal that the backend is running **Go version 1.20.10**, which allows attackers to check for known vulnerabilities specific to that runtime version.
2. **Architecture & Logic Disclosure:** The presence of specific custom metrics (e.g., `access_evaluation_duration`, `access_permissions_duration`) reveals internal application logic and architecture. It indicates the application performs complex access control evaluations, providing attackers with clues about how the system works.
3. **Resource Usage Profiling:** Metrics like `go_memstats_alloc_bytes` and `go_goroutines` expose the application's resource consumption, which can be used to assess the application's capacity or to craft targeted Denial of Service (DoS) attacks.
4. **Potential for Sensitive Data Leakage:** If developers add custom metrics in the future containing sensitive data (e.g., database connection strings, API tokens, user identifiers), they would be immediately exposed to the public.
# Remediation
To secure the Prometheus metrics endpoint, the following actions should be taken:

1. **Restrict Network Access:** Implement firewall rules, security groups, or a reverse proxy configuration to restrict access to the `/metrics` endpoint. It should only be accessible by the internal Prometheus scraper or trusted monitoring IP addresses.
2. **Implement Authentication:** If the metrics endpoint must remain publicly accessible, place it behind an authentication mechanism (e.g., HTTP Basic Auth via a reverse proxy, mTLS, or an API key).
3. **Bind to Localhost:** If the application server hosts its own metrics, configure the metrics listener to bind only to the local loopback interface (`127.0.0.1`) rather than `0.0.0.0` or a public IP, and use a local agent (like Node Exporter or a sidecar proxy) to scrape and securely forward the data.
4. **Review Exposed Metrics:** Audit the application code to ensure no sensitive business logic, PII, or secrets are being exported as Prometheus metrics.
