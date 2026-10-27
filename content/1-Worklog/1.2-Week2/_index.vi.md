---
title: "Worklog Tuần 2"
date: 2026-09-21
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

## Mục tiêu tuần 2

- Nắm vững kiến trúc mạng VPC: Custom VPC, Public Subnet, Private Subnet, Internet Gateway và Route Table.
- Triển khai máy chủ trong Public Subnet và Private Subnet, kiểm chứng tính cô lập bảo mật của Private Subnet.
- Cấu hình định tuyến nâng cao với NAT Gateway và Elastic IP, cho phép Private Subnet kết nối Internet một chiều.
- Tìm hiểu và thực hành dịch vụ DNS Amazon Route 53: Tạo Public Hosted Zone, tạo A record và kiểm chứng phân giải tên miền với `nslookup`.
- Thiết lập phân phối nội dung toàn cầu qua Amazon CloudFront CDN kết nối S3 Bucket Origin.
- Tối ưu bảo mật mạng Edge với Lambda@Edge: Triển khai hàm Lambda Node.js, cấu hình IAM Role, trigger Viewer Response để chèn HTTP Security Headers.
- Thực hành nâng cao về Security Group: Thiết lập Security Group nhiều tầng, kiểm thử kết nối nội bộ (inter-subnet ICMP/SSH) qua Bastion host / Web Server.

### Các công việc cần triển khai trong tuần này

