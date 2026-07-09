# Qualys TotalCloud Demo Lab - AWS Free Tier/Sandbox

เป้าหมายของ lab นี้คือสร้าง AWS environment เล็ก ๆ ที่มี misconfiguration แบบควบคุมได้ เพื่อใช้สาธิต Qualys TotalCloud โดยเริ่มจาก `CSPM` และ `CIEM`

> ใช้กับ AWS sandbox account เท่านั้น ห้ามใช้ production account และห้ามใส่ข้อมูลจริงลงใน resource ที่ตั้งใจทำให้อ่อน

## ภาพรวม Demo Story

Flow ที่ควรเล่า:

```text
AWS Account -> Qualys Cloud Connector -> Inventory -> CSPM/CIEM Findings -> TruRisk Priority -> Remediation -> Report
```

สิ่งที่เราจะสร้าง:

- S3 bucket ที่ตั้งค่า public read ผ่าน bucket policy
- Security Group ที่เปิด `22/SSH` และ `3389/RDP` จาก `0.0.0.0/0`
- IAM Role ที่มี `AdministratorAccess`
- S3 bucket ที่ไม่ได้ตั้งค่า security best practices เพิ่ม เช่น versioning/logging/default encryption

สิ่งที่ไม่สร้างใน lab แรก:

- ไม่สร้าง EC2 vulnerable จริง
- ไม่สร้าง access key
- ไม่ใส่ข้อมูลสำคัญ
- ไม่ทำ exploit หรือ brute force ใด ๆ

## ขั้นที่ 0 - กันค่าใช้จ่ายก่อน

ก่อนสร้าง resource ให้ทำก่อน:

1. เข้า AWS Console
2. ไปที่ `Billing and Cost Management`
3. เปิด `AWS Budgets`
4. สร้าง budget แบบ monthly cost เช่น `1 USD` หรือ `5 USD`
5. ใส่อีเมลแจ้งเตือนที่ 50%, 80%, 100%

อ้างอิง AWS ระบุว่า AWS Budgets ใช้ตั้ง cost/usage budgets และแจ้งเตือนเมื่อเกิน threshold ได้ และ AWS มี tutorial สำหรับ track Free Tier usage กับ budgets

## ขั้นที่ 1 - เลือก Region

แนะนำใช้ region เดียวเพื่อให้หา resource ง่าย:

- `ap-southeast-1` Singapore
- หรือ region ที่คุณใช้ประจำ

อย่ากระจายหลาย region ในรอบแรก เพราะตอน demo จะตาม finding ยาก

## ขั้นที่ 2 - สร้าง Demo Stack ด้วย CloudFormation

1. เข้า AWS Console
2. ไปที่ `CloudFormation`
3. กด `Create stack`
4. เลือก `With new resources`
5. เลือก `Upload a template file`
6. Upload ไฟล์ `tc-demo-lab-cspm-ciem.yaml`
7. Stack name: `qualys-tc-demo`
8. Parameter:
   - `DemoName`: `qualys-tc-demo`
   - `VpcId`: เลือก default VPC
9. ติ๊กยอมรับ IAM capability ถ้าหน้า CloudFormation ถาม เพราะ template สร้าง IAM Role
10. กด Create stack

รอจน Stack status เป็น `CREATE_COMPLETE`

## ขั้นที่ 3 - เช็คว่า Resource ถูกสร้าง

ไปตรวจ:

- `S3` มี bucket ชื่อประมาณ `qualys-tc-demo-<account-id>-<region>`
- `EC2 > Security Groups` มี `qualys-tc-demo-open-management-sg`
- `IAM > Roles` มี `qualys-tc-demo-overprivileged-role`

อย่า upload ข้อมูลจริงลง S3 bucket นี้

## ขั้นที่ 4 - เชื่อม Qualys TotalCloud กับ AWS

ใน Qualys TotalCloud:

1. ไปที่ `TotalCloud` หรือ `Connectors`
2. สร้าง AWS Connector
3. เลือก account-level connector
4. Qualys จะแสดง:
   - Qualys AWS Account ID
   - External ID
   - Required IAM policy/role instruction
5. กลับไป AWS แล้วสร้าง IAM Role แบบ cross-account ตามค่าที่ Qualys ให้
6. ใส่ `Role ARN` กลับใน Qualys
7. Test connection
8. Run connector / sync inventory

หลักการคือ Qualys ใช้ cross-account role และ external ID เพื่อ assume role เข้าไปอ่าน metadata/configuration โดยไม่ต้องใช้ long-term access key

## ขั้นที่ 5 - Demo ใน Qualys TotalCloud

ลำดับการกด demo:

1. เปิด `Dashboard`
   - พูดว่า "นี่คือภาพรวม cloud risk posture"

2. เปิด `Inventory`
   - filter ด้วย tag `Purpose = QualysTotalCloudDemo`
   - หรือค้นหา resource ชื่อ `qualys-tc-demo`

