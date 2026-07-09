# TotalCloud Demo Checklist

## ก่อน Demo

- [ ] ใช้ AWS sandbox account เท่านั้น
- [ ] ตั้ง AWS Budget alert แล้ว
- [ ] เลือก region เดียว
- [ ] Deploy CloudFormation stack สำเร็จ
- [ ] Qualys AWS Connector test connection ผ่าน
- [ ] Inventory sync แล้ว
- [ ] หา resource ด้วยชื่อ `qualys-tc-demo` หรือ tag `Purpose=QualysTotalCloudDemo` ได้

## Finding ที่ต้องเตรียมเปิด

- [ ] S3 bucket public
- [ ] S3 public access block disabled
- [ ] Security Group เปิด SSH จาก `0.0.0.0/0`
- [ ] Security Group เปิด RDP จาก `0.0.0.0/0`
- [ ] IAM Role มี `AdministratorAccess`
- [ ] เปิด remediation guidance ได้
- [ ] เปิด dashboard/report ได้

## คำถามลูกค้าที่ควรซ้อมตอบ

Q: Connector ได้สิทธิ์อะไรใน AWS?  
A: ใช้ IAM Role แบบ cross-account และ external ID ให้ Qualys assume role เข้ามาอ่าน metadata/configuration ตาม permission ที่กำหนด ไม่ต้องใช้ access key ถาวร

Q: ต้องเริ่มแบบ read-only ได้ไหม?  
A: ได้ และควรเริ่มแบบ read-only สำหรับ visibility/CSPM ก่อน ถ้าจะทำ remediation automation ค่อยเพิ่ม permission ตาม governance

Q: Finding เยอะแล้วจะรู้ได้ไงว่าอะไรต้องแก้ก่อน?  
A: ใช้ TruRisk รวม context เช่น exposure, vulnerability, permission, threat signal และ asset criticality เพื่อ prioritize

Q: Public S3 bucket เสี่ยงยังไง?  
A: ถ้ามีข้อมูลสำคัญอยู่ อาจถูกเข้าถึงจาก internet ได้ ควรเปิด Block Public Access และจำกัด bucket policy

Q: Security Group เปิด 22/3389 จาก internet เสี่ยงยังไง?  
A: เป็น management port ที่ internet ทั้งโลกพยายามเชื่อมต่อได้ เพิ่มความเสี่ยง brute force หรือ unauthorized access ควรจำกัด source IP ผ่าน VPN/bastion/admin network

Q: TotalCloud ต่างจาก VMDR ยังไง?  
A: VMDR เน้น vulnerability management ของ asset/workload ส่วน TotalCloud เป็น CNAPP ที่รวม cloud posture, workload, identity, container, SaaS, detection และ TruRisk context

