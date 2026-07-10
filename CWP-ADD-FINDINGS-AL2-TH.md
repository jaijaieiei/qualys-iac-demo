# CWP: ทำให้มี Finding ง่ายขึ้นด้วย Amazon Linux 2

เครื่อง `Amazon Linux 2023` ที่สร้างไว้ก่อนหน้าเป็น OS ใหม่และสะอาด จึงอาจไม่เจอ vulnerability ใน Qualys ทันที

ทางที่ปลอดภัยกว่าการยัด exploit ลงเครื่อง คือสร้าง EC2 อีกตัวเป็น `Amazon Linux 2` แล้วติด Qualys Cloud Agent เหมือนเดิม

เหตุผล:

- AWS ระบุว่า Amazon Linux 2 end of support คือ `June 30, 2026`
- ตอนนี้เป็นหลังวันดังกล่าวแล้ว จึงเหมาะสำหรับ demo เรื่อง OS lifecycle / EOL / missing patch risk
- ยังใช้ `t3.micro` หรือ `t2.micro` ได้
- Security Group ไม่มี inbound access จึงไม่เปิดบริการ vulnerable ออก Internet

## สร้าง Stack

AWS CloudFormation:

```text
Create stack > Upload a template file
Template: tc-demo-cwp-al2-vulnerable-agent.yaml
Stack name: qualys-cwp-al2-demo
DemoName: qualys-cwp-al2-demo
InstanceType: t3.micro
VPC/Subnet: เลือก default VPC/subnet ใน region เดียวกับที่ใช้ demo
```

ติ๊ก IAM capability แล้วกด Create stack

## ติด Cloud Agent

ใช้ไฟล์ RPM และ activation command เดิม:

```bash
cd /tmp
curl -L "PRESIGNED_URL" -o QualysCloudAgent.rpm
sudo rpm -ivh QualysCloudAgent.rpm
sudo /usr/local/qualys/cloud-agent/bin/qualys-cloud-agent.sh ActivationId=... CustomerId=... ServerUri=...
sudo systemctl status qualys-cloud-agent
```

## ดูผล

รอ 30 นาที - 2 ชั่วโมง แล้วดู:

```text
Cloud Agent > Agents
VMDR / Vulnerability Management > Assets
TotalCloud > Inventory > AWS > Instance
```

สิ่งที่คาดหวัง:

```text
OS lifecycle/EOL finding
missing patch/package finding
software inventory มากกว่า AL2023
```

ถ้ายังไม่ขึ้นทันที ให้รอรอบประมวลผลของ Cloud Agent/VMDR ก่อน
