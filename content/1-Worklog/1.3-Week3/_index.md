---
title: "Week 3 Worklog"
date: 2026-09-28
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

## Week 3 Objectives: Database & Scale

- **Amazon RDS:** Provision a relational MySQL database on Amazon RDS, configure security parameters, and establish secure connectivity from EC2 instances.
- **Amazon DynamoDB:** Design a NoSQL table, master the Partition Key mechanism, and analyze the performance and cost differences between Scan and Query operations.
- **ElastiCache:** Deploy an in-memory Redis cluster with Multi-AZ and Auto-failover capabilities to optimize read performance and reduce primary database load.
- **Auto Scaling:** Author a Launch Template defining standard server configurations and configure an EC2 Auto Scaling Group to dynamically maintain target capacity.
- **Load Balancer:** Set up an Application Load Balancer (ALB) to evenly distribute incoming user traffic across web servers in multiple Availability Zones (Multi-AZ).
- **Comprehensive Lab 3:** Architect a robust High Availability infrastructure (ALB + Auto Scaling EC2 + RDS) and validate Self-Healing / Auto-Recovery mechanisms.

### Tasks Planned for This Week

| Day | Topic | Task | Start Date | End Date | Reference |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **Mon** | **Amazon RDS** | Provision MySQL on RDS, configure secure endpoint, connect from EC2 via MariaDB client | 28/09/2026 | 28/09/2026 | [Amazon RDS](https://docs.aws.amazon.com/rds/) |
| **Tue** | **Amazon DynamoDB** | Create NoSQL table (`LabT3_Products`), set up Partition Key, practice Scan vs Query operations | 29/09/2026 | 29/09/2026 | [Amazon DynamoDB](https://docs.aws.amazon.com/dynamodb/) |
| **Wed** | **ElastiCache** | Deploy Redis Cache cluster (`labt4-rediscache`), enable Multi-AZ, Auto-failover, and encryption | 30/09/2026 | 30/09/2026 | [Amazon ElastiCache](https://docs.aws.amazon.com/elasticache/) |
| **Thu** | **Auto Scaling** | Create Launch Template (`LabT5-AutoScaling`), configure EC2 Auto Scaling Group capacity management | 01/10/2026 | 01/10/2026 | [EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/) |
| **Fri** | **Load Balancer** | Deploy Application Load Balancer (`Web-LB`), create Target Group, verify multi-AZ load balancing | 02/10/2026 | 02/10/2026 | [Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/) |
| **Sat** | **Comprehensive Lab 3** | Deploy High Availability architecture: ALB + Auto Scaling EC2 + Database, test Self-Healing | 03/10/2026 | 03/10/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |

### Week 3 Results

- Successfully provisioned an Amazon RDS MySQL relational database (`labt2-db`), connecting securely from an EC2 instance (`MyLabT2`) using the MariaDB client.
- Designed a NoSQL database on Amazon DynamoDB (`LabT3_Products`) with Partition Key `ProductID`, proving that Query operations consume only 0.5 RCU compared to 2 RCUs consumed by Scan.
- Provisioned an enterprise-grade Amazon ElastiCache for Redis cluster (`labt4-rediscache`) featuring 2 nodes, Multi-AZ with Auto-failover, and in-transit & at-rest encryption.
- Automated compute provisioning using Launch Template `LabT5-AutoScaling`, automatically spinning up EC2 instances within an Auto Scaling Group.
- Deployed an Application Load Balancer (`Web-LB`), achieving smooth Round-Robin traffic distribution across Web Server 1 (Zone A) and Web Server 2 (Zone B).
- Constructed an end-to-end High Availability architecture (ALB + Auto Scaling EC2 + RDS), verifying Self-Healing: when an instance was terminated, the Auto Scaling Group automatically launched a replacement to maintain 100% service availability.

---

#### Exercise 1: Amazon RDS — Provision MySQL on RDS & Connect from EC2

Provisioned a managed relational database on **Amazon RDS** running MySQL named **`labt2-db`** in Region `us-east-1`. The service allocated a dedicated database endpoint:
- **Endpoint:** `labt2-db.c0pysocmgc4s.us-east-1.rds.amazonaws.com`
- **Port:** `3306`
- **Master username:** `admin`
- **IAM Authentication:** Disabled | **Internet access gateway:** Disabled (private subnet isolation)

AWS RDS provides connection instructions and links to download the root SSL/TLS certificate bundle `global-bundle.pem` for encrypted database sessions.

> **Screenshot:** ![Amazon RDS MySQL Connection Endpoint & Configuration](/images/1-Worklog/week-03/01-rds-database-connect-info.png)

Connected to EC2 instance **`MyLabT2`** (`i-0d40f82022fa610a8`, Public IPv4: `54.87.188.14`, Private IPv4: `172.31.43.186`). Installed the client library and executed the connection command:

```bash
# Install MariaDB client on Amazon Linux 2023
sudo dnf install mariadb105 -y

# Connect to Amazon RDS MySQL database
mysql -h labt2-db.c0pysocmgc4s.us-east-1.rds.amazonaws.com -P 3306 -u admin -p

# Enter password
# Connection established:
# Welcome to the MariaDB monitor. Commands end with ; or \g.
# Your MySQL connection id is 35
# Server version: 8.4.9 Source distribution
# MySQL [(none)]>
```

Successful connection verified that the RDS Security Group accurately allowed Inbound traffic on port 3306 scoped to the EC2 instance Security Group.

> **Screenshot:** ![EC2 Successfully Connected to Amazon RDS MySQL](/images/1-Worklog/week-03/02-ec2-rds-mysql-connected.png)

---

#### Exercise 2: Amazon DynamoDB — NoSQL Table Design, Partition Key & Querying

Created a NoSQL table named **`LabT3_Products`** on **Amazon DynamoDB**. Configured **Partition Key** as **`ProductID`** (String type) serving as the primary identifier. Populated items with dynamic, schemaless attributes:
- Product 1: `ProductID = SP01`, `ProductName = Laptop Dell`, `Price = 15000000`
- Product 2: `ProductID = SP02`, `ProductName = Logitech mouse`, `Color = Black`

Executed a **Scan** operation: The engine scanned all records in the table, returned 2 items, and consumed **2 RCUs** (Read Capacity Units).

> **Screenshot:** ![DynamoDB Scan Operation Consuming 2 RCUs](/images/1-Worklog/week-03/03-dynamodb-scan-products.png)

Executed a **Query** operation: Filtered specifically by Partition Key `ProductID = SP01`. The engine performed a direct hash lookup, scanning exactly 1 item and returning 1 item, consuming only **0.5 RCU** (a 75% read cost reduction compared to Scan).

> **Screenshot:** ![DynamoDB Query Operation Optimizing Read Cost to 0.5 RCU](/images/1-Worklog/week-03/04-dynamodb-query-partition-key.png)

The detailed Query result showed exact product information for `SP01` with sub-millisecond retrieval latency.

> **Screenshot:** ![Detailed Query Result for Product SP01](/images/1-Worklog/week-03/05-dynamodb-query-result.png)

---

#### Exercise 3: ElastiCache — Provision Redis Cache Cluster to Offload Database

Provisioned an in-memory caching cluster using **Amazon ElastiCache for Redis** named **`labt4-rediscache`** configured with enterprise features:
- **Engine:** Redis version `7.1.0`
- **Node type:** `cache.t3.micro`
- **Number of nodes:** `2` (1 Primary Node + 1 Read Replica Node)
- **Multi-AZ:** Enabled (Provides high availability across multiple Availability Zones)
- **Auto-failover:** Enabled (Automatically promotes replica if primary node fails)
- **Encryption:** Encryption at rest (Enabled), Encryption in transit (Required)
- **Primary endpoint:** `master.labt4-rediscache.rfu6sh.use1.cache.amazonaws.com:6379`
- **Reader endpoint:** `replica.labt4-rediscache.rfu6sh.use1.cache.amazonaws.com:6379`
- **Status:** ✅ Available

The ElastiCache cluster allows sub-millisecond response caching, offloading repetitive read-heavy SQL queries from the RDS database.

> **Screenshot:** ![ElastiCache Redis Cluster with Multi-AZ & Auto-failover](/images/1-Worklog/week-03/06-elasticache-redis-cluster.png)

---

#### Exercise 4: Auto Scaling — Create Launch Template & EC2 Auto Scaling Group

Created a **Launch Template** named **`LabT5-AutoScaling`** (Template ID: `lt-0034af3e5980ae08d`, Version 1). The Launch Template stores standardized instance launch specifications:
- AMI: Amazon Linux 2023
- Instance type: `t2.micro`
- Network configuration: Security Group allowing HTTP port 80
- User Data script: Automatically installs and enables Apache web server on boot

> **Screenshot:** ![Launch Template LabT5-AutoScaling Created](/images/1-Worklog/week-03/07-launch-template-created.png)

Created an **Auto Scaling Group** based on the template with Desired Capacity = 1, Minimum = 1, Maximum = 3. The Auto Scaling Group automatically provisioned instance **`i-05e9debdebe0517`** in Availability Zone `us-east-1d`. The instance reached **Running** state with all initial health checks passing.

> **Screenshot:** ![EC2 Instance Automatically Launched by Auto Scaling Group](/images/1-Worklog/week-03/08-asg-instance-running.png)

---

#### Exercise 5: Load Balancer — Configure Application Load Balancer (ALB) for Multi-AZ Traffic

Created an **Application Load Balancer (ALB)** named **`Web-LB`** with the following parameters:
- **Type:** Application Load Balancer (Layer 7 HTTP/HTTPS)
- **Scheme:** Internet-facing (Receives client traffic from public internet)
- **VPC:** `vpc-002805dad0cf2c1ee`
- **Availability Zones:** Spanning 2 separate subnets: `subnet-06775fc833935f53a` (`us-east-1a`) and `subnet-0213687910a63f131` (`us-east-1b`)
- **DNS Name:** `Web-LB-136874904.us-east-1.elb.amazonaws.com` (A Record)

> **Screenshot:** ![Application Load Balancer Web-LB Configuration Details](/images/1-Worklog/week-03/09-alb-provisioning-details.png)

Configured a Target Group containing two web servers placed across different Availability Zones. Accessing the ALB DNS endpoint demonstrated round-robin traffic routing:
- First request: Routed to **Máy chủ WEB 1 - Khu vuc A**.

> **Screenshot:** ![ALB Routing Request to Web Server 1 in Zone A](/images/1-Worklog/week-03/10-alb-target-web1-response.png)

- Subsequent request (Browser refresh): Evenly balanced to **Máy chủ WEB 2 - Khu vuc B**.

> **Screenshot:** ![ALB Alternating Request to Web Server 2 in Zone B](/images/1-Worklog/week-03/11-alb-target-web2-response.png)

---

#### Exercise 6: Comprehensive Lab 3 — High Availability Architecture: ALB + Auto Scaling EC2 + Database & Self-Healing

Constructed a full production-like architecture integrating **Application Load Balancer**, **Auto Scaling Group**, and a **Multi-AZ Database**. The Auto Scaling Group dynamically maintained 2 healthy instances distributed across multiple Availability Zones:
- `i-060adb40952d798d8` in `us-east-1a` (Running, 2/2 checks passed)
- `i-05fd991ecb5911ea0` in `us-east-1b` (Running, 2/2 checks passed)

> **Screenshot:** ![Multi-AZ Auto Scaling Group Instances Healthy](/images/1-Worklog/week-03/12-ha-asg-instances-multi-az.png)

Verified website accessibility via ALB DNS, displaying: *"Hello from Auto Scaling Group"*.

> **Screenshot:** ![Web Access via Load Balancer to Auto Scaling Group](/images/1-Worklog/week-03/13-ha-alb-asg-web-success.png)

**Self-Healing / Auto-Recovery Verification:**
Conducted resilience testing by terminating instance `i-012375caa815c8465` (`Web-ALB`). The ALB health check detected the termination, and the Auto Scaling Group immediately provisioned replacement instance `i-059fb2b8e7c73f56f`, preserving the desired capacity of 2 instances without downtime.

> **Screenshot:** ![Auto Scaling Group Triggering Self-Healing Replacement Instance](/images/1-Worklog/week-03/14-ha-asg-auto-healing-recovery.png)

Upon completing verification, terminated test instances and deleted resources to adhere to AWS Free Tier cost-saving practices.

> **Screenshot:** ![Terminating Lab Compute Resources to Avoid Costs](/images/1-Worklog/week-03/15-cleanup-terminated-instances.png)

---

#### Challenges Encountered

| Issue | Root Cause | Resolution |
| :--- | :--- | :--- |
| EC2 unable to connect to Amazon RDS MySQL (Connection timed out) | RDS Security Group missing inbound rule for port 3306 | Added Inbound Rule to RDS Security Group: Type MySQL/Aurora (port 3306) with source pointing to EC2 Security Group |
| DynamoDB Scan consumes high RCUs and incurs unnecessary read costs | Scan inspects every record sequentially rather than utilizing indexes | Refactored query to use **Query** with Partition Key (`ProductID = SP01`), cutting RCU consumption by 75% |
| Unable to connect to ElastiCache Redis cluster | Cluster is deployed in private subnets with in-transit TLS encryption enabled | Connected via an EC2 instance in the same VPC using `--tls` flag with `redis-cli` |
| Load Balancer reports targets as unhealthy | Web server process stopped or EC2 Security Group blocked port 80 | Verified Apache service status (`sudo systemctl status httpd`) and opened Inbound HTTP port 80 to the ALB Security Group |
| Replacement instance from Auto Scaling Group does not serve web traffic | Launch Template omitted web server installation script | Added bootstrap command `yum install -y httpd` into Launch Template User Data |

### Week 4 Plan

- Explore Serverless compute with AWS Lambda and Amazon API Gateway.
- Architect Event-Driven workflows with Amazon SNS and Amazon SQS.
- Implement asynchronous message decoupling patterns.
- Study event routing using Amazon EventBridge.
