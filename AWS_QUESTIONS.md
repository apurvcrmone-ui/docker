# ☁️ AWS (Amazon Web Services) Reference & Interview Guide

A comprehensive guide covering AWS core services, architecture, security, troubleshooting, and best practices.

---

## 📌 Table of Contents
1. [IAM (Identity & Access Management)](#-1-iam-identity--access-management)
2. [S3 (Simple Storage Service)](#-2-s3-simple-storage-service)
3. [CloudWatch](#-3-cloudwatch)
4. [KMS (Key Management Service)](#-4-kms-key-management-service)
5. [Route 53 (DNS)](#-5-route-53-dns)
6. [SNS & SQS (Messaging & Queuing)](#-6-sns--sqs-messaging--queuing)
7. [CloudTrail (Auditing)](#-7-cloudtrail-auditing)
8. [CloudFormation (IaC)](#-8-cloudformation-iac)
9. [AWS Architecture Patterns](#-9-aws-architecture-patterns)
10. [AWS Security Best Practices](#-10-aws-security-best-practices)
11. [AWS Troubleshooting Workflows](#-11-aws-troubleshooting-workflows)
12. [Must-Know AWS Comparison Tables](#-12-must-know-aws-comparison-tables)

---

## 🔐 1. IAM (Identity & Access Management)

### Q1. What is IAM?
AWS IAM is a service that helps you securely control access to AWS resources. It controls **authentication** (who can sign in) and **authorization** (what permissions they have) to resource operations.

### Q2. IAM User vs. IAM Role?
* **IAM User:** A permanent identity with long-term credentials (password, access keys) representing a specific person or service.
* **IAM Role:** A temporary identity with no long-term credentials. It is assumed by trusted entities (like EC2 instances, Lambda functions, or cross-account users) to obtain temporary security credentials via the AWS STS (Security Token Service).

### Q3. IAM Role vs. IAM Policy?
* **IAM Policy:** A JSON document that defines permissions (Allows or Denies actions on specific resources). A policy does nothing on its own until attached.
* **IAM Role:** An identity container. You attach policies to the role to declare what permissions anyone assuming the role will inherit.

### Q4. What is Least Privilege?
The security principle of granting users, roles, or services **only the minimum permissions** necessary to complete their specific tasks. This limits exposure and prevents accidental or malicious damage.

### Q5. Why use IAM Roles instead of Access Keys?
* **No hardcoded credentials:** Access keys are long-lived and easily exposed (e.g., pushed to GitHub). Roles use temporary credentials that rotate automatically.
* **Operational overhead:** Rotating keys manually is tedious and high-risk. Roles manage rotation automatically through AWS Security Token Service (STS).

### Q6. What causes AccessDenied errors?
An `AccessDenied` error occurs when a request is blocked by AWS's authorization engine. Common causes:
1. No explicit `Allow` statement matches the action/resource.
2. An **Explicit Deny** matches the action/resource.
3. Service Control Policies (SCPs) at the Organization level block the action.
4. Permissions Boundaries or Session Policies limit the permitted actions.
5. VPC Endpoint Policies restrict access.

### Q7. What is an Explicit Deny?
In AWS, a request defaults to Deny. The evaluation logic is:
```
Default Deny ──> Any Explicit Deny? (Yes) ──> Final Decision: DENY
                  │ (No)
                  └──> Any Explicit Allow? (Yes) ──> Final Decision: ALLOW
                        │ (No)
                        └──> Final Decision: DENY
```
An **Explicit Deny** is a statement in any applicable policy containing `"Effect": "Deny"`. It overrides all `Allow` statements, regardless of where or when they are defined.

---

## 🪣 2. S3 (Simple Storage Service)

### Q8. What is Amazon S3?
Amazon S3 is an object storage service offering industry-leading scalability, data availability, security, and performance. It stores data as objects within containers called "buckets".

### Q9. What is S3 Versioning?
A feature that keeps multiple variants of an object in the same bucket. It allows you to preserve, retrieve, and restore every version of every object stored in S3, protecting against accidental deletions and overwrites.

### Q10. What is an S3 Lifecycle Policy?
A set of rules that automates the transition of objects to more cost-effective storage classes or schedules their permanent deletion after a specified time frame. For example:
* Transition objects from *S3 Standard* to *Standard-IA* after 30 days.
* Move them to *Glacier* after 90 days.
* Expire/delete them after 365 days.

### Q11. S3 Standard vs. Standard-IA vs. Glacier?
* **S3 Standard:** High durability, availability, and performance for active, frequently accessed data.
* **Standard-IA (Infrequent Access):** Lower storage cost but includes a retrieval fee. Best for data accessed less than once a month but requires rapid access when needed.
* **S3 Glacier (Flexible/Deep Archive):** Ultra-low-cost archival storage. Retrieval times range from minutes (Expedited) to hours (Bulk).

### Q12. How do you recover a deleted file in S3?
If S3 Versioning is enabled:
1. Go to the S3 bucket and turn on **Show Versions**.
2. Find the deleted file. You will see a **Delete Marker** as the latest version.
3. Delete the **Delete Marker** version. The previous version of the file will automatically become the active version.

### Q13. How do you reduce S3 storage costs?
1. Enable **S3 Lifecycle Policies** to transition aging files to cold storage classes.
2. Use **S3 Intelligent-Tiering** to automatically optimize costs for data with unpredictable access patterns.
3. Clean up incomplete **multipart uploads** using lifecycle rules.
4. Delete old object versions if versioning is active.

### Q14. What is S3 Object Lock?
An S3 feature that stores objects using a **WORM** (Write Once, Read Many) model. It blocks object deletion or overwriting for a fixed retention period or indefinitely (Compliance or Governance modes), helping prevent ransomware or rogue deletion activities.

### Q15. How do you secure an S3 bucket?
1. **Enable Block Public Access (BPA)** at the bucket and account levels.
2. Use **Bucket Policies** to enforce SSL-only traffic (`aws:SecureTransport: false` deny).
3. Enable **Encryption at Rest** (using SSE-S3 or SSE-KMS).
4. Implement **least-privilege IAM policies** for bucket access.
5. Enable S3 Versioning and Object Lock to prevent accidental loss.

---

## 📈 3. CloudWatch

### Q16. What is CloudWatch?
Amazon CloudWatch is a monitoring and observability service that collects data in the form of logs, metrics, and events, providing actionable insights for AWS applications and infrastructure.

### Q17. CloudWatch Metrics vs. Logs?
* **Metrics:** Numeric time-series data representing resource performance variables (e.g., `CPUUtilization`, `DiskWriteBytes`). Used for graphing and alarm thresholds.
* **Logs:** Structured or unstructured text-based records generated by application, system, or AWS service events (e.g., Nginx access logs, application error traces).

### Q18. What is a CloudWatch Alarm?
A resource that monitors a single CloudWatch metric over a specified time period. It performs one or more actions (such as sending an email alert via SNS, triggering Auto Scaling, or stopping an EC2 instance) when the metric crosses a set threshold.

### Q19. How do you send email alerts from CloudWatch?
1. Create an **Amazon SNS (Simple Notification Service) Topic**.
2. Subscribe your email address to the SNS Topic and confirm the subscription.
3. Create a **CloudWatch Alarm** on a metric (e.g., `CPUUtilization > 80%`).
4. Configure the alarm's "Actions" to send a notification to the SNS Topic when in the `ALARM` state.

### Q20. How do you monitor EC2 CPU utilization?
EC2 instances automatically report basic hypervisor-level metrics to CloudWatch (like CPU utilization) every 5 minutes for free, or every 1 minute if **Detailed Monitoring** is enabled. No agent installation is required for CPU monitoring.

### Q21. How do you troubleshoot an EC2 instance using CloudWatch?
1. **Check Status Check Metrics**: Monitor `StatusCheckFailed_System` (hypervisor hardware issues) and `StatusCheckFailed_Instance` (OS/software issues).
2. **Review Resource Metrics**: Inspect CPU, Disk, and Network metrics to spot bottlenecks.
3. **Analyze Logs**: Install the **CloudWatch Agent** on the instance to stream OS logs (`/var/log/syslog`, `/var/log/nginx/error.log`) and Memory metrics (which are not available at the hypervisor level) directly to CloudWatch Logs.

---

## 🔑 4. KMS (Key Management Service)

### Q22. What is AWS KMS?
AWS Key Management Service is a managed service that makes it easy to create, manage, and control cryptographic keys (symmetric/asymmetric) used to encrypt your data.

### Q23. Why do we use KMS?
* **Centralized Key Management:** Create, rotate, and define permissions for encryption keys.
* **Envelope Encryption:** Encrypts data with a data key, which is itself encrypted by a master key (KMS wrapper).
* **Auditability:** Automatically integrates with AWS CloudTrail to log every single key usage request.

### Q24. How do you encrypt an S3 bucket?
1. Under bucket settings, enable **Default Encryption**.
2. Select encryption type: **SSE-S3** (Amazon-managed keys) or **SSE-KMS** (Customer-managed KMS keys).
3. Optionally, enforce encryption via bucket policy by denying `PutObject` requests that do not specify encryption headers.

### Q25. What is SSE-KMS?
**Server-Side Encryption with AWS KMS**. It uses KMS keys to encrypt objects, providing a separate permission envelope (users need both S3 access permissions and KMS Decrypt permissions to read files) and a full audit trail of key usage.

### Q26. Why might an encrypted S3 bucket return AccessDenied?
If the bucket is encrypted with a **Customer Managed Key (SSE-KMS)**, a user attempting to read/write files will get `AccessDenied` if:
* They have S3 permission policies but lack the **`kms:Decrypt`** or **`kms:GenerateDataKey`** permissions on the KMS key.
* The KMS Key Policy itself does not explicitly allow the user's role/account to use the key.

---

## 🗺️ 5. Route 53 (DNS)

### Q27. What is Route 53?
Route 53 is AWS's highly available and scalable Domain Name System (DNS) web service. It translates human-readable names (like `example.com`) into numeric IP addresses (like `192.0.2.1`).

### Q28. A Record vs. CNAME?
* **A Record (Address):** Maps a hostname directly to an IPv4 address. (e.g., `app.com` ──> `192.0.2.1`).
* **CNAME (Canonical Name):** Maps a hostname to another hostname. (e.g., `www.app.com` ──> `app.com`).
  > [!WARNING]
  > CNAMEs cannot be created at the zone apex (root domain level like `example.com`).

### Q29. What is Alias Record?
An AWS-specific DNS record extension. Like a CNAME, it maps a hostname to an AWS resource (such as an ELB or S3 bucket), but **can be used at the zone apex (root domain)**. Route 53 automatically tracks IP changes of target resources.

### Q30. What is Simple Routing?
A basic routing policy that resolves a domain name to one or more IP addresses static to the record. If multiple IPs are specified, Route 53 returns all values in a random order to the client.

### Q31. What is Weighted Routing?
Enables you to associate multiple resources with a single domain name and specify what **proportion** of traffic is routed to each resource based on weight. (e.g., route 90% of traffic to v1 servers and 10% to v2 servers for canary testing).

### Q32. What is Failover Routing?
Used to configure active-passive failover. Route 53 monitors a health check on the primary resource; if it fails, Route 53 automatically redirects users to a secondary standby resource (e.g., static error page on S3).

### Q33. What is Latency-Based Routing?
Routes DNS queries to the AWS region that provides the lowest network latency for the user, improving application response times for global users.

### Q34. When would you use each Route 53 routing policy?
* **Simple**: For single resources (like a single IP address).
* **Weighted**: For blue/green deployments, canary releases, or resource load balancing.
* **Failover**: For disaster recovery (redirecting to backup servers).
* **Latency**: For high-performance, multi-region setups.
* **Geolocation**: To serve content in specific languages or restrict licenses based on user country.
* **Geoproximity**: Route traffic based on geographical distance using bias coordinates.
* **Multi-value Answer**: For simple, DNS-level load balancing with health checks.

---

## ✉️ 6. SNS & SQS (Messaging & Queuing)

### Q35. What is SNS?
Amazon Simple Notification Service is a managed **Pub/Sub (Publish/Subscribe)** messaging service. A publisher pushes messages to a "Topic", which broadcasts them to multiple subscribers (Lambda, SQS, email, HTTP, SMS).

### Q36. What is SQS?
Amazon Simple Queue Service is a fully managed **message queuing** service used to decouple microservices. A producer sends messages to a queue, and consumers pull messages to process them asynchronously.

### Q37. SNS vs. SQS?
* **SNS (Pub/Sub):** "Push" model. Messages are sent immediately to all subscribers. No persistence (messages are lost if not delivered).
* **SQS (Queue):** "Pull" model. Consumers poll the queue. Messages are stored persistantly until processed and deleted (up to 14 days).

### Q38. What is a Dead Letter Queue (DLQ)?
A specialized SQS queue or SNS target where messages are redirected if they fail to process successfully after a specified number of attempts (e.g., if a consumer keeps crashing while processing a message). This prevents queue blockage ("poison pill" messages).

### Q39. How do SNS and SQS work together?
Using the **Fanout Pattern**:

```
                  ┌──> SQS Queue 1 ──> Consumer A
                  │
Publisher ──> SNS Topic ──> SQS Queue 2 ──> Consumer B
                  │
                  └──> SQS Queue 3 ──> Consumer C
```

Publishing a message once to the SNS topic automatically duplicates it into multiple SQS queues, allowing multiple distinct consumer applications to process the same message in parallel.

### Q40. When would you use SNS?
Use SNS when you need to **broadcast** a single event to multiple consumers simultaneously (e.g., order placed event sending an email, notifying inventory service, and writing to shipping service).

### Q41. When would you use SQS?
Use SQS when you want to **decouple** applications, buffer heavy traffic spikes, and process tasks asynchronously (e.g., offloading video encoding requests to a pool of background worker servers).

---

## 🕵️ 7. CloudTrail (Auditing)

### Q42. What is CloudTrail?
AWS CloudTrail is an auditing service that records API calls made within your AWS account. It documents the identity of the API caller, the time of the call, the source IP address, and the parameters of the request.

### Q43. CloudTrail vs. CloudWatch?
* **CloudTrail:** Records **Who** did **What** (actions and API audit trails).
* **CloudWatch:** Records **How** the resource is performing (system performance, metrics, and application logs).

### Q44. How do you find who deleted an AWS resource?
1. Open the **CloudTrail Console** and navigate to **Event History**.
2. Filter the history by `Event name` (e.g., `DeleteBucket`, `TerminateInstances`) or `Resource name`.
3. Open the event details to inspect the `userIdentity` block, which contains the IAM User/Role name, source IP, and access key used.

### Q45. What information does CloudTrail store?
* **Who:** IAM User, Role, or federated credentials of the caller.
* **What:** The API action name (e.g., `CreateSecurityGroup`).
* **When:** UTC timestamp of the request.
* **Where:** The target AWS region and source IP address.
* **Result:** Request parameters and response elements (success/fail details).

---

## 🏗️ 8. CloudFormation (IaC)

### Q46. What is CloudFormation?
AWS CloudFormation is an Infrastructure as Code (IaC) service that allows you to model, provision, and manage AWS resources using declarative YAML or JSON template files.

### Q47. What is a CloudFormation Stack?
A single unit of resource management. You create, update, or delete a group of AWS resources (like databases, EC2 instances, and VPCs) by managing the corresponding CloudFormation Stack.

### Q48. What is a CloudFormation Template?
A JSON or YAML file that acts as a blueprint declaring what AWS resources you want to create and configure.

### Q49. What happens during a Stack Update?
CloudFormation compares the new template with the currently deployed stack. It computes a diff and updates only the modified resources. If an update fails, CloudFormation automatically performs a **rollback** to restore the stack's previous working state.

### Q50. CloudFormation vs. Terraform?
* **CloudFormation:** Native to AWS. Managed state (no manual state files). Free to use.
* **Terraform:** Cloud-agnostic (supports Azure, GCP, Kubernetes, etc.). Written in HCL. Requires managing a state file (`terraform.tfstate`).

### Q51. What is a Change Set?
A preview of the actions CloudFormation will perform during a stack update. It lets you review resource creations, modifications, or deletions before committing the changes.

---

## 🏛️ 9. AWS Architecture Patterns

### Q52. Design a highly available web application on AWS.
A classic **three-tier highly available architecture**:

```
                       [Route 53]
                            │
               [Application Load Balancer]
             ┌──────────────┴──────────────┐
       [Public Subnet AZ1]           [Public Subnet AZ2]
        └─> Nginx Instance            └─> Nginx Instance
             ┌──────────────┬──────────────┐
       [Private Subnet AZ1]          [Private Subnet AZ2]
        └─> App Server Instance       └─> App Server Instance
             ┌──────────────┬──────────────┐
       [Database Subnet AZ1]         [Database Subnet AZ2]
        └─> Primary DB (Active) <──> Standby DB (Replica)
```

1. **Route 53:** For DNS routing.
2. **Application Load Balancer (ALB):** Spreads incoming HTTP/HTTPS traffic across multi-AZ instances.
3. **Auto Scaling Group (ASG):** Scales EC2 instances across multiple Availability Zones in private subnets.
4. **Multi-AZ Amazon RDS:** Active-Passive database setup. Automatically replicates data to a secondary AZ database for instant failover.
5. **Multi-AZ NAT Gateways:** Enables private subnet instances to connect securely to the internet for package updates.

### Q53. How would you build a low-latency global application?
1. **Amazon CloudFront:** Deploy a Content Delivery Network (CDN) to cache static and dynamic assets at edge locations close to users.
2. **Amazon Route 53 Latency Routing:** Route user DNS queries to the nearest AWS region.
3. **AWS Global Accelerator:** Routes traffic over AWS's high-speed global network fiber instead of the public internet.
4. **Amazon DynamoDB Global Tables:** Multi-region active-active database replication.

### Q54. How would you protect data from accidental deletion?
1. Enable **S3 Versioning** and **MFA Delete** on critical buckets.
2. Enable **RDS Backups** and snapshot copying to secondary AWS accounts.
3. Use **AWS Backup** for centralized, automated backup management.
4. Apply **termination protection** to EC2 instances, databases, and CloudFormation Stacks.

---

## 🔒 10. AWS Security Best Practices

### Q55. How do you secure an EC2 instance?
1. **VPC Placement:** Run instances in private subnets.
2. **Security Groups:** Enforce least-privilege access rules (e.g., restrict port 22/SSH to specific IP blocks only).
3. **Session Manager:** Disable SSH key keys entirely and use AWS Systems Manager Session Manager for console access.
4. **Patch Management:** Keep OS updated.
5. **IAM Role Instance Profile:** Grant temporary privileges instead of saving AWS keys inside the OS config.

### Q56. What is encryption at rest?
Securing data while stored on disk or physical media (using algorithms like AES-256). Handled automatically in AWS using KMS key integration on S3, EBS, and RDS.

### Q57. What is encryption in transit?
Securing data as it travels across network paths (using TLS/SSL). Implemented by enforcing HTTPS on Load Balancers, CloudFront, and internal endpoints.

### Q58. Secrets Manager vs. Parameter Store?
* **Systems Manager Parameter Store:** Good for standard configuration data and basic secret storage. Free for standard parameters. Does not support automatic rotation out-of-the-box.
* **Secrets Manager:** Designed specifically for database/API credentials. Paid service. Supports **automatic rotation** of credentials using AWS Lambda integrations.

---

## 🛠️ 11. AWS Troubleshooting Workflows

### Q59. EC2 is not reachable. What do you check?
1. **Route Table**: Does the subnet have a route to the Internet Gateway (`0.0.0.0/0 -> igw-xxxx`) if public, or a route to a NAT Gateway if private?
2. **Security Group**: Does the inbound security group rule allow traffic on the requested port (e.g., HTTP 80, SSH 22) from your IP?
3. **Network ACL (NACL)**: Are both inbound and outbound rules allowing traffic? (NACLs are stateless, so you must explicitly allow ephemeral return ports).
4. **Status Checks**: In the EC2 console, did the instance fail system status checks? (indicates hardware or hypervisor host failures).

### Q60. S3 returns AccessDenied. What do you check?
1. **IAM Policy**: Does the caller have S3 permissions (`s3:GetObject`, `s3:PutObject`) allowed?
2. **Bucket Policy**: Is there a bucket policy blocking or restricting access (such as denying non-SSL traffic or external IPs)?
3. **S3 Block Public Access**: If trying to access publicly, is block public access active?
4. **KMS Permissions**: If the bucket uses SSE-KMS, does the user have permission to use the KMS decryption key?

### Q61. Application cannot connect to RDS. What do you check?
1. **VPC Routing**: Are the EC2 and RDS instances in the same VPC? If not, is VPC Peering or Transit Gateway configured?
2. **RDS Security Group**: Does the RDS security group allow inbound traffic on the database port (e.g., PostgreSQL 5432, MySQL 3306) from the EC2 security group ID?
3. **Subnet Group**: Is the RDS instance placed in private subnets?
4. **Authentication**: Verify DB username, password, and database name credentials.

---

## 📊 12. Must-Know AWS Comparison Tables

### A. Security Group vs. Network ACL (NACL)
| Feature | Security Group (SG) | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Level** | Operates at the **Instance** level. | Operates at the **Subnet** level. |
| **Statefulness** | **Stateful** (Return traffic is allowed automatically). | **Stateless** (Must allow both inbound and outbound traffic). |
| **Rules** | Supports **Allow** rules only. | Supports both **Allow** and **Deny** rules. |
| **Evaluation** | Evaluates all rules before deciding. | Evaluates rules in numerical order (first match wins). |

---

### B. NAT Gateway vs. Internet Gateway (IGW)
| Component | Internet Gateway (IGW) | NAT Gateway (NAT) |
| :--- | :--- | :--- |
| **Purpose** | Connects VPC subnets to the public internet. | Allows private subnets to send outbound traffic to the internet. |
| **Communication** | Bidirectional (Allows both inbound and outbound traffic). | Unidirectional (Outbound only; blocks incoming traffic from internet). |
| **Subnet Placement** | Attached to VPC, associated with Route Table. | Must be placed in a **Public Subnet**. |

---

### C. Multi-AZ Deployment vs. Read Replicas
| Parameter | Multi-AZ | Read Replicas |
| :--- | :--- | :--- |
| **Primary Goal** | **High Availability** & Disaster Recovery. | **Performance Scaling** (offloads read queries). |
| **Replication Type** | **Synchronous** (highly consistent). | **Asynchronous** (eventually consistent). |
| **Active/Passive** | Only 1 DB instance is active for reads/writes. | All read replicas are active and readable. |
| **Failover** | Automatic database failover to standby. | Manual promotion to primary instance if needed. |
