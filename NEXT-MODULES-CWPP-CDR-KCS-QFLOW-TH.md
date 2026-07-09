# Qualys TotalCloud Demo Lab: CWPP, CDR, KCS, Qualys Flow

สถานะก่อนเริ่ม:

- CSPM เห็น Cloud Posture แล้ว
- CIEM เห็น IAM User / IAM Role risk แล้ว
- IaC scan ผ่าน CLI และ GitHub Connector แล้ว

ไฟล์ CloudFormation สำหรับ workload:

`tc-demo-workload-cwpp-kcs.yaml`

## ภาพรวม Lab

```text
AWS EC2 demo host
  -> CWPP: ติด Qualys Cloud Agent หรือใช้ FlexScan/API/Snapshot scan
  -> KCS: ใช้ Docker host + Qualys Container Sensor scan image/container

AWS GuardDuty
  -> CDR: generate sample findings แล้ว ingest เข้า TotalCloud Investigate

Qualys Flow
  -> สร้าง QFlow จาก template หรือ scratch เพื่อ query/remediate Security Group finding
```

## Cost Guardrails

- ใช้ `t3.small` สำหรับ demo 2-4 ชั่วโมงแล้วลบทิ้ง
- EC2, EBS, public IPv4, GuardDuty หลัง free trial อาจมีค่าใช้จ่าย
- ตั้ง AWS Budget ไว้แล้วให้เช็คก่อนเริ่ม
- หลัง demo ให้ลบ stack `qualys-tc-workload` และปิด GuardDuty ถ้าไม่ใช้ต่อ

## Module 1: CDR

เป้าหมาย: ทำให้หน้า `TotalCloud > Investigate` มี detection/threat ให้โชว์

ทางเร็วสุด:

1. AWS Console > `GuardDuty`
2. Region: `us-east-1`
3. Enable GuardDuty
4. `Settings > Sample findings > Generate sample findings`
5. `GuardDuty > Findings` ต้องเห็น finding ขึ้นต้นด้วย `[SAMPLE]`
6. Qualys > `TotalCloud > Configure > Threat Scanners > AWS`
7. เช็คว่ามี CDR/GuardDuty onboarding หรือ deployment สำหรับ account นี้
8. กลับไป `TotalCloud > Investigate > Detections`

บทพูด:

> CDR คือ detection/response layer ของ TotalCloud ใช้ดู active threats หรือ suspicious activity เช่น GuardDuty finding, suspicious communication, port scan หรือ C2 communication ต่างจาก CSPM ที่เป็น posture/misconfiguration

## Module 2: CWPP

เป้าหมาย: เห็น vulnerability ของ EC2 instance ใน TotalCloud

ทางเลือก:

1. Cloud Agent: แม่นและชัดสุด แต่ต้องใช้ `Activation ID` และ `Customer ID` จาก Qualys
2. FlexScan API/Snapshot: agentless แต่ต้อง enable VM scanning/FlexScan ที่ AWS connector และสิทธิ์ IAM เพิ่ม

ขั้นตอนสร้าง workload:

1. AWS > CloudFormation > Create stack
2. Upload `tc-demo-workload-cwpp-kcs.yaml`
3. Stack name: `qualys-tc-workload`
4. เลือก default VPC และ public subnet
5. Instance type: `t3.small`
6. Create stack

หลัง instance ขึ้น:

1. Qualys > Cloud Agent > Activation Keys
2. สร้างหรือเลือก key สำหรับ VMDR/Cloud Agent
3. Copy Linux install command
4. AWS > Systems Manager > Session Manager > Start session เข้า instance
5. รัน command ติดตั้ง Cloud Agent
6. รอ 15-60 นาที
7. Qualys > TotalCloud > Inventory > AWS > Instance
8. เปิด instance แล้วดู Vulnerabilities / TruRisk / hasAgent

บทพูด:

> CWPP ดูความเสี่ยงของ workload จริง เช่น EC2 มี package vulnerability อะไรบ้าง ต่างจาก CSPM ที่ดู configuration ของ cloud resource

## Module 3: KCS

เป้าหมาย: เห็น container image/container risk

ทางที่เหมาะกับ demo:

1. ใช้ EC2 Docker host จาก stack `qualys-tc-workload`
2. Qualys > Container Security > Configurations > Sensors
3. Download Sensor
4. เลือก sensor type: `General` หรือ `CI/CD`
5. เลือก Docker/Standalone Linux
6. Copy docker run/install command
7. รันบน EC2 ผ่าน Session Manager
8. Sensor จะ discover image/container เช่น `nginx:1.14`
9. กลับไป Qualys > Container Security / TotalCloud KCS view เพื่อดู image/container vulnerability

บทพูด:

> KCS คือ Kubernetes and Container Security ใช้ดู risk ของ container image, running container, registry, และ runtime events ต่างจาก CWPP ที่เน้น VM/workload host

## Module 4: Qualys Flow

เป้าหมาย: โชว์ automation หลังเจอ risk

ทางปลอดภัย:

1. Qualys app launcher > `Qualys Flow`
2. `QFlows > Create QFlow > Using a template`
3. หา template ที่เกี่ยวกับ Security Group เช่น `Security Group allowing outside IPs`
4. เลือก AWS account `749625535483`
5. เลือก region `us-east-1`
6. Save
7. Run แบบ manual/dry-run ถ้ามี
8. ดู execution result

บทพูด:

> Qualys Flow คือ no-code/low-code automation ใช้สร้าง playbook เช่น query resource, filter resource ที่เสี่ยง, notify, remediate หรือเอาไปเป็น custom runtime control ใน TotalCloud

## Demo Story ที่ควรเล่า

```text
1. IaC: เจอ risk ตั้งแต่ code ก่อน deploy
2. CSPM: deploy แล้ว Qualys ตรวจ posture/misconfiguration
3. CIEM: ดู identity/permission ว่า role/user มีสิทธิ์มากเกินไปไหม
4. CWPP: ดู vulnerability ของ EC2 workload
5. KCS: ดู vulnerability ของ container image/container
6. CDR: ดู active threat/detection
7. QFlow: automate response/remediation
```

## Cleanup

หลัง demo:

1. CloudFormation > delete `qualys-tc-workload`
2. ถ้าไม่ใช้ต่อ ปิด GuardDuty ใน region ที่เปิด
3. ลบ/disable Container Sensor ถ้าไม่ใช้ต่อ
4. ตรวจ Billing > Cost Explorer / Credits
