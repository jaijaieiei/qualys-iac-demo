# ทดสอบ Qualys TotalCloud Inventory ใน AWS Thailand Region

AWS Thailand Region:

```text
Asia Pacific (Thailand)
Region code: ap-southeast-7
```

อ้างอิง AWS: Asia Pacific (Thailand) Region เปิดใช้งานทั่วไปแล้วและใช้ API name `ap-southeast-7` พร้อม 3 Availability Zones

## เป้าหมาย

สร้าง resource ต้นทุนต่ำใน `ap-southeast-7` เพื่อเช็คว่า Qualys TotalCloud connector scan/inventory เจอ resource ใน Thailand region หรือไม่

## Resource ที่สร้าง

ไฟล์:

```text
tc-demo-thailand-region-inventory.yaml
```

จะสร้าง:

- VPC ใหม่ใน Thailand region
- Security Group เปิด `22/SSH` และ `3389/RDP` จาก `0.0.0.0/0` แต่ไม่มี EC2 ผูกอยู่
- S3 bucket ว่างใน Thailand region พร้อม weak public access block setting

ไม่สร้าง:

- EC2
- NAT Gateway
- Load Balancer
- RDS
- EKS

## ขั้นตอนใน AWS

1. มุมขวาบนของ AWS Console เปลี่ยน Region เป็น:

```text
Asia Pacific (Thailand) ap-southeast-7
```

ถ้าไม่เห็น Thailand region:

- ไปที่ `Account settings` หรือ `AWS Regions`
- Enable `Asia Pacific (Thailand)`
- รอ region status เป็น enabled

2. ไปที่:

```text
CloudFormation -> Create stack -> With new resources
```

3. เลือก:

```text
Choose an existing template
Upload a template file
```

4. Upload:

```text
tc-demo-thailand-region-inventory.yaml
```

5. ตั้งค่า:

```text
Stack name: qualys-tc-thailand-demo
DemoName: qualys-tc-th
```

6. กด Next ไปจนถึง Review แล้ว Submit

7. รอ:

```text
CREATE_COMPLETE
```

## เช็คใน Qualys

หลัง AWS stack สร้างเสร็จ:

1. กลับไป Qualys
2. ไปที่:

```text
TotalCloud -> Configure -> CSPM Connectors
```

3. เปิด connector account ของคุณ
4. กด sync / refresh ถ้ามี
5. รอสักพัก

จากนั้นไปที่:

```text
Inventory / Resources / Posture
```

ค้นหา:

```text
qualys-tc-th
```

หรือ filter:

```text
Account ID = account ของคุณ
Region = ap-southeast-7
Service = S3 / VPC / EC2
```

## Expected result

ถ้า Qualys scan region ไทยเจอ ควรเห็น:

- S3 bucket:

```text
qualys-tc-th-<account-id>-ap-southeast-7
```

- VPC:

```text
qualys-tc-th-vpc
```

- Security Group:

```text
qualys-tc-th-open-management-sg
```

และอาจมี posture finding เช่น:

- Security group allows unrestricted SSH
- Security group allows unrestricted RDP
- S3 public access block disabled
- S3 versioning/logging/encryption related controls

## ถ้า Qualys ไม่เจอ

ตรวจ 4 จุด:

1. AWS Thailand region enabled แล้วหรือยัง
2. CloudFormation stack อยู่ใน `ap-southeast-7` จริงไหม
3. Qualys connector sync หลังจากสร้าง resource แล้วหรือยัง
4. Connector permission policy ครอบคลุม region ใหม่หรือไม่

ถ้ายังไม่เจอ ให้รอ sync cycle เพิ่ม หรือสร้าง connector ใหม่/แก้ connector region scope ถ้า tenant มี region selection

## Cleanup

หลัง test:

```text
CloudFormation -> qualys-tc-thailand-demo -> Delete
```

ตรวจว่า S3 bucket, VPC และ Security Group ถูกลบแล้ว

