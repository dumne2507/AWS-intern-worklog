---
title: "Worklog Tuần 7"
date: 2026-10-26
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

## Mục tiêu tuần 7: IaC (Code hạ tầng)

- **AWS CloudFormation:** Tìm hiểu mô hình hạ tầng dưới dạng mã (Infrastructure as Code - IaC), viết bản mẫu khai báo định dạng YAML khởi tạo tự động toàn bộ hạ tầng mạng VPC, Subnet, Route Table, Internet Gateway, Security Group và máy chủ ảo EC2 cài đặt sẵn Web Server Apache.
- **AWS CDK Cơ bản (Cloud Development Kit):** Làm quen với CDK – framework phát triển hạ tầng hiện đại bằng ngôn ngữ lập trình bậc cao (Python). Thực hiện quy trình khởi tạo dự án, cài đặt môi trường ảo (virtualenv), thực thi `cdk bootstrap` triển khai CDKToolkit và deploy S3 Bucket đầu tiên.
- **AWS CDK Nâng cao:** Lập trình kiến trúc mạng và máy chủ đa tầng có tính liên kết chặt chẽ (Two-Tier Architecture): tạo VPC, chia Public/Private Subnet, xây dựng các Security Group tham chiếu chéo (Security Group Chaining) và khởi chạy đồng thời Web Server cùng App Server.
- **VS Code Toolkit:** Cài đặt và tích hợp extension **AWS Toolkit for Visual Studio Code**, tạo người dùng IAM `LabT5-Vscode-toolkit` cấp Access Key, cấu hình xác thực profile AWS và tận dụng Language Server Protocol (LSP) hỗ trợ kiểm tra cú pháp, gợi ý code template trực tiếp trong IDE.
- **Workshop IaC:** Chuyển đổi toàn bộ kiến trúc mạng VPC thủ công của Lab Tuần 2 thành file mã nguồn YAML (`vpc-lab.yaml`) chuẩn CloudFormation, tự động hóa 100% quy trình cấp phát mạng chỉ trong một lần thực thi Stack.
- **Lab Tổng hợp 7:** Đóng gói toàn bộ kiến trúc Mạng + Máy chủ vào dự án AWS CDK Python (`lab-t7-cdk`), triển khai toàn bộ hệ thống bằng đúng 1 dòng lệnh duy nhất `cdk deploy`, kiểm tra hoạt động của Web Application và thực hành dọn dẹp sạch sẽ tài nguyên bằng `cdk destroy`.

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **2** | **AWS CloudFormation** | Viết template YAML khởi tạo VPC, Subnet, Security Group và EC2 Web Server tự động | 26/10/2026 | 26/10/2026 | [AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/) |
| **3** | **AWS CDK Cơ bản** | Cài đặt AWS CDK, cấu hình dự án Python CDK, chạy `cdk bootstrap` và deploy S3 Bucket | 27/10/2026 | 27/10/2026 | [AWS CDK Python](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-python.html) |
| **4** | **AWS CDK Nâng cao** | Lập trình kiến trúc liên kết Web/App, ràng buộc Security Group, deploy và destroy stack | 28/10/2026 | 28/10/2026 | [AWS CDK Networking](https://docs.aws.amazon.com/cdk/api/v2/python/) |
| **5** | **VS Code Toolkit** | Cài đặt AWS Toolkit, tạo IAM User `LabT5-Vscode-toolkit`, cấu hình profile và CloudFormation LSP | 29/10/2026 | 29/10/2026 | [AWS Toolkit for VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/) |
| **6** | **Workshop IaC** | Chuyển đổi VPC Tuần 2 thành template CloudFormation `vpc-lab.yaml`, deploy stack thành công | 30/10/2026 | 30/10/2026 | [CloudFormation VPC Template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html) |
| **7** | **Lab Tổng hợp 7** | Triển khai toàn bộ hệ thống VPC + EC2 Web Server chỉ bằng 1 dòng lệnh `cdk deploy` | 31/10/2026 | 31/10/2026 | [AWS CDK Deployment](https://aws.amazon.com/cdk/) |

### Kết quả đạt được tuần 7

- Viết thành công bản mẫu CloudFormation YAML triển khai stack **`LabT2-Infrastructure`** bao gồm 30 sự kiện `CREATE_COMPLETE`, tự động cấp phát VPC `vpc-02fdae5de629e3e13 (LabT2-VPC)`, Public Subnet `subnet-01e4598557faf4735` và máy chủ `LabT2-EC2-Server` (`i-03e27f7b8dbd6383a`, Public IP: `54.234.168.240`). Ứng dụng web phản hồi chính xác thông điệp chúc mừng trên trình duyệt.
- Cài đặt hoàn tất AWS CDK CLI trong môi trường CloudShell, kích hoạt thành công stack nền tảng **`CDKToolkit`** kèm S3 bucket lưu trữ assets `cdk-hnb659fds-assets-468502414950-us-east-1`. Triển khai thành công stack Python **`MyCdkPyStack`** tạo S3 bucket `mycdkpystack-myfirstbucketb8884501-csoyhw5ftzwa` trong 20.62 giây.
- Xây dựng kiến trúc hạ tầng nâng cao đa tầng bằng Python CDK với stack **`MyCdkLabt4Stack`**: khởi tạo mạng VPC `vpc-07408be225170a368` (`10.0.0.0/16`), thiết lập chuỗi phụ thuộc Security Group (`AppSG` chỉ nhận traffic từ `${WebSG.GroupId}`), đồng thời khởi chạy thành công cặp máy chủ `WebServer` (`i-01e6f11153d530bb6`, Public IP: `100.56.245.214`) và `AppServer` (`i-05d354d92f5427532`, Private IP: `10.0.2.154`). Thực hành dọn dẹp sạch sẽ tài nguyên bằng `cdk destroy`.
- Tích hợp thành công **AWS Toolkit** trên Visual Studio Code, tạo IAM user `LabT5-Vscode-toolkit` với Access Key `AKIAW2FHLQZTHZVRABAR`, kích hoạt Language Server Protocol `cloudformation-languageserver` v1.14.0 giúp hiển thị tài liệu cú pháp và quản lý tài nguyên AWS trực quan ngay trong IDE.
- Hoàn thành bài Workshop IaC chuyển đổi kiến trúc Tuần 2 sang template CloudFormation `vpc-lab.yaml`, triển khai thành công stack **`LabT6-workshop-IaC`** (28 sự kiện `CREATE_COMPLETE`) tự động hóa toàn bộ tài nguyên: VPC `LabT6-VPC-Dung`, Public Subnet, Route Table `LabT6-Public-RT`, Internet Gateway và các liên kết định tuyến.
- Hoàn thành xuất sắc Lab Tổng hợp 7: Khởi tạo dự án Python CDK `lab-t7-cdk`, lập trình toàn diện VPC và Web Server EC2 `LabT7CdkStack/LabT7WebServer` (`i-00931b095038d913f`). Triển khai toàn bộ hạ tầng chỉ với 1 câu lệnh `cdk deploy --require-approval never` trong 179.47 giây, xác thực trang web hoạt động hoàn hảo tại `http://100.52.181.97` với thông báo *"Lab T7 Thanh Cong - Trien khai VPC & EC2 bang AWS CDK!"*, và thực hiện hủy hạ tầng an toàn với `cdk destroy --force`.

---

#### Bài tập 1: AWS CloudFormation — Tự động hóa Khởi tạo VPC & EC2 bằng Template Khai báo

Trong phương pháp truyền thống, kỹ sư phải click chuột qua hàng chục bước thủ công trên AWS Console để tạo VPC, Subnet, Route Table, Internet Gateway, Security Group và gắn vào EC2. Với **AWS CloudFormation**, toàn bộ kiến trúc hạ tầng được mô tả dưới dạng tệp khai báo (Declarative Template) chuẩn định dạng YAML/JSON:

**1. Cấu trúc template CloudFormation triển khai:**
File template định nghĩa các khối tài nguyên cốt lõi:
- `AWS::EC2::VPC`: Khởi tạo dải mạng ảo riêng biệt mang tag `LabT2-VPC`.
- `AWS::EC2::Subnet`: Khởi tạo Public Subnet `LabT2-PublicSubnet`.
- `AWS::EC2::InternetGateway` & `AWS::EC2::VPCGatewayAttachment`: Tạo và đính kèm cổng Internet vào VPC.
- `AWS::EC2::RouteTable` & `AWS::EC2::Route`: Thiết lập bảng định tuyến mở luồng mạng `0.0.0.0/0` hướng ra Internet Gateway.
- `AWS::EC2::SecurityGroup`: Cho phép traffic HTTP inbound cổng 80 và SSH cổng 22.
- `AWS::EC2::Instance`: Khởi chạy máy chủ ảo `LabT2-EC2-Server` sử dụng đoạn mã UserData tự động cài đặt Apache HTTP Server (`httpd`) và tạo trang chủ web.

**2. Quá trình thực thi Stack:**
Đăng nhập CloudFormation Console, tải lên template và tiến hành tạo Stack mang tên **`LabT2-Infrastructure`**.
- Hệ thống điều phối tuần tự và ghi nhận tổng cộng **30 sự kiện (Events)**.
- Mọi tài nguyên từ `PublicRoute`, `SubnetRouteTableAssociation`, `SecurityGroup` cho tới `EC2Instance` đều hoàn thành với trạng thái **`CREATE_COMPLETE`**.

> **Screenshot:** ![Lịch sử sự kiện khởi tạo thành công 30 tài nguyên trên CloudFormation Stack LabT2-Infrastructure](/images/1-Worklog/week-07/01-cfn-stack-events-create-complete.png)

**3. Kiểm tra tài nguyên máy chủ EC2:**
Truy cập Amazon EC2 Console để xác minh máy chủ vừa được CloudFormation tự động sinh ra:
- **Tên máy chủ:** `LabT2-EC2-Server`
- **Instance ID:** `i-03e27f7b8dbd6383a`
- **Trạng thái:** **`Running`** (Instance state: Running)
- **Loại phiên bản:** `t2.micro` tại Availability Zone `us-east-1c`.

> **Screenshot:** ![Máy chủ EC2 LabT2-EC2-Server hoạt động ổn định sau khi tạo bởi CloudFormation](/images/1-Worklog/week-07/02-cfn-ec2-instance-running.png)

**4. Xác minh chi tiết cấu hình mạng:**
Xem chi tiết phần thông tin máy chủ:
- **Public IPv4:** `54.234.168.240`
- **Private IPv4:** `10.0.1.129`
- **VPC ID:** `vpc-02fdae5de629e3e13 (LabT2-VPC)`
- **Subnet ID:** `subnet-01e4598557faf4735 (LabT2-PublicSubnet)`
- **Hostname:** `ip-10-0-1-129.ec2.internal`

> **Screenshot:** ![Bảng chi tiết thông tin mạng VPC, Subnet và địa chỉ IP Public của máy chủ](/images/1-Worklog/week-07/03-cfn-ec2-instance-summary.png)

**5. Kiểm chứng hoạt động của ứng dụng Web:**
Mở trình duyệt web và điều hướng tới địa chỉ `http://54.234.168.240`. Web Server phản hồi ngay lập tức với tiêu đề in đậm: **`Lab T2 - AWS CloudFormation - Congratulation on completing labT2 - Thanh Cong!`**.

> **Screenshot:** ![Trang web phản hồi thành công xác nhận kịch bản UserData đã cài đặt Apache hoàn tất](/images/1-Worklog/week-07/04-cfn-web-server-browser-success.png)

Toàn bộ hệ thống hoạt động hoàn chỉnh mà không cần chạm tay cấu hình thủ công bất kỳ thành phần nào.

---

#### Bài tập 2: AWS CDK Cơ bản — Khởi tạo Dự án IaC bằng Ngôn ngữ Lập trình Bậc cao (Python)

Mặc dù CloudFormation YAML rất mạnh mẽ, nhưng việc viết hàng nghìn dòng cấu hình tĩnh dễ gây lặp lại mã và khó tái sử dụng. **AWS Cloud Development Kit (CDK)** mang lại bước tiến lớn khi cho phép sử dụng ngôn ngữ lập trình phổ biến (như Python, TypeScript) với đầy đủ cấu trúc hướng đối tượng (OOP), hàm và biến để định nghĩa hạ tầng:

**1. Chuẩn bị môi trường & CDK Bootstrap:**
Trong AWS CloudShell, khởi tạo môi trường Python và thực thi lệnh khởi tạo nền tảng:
```bash
mkdir my-cdk-py && cd my-cdk-py
cdk init app --language python
source .venv/bin/activate
pip install -r requirements.txt
cdk bootstrap
```
Quá trình `cdk bootstrap` triển khai stack nền tảng mang tên **`CDKToolkit`** trên CloudFormation, đồng thời tự động tạo một kho lưu trữ **Amazon S3 Bucket** mang tên `cdk-hnb659fds-assets-468502414950-us-east-1` để chứa các file assets, templates phục vụ cho việc triển khai ứng dụng.

**2. Lập trình khởi tạo S3 Bucket đầu tiên:**
Trong file `my_cdk_py/my_cdk_py_stack.py`, bổ sung định nghĩa S3 Bucket đơn giản:
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

**3. Triển khai hạ tầng với `cdk deploy`:**
Chạy lệnh triển khai trực tiếp từ terminal CloudShell:
```bash
cdk deploy
```
- **Thời gian tổng hợp (Synthesis time):** 18.2s
- **Tiến trình triển khai:** `MyCdkPyStack: deploying... [1/1]`
- **Thời gian triển khai thực tế (Deployment time):** 20.62s
- **Tổng thời gian hoàn tất:** 46.56s
- **Stack ARN:** `arn:aws:cloudformation:us-east-1:468502414950:stack/MyCdkPyStack/50237ef0-dcde-11f1-98cd-0effe6e65f71`

> **Screenshot:** ![Thực thi cdk deploy thành công dự án Python CDK MyCdkPyStack trên AWS CloudShell](/images/1-Worklog/week-07/05-cdk-python-deploy-cloudshell.png)

**4. Xác minh tài nguyên S3 và CloudFormation:**
- Trên Amazon S3 Console, hệ thống hiển thị bucket CDK bootstrap `cdk-hnb659fds-assets-468502414950-us-east-1` cùng bucket mới sinh ra theo mã code: `mycdkpystack-myfirstbucketb8884501-csoyhw5ftzwa`.
- Trên AWS CloudFormation Console, ghi nhận 2 Stacks hoạt động song song ở trạng thái **`CREATE_COMPLETE`**: `CDKToolkit` và `MyCdkPyStack`.

> **Screenshot:** ![Danh sách các S3 Bucket được quản lý và tạo ra tự động bởi AWS CDK](/images/1-Worklog/week-07/06-cdk-s3-buckets-created.png)

> **Screenshot:** ![CloudFormation Stacks ghi nhận trạng thái CREATE_COMPLETE của MyCdkPyStack và CDKToolkit](/images/1-Worklog/week-07/07-cdk-cloudformation-stacks-complete.png)

---

#### Bài tập 3: AWS CDK Nâng cao — Xây dựng Kiến trúc Mạng & Máy chủ Đa tầng Có tính Liên kết

Bài tập nâng cao vận dụng sức mạnh liên kết tài nguyên của AWS CDK để xây dựng mô hình **Two-Tier Architecture** gồm tầng Web đối ngoại (Public) và tầng Ứng dụng nghiệp vụ (Private):

**1. Thiết lập các ràng buộc bảo mật (Security Group Chaining):**
Trong code CDK, chúng ta định nghĩa 2 Security Groups có quan hệ phụ thuộc:
- **WebSG (`sg-006abbaf10746f8fd`):** Cho phép kết nối HTTP cổng 80 từ toàn bộ Internet (`0.0.0.0/0`).
- **AppSG (`sg-0da1562dfc4762a4e`):** Chỉ cho phép kết nối cổng TCP 8080 bắt nguồn duy nhất từ `WebSG` (`${WebSG.GroupId}`), từ chối mọi kết nối trực tiếp từ bên ngoài.

Khi thực thi `cdk deploy`, CDK tự động phát hiện các thay đổi liên quan đến bảo mật (Security-sensitive updates) và hiển thị bảng đối chiếu rõ ràng:

> **Screenshot:** ![Bảng đối chiếu thay đổi quy tắc an ninh Security Group trước khi xác nhận triển khai](/images/1-Worklog/week-07/08-cdk-multi-tier-deploy-sg-changes.png)

Sau khi xác nhận `y`, hệ thống tiến hành tạo toàn bộ hạ tầng trong thời gian **176.86s** (Tổng thời gian: 212.08s):
- **Stack ARN:** `arn:aws:cloudformation:us-east-1:468502414950:stack/MyCdkLabt4Stack/b7969050-dce7-11f1-bc44-0e70392daf69`
- **Outputs ghi nhận:**
  - `MyCdkLabt4Stack.AppPrivateIP = 10.0.2.154`
  - `MyCdkLabt4Stack.WebPublicIP = 100.56.245.214`

**2. Kiểm tra hạ tầng mạng VPC và Security Groups:**
- Mạng VPC mới được tạo tự động mang tên **`MyCdkLabt4Stack/LabVpc`** (`vpc-07408be225170a368`) với dải địa chỉ CIDR chuẩn `10.0.0.0/16`.
- Bảng Security Groups hiển thị chuẩn xác `MyCdkLabt4Stack-WebSG` ("Cho phep HTTP tu internet") và `MyCdkLabt4Stack-AppSG` ("Chi nhan traffic tu Web Server").

> **Screenshot:** ![VPC LabVpc được khởi tạo với dải mạng 10.0.0.0/16 thông qua mã nguồn CDK](/images/1-Worklog/week-07/11-cdk-vpc-created-available.png)

> **Screenshot:** ![Hai Security Groups liên kết chặt chẽ cho tầng Web và tầng App trên AWS Console](/images/1-Worklog/week-07/10-cdk-security-groups-interlinked.png)

**3. Kiểm tra cụm máy chủ EC2:**
Tại giao diện EC2 Console, 2 máy chủ ảo hoạt động đồng thời:
- `MyCdkLabt4Stack/WebServer` (`i-01e6f11153d530bb6` - `t3.micro`, Running, 3/3 checks passed).
- `MyCdkLabt4Stack/AppServer` (`i-05d354d92f5427532` - `t3.micro`, Running, 3/3 checks passed).

> **Screenshot:** ![Cặp máy chủ WebServer và AppServer hoạt động đồng bộ đạt 3/3 checks passed](/images/1-Worklog/week-07/09-cdk-multi-tier-ec2-instances.png)

> **Screenshot:** ![CloudFormation Stack MyCdkLabt4Stack ghi nhận trạng thái CREATE_COMPLETE](/images/1-Worklog/week-07/12-cdk-multi-tier-cloudformation-stack.png)

**4. Thực hành dọn dẹp hạ tầng an toàn với `cdk destroy`:**
Một trong những ưu điểm vượt trội của IaC là khả năng xóa sạch toàn bộ môi trường mà không lo sót tài nguyên rác phát sinh chi phí:
```bash
cdk destroy
# Xác nhận: Are you sure you want to delete: MyCdkLabt4Stack (y/n) y
# MyCdkLabt4Stack: destroying... [1/1]
# MyCdkLabt4Stack: destroyed
```
Toàn bộ VPC, Subnets, Routing Tables, Gateways, EC2 và Security Groups được tháo dỡ tuần tự theo đúng chu trình phụ thuộc ngược.

> **Screenshot:** ![Thực thi lệnh cdk destroy dọn dẹp sạch sẽ toàn bộ tài nguyên môi trường Lab T4](/images/1-Worklog/week-07/13-cdk-destroy-cleanup-resources.png)

---

#### Bài tập 4: VS Code Toolkit — Tích hợp AWS Toolkit và Language Server vào Môi trường Phát triển

Để nâng cao năng suất viết mã hạ tầng, tiện ích **AWS Toolkit for Visual Studio Code** cho phép lập trình viên kết nối trực tiếp đến tài khoản AWS, duyệt cây tài nguyên và nhận hỗ trợ gợi ý cú pháp theo thời gian thực:

**1. Khởi tạo tài khoản người dùng IAM riêng biệt:**
Truy cập AWS IAM Console để tạo IAM User chuyên dụng phục vụ việc tích hợp vào IDE:
- **Tên người dùng:** `LabT5-Vscode-toolkit`
- **User ARN:** `arn:aws:iam::468502414950:user/LabT5-Vscode-toolkit`
- **Truy cập Console:** Disabled (Chỉ dùng Programmatic access)
- **Thông tin xác thực (Security credentials):** Tạo Access Key `AKIAW2FHLQZTHZVRABAR` ở trạng thái Active.

> **Screenshot:** ![Tạo người dùng IAM LabT5-Vscode-toolkit cấp Access Key phục vụ kết nối VS Code](/images/1-Worklog/week-07/14-iam-user-vscode-toolkit.png)

**2. Tích hợp và cấu hình AWS Toolkit trên VS Code:**
- Cài đặt extension AWS Toolkit trong Visual Studio Code.
- Cấu hình thông tin xác thực AWS credentials với profile mang tên **`LabT5-Vscode-toolkit`** trỏ về Region `us-east-1` (N. Virginia). Thanh trạng thái dưới cùng của VS Code hiển thị biểu tượng: `AWS: profile:LabT5-Vscode-toolkit`.
- Tiện ích tự động tải và khởi chạy trình phân tích ngôn ngữ **`cloudformation-languageserver`** phiên bản `v1.14.0` (`cfn-lsp-server-standalone.js`).
- Khung **AWS Explorer** ở cột bên trái cho phép tương tác trực tiếp với các dịch vụ S3, CDK Apps, CloudFormation Stacks mà không cần mở trình duyệt web.

> **Screenshot:** ![Giao diện VS Code tích hợp thành công AWS Toolkit và nạp CloudFormation Language Server](/images/1-Worklog/week-07/15-vscode-aws-toolkit-connected.png)

---

#### Bài tập 5: Workshop IaC — Chuyển đổi Kiến trúc Mạng Thủ công sang Code CloudFormation

Tại Tuần 2, toàn bộ hạ tầng mạng VPC phải tạo thủ công qua giao diện Console mất nhiều thời gian và dễ nhầm lẫn. Bài workshop này thực hiện "Code hóa" 100% kiến trúc mạng đó thành một tệp YAML chuẩn mang tên **`vpc-lab.yaml`**:

**1. Soạn thảo tệp mẫu hạ tầng `vpc-lab.yaml` trên VS Code:**
Tận dụng tính năng hỗ trợ của AWS Toolkit (tự động gợi ý thuộc tính bắt buộc như `VpcId`, kiểm tra lỗi cú pháp và hiển thị tài liệu mẫu `AWS::EC2::RouteTable` qua tooltip):

```yaml
# 6. Tạo Route Table cho Public Subnet
PublicRouteTable:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref MyVPC
    Tags:
      - Key: Name
        Value: LabT6-Public-RT

# 7. Cấu hình Route: Đẩy luồng mạng 0.0.0.0/0 ra Internet Gateway
PublicRoute:
  Type: AWS::EC2::Route
  DependsOn: AttachGateway
  Properties:
    RouteTableId: !Ref PublicRouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway

# 8. Liên kết Route Table đó vào Public Subnet
PublicSubnetRouteTableAssociation:
  Type: AWS::EC2::SubnetRouteTableAssociation
  Properties:
    SubnetId: !Ref PublicSubnet
    RouteTableId: !Ref PublicRouteTable
```

Lưu ý quan trọng: Thuộc tính `DependsOn: AttachGateway` được khai báo rõ ràng để đảm bảo định tuyến Internet chỉ được tạo sau khi Internet Gateway đã gắn kết nối thành công với VPC.

> **Screenshot:** ![Soạn thảo template vpc-lab.yaml trên VS Code với sự hỗ trợ tài liệu trực tiếp từ AWS Toolkit](/images/1-Worklog/week-07/16-vscode-cfn-yaml-template-authoring.png)

**2. Triển khai Stack `LabT6-workshop-IaC`:**
Thực hiện triển khai template lên AWS CloudFormation:
- **Tên Stack:** `LabT6-workshop-IaC`
- **Mô tả:** `Lab T6 - Chuyen doi VPC thu cong sang IaC bang CloudFormation`
- **Lịch sử sự kiện:** Quá trình tạo diễn ra mượt mà với 28 sự kiện, kết thúc với sự kiện tổng thể mang trạng thái **`CREATE_COMPLETE`**.

> **Screenshot:** ![CloudFormation Stack LabT6-workshop-IaC hoàn thành 28 sự kiện khởi tạo hạ tầng mạng](/images/1-Worklog/week-07/17-cfn-workshop-stack-create-complete.png)

Hạ tầng mạng chuẩn hóa đã sẵn sàng để tái sử dụng hoặc nhân bản sang nhiều môi trường (Dev, Staging, Prod) mà không cần cấu hình lại bằng tay.

---

#### Bài tập 6: Lab Tổng hợp 7 — Triển khai Hạ tầng Hoàn chỉnh (Mạng + Máy chủ) Chỉ bằng 1 Dòng Lệnh `cdk deploy`

Lab Tổng hợp 7 là bài thực hành đỉnh cao kết hợp toàn bộ kỹ năng IaC: Viết mã nguồn hoàn chỉnh bằng Python CDK để dựng toàn bộ hệ thống gồm Mạng VPC, Subnet, Routing, Security Groups và Web Server EC2; triển khai chỉ bằng **một câu lệnh duy nhất**; xác thực ứng dụng và hủy hạ tầng tự động.

**Bước 1: Lập trình hạ tầng hoàn chỉnh với Python CDK:**
Tạo dự án `lab-t7-cdk` và xây dựng stack `LabT7CdkStack` trong file `lab_t7_cdk_stack.py`:
```python
class LabT7CdkStack(Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # 1. Khởi tạo VPC & Subnets
        vpc = ec2.Vpc(self, "LabT7VPC", ...)

        # 2. Khởi tạo Security Group mở port 80
        web_sg = ec2.SecurityGroup(self, "WebServerSG", vpc=vpc, ...)
        web_sg.add_ingress_rule(ec2.Peer.any_ipv4(), ec2.Port.tcp(80))

        # 3. Khởi tạo Web Server EC2
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

**Bước 2: Triển khai 1 câu lệnh duy nhất từ VS Code Terminal:**
Mở PowerShell tích hợp trong VS Code và thực thi lệnh:
```powershell
(.venv) PS C:\Users\Admin\lab-t7-cdk> cdk deploy --require-approval never
```
- Quá trình tổng hợp và triển khai tự động nạp toàn bộ 19 tài nguyên.
- **Thời gian triển khai:** `179.47s` (Tổng thời gian: `211.51s`).
- **Outputs trả về:** `LabT7CdkStack.WebServerPublicIP = http://100.52.181.97`
- **Stack ARN:** `arn:aws:cloudformation:us-east-1:468502414950:stack/LabT7CdkStack/e4c4fa10-e0db-11f1-8dbb-0afffd563d2b`

> **Screenshot:** ![Lệnh cdk deploy thực thi thành công toàn bộ hạ tầng mạng và máy chủ từ VS Code terminal](/images/1-Worklog/week-07/18-cdk-deploy-full-stack-vscode.png)

**Bước 3: Xác minh các tài nguyên trên AWS Console:**
- **CloudFormation Stacks:** Cả 3 Stacks phục vụ các bài thực hành đều hiển thị xanh tươi ở trạng thái `CREATE_COMPLETE`: `LabT7CdkStack`, `CDKToolkit` và `LabT6-workshop-IaC`.

> **Screenshot:** ![Danh mục các CloudFormation Stacks ghi nhận trạng thái hoạt động hoàn hảo](/images/1-Worklog/week-07/19-cfn-stacks-overview-t7.png)

- **Amazon VPC:** VPC `LabT7CdkStack/LabT7VPC` (`vpc-02fe48766e7345ae7`) đã sẵn sàng cùng `LabT6-VPC-Dung` (`vpc-0381c6cc2ff3a8741`).

> **Screenshot:** ![VPC LabT7VPC được khởi tạo thành công ở trạng thái Available](/images/1-Worklog/week-07/20-vpc-labt7-created-available.png)

- **Amazon EC2:** Máy chủ `LabT7CdkStack/LabT7WebServer` (`i-00931b095038d913f`) đang hoạt động ở trạng thái `Running` tại `us-east-1a`.

> **Screenshot:** ![Máy chủ Web Server tạo bởi CDK hoạt động ở trạng thái Running trên EC2 Console](/images/1-Worklog/week-07/21-ec2-labt7-web-server-running.png)

**Bước 4: Kiểm chứng truy cập ứng dụng trên trình duyệt:**
Truy cập đường dẫn public IP `http://100.52.181.97`. Trình duyệt web hiển thị tức thì dòng thông điệp xác thực:
**`Lab T7 Thanh Cong - Trien khai VPC & EC2 bang AWS CDK!`**

> **Screenshot:** ![Trình duyệt web xác nhận ứng dụng web triển khai tự động qua AWS CDK phản hồi hoàn hảo](/images/1-Worklog/week-07/22-browser-web-app-cdk-success.png)

**Bước 5: Tự động hóa hủy hạ tầng an toàn:**
Sau khi hoàn tất bài nghiệm thu, chỉ cần thực thi 1 câu lệnh để dọn dẹp sạch sẽ toàn bộ môi trường:
```powershell
(.venv) PS C:\Users\Admin\lab-t7-cdk> cdk destroy --force
```
CDK tự động lần lượt xóa toàn bộ 19 tài nguyên (`DELETE_COMPLETE`) từ EC2 Instance, IAM Role, Security Group, Route Table, Internet Gateway cho tới VPC, trả về thông báo: **`LabT7CdkStack: destroyed`**.

> **Screenshot:** ![Lệnh cdk destroy --force tháo dỡ sạch sẽ toàn bộ 19 tài nguyên chỉ trong vài phút](/images/1-Worklog/week-07/23-cdk-destroy-full-stack-cleanup.png)

---

#### Khó khăn gặp phải & Cách giải quyết

| Vấn đề | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| Lỗi phụ thuộc `Network vpc-... is not attached to an internet gateway` trong CloudFormation | Resource `AWS::EC2::Route` được tạo trước khi `VPCGatewayAttachment` hoàn tất gắn kết Internet Gateway vào VPC | Thêm thuộc tính `DependsOn: AttachGateway` vào định nghĩa Route để ép CloudFormation đợi đính kèm Gateway xong mới tạo định tuyến |
| Lệnh `cdk deploy` bị treo ở bước hỏi xác nhận quyền bảo mật `(y/n)` | Mặc định CDK yêu cầu xác nhận tương tác khi có sự thay đổi mở rộng Security Group hoặc tạo mới IAM Role | Bổ sung cờ cấu hình `--require-approval never` khi thực thi lệnh `cdk deploy` trong các quy trình tự động |
| VS Code báo lỗi `Unable to authenticate connection / InvalidClientTokenId` | Chuỗi Access Key hoặc Secret Key của IAM User bị gõ sai hoặc profile AWS chưa được kích hoạt đúng vùng | Tạo lại cặp Access Key mới cho user `LabT5-Vscode-toolkit`, cấu hình lại thông qua bảng lệnh `AWS: Connect to AWS` và chọn đúng Region `us-east-1` |
| Máy chủ EC2 tạo bởi CDK không nhận được Public IP để truy cập web | Cấu hình subnet mặc định trong CDK có thể trỏ vào Private Subnet hoặc không bật cờ tự động cấp phát Public IPv4 | Chỉ định rõ ràng tham số `vpc_subnets=ec2.SubnetSelection(subnet_type=ec2.SubnetType.PUBLIC)` trong khai báo khởi tạo `ec2.Instance` |
| Quá trình xóa tài nguyên thủ công trên Console bị lỗi `DependencyViolation` | Các thành phần mạng có sự phụ thuộc lẫn nhau phức tạp (Subnet gắn vào Route Table, Security Group tham chiếu chéo) | Luôn sử dụng lệnh `cdk destroy --force` hoặc xóa CloudFormation Stack để AWS tự động phân tích đồ thị phụ thuộc và xóa tuần tự theo thứ tự chuẩn |

### Kế hoạch tuần 8: Tự động hóa & CI/CD trên AWS

- Tìm hiểu giải pháp triển khai mã nguồn liên tục với bộ dịch vụ **AWS CodePipeline**, **AWS CodeBuild** và **AWS CodeDeploy**.
- Thiết lập quy trình tự động hóa kiểm thử mã nguồn và build Docker container image.
- Quản lý kho lưu trữ hình ảnh container an toàn với **Amazon Elastic Container Registry (ECR)**.
- Triển khai ứng dụng container hóa không cần quản lý máy chủ với **Amazon ECS on AWS Fargate**.
- Cấu hình thông báo tự động trạng thái build và pipeline thông qua **Amazon SNS** gửi về Email/Discord.
