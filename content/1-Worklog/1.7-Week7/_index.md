---
title: "Week 7 Worklog"
date: 2026-10-26
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

## Week 7 Objectives: Infrastructure as Code (IaC)

- **AWS CloudFormation:** Explore the Infrastructure as Code (IaC) paradigm, author declarative YAML templates to automate the provisioning of complete networking topologies (VPC, Subnet, Route Table, Internet Gateway, Security Group) and an EC2 instance with an automated Apache Web Server bootstrap.
- **AWS CDK Basics (Cloud Development Kit):** Get hands-on with AWS CDK – a modern software development framework to define cloud infrastructure using familiar programming languages (Python). Perform project initialization, virtual environment setup, run `cdk bootstrap` to provision CDKToolkit, and deploy an initial Amazon S3 bucket.
- **AWS CDK Advanced:** Program a tightly coupled, multi-tier cloud infrastructure (Two-Tier Architecture): create custom VPCs, carve out Public and Private Subnets, configure Security Group chaining (`AppSG` accepting traffic only from `${WebSG.GroupId}`), and launch concurrent Web and App servers.
- **VS Code Toolkit:** Install and configure the **AWS Toolkit for Visual Studio Code** extension, provision a dedicated IAM user `LabT5-Vscode-toolkit` with programmatic Access Keys, authenticate IDE AWS profiles, and leverage Language Server Protocol (LSP) for in-editor template linting and resource exploration.
- **IaC Workshop:** Convert the manual VPC networking architecture from Week 2 into a clean, reusable CloudFormation YAML template (`vpc-lab.yaml`), achieving 100% automated network provisioning in a single stack execution.
- **Comprehensive Lab 7:** Package full-stack networking and compute infrastructure into a Python CDK project (`lab-t7-cdk`), deploy the complete architecture using a single command `cdk deploy`, verify live web application responsiveness, and practice automated infrastructure teardown with `cdk destroy`.

### Tasks to be Carried Out This Week