3. เปิด `CSPM / Posture / Findings`
   - หา finding ประเภท:
     - S3 bucket public
     - Security group allows unrestricted SSH/RDP
     - S3 bucket missing versioning/logging/encryption

4. เปิด `CIEM / Identity Risk` ถ้ามี license/view
   - หา IAM role ที่มี `AdministratorAccess`

5. เปิด `TruRisk / Insights / Risk Priority`
   - อธิบายว่า risk signal หลายตัวถูกนำมาจัดลำดับ

6. เปิด remediation detail
   - อ่าน remediation guidance
   - อธิบายว่าจะส่ง ticket/workflow หรือแก้ config อย่างไร

## บทพูด Demo 10 นาที

Opening:

"วันนี้ผมจะใช้ AWS sandbox account สาธิตว่า Qualys TotalCloud ช่วยให้ทีม security เห็น cloud asset ตรวจ misconfiguration และจัดลำดับความเสี่ยงได้อย่างไร"

Connector:

"ก่อนตรวจ cloud ได้ เราต้องสร้าง Cloud Connector เพื่อให้ Qualys อ่าน metadata และ configuration จาก AWS account ผ่าน IAM role และ external ID โดยไม่ต้องใช้ access key ถาวร"

Inventory:

"หลัง sync แล้ว TotalCloud จะสร้าง inventory ทำให้เราเห็นว่าใน account นี้มี resource อะไร เช่น S3, Security Group และ IAM Role"

Finding 1 - S3 Public:

"ตัวอย่างแรกคือ S3 bucket ที่เปิด public read ผ่าน bucket policy ความเสี่ยงคือถ้ามีข้อมูลสำคัญอยู่ใน bucket นี้ อาจถูกเข้าถึงจาก internet ได้ วิธีแก้คือเปิด Block Public Access และจำกัด bucket policy"

Finding 2 - Open Security Group:

"ตัวอย่างต่อมาคือ Security Group ที่เปิด SSH port 22 และ RDP port 3389 จาก 0.0.0.0/0 ทำให้ internet ทั้งโลกพยายามเชื่อมต่อ management port ได้ วิธีแก้คือจำกัด source IP เฉพาะ VPN, bastion หรือ admin network"

Finding 3 - Overprivileged Role:

"ตัวอย่างนี้คือ IAM Role ที่มี AdministratorAccess ซึ่งกว้างเกินความจำเป็น ถ้า workload หรือ trust relationship ถูก abuse ผู้โจมตีอาจมีสิทธิ์สูงใน account วิธีแก้คือใช้ least privilege และแยก role ตามหน้าที่"

TruRisk:

"จุดสำคัญคือ TotalCloud ไม่ได้แค่แสดง finding แต่ช่วยจัดลำดับว่าอะไรสำคัญกว่า เช่น resource ที่ public, มี permission สูง หรือเกี่ยวข้องกับ workload สำคัญ จะควรถูก prioritize ก่อน"

Close:

"สรุป TotalCloud ช่วยตั้งแต่ discover asset, assess risk, prioritize ด้วย context และส่ง remediation guidance ให้ทีม cloud owner แก้ไขได้เป็นระบบ"

## Expected Findings

หลัง sync อาจใช้เวลาสักพัก ขึ้นกับ tenant และ connector schedule

Finding ที่คาดว่าจะเห็น:

- Public S3 bucket / public bucket policy
- S3 Block Public Access disabled
- S3 versioning disabled
- S3 server access logging disabled
- Security Group allows unrestricted SSH from `0.0.0.0/0`
- Security Group allows unrestricted RDP from `0.0.0.0/0`
- IAM role with administrative privileges

ชื่อ finding อาจไม่ตรงกัน 100% ตาม policy pack และ license ที่เปิดใช้ใน Qualys

## Cleanup

หลัง demo ให้ลบ resource:

1. ไปที่ `CloudFormation`
2. เลือก stack `qualys-tc-demo`
3. กด `Delete`
4. รอ `DELETE_COMPLETE`
5. ตรวจ S3, Security Group, IAM Role ว่าหายแล้ว
6. ตรวจ Billing dashboard วันถัดไป

ถ้า S3 bucket ลบไม่ออก ให้เข้า bucket แล้วลบ object ทั้งหมดก่อน จากนั้น delete stack ใหม่

## Roadmap ถัดไป

หลังเข้าใจ CSPM/CIEM แล้วค่อยเพิ่ม:

1. `CWP/VMDR/FlexScan` - สร้าง EC2 test instance เพื่อตรวจ CVE
2. `KCS` - ใช้ container image vulnerable ใน registry/test cluster
3. `CDR` - ใช้ threat simulation ที่ vendor แนะนำเท่านั้น ไม่ทำ malware จริง
4. `SSPM` - ต่อ Microsoft 365/Google Workspace sandbox ถ้ามี tenant

สำหรับรอบแรกให้เอา `CSPM + CIEM + TruRisk + Remediation` ให้แน่นก่อน

