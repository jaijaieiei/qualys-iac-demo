# OWASP Juice Shop on AWS - Restricted Access Lab

เป้าหมาย: สร้าง vulnerable web app สำหรับ demo แบบจำกัด IP ไม่เปิด public ทั้งโลก

```text
EC2 Amazon Linux 2023
  -> Docker
  -> OWASP Juice Shop container
  -> Security Group เปิด port 3000 เฉพาะ public IP ของคุณเท่านั้น
```

## ใช้ทำ demo อะไรได้

- ใช้เป็น web target สำหรับ Qualys WAS หรือ web/app security demo
- ใช้เป็น container/image target สำหรับ KCS ถ้าติด Container Sensor ต่อ
- ใช้เป็น workload context สำหรับ CWP ถ้าติด Cloud Agent บน host

## Guardrails

- Security Group ไม่มี SSH inbound
- Port 3000 เปิดเฉพาะ `AllowedHttpCidr`
- อย่าเปลี่ยนเป็น `0.0.0.0/0`
- หลัง demo ให้ delete stack

## Deploy

AWS Console:

```text
Region: us-east-1 หรือ region ที่ต้องการ
CloudFormation > Create stack > Upload a template file
Template: tc-demo-juice-shop-restricted.yaml
Stack name: qualys-juice-shop
DemoName: qualys-juice-shop
InstanceType: t3.micro
AllowedHttpCidr: public-ip-ของคุณ/32
VpcId/SubnetId: เลือก default VPC/subnet ของ region นั้น
```

หน้า Configure stack options:

```text
Tags: blank
IAM role: blank
Rollback: default
Capabilities: tick acknowledge IAM resources
```

รอ `CREATE_COMPLETE`

## เปิดเว็บ

ไปที่ tab `Outputs` แล้วเปิด:

```text
JuiceShopUrl
```

ตัวอย่าง:

```text
http://<public-ip>:3000
```

ถ้าเปิดไม่ได้:

1. เช็คว่าเปิดจาก public IP เดียวกับ `AllowedHttpCidr`
2. เช็ค EC2 instance status เป็น `running`
3. เข้า Session Manager แล้วรัน:

```bash
sudo docker ps
sudo docker logs --tail 50 juice-shop
```

## Update IP ถ้าเน็ตเปลี่ยน

ถ้า public IP คุณเปลี่ยน:

1. หา IP ใหม่จาก `https://api.ipify.org`
2. CloudFormation > stack `qualys-juice-shop`
3. Update stack > Use current template
4. เปลี่ยน `AllowedHttpCidr` เป็น `NEW_PUBLIC_IP/32`

## Cleanup

หลัง demo:

```text
CloudFormation > qualys-juice-shop > Delete
```

ถ้ามี Qualys scanner/agent/sensor ที่ผูกไว้ ให้ disable/remove ตาม lab ที่ใช้ต่อด้วย
