## 1.0  SIEM (Security Information and Event Management)

### 1.1 What is SIEM 
SIEM is a software used in organization to collect logs across organizations , organize the information , store them in a 1 place, correlates them to detect threats and raise alerts for analysts to investigate.

SIEM consist of 2 parts SIM and SEM :
1. SIM - security information management , responsible for long term log storage , reporting and compliance.
2. SEM - security event management , responsible for real time monitoring , correlation and alerting.
In short SIEM is the brain , it collects all data from endpoints ,  network , cloud and identity system and normalize into a common format , correlates events to detect attack patters and generate alerts for analysts to triage.

---

### 1.2 Purpose of existance : based on real world case
What problem does it solve , let say an organization owns 500 laptops , 80 servers, a firewall , a VPN, microsoft 365 , AWS and active directory. Each component would produces logs in a different format and in different place thus causing the attackers footprints to be scattered all over. Single person cannot check all logs .
- phishing email - email gateway log
- user clicked a link - proxy log
- malicious process started - endpoint log
- new account or admin account was created - active directory log
- data left the network  - firewall log

1.2.1 Target Corporation Data Breach 2013
Target had firefly malware detection system that did generate alerts when attackers installed a card stealing malware. The alerts were not acted on time and caused about 40 million payments cards being stolen. A tool that doesnt provide triage process will still result a failure in protecting even if the analysts are trained.(need to do case study)

1.2.2  Equifax Data Breach 2017
An expired certificate on a network inspection means that encrypted traffic wasn't being inspected for months. SIEM is implemented but the lack of visibility is the true problem. If a log goes missing or log source is broken then it is same as we being blind, meaning log source health is important.( must do another case study and learn about log source health improvement)

---

### 1.3 Core functions of SIEM
1. Log collection and aggregation(collectiong and combining all log into a place) - pull data from everywhere and combining them into 1 place and language/normalizing them.
2. Parsing and normalization - organize and turn messy raw data and text into structured ,consistent fields.
3. Storage and retention - hot storage , cold storage and warm storage and must meet compliance
4. Search and investigation - capable to query billion of events in seconds
5. Correlation and detection - rules ,statistics and behavioral analytics
6. Alerting and incident management - notification ,ticket ,escalate
7. Dashboard and reporting - SOC metrics ,compliance evidence ,executive reports.

---

### 1.4 SIEM VS Related technologies

| Technology                              | Function                                             | Relation to SIEM                                                     |
| --------------------------------------- | ---------------------------------------------------- | -------------------------------------------------------------------- |
| Log Management                          | Collect and store logs                               | SIEM = Log management + security analytics                           |
| IPS/IDS (Snort,Suriacata)               | Inspect network traffics for knows attack signatures | Feed the alerts into SIEM                                            |
| ERD (CrowdStrike,Defender for Endpoint) | Deep endpoint telemetry , detection and response     | Major data source for SIEM and can isolate hosts                     |
| XDR                                     | ERD + email + identity + cloud all correlated        | Overlaps with SIEM but it focuses primarly on cendor and third party |
| SOAR                                    | Automate response through playbook                   | Consumes SIEM alerts and automates the actions                       |
| Firewall                                | Filter traffic , block or permits                    | Generate logs for SIEM                                               |
| UEBA                                    | Learn normal user behavior and flags deviations      | Mostly SIEM module                                                   |
| Threat Intel Platform                   | Manage IOC and Threat feeds                          | Enriches SIEM events                                                 |

---

### 1.5 Architecture

Data source - Endpoint, Servers, Firewalls, cloud and Saas ------>
Collection - Agents , Syslog , APIs ,Forwarders ------>
Processing - Parse , Normalize , Enrich(GeoIP , TI, asset) ------>
Storage - Hot or warm , Cold , Archive ------>
Analytics - Correlation , Rules/ML , Threat Intel , UEBA or Search ------>
Action - Alerts , Tickets , SOAR , Dashboards.

---

### 1.6 Terminology to remember
1. Event : the single recorded ocurrence or 1 log line
2. Log : A record of events produced by the system
3. Alert : SIEM generated notification when an event or pattern matches a detection rule
4. Incident : One or more confirmed ,related alerts representing a securit problem
5. IOC (indicator of compromise) : an artifact such as IP, hash, domain or filename
6. IOA (indicator of attack) : a behavior such as word spawning powershell
7. EPS (events per second) :The sizing and licensing metric
8. use case : a defined detection scenario such as brute force
9. Baseline : what a normal looks like
10. Telemetry : a stream of data from sources

