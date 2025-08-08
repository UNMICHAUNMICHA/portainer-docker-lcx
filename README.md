# portainer-docker-lcx
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 \
  --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data portainer/portainer-ce:latest

วิธีติดตั้ง Docker ใน Debian/Ubuntu LXC แบบแนะนำ (ด้วย official Docker repository):
อัพเดตแพ็กเกจ


apt update
apt upgrade -y
ติดตั้ง dependencies


apt install -y ca-certificates curl gnupg lsb-release
เพิ่ม Docker GPG key


mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
เพิ่ม Docker repository (สำหรับ Ubuntu 24.04 เป็นตัวอย่าง หากใช้ Debian หรือเวอร์ชันอื่น ให้แก้ชื่อโค้ดเนมให้ถูก)


echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
อัพเดต apt และติดตั้ง Docker Engine

apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
ตรวจสอบ Docker

docker --version
systemctl status docker

ตั้งค่า Container config เพิ่มเติม (ถ้าไม่ใช่ privileged)
ถ้าอยากใช้ unprivileged ต้องแก้ไขไฟล์ config container เช่น


nano /etc/pve/lxc/<CTID>.conf
เพิ่มบรรทัดเหล่านี้เพื่อเปิด cgroup และให้ Docker ทำงานได้


features: nesting=1,keyctl=1
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
