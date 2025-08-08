วิธีติดตั้ง Docker + Docker Compose และรัน Portainer Stack บน Proxmox LXC (Debian/Ubuntu)
คู่มือนี้จะแนะนำขั้นตอนการติดตั้ง Docker และ Docker Compose รวมถึงการรัน Portainer Stack บน Proxmox LXC Container ที่ใช้ระบบปฏิบัติการ Debian หรือ Ubuntu อย่างละเอียดและชัดเจน

1. เตรียม LXC Container

สร้าง LXC Container  

สร้าง LXC Container โดยเลือกระบบปฏิบัติการ Debian (แนะนำ Debian 12) หรือ Ubuntu (แนะนำ 22.04/24.04) ใน Proxmox  
ตรวจสอบว่า LXC มีการตั้งค่าเน็ตเวิร์กและที่เก็บข้อมูลเพียงพอ


ตั้งค่า Container Config  

แก้ไขไฟล์ /etc/pve/lxc/<CTID>.conf (แทน <CTID> ด้วย ID ของ Container)  
หากใช้ privileged container, เพิ่มบรรทัดต่อไปนี้เพื่อเปิดใช้งาน nesting และ keyctl:features: nesting=1,keyctl=1


หากใช้ unprivileged container, เพิ่มบรรทัดต่อไปนี้เพื่อเปิด cgroup และให้ Docker ทำงานได้:features: nesting=1,keyctl=1
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:




รีสตาร์ท LXC Container  

รีสตาร์ท Container เพื่อให้การตั้งค่ามีผล:pct restart <CTID>






2. ติดตั้ง Docker

อัพเดตระบบและติดตั้ง Dependencies  

รันคำสั่งเพื่ออัพเดตระบบและติดตั้งแพ็คเกจที่จำเป็น:apt update && apt upgrade -y
apt install -y ca-certificates curl gnupg lsb-release




เพิ่ม Docker GPG Key และ Repository  

สร้างโฟลเดอร์สำหรับเก็บ GPG key:mkdir -p /etc/apt/keyrings


ดาวน์โหลดและเพิ่ม GPG key ของ Docker:curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg


เพิ่ม Docker repository:echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null




ติดตั้ง Docker Engine และ Docker Compose Plugin  

อัพเดตแพ็คเกจและติดตั้ง Docker:apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin




ตรวจสอบการติดตั้ง Docker  

ตรวจสอบเวอร์ชันของ Docker เพื่อยืนยันการติดตั้ง:docker --version






3. สร้างไฟล์ docker-compose.yml สำหรับ Portainer

สร้างโฟลเดอร์สำหรับเก็บไฟล์  

สร้างโฟลเดอร์และย้ายไปยังโฟลเดอร์นั้น:mkdir -p ~/portainer
cd ~/portainer




สร้างไฟล์ docker-compose.yml  

สร้างไฟล์ docker-compose.yml ด้วยเนื้อหาดังนี้:cat > docker-compose.yml <<EOF
version: "3.8"

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      - "8000:8000"
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
EOF






4. รัน Portainer Stack

รัน Portainer โดยใช้คำสั่ง:docker compose up -d




5. เข้าใช้งาน Portainer

เข้าถึง Portainer ผ่านเว็บเบราว์เซอร์  

เปิดเบราว์เซอร์และไปที่:https://<IP-ของ-LXC>:9443


แทน <IP-ของ-LXC> ด้วยที่อยู่ IP ของ LXC Container


ตั้งค่าผู้ใช้แรกเริ่ม  

เมื่อเข้าสู่ Portainer ครั้งแรก ระบบจะให้ตั้งค่ารหัสผ่านสำหรับผู้ใช้ admin  
หลังจากตั้งค่าเสร็จสิ้น คุณสามารถเริ่มจัดการ Docker Container ได้ทันที




หมายเหตุสำคัญ

LXC Nesting: ต้องเปิดใช้งาน nesting ในไฟล์ config ของ LXC (ตามขั้นตอนที่ 1) เพื่อให้ Docker ทำงานได้อย่างถูกต้อง
Unprivileged Container: การตั้งค่าเพิ่มเติมสำหรับ unprivileged container (เช่น lxc.apparmor.profile: unconfined) จำเป็นเพื่อให้ Docker ทำงานได้
Docker Compose: หากคำสั่ง docker compose ไม่ทำงาน ให้ลองใช้ docker-compose (ต้องติดตั้งแยกโดยใช้ apt install docker-compose)
ความปลอดภัย: 
แนะนำให้ตั้งค่า firewall เพื่อจำกัดการเข้าถึงพอร์ต 8000 และ 9443
ควรติดตั้ง SSL certificate ที่เหมาะสมสำหรับการใช้งานในสภาพแวดล้อมจริง


การจัดการ: Portainer เป็นเครื่องมือที่มีประโยชน์สำหรับจัดการ Docker containers, stacks, และ volumes ผ่าน GUI


สรุป
เพียงทำตามขั้นตอนด้านบน คุณจะสามารถติดตั้ง Docker และ Docker Compose รวมถึงรัน Portainer Stack บน Proxmox LXC ได้อย่างราบรื่น ไม่ว่าจะใช้ privileged หรือ unprivileged container คู่มือนี้เหมาะสำหรับผู้เริ่มต้นและผู้ใช้ขั้นสูงที่ต้องการจัดการ container ผ่าน GUI ที่ใช้งานง่าย
