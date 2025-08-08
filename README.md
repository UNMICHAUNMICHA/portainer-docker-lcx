# portainer-docker-lcx
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 \
  --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data portainer/portainer-ce:latest



ตั้งค่า Container config เพิ่มเติม (ถ้าไม่ใช่ privileged)
ถ้าอยากใช้ unprivileged ต้องแก้ไขไฟล์ config container เช่น

swift
Copy
Edit
nano /etc/pve/lxc/<CTID>.conf
เพิ่มบรรทัดเหล่านี้เพื่อเปิด cgroup และให้ Docker ทำงานได้

makefile
Copy
Edit
features: nesting=1,keyctl=1
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
