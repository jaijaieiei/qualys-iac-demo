# แผนด่วนสำหรับเดโม Qualys TotalCloud พรุ่งนี้

สถานการณ์: AWS account เป็น `Paid plan` และยังไม่มี credit แต่ต้องใช้ lab พรุ่งนี้

แนวทาง: ใช้ account นี้ได้ แต่ทำเฉพาะ resource ที่ต้นทุนต่ำมาก/แทบไม่มีค่าใช้จ่าย และตั้ง budget alert ก่อน

## หลักความปลอดภัยด้านค่าใช้จ่าย

ทำได้:

- IAM Role
- Security Group
- S3 bucket ว่าง
- CloudFormation stack
- Qualys connector แบบ read-only

ห้ามทำก่อนเดโม:

- EC2 instance
- RDS
- NAT Gateway
- Load Balancer
- EKS / ECS cluster
- GuardDuty / Security Hub ถ้าไม่มั่นใจเรื่อง trial/price
- Upload ไฟล์ใหญ่ลง S3

## Step 1 - ตั้ง Budget ก่อน

ใน AWS Console:

```text
Billing and Cost Management -> Budgets -> Create budget
```

ตั้งค่า:

```text
Budget name: qualys-lab-budget
Budget amount: 1 USD
Alert: 50%, 80%, 100%
Email: อีเมลของคุณ
```

หมายเหตุ: Budget เป็น alert ไม่ใช่ hard stop แต่ช่วยให้รู้เร็วถ้ามีค่าใช้จ่าย

## Step 2 - ใช้ Template แบบ Safer

ใช้ไฟล์:

```text
tc-demo-lab-safer-zero-cost.yaml
```

ไฟล์นี้สร้าง:

- S3 bucket ว่างที่ปิด Block Public Access ระดับ bucket เพื่อให้มี posture weakness แต่ไม่ใส่ public bucket policy
- Security Group เปิด `22` และ `3389` จาก `0.0.0.0/0` แต่ไม่มี EC2 ผูกอยู่
- IAM Role ที่มี `AdministratorAccess` สำหรับ CIEM finding

## Step 3 - Deploy CloudFormation

1. ไปที่ `CloudFormation`
2. กด `Create stack`
3. เลือก `Upload a template file`
4. Upload `tc-demo-lab-safer-zero-cost.yaml`
5. Stack name:

```text
qualys-tc-demo
```

6. Parameter:

```text
DemoName = qualys-tc-demo
VpcId = default VPC
```

7. ติ๊กยอมรับ IAM capability ถ้ามีถาม
8. กด Create stack
9. รอ `CREATE_COMPLETE`

## Step 4 - ต่อ Qualys TotalCloud

ใน Qualys:

```text
TotalCloud -> Connectors -> AWS Connector
```

ใช้ flow ที่ Qualys ให้:

- AWS Account ID
- External ID
- IAM role / CloudFormation onboarding template ของ Qualys
- Test connection
- Run sync / wait inventory

เริ่มแบบ read-only ก่อน

## Step 5 - สิ่งที่จะโชว์

Demo flow:

```text
Connector -> Inventory -> Findings -> TruRisk/Prioritization -> Remediation Guidance -> Report
```

Finding ที่คาดว่าจะใช้พูด:

- S3 bucket public access block disabled
- Security Group เปิด SSH/RDP จาก internet
- IAM Role มี AdministratorAccess

ชื่อ finding ใน Qualys อาจต่างกันตาม policy pack/license

## Step 6 - บทพูดสั้น

"วันนี้ผมใช้ AWS sandbox account เพื่อสาธิต TotalCloud แบบปลอดภัย เราไม่ได้สร้าง server จริงหรือใส่ข้อมูลสำคัญ แต่ตั้งค่า cloud resource บางตัวให้อ่อน เพื่อให้เห็นว่า TotalCloud ตรวจ visibility, posture และ identity risk ได้อย่างไร"

"เริ่มจาก Cloud Connector เพื่อให้ Qualys อ่าน metadata จาก AWS จากนั้น TotalCloud จะสร้าง inventory และตรวจ configuration เช่น S3 public access setting, security group ที่เปิด management port และ IAM role ที่มีสิทธิ์สูง"

"สิ่งสำคัญคือ TotalCloud ไม่ได้แค่แสดง finding แต่ช่วยจัดลำดับด้วย risk context และให้ remediation guidance เพื่อให้ทีม cloud owner แก้ไขได้เป็นระบบ"

## Step 7 - Cleanup หลังเดโม

หลังจบ demo:

```text
CloudFormation -> qualys-tc-demo -> Delete
```

ตรวจซ้ำ:

- S3 bucket หายแล้ว
- Security Group หายแล้ว
- IAM Role หายแล้ว
- Billing dashboard ไม่มีค่าใช้จ่ายผิดปกติ

