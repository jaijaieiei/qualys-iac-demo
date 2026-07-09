# CWP Demo: Qualys Cloud Agent on AWS EC2

เป้าหมาย: ทำให้ Qualys เห็น workload จริง และใช้เล่า CWP/CWPP ได้

```text
AWS EC2 micro instance
  -> Install Qualys Cloud Agent
  -> Agent sends inventory/vulnerability telemetry to Qualys
  -> TotalCloud/CWP shows workload risk and TruRisk context
```

## 1. สร้าง EC2 แบบประหยัดเครดิต

AWS Console:

```text
Region: us-east-1 หรือ ap-southeast-1
CloudFormation > Create stack > Upload a template file
Template: tc-demo-cwp-free-tier-agent.yaml
Stack name: qualys-cwp-demo
DemoName: qualys-cwp-demo
VPC/Subnet: เลือก default VPC/subnet
InstanceType: t3.micro หรือ t2.micro
```

หน้า Configure stack options:

```text
Tags: blank
IAM role: blank
Rollback: default
Capabilities: tick acknowledge IAM resources
```

รอ `CREATE_COMPLETE`

## 2. หา Activation ID / Customer ID ใน Qualys

Qualys:

```text
App launcher > Cloud Agent
Agent Management > Activation Keys
New Key หรือเลือก key เดิม
Install Agent > Linux
```

จด 3 ค่า:

```text
ActivationId
CustomerId
ServerUri
```

อย่าส่งค่าเหล่านี้ใน chat ถ้าเป็น credential จริง

อ้างอิง Qualys ระบุว่า Cloud Agent ต้องใช้ `ActivationId` และ `CustomerId` ตอน provision agent

## 3. เข้า EC2 ผ่าน Session Manager

AWS:

```text
Systems Manager > Session Manager > Start session
เลือก instance: qualys-cwp-demo-ec2-agent-host
Start session
```

เปลี่ยนเป็น root:

```bash
sudo su -
```

## 4. ติดตั้ง Qualys Cloud Agent

ใน Qualys หน้า Install Agent จะมี command download/install ที่ถูกต้องสำหรับ tenant ของคุณ ให้ใช้ command จาก UI เป็นหลัก

รูปแบบทั่วไป:

```bash
rpm -ivh qualys-cloud-agent.x86_64.rpm
/usr/local/qualys/cloud-agent/bin/qualys-cloud-agent.sh ActivationId=<ACTIVATION_ID> CustomerId=<CUSTOMER_ID> ServerUri=<SERVER_URI>
```

ตรวจ service:

```bash
systemctl status qualys-cloud-agent
```

## 5. ดูผลใน Qualys

รอ 10-30 นาที แล้วดู:

```text
Cloud Agent > Agents
```

ค้นหา:

```text
qualys-cwp-demo
```

จากนั้นดูใน TotalCloud:

```text
TotalCloud > Inventory > AWS > Instance
TotalCloud > Posture / Workload / Vulnerabilities
```

## บทพูด CWP

> CWP หรือ Cloud Workload Protection ใช้ดูความเสี่ยงใน workload จริง เช่น EC2/VM ว่ามี vulnerability หรือ software risk อะไรบ้าง ต่างจาก CSPM ที่ดู configuration ของ cloud resource และต่างจาก CIEM ที่ดู identity/permission risk

## Cleanup

หลัง demo:

```text
CloudFormation > qualys-cwp-demo > Delete
Cloud Agent > Agents > deactivate/remove agent record ถ้าต้องการ
```