| Day | Topic | Task | Start Date | Completion Date | Reference Material |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **Mon** | **AWS CloudFormation** | Author YAML template to provision VPC, Subnet, Security Group, and EC2 Web Server | 10/26/2026 | 10/26/2026 | [AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/) |
| **Tue** | **AWS CDK Basics** | Install AWS CDK, configure Python project, execute `cdk bootstrap`, and deploy S3 Bucket | 10/27/2026 | 10/27/2026 | [AWS CDK Python](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-python.html) |
| **Wed** | **AWS CDK Advanced** | Program multi-tier Web/App architecture, chain Security Groups, deploy and destroy stack | 10/28/2026 | 10/28/2026 | [AWS CDK Networking](https://docs.aws.amazon.com/cdk/api/v2/python/) |
| **Thu** | **VS Code Toolkit** | Install AWS Toolkit, create IAM User `LabT5-Vscode-toolkit`, configure profile & CFN LSP | 10/29/2026 | 10/29/2026 | [AWS Toolkit for VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/) |
| **Fri** | **IaC Workshop** | Convert Week 2 manual VPC into template `vpc-lab.yaml`, successfully deploy stack | 10/30/2026 | 10/30/2026 | [CloudFormation VPC Template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html) |
| **Sat** | **Comprehensive Lab 7** | Deploy end-to-end VPC + EC2 Web Server fleet using a single `cdk deploy` command | 10/31/2026 | 10/31/2026 | [AWS CDK Deployment](https://aws.amazon.com/cdk/) |

### Week 7 Achievements

- Successfully authored declarative CloudFormation YAML template deploying stack **`LabT2-Infrastructure`** spanning 30 `CREATE_COMPLETE` events, automatically provisioning VPC `vpc-02fdae5de629e3e13 (LabT2-VPC)`, Public Subnet `subnet-01e4598557faf4735`, and server `LabT2-EC2-Server` (`i-03e27f7b8dbd6383a`, Public IP: `54.234.168.240`). Verified web application responding with expected congratulations header.
- Installed AWS CDK CLI in CloudShell, bootstrapped foundational stack **`CDKToolkit`** with asset bucket `cdk-hnb659fds-assets-468502414950-us-east-1`. Deployed Python CDK stack **`MyCdkPyStack`** generating S3 bucket `mycdkpystack-myfirstbucketb8884501-csoyhw5ftzwa` in 20.62 seconds.
- Architected advanced multi-tier infrastructure using Python CDK stack **`MyCdkLabt4Stack`**: created VPC `vpc-07408be225170a368` (`10.0.0.0/16`), enforced strict Security Group chaining (`AppSG` accepting traffic exclusively from `${WebSG.GroupId}`), and provisioned `WebServer` (`i-01e6f11153d530bb6`, Public IP: `100.56.245.214`) along with `AppServer` (`i-05d354d92f5427532`, Private IP: `10.0.2.154`). Performed clean teardown via `cdk destroy`.
- Integrated **AWS Toolkit** into Visual Studio Code, configured IAM user `LabT5-Vscode-toolkit` with Access Key `AKIAW2FHLQZTHZVRABAR`, initialized Language Server Protocol `cloudformation-languageserver` v1.14.0 for real-time schema validation and AWS resource tree exploration.
- Completed IaC Workshop converting Week 2 manual VPC topology into CloudFormation template `vpc-lab.yaml`, deploying stack **`LabT6-workshop-IaC`** (28 `CREATE_COMPLETE` events) automating all resources: VPC `LabT6-VPC-Dung`, Public Subnet, Route Table `LabT6-Public-RT`, and Internet Gateway route associations.
- Successfully completed Comprehensive Lab 7: Initialized Python CDK project `lab-t7-cdk`, defined full VPC networking and Web Server EC2 `LabT7CdkStack/LabT7WebServer` (`i-00931b095038d913f`). Deployed entire infrastructure in 179.47 seconds using single command `cdk deploy --require-approval never`, validated live web response at `http://100.52.181.97` (*"Lab T7 Thanh Cong - Trien khai VPC & EC2 bang AWS CDK!"*), and performed zero-residue cleanup via `cdk destroy --force`.

---

#### Exercise 1: AWS CloudFormation — Automated VPC & EC2 Provisioning via Declarative Templates

In conventional cloud administration, engineers manually configure dozens of settings across the AWS Console to provision VPCs, Subnets, Route Tables, Internet Gateways, Security Groups, and EC2 instances. **AWS CloudFormation** eliminates manual toil by expressing the desired end-state architecture in declarative YAML/JSON templates:

**1. CloudFormation Template Architecture:**
The template specifies modular AWS resource declarations:
- `AWS::EC2::VPC`: Provisions an isolated virtual network tagged `LabT2-VPC`.
- `AWS::EC2::Subnet`: Creates Public Subnet `LabT2-PublicSubnet`.
- `AWS::EC2::InternetGateway` & `AWS::EC2::VPCGatewayAttachment`: Creates and attaches the IGW to the VPC.
- `AWS::EC2::RouteTable` & `AWS::EC2::Route`: Configures a routing table directing outbound traffic `0.0.0.0/0` to the Internet Gateway.
- `AWS::EC2::SecurityGroup`: Authorizes inbound HTTP traffic on port 80 and SSH on port 22.
- `AWS::EC2::Instance`: Boots virtual server `LabT2-EC2-Server` with automated UserData bootstrapping Apache HTTP Server (`httpd`) and generating index HTML content.

**2. Stack Execution Lifecycle:**
Uploaded template to CloudFormation Console and executed stack creation named **`LabT2-Infrastructure`**.
- Orchestration engine sequenced and executed **30 lifecycle events**.
- All resources—from `PublicRoute`, `SubnetRouteTableAssociation`, and `SecurityGroup` to `EC2Instance`—transitioned cleanly to **`CREATE_COMPLETE`**.

> **Screenshot:** ![Execution history showing 30 successfully completed resources on CloudFormation Stack LabT2-Infrastructure](/images/1-Worklog/week-07/01-cfn-stack-events-create-complete.png)

**3. EC2 Instance Verification:**
Inspected the Amazon EC2 Console to verify the auto-provisioned compute node:
- **Server Name:** `LabT2-EC2-Server`
- **Instance ID:** `i-03e27f7b8dbd6383a`
- **Instance State:** **`Running`**
- **Instance Type:** `t2.micro` within Availability Zone `us-east-1c`.

> **Screenshot:** ![EC2 server LabT2-EC2-Server in Running state provisioned by CloudFormation](/images/1-Worklog/week-07/02-cfn-ec2-instance-running.png)

**4. Networking Parameters Verification:**
Inspected instance metadata and network associations:
- **Public IPv4:** `54.234.168.240`
- **Private IPv4:** `10.0.1.129`
- **VPC ID:** `vpc-02fdae5de629e3e13 (LabT2-VPC)`
- **Subnet ID:** `subnet-01e4598557faf4735 (LabT2-PublicSubnet)`
- **Hostname:** `ip-10-0-1-129.ec2.internal`

> **Screenshot:** ![Instance summary detailing VPC, Subnet, and Public IPv4 configurations](/images/1-Worklog/week-07/03-cfn-ec2-instance-summary.png)

**5. Application Endpoint Verification:**
Opened a web browser and navigated to `http://54.234.168.240`. The web server responded instantly with the bold confirmation text: **`Lab T2 - AWS CloudFormation - Congratulation on completing labT2 - Thanh Cong!`**.

> **Screenshot:** ![Browser confirming successful web server response from automated UserData script](/images/1-Worklog/week-07/04-cfn-web-server-browser-success.png)

The complete application environment was operational without any manual intervention.

---

#### Exercise 2: AWS CDK Basics — Infrastructure as Code with High-Level Programming (Python)

While declarative YAML templates are robust, managing thousands of lines of static configuration introduces boilerplate redundancy. **AWS Cloud Development Kit (CDK)** provides a modern alternative, allowing software engineers to define cloud infrastructure using expressive, object-oriented programming languages such as Python and TypeScript:

**1. Environment Setup & CDK Bootstrap:**
In AWS CloudShell, initialized a Python CDK project and bootstrapped the environment:
```bash
mkdir my-cdk-py && cd my-cdk-py
cdk init app --language python
source .venv/bin/activate
pip install -r requirements.txt
cdk bootstrap
```
The `cdk bootstrap` command provisions a foundational CloudFormation stack named **`CDKToolkit`** and creates an Amazon S3 asset bucket (`cdk-hnb659fds-assets-468502414950-us-east-1`) to stage synthesized templates and deployment packages.

**2. Coding an S3 Storage Construct:**
In `my_cdk_py/my_cdk_py_stack.py`, defined an Amazon S3 bucket construct:
```python
from aws_cdk import (
    Stack,
    aws_s3 as s3,
)
from constructs import Construct

class MyCdkPyStack(Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)
        s3.Bucket(self, "MyFirstBucket", versioned=True)
```

**3. Deploying Stack via `cdk deploy`:**
Executed the deployment command from CloudShell:
```bash
cdk deploy
```
- **Synthesis Time:** 18.2s
- **Deployment Progress:** `MyCdkPyStack: deploying... [1/1]`
- **Deployment Time:** 20.62s
- **Total Duration:** 46.56s
- **Stack ARN:** `arn:aws:cloudformation:us-east-1:468502414950:stack/MyCdkPyStack/50237ef0-dcde-11f1-98cd-0effe6e65f71`

> **Screenshot:** ![Executing cdk deploy successfully for Python CDK project MyCdkPyStack on CloudShell](/images/1-Worklog/week-07/05-cdk-python-deploy-cloudshell.png)

**4. Validating S3 and CloudFormation Resources:**
- On Amazon S3 Console, verified the CDK asset bucket `cdk-hnb659fds-assets-468502414950-us-east-1` alongside the construct-generated bucket `mycdkpystack-myfirstbucketb8884501-csoyhw5ftzwa`.
- On AWS CloudFormation Console, confirmed both stacks (`CDKToolkit` and `MyCdkPyStack`) active in **`CREATE_COMPLETE`** status.

> **Screenshot:** ![Amazon S3 console listing CDK-managed buckets](/images/1-Worklog/week-07/06-cdk-s3-buckets-created.png)

> **Screenshot:** ![CloudFormation Console confirming CREATE_COMPLETE for MyCdkPyStack and CDKToolkit](/images/1-Worklog/week-07/07-cdk-cloudformation-stacks-complete.png)

---

#### Exercise 3: AWS CDK Advanced — Multi-Tier Interlinked Networking & Compute Architecture

This advanced exercise leverages CDK's cross-construct referencing capabilities to engineer a hardened **Two-Tier Architecture** comprising a public-facing Web tier and an isolated Application tier:

**1. Interlinked Security Group Governance (Chaining):**
Programmed two interdependent Security Groups:
- **WebSG (`sg-006abbaf10746f8fd`):** Accepts HTTP port 80 traffic from anywhere on the internet (`0.0.0.0/0`).
- **AppSG (`sg-0da1562dfc4762a4e`):** Restricts TCP port 8080 ingress strictly to instances belonging to `WebSG` (`${WebSG.GroupId}`), rejecting all direct external access.

During `cdk deploy`, CDK automatically detected security-sensitive updates and rendered an approval matrix:

> **Screenshot:** ![Security Group changes matrix displayed by CDK prior to deployment confirmation](/images/1-Worklog/week-07/08-cdk-multi-tier-deploy-sg-changes.png)

Upon confirming `y`, CDK provisioned the multi-tier infrastructure in **176.86s** (Total time: 212.08s):
- **Stack ARN:** `arn:aws:cloudformation:us-east-1:468502414950:stack/MyCdkLabt4Stack/b7969050-dce7-11f1-bc44-0e70392daf69`
- **Outputs Emitted:**
  - `MyCdkLabt4Stack.AppPrivateIP = 10.0.2.154`
  - `MyCdkLabt4Stack.WebPublicIP = 100.56.245.214`

**2. VPC Networking & Security Group Verification:**
- Custom VPC named **`MyCdkLabt4Stack/LabVpc`** (`vpc-07408be225170a368`) was provisioned with CIDR `10.0.0.0/16`.
- Verified Security Groups: `MyCdkLabt4Stack-WebSG` ("Cho phep HTTP tu internet") and `MyCdkLabt4Stack-AppSG` ("Chi nhan traffic tu Web Server").

> **Screenshot:** ![VPC LabVpc initialized with 10.0.0.0/16 CIDR via CDK construct](/images/1-Worklog/week-07/11-cdk-vpc-created-available.png)

> **Screenshot:** ![Chained Security Groups for Web and Application tiers on AWS Console](/images/1-Worklog/week-07/10-cdk-security-groups-interlinked.png)

**3. EC2 Multi-Tier Cluster Verification:**
Inspected both compute instances running concurrently on EC2 Console:
- `MyCdkLabt4Stack/WebServer` (`i-01e6f11153d530bb6` - `t3.micro`, Running, 3/3 checks passed).
- `MyCdkLabt4Stack/AppServer` (`i-05d354d92f5427532` - `t3.micro`, Running, 3/3 checks passed).

> **Screenshot:** ![Synchronized WebServer and AppServer instances passing 3/3 health checks](/images/1-Worklog/week-07/09-cdk-multi-tier-ec2-instances.png)

> **Screenshot:** ![CloudFormation Stack MyCdkLabt4Stack in CREATE_COMPLETE state](/images/1-Worklog/week-07/12-cdk-multi-tier-cloudformation-stack.png)

**4. Automated Teardown with `cdk destroy`:**
Demonstrated safe lifecycle management by removing all resources via a single command:
```bash
cdk destroy
# Prompt: Are you sure you want to delete: MyCdkLabt4Stack (y/n) y
# MyCdkLabt4Stack: destroying... [1/1]
# MyCdkLabt4Stack: destroyed
```
CDK systematically dismantled VPCs, Subnets, Routing Tables, Gateways, EC2 instances, and Security Groups in reverse dependency order without leaving orphaned assets.

> **Screenshot:** ![Executing cdk destroy to tear down all Lab T4 cloud infrastructure cleanly](/images/1-Worklog/week-07/13-cdk-destroy-cleanup-resources.png)

---

#### Exercise 4: VS Code Toolkit — Developer Environment Integration with AWS Toolkit & Language Server

To optimize development workflows, the **AWS Toolkit for Visual Studio Code** allows cloud developers to manage AWS accounts, explore cloud resources, and receive real-time template autocomplete directly within the IDE:

**1. Dedicated IAM User Provisioning:**
Created an IAM user for IDE integration on the AWS IAM Console:
- **User Name:** `LabT5-Vscode-toolkit`
- **User ARN:** `arn:aws:iam::468502414950:user/LabT5-Vscode-toolkit`
- **Console Access:** Disabled (Programmatic access only)
- **Security Credentials:** Generated Access Key `AKIAW2FHLQZTHZVRABAR` in Active state.

> **Screenshot:** ![Creating dedicated IAM User LabT5-Vscode-toolkit with active Access Key credentials](/images/1-Worklog/week-07/14-iam-user-vscode-toolkit.png)

**2. Configuring AWS Toolkit in VS Code:**
- Installed the AWS Toolkit extension in Visual Studio Code.
- Configured local credentials under profile name **`LabT5-Vscode-toolkit`** targeting region `us-east-1` (N. Virginia), indicated on the status bar: `AWS: profile:LabT5-Vscode-toolkit`.
- Initialized Language Server Protocol client **`cloudformation-languageserver`** v1.14.0 (`cfn-lsp-server-standalone.js`).
- Explored live AWS services (S3 buckets, CDK apps, CloudFormation stacks) through the **AWS Explorer** sidebar without opening an external browser.

> **Screenshot:** ![VS Code workspace integrated with AWS Toolkit and CloudFormation Language Server](/images/1-Worklog/week-07/15-vscode-aws-toolkit-connected.png)

---

#### Exercise 5: IaC Workshop — Converting Manual VPC Architecture into CloudFormation YAML

During Week 2, the VPC networking topology was provisioned manually through point-and-click operations. This workshop codified 100% of that architecture into a production-ready CloudFormation YAML template named **`vpc-lab.yaml`**:

**1. Template Authoring in VS Code:**
Leveraged AWS Toolkit capabilities (attribute auto-completion, schema linting, and interactive `AWS::EC2::RouteTable` documentation tooltips):

```yaml
# 6. Public Route Table
PublicRouteTable:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref MyVPC
    Tags:
      - Key: Name
        Value: LabT6-Public-RT

# 7. Internet Route Definition
PublicRoute:
  Type: AWS::EC2::Route
  DependsOn: AttachGateway
  Properties:
    RouteTableId: !Ref PublicRouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway

# 8. Subnet Route Table Association
PublicSubnetRouteTableAssociation:
  Type: AWS::EC2::SubnetRouteTableAssociation
  Properties:
    SubnetId: !Ref PublicSubnet
    RouteTableId: !Ref PublicRouteTable
```

Key Implementation Detail: Explicitly declared `DependsOn: AttachGateway` to prevent route creation race conditions before the Internet Gateway attachment finishes.

> **Screenshot:** ![Authoring vpc-lab.yaml in VS Code with inline AWS Toolkit documentation tooltips](/images/1-Worklog/week-07/16-vscode-cfn-yaml-template-authoring.png)

**2. Stack Deployment `LabT6-workshop-IaC`:**
Deployed the finalized template to AWS CloudFormation:
- **Stack Name:** `LabT6-workshop-IaC`
- **Description:** `Lab T6 - Chuyen doi VPC thu cong sang IaC bang CloudFormation`
- **Events Log:** All 28 events completed flawlessly, concluding with top-level stack state **`CREATE_COMPLETE`**.

> **Screenshot:** ![CloudFormation Stack LabT6-workshop-IaC successfully completing 28 infrastructure events](/images/1-Worklog/week-07/17-cfn-workshop-stack-create-complete.png)

The standardized network topology is now reproducible across Dev, Staging, and Production environments on demand.

---

#### Exercise 6: Comprehensive Lab 7 — Full-Stack (Network + Server) Deployment via Single Command `cdk deploy`

Comprehensive Lab 7 synthesizes all IaC capabilities acquired throughout the week: authoring a complete Python CDK stack encompassing VPC networking, subnets, routing, security groups, and an Apache EC2 web server; deploying everything with **a single CLI command**; and testing live web traffic.

**Step 1: End-to-End Infrastructure Definition in Python CDK:**
Created project `lab-t7-cdk` and authored `LabT7CdkStack` in `lab_t7_cdk_stack.py`:
```python
class LabT7CdkStack(Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # 1. Provision VPC & Subnets
        vpc = ec2.Vpc(self, "LabT7VPC", ...)

        # 2. Configure Security Group opening port 80
        web_sg = ec2.SecurityGroup(self, "WebServerSG", vpc=vpc, ...)
        web_sg.add_ingress_rule(ec2.Peer.any_ipv4(), ec2.Port.tcp(80))

        # 3. Provision Web Server EC2
        web_server = ec2.Instance(
            self, "LabT7WebServer",
            vpc=vpc,
            vpc_subnets=ec2.SubnetSelection(subnet_type=ec2.SubnetType.PUBLIC),
            instance_type=ec2.InstanceType.of(ec2.InstanceClass.T2, ec2.InstanceSize.MICRO),
            machine_image=ec2.MachineImage.latest_amazon_linux2023(),
            security_group=web_sg,
        )
        web_server.user_data.add_commands(
            "sudo dnf install -y httpd",
            "sudo systemctl enable --now httpd",
            "echo 'Lab T7 Thanh Cong - Trien khai VPC & EC2 bang AWS CDK!' > /var/www/html/index.html"
        )
```

**Step 2: One-Command Deployment from VS Code Terminal:**
In the VS Code integrated terminal, executed:
```powershell
(.venv) PS C:\Users\Admin\lab-t7-cdk> cdk deploy --require-approval never
```
- Automatically synthesized and provisioned all 19 cloud resources.
- **Deployment Duration:** `179.47s` (Total time: `211.51s`).
- **Stack Output:** `LabT7CdkStack.WebServerPublicIP = http://100.52.181.97`
- **Stack ARN:** `arn:aws:cloudformation:us-east-1:468502414950:stack/LabT7CdkStack/e4c4fa10-e0db-11f1-8dbb-0afffd563d2b`

> **Screenshot:** ![Single cdk deploy command rolling out complete infrastructure from VS Code terminal](/images/1-Worklog/week-07/18-cdk-deploy-full-stack-vscode.png)

**Step 3: Console Verification of Provisioned Resources:**
- **CloudFormation Stacks:** Confirmed all three lab stacks in **`CREATE_COMPLETE`** state: `LabT7CdkStack`, `CDKToolkit`, and `LabT6-workshop-IaC`.

> **Screenshot:** ![CloudFormation Console showing all three lab stacks in CREATE_COMPLETE state](/images/1-Worklog/week-07/19-cfn-stacks-overview-t7.png)

- **Amazon VPC:** VPC `LabT7CdkStack/LabT7VPC` (`vpc-02fe48766e7345ae7`) verified in `Available` state alongside `LabT6-VPC-Dung` (`vpc-0381c6cc2ff3a8741`).

> **Screenshot:** ![VPC LabT7VPC successfully initialized in Available state](/images/1-Worklog/week-07/20-vpc-labt7-created-available.png)

- **Amazon EC2:** Instance `LabT7CdkStack/LabT7WebServer` (`i-00931b095038d913f`) running actively in `us-east-1a`.

> **Screenshot:** ![Web Server instance created by CDK actively running on EC2 Console](/images/1-Worklog/week-07/21-ec2-labt7-web-server-running.png)

**Step 4: End-User Application Browser Validation:**
Navigated to public endpoint `http://100.52.181.97`. Browser displayed the deployment success banner:
**`Lab T7 Thanh Cong - Trien khai VPC & EC2 bang AWS CDK!`**

> **Screenshot:** ![Browser confirming live web application serving traffic successfully via AWS CDK](/images/1-Worklog/week-07/22-browser-web-app-cdk-success.png)

**Step 5: Automated Infrastructure Cleanup:**
Following testing validation, torn down the complete environment with one command:
```powershell
(.venv) PS C:\Users\Admin\lab-t7-cdk> cdk destroy --force
```
CDK removed all 19 underlying resources in sequence (`DELETE_COMPLETE`), concluding with output: **`LabT7CdkStack: destroyed`**.

> **Screenshot:** ![Executing cdk destroy --force to completely tear down 19 resources in minutes](/images/1-Worklog/week-07/23-cdk-destroy-full-stack-cleanup.png)

---

#### Challenges & Solutions

| Issue Encountered | Root Cause | Resolution |
| :--- | :--- | :--- |
| CloudFormation error: `Network vpc-... is not attached to an internet gateway` | `AWS::EC2::Route` creation evaluated before `VPCGatewayAttachment` finished binding IGW to the VPC | Added explicit `DependsOn: AttachGateway` attribute to Route resource to enforce proper provisioning sequence |
| `cdk deploy` hangs awaiting security prompt approval `(y/n)` | CDK defaults to interactive confirmation prompts when Security Group rules or IAM policies broaden | Passed configuration flag `--require-approval never` during automated CI/CD and script execution |
| VS Code error: `Unable to authenticate connection / InvalidClientTokenId` | Secret access keys corrupted or wrong regional target set in local credentials | Regenerated active Access Key pair for user `LabT5-Vscode-toolkit`, reconnected via `AWS: Connect to AWS`, and selected `us-east-1` |
| CDK EC2 instance fails to obtain Public IP for web traffic | Default subnet selection defaulted to private subnets without public IPv4 auto-assignment | Explicitly configured parameter `vpc_subnets=ec2.SubnetSelection(subnet_type=ec2.SubnetType.PUBLIC)` in the instance construct |
| Manual console deletion triggers `DependencyViolation` errors | Complex inter-resource dependencies (subnets bound to route tables, cross-referencing security groups) prevent out-of-order deletion | Leveraged IaC automation via `cdk destroy --force` or stack deletion, letting AWS orchestrate reverse topological deletion order |

### Next Week Plan: Automation & CI/CD on AWS

- Study continuous integration and continuous deployment using **AWS CodePipeline**, **AWS CodeBuild**, and **AWS CodeDeploy**.
- Construct automated pipelines for source code testing and Docker container image compilation.
- Securely store and version container images using **Amazon Elastic Container Registry (ECR)**.
- Deploy serverless containerized microservices using **Amazon ECS on AWS Fargate**.
- Set up real-time pipeline event notifications and build failure alerts via **Amazon SNS** to Email/Discord.