---
## 2.0 How SIEM logs are collected
### 2.1 Log pipeline

Generate --> Collect -->Transport --> Parse --> Normalize --> Enrich --> Store/Index --> Search/Detech

---

### 2.2 Log source

| Category             | Examples                                                | What data                                |
| -------------------- | ------------------------------------------------------- | ---------------------------------------- |
| Endpoints            | Windows Event Logs,Sysmon,Linux auditd,EDR              | Process execution,logons,file changes    |
| Server or Apps       | IIS,Apache or Nginx , SQL Server, webapps               | Web attacks,SQL injection,Errors         |
| Network devices      | Firewall( Palo Alto, Fortinet), routers, switches, VPNs | Connections, Blocked traffic ,Geo source |
| Network Security     | IPS/IDS ,Zeek, NetFlow                                  | Signatures and Protocol anomalies        |
| Identity             | Active Directory , Entra ID , Okta , RADIUS             | Logins , privalage changes and MFA       |
| Cloud                | AWSCloudTrail, Azure ,Activity Log , GCP audit          | API calls, Resource changes              |
| SaaS                 | Microsoft165, Google Workspace , SalesForce             | Mailbox rules, File sharing              |
| Email or websecurity | ProofPoint, Proxy(Zscaler,Squid)                        | Phishing , Malicious URLs                |
| DNS,DHCP             | Window DNS, Infoblox                                    | C2 lookup , IP to host mapping           |
| Security tools       | Antivirus , Vulnerability scanner, DLP                  | Detection and exposure                   |

---

### 2.3 Collection Method

2.3.1 Agent Based
Agent based are software that is installed on the source reads logs and ships them.
Pros - reliable, can buffer if the network drops, can encrypt, can read local files and registries, adds file integrity monitoring. Wazuh agent, Splunk Universal Forwarder, Elastic Agent/Beats, Azure Monitoring Agent(AMA) are example softwares.
Cons - management overhead (must deploy, patch, monitor), consumes endpoint resources

2.3.2 Agentless
No software in the source .Either the SIEM pulls or the source pushes the logs.
- Syslog (UDP/TCP 514, TLS 6514):The standard for network devices , Linus and appliances. UPD can be unreliable as the packet drops silently better to use TCP/TLS for security logs
- Windows Event Forwarding(WEC/WEF): Windows hosts push events to a central collector using WinRM with no third party agent.
- WMI or RPC pull: The collector queries the windows host remotely (older and heavier)
- SNMP traps:  Device health and status events.
- Database polling(JDBC/ODBC): Reading audit tables.

2.3.3 API-Based (essential for cloud and Saas)
Cloud services has no server to put an agent on so the SIEM will call their APIs.
- AWS CloudTrail --> S3 bucket --> SIEM pulls(or SQS notifications)
- Microsoft 365 --> Office 365 Management Activity API
- Entra ID --> Microsoft Graph or Diagnostic Settings --> Even Hub or Log Analytics
- Okta --> System Log API

2.3.4 Intermediate Layers
- Syslog Relay/collector : 1 harderned box receives over 200 syslog and forwards to the SIEM.
- Heavy Forwarder (Splunk) : Parses before indexing.
- Message Brokers (Kafka, Even Hubs) : Buffer and decouple at high scale.
- Log shippers (Logstash, Fluentd, Cribl) : Transform, filter and route. Cribl is popular for cutting SIEM licensing cost by dropping noise.

---

## 2.4 Parsing and Normalization

2.4.1 Parsing 
Parsing is extracting fields from raw text.
Techniques :
- Regex or Grok - Elastic/Logstash
- Decoders - Wazuh, using XML + Regex
- Props.conf or Transforms.conf and add-ons - Splunk
- KQL parse, Custom tables, DCR transformations - Sentinel
- Native structured format - JSON, CEF, LEEF, key=value

2.4.2 Common log formats
- Syslog (RFC 3164/5424) : ==🔵< priority > timestap host app: message.==
- CEF (Common Event Format , ArcSight) : ==🔵CEF:0|Vendor|Product|Version|SignatureID|Name|Severity|Extension==
- LEEF : IBM QRadar's equivelent
- JSON : the mordern default (CloudTrail, Okta, Zeek)
- Windows Event XML or EVTX

2.4.3 Normalization Schemas (Common data models)

| Platform               | Model                                      |
| ---------------------- | ------------------------------------------ |
| Splunk                 | CIM (Common Information Model)             |
| Elastic                | ECS (Elastic Common Schema)                |
| Microsoft Sentinel     | ASIM (Advanced Security Information Model) |
| Industry open standard | OCSF (Open Cybersecurity Schema Framework) |

