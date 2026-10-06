
## Title
Grafana Unauthenticated Access Exposes Dashboards and Infrastructure Information.

## Vulnerable Endpoint
`https://grafana.cncmonitor.com.br/api/search?type=dash-db`

## Description
The Grafana instance at the ffected asset is configured with anonymous access enabled. This misconfiguration allows unauthenticated users to access sensitive API endpoints without any credentials. Specifically, the `/api/search?type=dash-db` endpoint returns a complete list of all dashboards, including their titles, unique IDs (UIDs), and folder structures.

## Steps to Reproduce
1. Open a terminal or command prompt.
2. Send a simple GET request to the vulnerable endpoint using `curl`:
```
curl -i "https://grafana.cncmonitor.com.br/api/search?type=dash-db"
```
3. Observe that the server responds with an **HTTP 200 OK** status and a JSON body containing sensitive dashboard information, without requiring any authentication.

## Proof of Concept
The following is a snippet of the response received from the vulnerable endpoint, which confirms the exposure of dashboard metadata:
<img width="1920" height="1080" alt="Screenshot From 2026-05-26 05-22-02" src="https://github.com/Deepak-cyber-web/Bug-Bounty-Reports/blob/main/Practice_report/evidence/Screenshot%20From%202026-10-06%2007-44-03.png" />

## Impact
An unauthenticated attacker can access critical operational and infrastructure information. This exposure reveals a significant amount of sensitive data, including:

- **Internal Business Structure:** Dashboard folder names such as `Polimold`, `Unipac`, and `Solcera` reveal internal divisions and customer names.
- **Operational Technology (OT) Details:** Dashboard titles like `Atividade Maquinas`, `Contagem de Peças`, and references to specific industrial equipment models (`Mazak`, `Grob G 550`, `Romi GL 300`) expose the internal manufacturing environment.
- **Infrastructure Reconnaissance:** The `Connect IoT` folder and `mqtt` tags suggest the presence of IoT and MQTT infrastructure, providing a roadmap for further attacks.

This information disclosure provides a malicious actor with valuable reconnaissance, enabling them to understand the target's operations and potentially launch more targeted attacks.

## Remediation
To secure the Grafana instance, it is recommended to:

1. **Disable Anonymous Access:** In the Grafana configuration file (`grafana.ini`), set the `enabled` option in the `[auth.anonymous]` section to `false`.
2. **Restrict Network Access:** Limit access to the Grafana instance using firewall rules or a VPN, ensuring it is not publicly exposed unless absolutely necessary.
3. **Review Data Source Permissions:** As a precaution, review and rotate any credentials for data sources (e.g., databases, InfluxDB, Prometheus) that may be accessible through the exposed Grafana instance.