| Thứ | Chủ đề | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- | --- |
| 2 | VPC Cơ bản | Tạo Custom VPC (`My-Lab-VPC`), Public & Private Subnet, triển khai Web Server trên Public Subnet và EC2 trên Private Subnet | 21/09/2026 | 21/09/2026 | [Amazon VPC](https://docs.aws.amazon.com/vpc/) |
| 3 | Route Table & NAT Gateway | Cấu hình Route Table (`Public-RT`, `Private-RT`), tạo NAT Gateway gắn Elastic IP cho phép Private Subnet ra ngoài Internet | 22/09/2026 | 22/09/2026 | [AWS NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| 4 | Amazon Route 53 | Tạo Public Hosted Zone `labt4-thuchanh.com`, tạo A record trỏ về IP EC2, kiểm chứng phân giải DNS với `nslookup` | 23/09/2026 | 23/09/2026 | [Amazon Route 53](https://docs.aws.amazon.com/route53/) |
| 5 | Amazon CloudFront | Tạo CloudFront Distribution kết nối Origin S3 `bucketlabt5`, phân phối website tĩnh tốc độ cao qua Edge Locations | 24/09/2026 | 24/09/2026 | [Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/) |
| 6 | Lambda@Edge & Security | Viết hàm Lambda@Edge chèn Security Headers (`X-Frame-Options`, `X-Fpt-Sec-Lab`), kích hoạt qua CloudFront Viewer Response | 25/09/2026 | 25/09/2026 | [Lambda@Edge](https://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html) |
| 7 | Lab Tổng hợp Mạng & Bảo mật | Thiết lập Security Group phân tầng (`Web-SG`, `Private-SG`), kiểm thử kết nối nội bộ (Ping, SSH) giữa Public và Private Subnet | 26/09/2026 | 26/09/2026 | [AWS Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) |

### Kết quả đạt được tuần 2

- Xây dựng thành công kiến trúc VPC đa tầng gồm Public Subnet và Private Subnet, kiểm chứng phân tách mạng độc lập.
- Thiết lập hoàn chỉnh định tuyến với Internet Gateway và NAT Gateway, kiểm soát luồng traffic ra/vào mạng nội bộ an toàn.
- Cấu hình DNS Server với Amazon Route 53, gán bản ghi tên miền vào máy chủ EC2 thực tế và xác minh truy vấn DNS thành công.
- Tối ưu tốc độ tải trang web thông qua mạng phân phối nội dung toàn cầu Amazon CloudFront với S3 Origin.
- Nâng cao tính bảo mật bằng điện toán biên Lambda@Edge, tự động tiêm các HTTP Security Headers vào response trả về cho client.
- Thiết lập quy tắc tường lửa phân tầng Security Group, cho phép kiểm soát truy cập chặt chẽ giữa các thành phần Public Web và Private Backend.

---

#### Bài tập 1: Kiến trúc VPC cơ bản — Public & Private Subnet

Khởi tạo một Virtual Private Cloud (VPC) tùy chỉnh mang tên **`My-Lab-VPC`**. Trong VPC này, chia tách 2 Subnet độc lập:
- `Public-subnet`: Kết nối với Internet Gateway, dành cho các tài nguyên phục vụ công cộng.
- `Private-subnet`: Hoàn toàn cách ly với Internet bên ngoài, dành cho các tài nguyên bảo mật nội bộ.

Khởi chạy máy chủ Web Server trên `Public-subnet` và gán Public IPv4 `13.223.85.49`. Truy cập qua trình duyệt web hiển thị trang chào mừng: *"Chao mung den voi Web Server tren VPC rieng! - Manhdung2507"*.

> **Screenshot:** ![Web Server trên Public Subnet của Custom VPC](/images/1-Worklog/week-02/01-vpc-web-server-public.png)

Khởi chạy máy chủ ảo thứ hai **`Test-Private-EC2`** (`i-0a2a4b868f760e445`) bên trong `Private-subnet`. Instance này chỉ sở hữu địa chỉ Private IPv4 (`10.0.2.91`), hoàn toàn không có Public IPv4.

> **Screenshot:** ![Chi tiết EC2 trong Private Subnet](/images/1-Worklog/week-02/02-private-ec2-instance.png)

Tiến hành kiểm chứng bảo mật: Cố gắng truy cập trực tiếp từ trình duyệt Internet tới địa chỉ Private IP `10.0.2.91`. Trình duyệt báo lỗi **`ERR_CONNECTION_TIMED_OUT`** ("This site can't be reached"). Điều này chứng minh rằng máy chủ trong Private Subnet được bảo vệ an toàn và không thể bị dò quét hay tấn công trực tiếp từ Internet.

> **Screenshot:** ![Kiểm chứng không thể truy cập Private Subnet từ Internet](/images/1-Worklog/week-02/03-private-ec2-connection-timeout.png)

---

#### Bài tập 2: Định tuyến mạng với Route Table & NAT Gateway

Khởi tạo và quản lý bảng định tuyến cho hệ thống mạng. Tạo **`Private-RT`** (`rtb-05b1c5a9be77d018d`) dành riêng cho các tài nguyên nội bộ thuộc `LabT3-VPC`.

> **Screenshot:** ![Bảng định tuyến Private Route Table](/images/1-Worklog/week-02/04-private-route-table.png)

Phân chia dải địa chỉ mạng con (Subnets) trong `LabT3-VPC`:
- `Public-Subnet`: `10.0.1.0/24` (Availability Zone `us-east-1a`)
- `Private-Subnet`: `10.0.2.0/24` (Availability Zone `us-east-1a`)

> **Screenshot:** ![Danh sách Subnet Public và Private trong VPC](/images/1-Worklog/week-02/05-vpc-subnets-list.png)

Thực hiện liên kết mạng con (Subnet Association): Gắn bảng định tuyến **`Public-RT`** (`rtb-0d32fcfc31405b51b`) vào `Public-Subnet` (`subnet-0d86b9a896d860d5b`), bảng này có tuyến đường trỏ thẳng tới Internet Gateway (`igw`).

> **Screenshot:** ![Liên kết Subnet Association cho Public Route Table](/images/1-Worklog/week-02/06-public-subnet-association.png)

Tạo **NAT Gateway** mang tên **`LabT3-NAT`** (`nat-160fbcccddaa27944`). NAT Gateway bắt buộc phải đặt trong `Public-Subnet` và được cấp phát một địa chỉ Elastic IP cố định (`18.209.181.106`) để làm đại diện khi giao tiếp với Internet. Trạng thái sau khi khởi tạo đạt **Available**.

> **Screenshot:** ![NAT Gateway ở trạng thái Available với Elastic IP](/images/1-Worklog/week-02/07-nat-gateway-available.png)

Cấu hình bảng định tuyến `Private-RT`: Thêm tuyến đường (Route) với đích đến `0.0.0.0/0` trỏ tới Target là NAT Gateway `nat-160fbcccddaa27944`. Cấu hình này giúp các EC2 bên trong Private Subnet có thể tải bản vá, cài đặt gói thư viện từ Internet ra ngoài mà chiều ngược lại từ Internet vẫn hoàn toàn bị chặn.

```bash
# Minh họa kiểm tra kết nối từ Private EC2 sau khi cấu hình NAT Gateway
curl -I https://aws.amazon.com
# HTTP/2 200
# content-type: text/html; charset=utf-8
```

> **Screenshot:** ![Cấu hình tuyến đường qua NAT Gateway trong Private-RT](/images/1-Worklog/week-02/08-private-rt-nat-route.png)

---

#### Bài tập 3: Amazon Route 53 — Quản lý DNS & Định tuyến tên miền

Triển khai máy chủ web kiểm thử **`LabT4-Test`** (`i-0a86d42df283fe32e`) với địa chỉ Public IPv4: `98.86.220.109`.

> **Screenshot:** ![EC2 Web Server dùng làm đích cho Route 53](/images/1-Worklog/week-02/09-ec2-route53-instance.png)

Khởi tạo **Public Hosted Zone** trên dịch vụ **Amazon Route 53** cho tên miền `labt4-thuchanh.com` (Hosted Zone ID: `Z012720835D8Y5T663A3Y`). Hệ thống tự động cấp phát 4 Name Server (NS) của AWS và 1 bản ghi SOA.

> **Screenshot:** ![Khởi tạo Public Hosted Zone trên Route 53](/images/1-Worklog/week-02/10-route53-hosted-zone.png)

Tạo bản ghi định tuyến **A Record**:
- **Record name:** `www.labt4-thuchanh.com`
- **Record type:** `A - Routes traffic to an IPv4 address and some AWS resources`
- **Value:** `98.86.220.109`
- **TTL (Seconds):** `300`
- **Routing policy:** Simple

> **Screenshot:** ![Tạo bản ghi A Record trỏ về IP máy chủ](/images/1-Worklog/week-02/11-route53-record-created.png)

Kiểm chứng phân giải tên miền: Mở Command Prompt trên máy local, thực hiện truy vấn DNS trực tiếp tới Authoritative Name Server của Route 53 (`ns-388.awsdns-48.com`):

```cmd
nslookup www.labt4-thuchanh.com ns-388.awsdns-48.com
```

Kết quả trả về địa chỉ IP chính xác: `98.86.220.109`, xác minh cấu hình Route 53 đã hoạt động hoàn hảo.

> **Screenshot:** ![Kết quả nslookup xác minh phân giải DNS thành công](/images/1-Worklog/week-02/12-nslookup-dns-verification.png)

---

#### Bài tập 4: Amazon CloudFront — Mạng phân phối nội dung (CDN)

Tạo **Amazon CloudFront Distribution** mang tên **`Labt5`** (Distribution ID: `E3G4O9N747F2M4`). CloudFront tự động cấp phát domain name CDN: `d2vfkjpwxqkl8v.cloudfront.net` với trạng thái **Enabled**.

> **Screenshot:** ![Tạo CloudFront Distribution thành công](/images/1-Worklog/week-02/13-cloudfront-distribution.png)

Tạo S3 Bucket **`bucketlabt5`** (`arn:aws:s3:::bucketlabt5`) tại Region `us-east-1`, thiết lập làm Origin cho CloudFront distribution để lưu trữ các tài nguyên tĩnh của website.

> **Screenshot:** ![S3 Bucket làm Origin cho CloudFront](/images/1-Worklog/week-02/14-s3-bucket-cloudfront.png)

Soạn thảo và tải file `index.html` lên S3 Bucket `bucketlabt5`. File HTML chứa nội dung thông báo kiểm thử CDN của sinh viên Mạnh Dũng.

> **Screenshot:** ![Upload file index.html lên S3 Origin](/images/1-Worklog/week-02/15-s3-objects-index-html.png)

Truy cập website tĩnh thông qua đường dẫn CDN toàn cầu `https://d2vfkjpwxqkl8v.cloudfront.net/index.html`. Trang web phản hồi tức thì với giao diện: *"Triển khai CDN Thành công! Trang web đang được phân phối qua mạng lưới Edge Locations toàn cầu của AWS CloudFront."*

> **Screenshot:** ![Website hoạt động mượt mà qua CloudFront CDN](/images/1-Worklog/week-02/16-cloudfront-website-live.png)

---

#### Bài tập 5: Lambda@Edge — Xử lý Edge Computing & HTTP Security Headers

Khởi tạo Lambda Function mang tên **`Edge-Security-Header`** với runtime `Node.js 24.x`, kiến trúc `x86_64` tại Region `us-east-1` (Vùng bắt buộc đối với dịch vụ Lambda@Edge).

> **Screenshot:** ![Khởi tạo Lambda Function cho Lambda@Edge](/images/1-Worklog/week-02/17-lambda-create-function.png)

Cấu hình quyền hạn (Execution Role): Chọn role có sẵn **`lambda-edge-role`**. Role này được thiết lập Trust Relationship cho phép cả hai dịch vụ `lambda.amazonaws.com` và `edgelambda.amazonaws.com` assume role.

> **Screenshot:** ![Chọn IAM Execution Role cho Lambda@Edge](/images/1-Worklog/week-02/18-lambda-role-selection.png)

Soạn thảo và gắn Policy Statement trên IAM Policy Editor, đảm bảo quyền ghi log vào Amazon CloudWatch Logs khi hàm chạy tại các Edge Locations trên khắp thế giới.

> **Screenshot:** ![Thiết lập IAM Policy Statement Editor](/images/1-Worklog/week-02/19-lambda-policy-statement.png)

Triển khai mã nguồn Node.js chèn các Security Headers và tạo CloudFront Trigger:
- **Distribution:** `E3G4O9N747F2M4` (CloudFront)
- **CloudFront event:** `Viewer response`
- Xác nhận triển khai hàm sang phiên bản mới (Deploy to Lambda@Edge).

```javascript
export const handler = async (event) => {
    const response = event.Records[0].cf.response;
    const headers = response.headers;

    // Chèn HTTP Security Headers
    headers['strict-transport-security'] = [{ key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubdomains; preload' }];
    headers['x-content-type-options'] = [{ key: 'X-Content-Type-Options', value: 'nosniff' }];
    headers['x-frame-options'] = [{ key: 'X-Frame-Options', value: 'DENY' }];
    headers['x-xss-protection'] = [{ key: 'X-XSS-Protection', value: '1; mode=block' }];
    headers['x-fpt-sec-lab'] = [{ key: 'X-Fpt-Sec-Lab', value: 'Hoan-thanh-Lab-T6' }];

    return response;
};
```

> **Screenshot:** ![Cấu hình CloudFront Trigger Viewer Response](/images/1-Worklog/week-02/20-lambda-edge-trigger.png)

Mở DevTools (F12) trên trình duyệt, kiểm tra tab Network khi gửi request tới `https://d2vfkjpwxqkl8v.cloudfront.net/index.html`. Trong phần Response Headers, các header bảo mật do Lambda@Edge tiêm vào hiển thị đầy đủ: `X-Fpt-Sec-Lab: Hoan-thanh-Lab-T6` và `X-Frame-Options: DENY`.

> **Screenshot:** ![Xác minh Response Headers bảo mật được tiêm thành công](/images/1-Worklog/week-02/21-lambda-edge-headers-verified.png)

---

#### Bài tập 6: Lab Tổng hợp — Phân tầng Security Groups & Kiểm thử kết nối liên Subnet

Thiết lập Security Group cho Web Server công khai mang tên **`Web-SG`** trong VPC `Mylabt7`. Cấu hình Inbound Rules mở các cổng cần thiết cho client bên ngoài:
- **HTTP (TCP 80)**: Source `0.0.0.0/0`
- **HTTPS (TCP 443)**: Source `0.0.0.0/0`
- **SSH (TCP 22)**: Source `0.0.0.0/0` (dùng để quản trị)

> **Screenshot:** ![Tạo Security Group Web-SG cho Public Web Server](/images/1-Worklog/week-02/22-security-group-web-sg.png)

Thiết lập Security Group cho máy chủ nội bộ **`Private-SG`** (`sg-00f606d0da92bd89c`). Để bảo mật tuyệt đối, Inbound Rules chỉ cho phép:
- **All ICMP - IPv4**: Cho phép ping kiểm tra trạng thái mạng nội bộ.
- **SSH (TCP 22)**: Cho phép kết nối quản trị từ máy chủ Web (Bastion Host) trong Public Subnet.

> **Screenshot:** ![Cấu hình Inbound Rules cho Private-SG](/images/1-Worklog/week-02/24-private-security-group-rules.png)

Kiểm thử kết nối liên Subnet (Inter-subnet Connectivity): Kết nối vào máy chủ **`LabT7-Web-Server`** (`10.0.1.123`) thông qua EC2 Instance Connect, sau đó thực hiện lệnh `ping` tới địa chỉ IP máy chủ nội bộ `10.0.2.214`. Các gói tin phản hồi ổn định với độ trễ cực thấp (~0.3ms), chứng minh hệ thống định tuyến nội bộ của VPC hoạt động thông suốt và các Security Group được cấu hình chính xác.

```bash
# Ping từ Web Server (Public) sang Private Instance
ping 10.0.2.214
# PING 10.0.2.214 (10.0.2.214) 56(84) bytes of data.
# 64 bytes from 10.0.2.214: icmp_seq=1 ttl=127 time=1.31 ms
# 64 bytes from 10.0.2.214: icmp_seq=2 ttl=127 time=0.319 ms
# ...
```

> **Screenshot:** ![Thực hiện ping thành công giữa Public Subnet và Private Subnet](/images/1-Worklog/week-02/23-inter-subnet-ping-test.png)

---

#### Khó khăn gặp phải & Cách giải quyết

| Vấn đề | Nguyên nhân | Cách giải quyết |
| :--- | :--- | :--- |
| Không thể truy cập máy chủ Private EC2 từ trình duyệt | Private Subnet không có Internet Gateway và không cấp Public IP | Đây là thiết kế bảo mật chuẩn; để quản trị hoặc tải gói tin, sử dụng Bastion Host hoặc cấu hình NAT Gateway |
| Route Table Private không thể kết nối ra ngoài Internet | Chưa cấu hình default route `0.0.0.0/0` trỏ tới NAT Gateway | Vào `Private-RT`, thêm route `0.0.0.0/0` với Target là ID của NAT Gateway `nat-160fbcccddaa27944` |
| Route 53 A record chưa phân giải được ngay trên máy local | Máy local vẫn đang cache DNS từ ISP cũ | Sử dụng `nslookup` chỉ định trực tiếp Name Server của AWS (`ns-388.awsdns-48.com`) hoặc xoá cache DNS bằng `ipconfig /flushdns` |
| Lambda@Edge không hiển thị tùy chọn trigger CloudFront | Lambda function không nằm tại Region `us-east-1` (N. Virginia) | Lambda@Edge yêu cầu khởi tạo tại region `us-east-1` và cần gán IAM Role có quyền trust `edgelambda.amazonaws.com` |
| Ping giữa 2 subnet trong cùng VPC bị timeout | Security Group của máy đích chưa mở giao thức ICMP | Mở rule `All ICMP - IPv4` trong Inbound rules của Security Group máy đích với nguồn từ Subnet nội bộ |

### Kế hoạch tuần 3

- Tìm hiểu và triển khai Elastic Load Balancing (Application Load Balancer - ALB).
- Cấu hình Auto Scaling Group (ASG) tự động co giãn tài nguyên theo tải CPU.
- Thiết lập Health Check và Target Groups phân phối lưu lượng truy cập.
- Tìm hiểu cơ chế giám sát và cảnh báo hệ thống với Amazon CloudWatch Alarms.