2.4.4 Must know Windows Event IDs

| Event IDs      | Meaning                             | Reason                                        |
| -------------- | ----------------------------------- | --------------------------------------------- |
| 4624           | Successful logon                    | Baseline, Check logon type and source         |
| 4625           | Failed logon                        | Bruteforce, password spray]                   |
| 4634/4647      | Logoff                              | Session reconstruction                        |
| 4648           | Login with explicit credentials     | Lateral moveman (runas, PsExec)               |
| 4672           | Special privilages assigned         | Admin logon                                   |
| 4688           | New process created                 | Execution (enable command line auditing)      |
| 4697/7045      | Service installed                   | Persistence, PsExec, ransomware               |
| 4698           | Scheduled task created              | Persistence                                   |
| 4720           | User account created                | Backdoor accounts                             |
| 4728/4732/4756 | Member added to privilaged group    | Privilege escalation                          |
| 4768/4769      | Kerberos TGT/service ticket request | Kerberoasting (4769 with RC4 encryption 0x17) |
| 4776           | NTLM credential validation          | Pass-the-hash and NTLM abuse                  |
| 1102           | Security log cleared                | Anti-forensics (major red flag)               |
| 4104           | Powershell script block logging     | Malicious Powershell                          |

2.4.5 Logon Types

| Type | Meaning           | Example                                  |
| ---- | ----------------- | ---------------------------------------- |
| 2    | Interactive       | Sitting at the keyboard                  |
| 3    | Network           | File Share access, PsExec                |
| 4    | Batch             | Schedule task                            |
| 5    | Service           | Service account start                    |
| 7    | Unlock            | Screen unlock                            |
| 9    | NewCredentials    | runas /netonly , pass-the-hash indicator |
| 10   | RemoteInteractive | RDP                                      |
| 11   | CachedInteractive | Offline domain Logon                     |
Sysmon is a free microsoft sysinternals tool, it adds what native window logs lack (==🟢i have no diea what this is must read up all==):
1. Event 1 - process create with hashes and parent
2. Event 3 - network connection 
3. Event 7 - image load 
4. Event 8 - CreateRemoteThread
5. Event 10 - Process access such as LSASS dumping
6. Event 11 - File create
7. Event 13 - Registry set
8. Event 22 - DNS query

### 2.5 Real operational challenges
1. Volume and cost - Splunk licenses GB/Day and Sentinel by GB ingested, so noisy sources blow up the budget. Solution is to filter at source or with CRIBL, tier storage and keep only the security relevent events.
2. Missing audit policy - Windows do not log process creation or Powershell blocks by default so if auditing is not enabled then SIEM rule cannot detect it.
3. Time sync - If a server is off by 8 minutes then the correlation breaks and the timeline does not match up ,  best to use NTP everywhere and watch time zones.
4. Parsing Failures - A vendor updates the log format and the fields starts going blank , so rules silently stop firing , must monitor this.
5. Log source silence - A source that stops sending log itself is an alert. A common detection is no logs for past 30 minutes from domain controller is suspicious.
6. UDP loss - Under load UDP system drops events.
7. Log integrity - Attackers clear logs which is event 1102 , so shipping logs off the original host in real time can give a duplicate and the original logs will then be tamper-evident or in other words when tempered with can be caught.
8. Privacy / legal - Logs usually contain username , ip, and sometimes PII so retantion and access must follow policy and law such as GDPR and PDPA.

---
## 3.0 Correlation and alerts

Correlation - Correlation is linking multiple events , across time and across sources to identify pattern that indicate an attack and that individual event would reveal. One failed login can be mistyped password but 500 failed log in from 1 IP over 3 minutes  that is followed by success is a compromise that pattern is the detection.

### 3.1 Types of Correlation 

| Types of Correlation                              | Meaning                                                                                                                                                                                                                                                                  |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Single-event match (signature)                    | One event matches a rule , Example - event 1102 audit logs cleared alert. Simple, fast and clear                                                                                                                                                                         |
| Threshold or aggregation                          | Count events in a timed window. Example - more than 10 x 4625 from same source IP in 5 minutes --> brute force.                                                                                                                                                          |
| Sequence or multi stage                           | A then B then C within a window. Example - 10 x failed logon    ->successful logon -> new process -> outbound connection.                                                                                                                                                |
| Cross-source correlation (the SIEM real strenght) | Combines different log types. Example - VPN login from Russia + the same badge into office 20 minutes ago , that is impossible to travel.                                                                                                                                |
| Statistical or anomaly                            | Compare a baseline. Example - A user normally upload 50MB/day and today 40GB.                                                                                                                                                                                            |
| Behavioral or UEBA                                | <br>Machine learning builds profile for each users and entity such as peer-group comparison , time of day , resource accessed and assign risk score.                                                                                                                     |
| Threat intelligence correlation                   | Match events against IOC feeds such as MISP, AlienVault OTX, commercial feeds. Example - the firewall log shows a connection to an IP listed as CobaltStrike C2 server.                                                                                                  |
| Risk based correlation (RBA)                      | Instead of alerting on every week signal each observation adds risk point to an entity either user or host. When the cumulative score crosses a threshold a high confidence alert files. This is Splunk ES's Risk Based Alerting  and it massively reduce alert fatigue. |

### 3.2 Detection rule logic with Examples

3.2.1 Use case 1 : Brute force followed by a successful logon
10 failed logon from the same source IP in 5 minutes followed by a successful logon from that source for the same account. 

Splunk SPL 
![[Pasted image 20261126181321.png]]

Microsoft Sentinel KQL 
![[Pasted image 20261126181427.png]]
Wazuh rule (XML in /var/ossec/etc/rules/local_rules.xml)

![[Pasted image 20261126181607.png]]

3.2.2 Use case 2 : Password spraying (Entra ID/Azure ID)
Few attempts per account but many account from 1 source IP , designed to avoid lockout.

Microsoft Sentinel KQL
![[Pasted image 20261126181839.png]]
This is pretty much brute force same same but different scenario where instead of trying 100 password for 1 username we use 100 username for 1 password.

3.2.3 Use case 3 : Suspicious Office -> Powershell (Phishing execution)
Parent process winword.exe or excel.exe spawns powershell.exe or cmd.exe. Legitimate use are rare. (Sysmon event 1 or Windows 4688)

Splunk SPL 
![[Pasted image 20261126182351.png]]
T1566.001 -> spearfishing attachment & T1059.001 -> Powershell

3.2.4 Use case 4 : Ransomware precursors
- vssadmin delete shadows /all /quiet or wmic shadowcopy delete - deleting backups
- Hundreds of file renames with new file extension within a minute.
- bcdedit /set {default} recoveryenabled no

3.2.5 Impossible travel
A user sign in from KL at 9 am and then from Germany at 9.25 am . The distance in 25 minutes is not feasible . Can be VPN or Proxy use can be verified before escalating.

3.2.6 DNS tunneling or C2 beaconing
Very long , high entropy subdomains , high query volume to one domain or regular interval connection plus minus 60 seconds to a rare external IP.

### 3.3 Event vs Alert VS Incident

Event : Something happened usually million per day.
Alert : Something suspicious happened possibly hundreds per day
Incident : A confirmed security issue needing response.

An alert is a notification that is generated when collected events match a detection logic signaling potentially malicious or policy violating activity that requires a human or possibly a automated review.

3.3.1 Anatomy of a good alert
- Name/title : "Brute Force Success: 4625 ×N then 4624"
- Severity : Low/Medium/High/Critical
- Timestamps and time window
- Entities : user, host, source/destination IP, process, file hash
- Evidence : the triggering raw events
- MITRE mapping : tactic and technique
- Context : asset criticality, user role, threat intel match
- Recommended actions/playbook link
- Owner/status: New → In Progress → Resolved

3.3.2 Alert Classification

|             | Attack actually happening                       | No attack                           |
| ----------- | ----------------------------------------------- | ----------------------------------- |
| Alert fired | True positive (TP) - correct                    | False positive (FP) - wasted effort |
| No alert    | False negative (FN) - missed attack , dangerous | True negative (TN) - correct        |
Benign True Positive - These are rule that fired correctly on authorised activity such as pentesting.

### 3.4 Alert lifecycle and triage methodology

3.4.1 Alert lifecycle

Detection → Triage → Investigation → Classification → Response/Escalation → Closure → Tuning/Lessons Learned

3.4.2 Triage checklist
1. What fired ? - read the rule logic and understand it
2. Who or what it involves ? - User , host , IP and what is the criticality
3. Is it expected ? - Check the change calendar, ticket , user role ,  maintanance window 
4. What is the context ? - Threat intel on IP or hash or domain 
5. What happened before and after ? - Pivot on the user host
6. Did it successd ? - Attempt or was there a compromise
7. Scope - Are there other hosts affected
8. Decide - TP/FP/benign , contain or close or escalate
9. Document - document everything with as much detail so anyone can pick it up exactly where u left off
