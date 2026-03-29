---
date_created: 2026-03-20
date_modified: 2026-03-20
document_type: note
project: Proxmox
tags:
  - proxmox
  - "#vm"
  - immich
  - ubuntu
---
[[Projects/Proxmox/Home|Home]] | [[Projects/Proxmox/Meetings/All Meetings|Meetings]] | [[Projects/Proxmox/Notes/All Notes|Notes]] | [[Projects/Proxmox/References|References]]
# Install Immich in vm
**Overview**
Description:: Покрокова інструкція зі створення Ubuntu VM у Proxmox, міграції конфігів і бекапу Immich зі старого Raspberry Pi 4, а також підключення Tailscale для безпечного віддаленого доступу.

## Note

### 1. Створення Ubuntu VM у Proxmox

#### Рекомендована конфігурація VM
- CPU: 4 cores
- RAM: 8 GB
- CPU type: `host`
- BIOS: `OVMF (UEFI)`
- Machine: `q35`
- SCSI Controller: `VirtIO SCSI single`
- Disk 1 (system): 100–150 GB
- Disk 2 (library/data): решта доступного місця
- Network: `VirtIO`, bridge `vmbr0`

#### Кроки у веб-інтерфейсі Proxmox
1. Завантажити `Ubuntu Server 24.04 LTS` ISO в `local` → `ISO Images`.
2. Натиснути `Create VM`.
3. На вкладці `General` задати ім’я, наприклад `immich-vm`.
4. На вкладці `OS` вибрати ISO Ubuntu Server.
5. На вкладці `System` виставити `q35`, `OVMF (UEFI)`, додати EFI disk.
6. На вкладці `Disks` для системного диска виставити:
   - Bus/Device: `SCSI`
   - Disk size: 100–150 GB
   - `Discard`: On
   - `SSD emulation`: On
   - `IO thread`: On
7. На вкладці `CPU` виставити `1 socket`, `4 cores`, `Type: host`.
8. На вкладці `Memory` виставити `8192 MB`.
9. На вкладці `Network` виставити `VirtIO` і `vmbr0`.
10. Після створення VM через `Hardware` → `Add` → `Hard Disk` додати другий диск під library.
11. Переконатися, що системний диск — `scsi0`, а data disk — `scsi1`.
12. Увімкнути `QEMU Guest Agent` в `Options` і пізніше встановити його в Ubuntu.

### 2. Інсталяція Ubuntu у VM
1. Запустити VM і відкрити `Console`.
2. Під час інсталяції встановлювати Ubuntu тільки на перший диск.
3. Другий диск не чіпати на етапі інсталяції.
4. Увімкнути `OpenSSH server` під час встановлення.
5. Після першого boot перевірити диски:
```bash
lsblk
```
6. Якщо root створено через LVM не на весь диск, розширити його:
```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

### 3. Форматування другого диска
> Перед виконанням перевірити правильну назву другого диска через `lsblk`.

1. Подивитися диски:
```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,MODEL
sudo fdisk -l
```
2. Якщо другий диск, наприклад `/dev/sdb`, порожній — створити GPT і одну партицію:
```bash
sudo parted /dev/sdb --script mklabel gpt
sudo parted /dev/sdb --script mkpart primary ext4 0% 100%
```
3. Форматувати партицію:
```bash
sudo mkfs.ext4 -L immich-library /dev/sdb1
```
4. Створити точку монтування:
```bash
sudo mkdir -p /mnt/immich-library
```
5. Отримати UUID:
```bash
sudo blkid /dev/sdb1
```
6. Додати диск у `/etc/fstab`:
```fstab
UUID=ТУТ_UUID /mnt/immich-library ext4 defaults,nofail 0 2
```
7. Змонтувати:
```bash
sudo mount -a
df -h
lsblk -f
```
8. Дати права користувачу:
```bash
sudo chown -R $USER:$USER /mnt/immich-library
mkdir -p /mnt/immich-library/uploads
mkdir -p /mnt/immich-library/postgres
```

### 4. Перенесення файлів Immich зі старого Raspberry Pi

#### Варіант через rsync
Найзручніше запускати `rsync` або на новій VM, або на Raspberry Pi. Напрям копіювання визначається порядком `source destination`.

##### Якщо запускати на Raspberry Pi
Скопіювати вміст папки `~/immich-app` на нову VM:
```bash
rsync -avz ~/immich-app/ username@IP_НОВОЇ_VM:~/immich-app/
```

##### Якщо потрібно спочатку перевірити, що буде скопійовано
```bash
rsync -avzn ~/immich-app/ username@IP_НОВОЇ_VM:~/immich-app/
```

##### Якщо не хочеш перезаписувати вже існуючі файли
```bash
rsync -avz --ignore-existing ~/immich-app/ username@IP_НОВОЇ_VM:~/immich-app/
```

#### SSH ключі для rsync без пароля
На машині, з якої запускається `rsync`, створити ключ:
```bash
ssh-keygen -t ed25519 -C "immich-migration"
```
Скопіювати публічний ключ на іншу машину:
```bash
ssh-copy-id username@IP_ІНШОЇ_МАШИНИ
```
Перевірити вхід без пароля:
```bash
ssh username@IP_ІНШОЇ_МАШИНИ
```
Після цього `rsync` зможе працювати без запиту пароля.

#### Які файли переносилися
- `.env`
- `apikey.txt`
- `docker-compose.yml`
- за потреби вся папка `~/immich-app`

> Після копіювання треба відкрити `.env` і перевірити, що шляхи вказують уже на нову VM, а не на старий Raspberry Pi.

### 5. Встановлення Docker на Ubuntu VM
1. Встановити базові пакети:
```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
2. Додати Docker repository:
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"${UBUNTU_CODENAME:-$VERSION_CODENAME}\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
3. Встановити Docker Engine і Compose plugin:
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
4. Перевірити Docker:
```bash
sudo docker run hello-world
docker compose version
```
5. Додати користувача в групу docker:
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 6. Запуск Immich на новій VM
1. Створити папку:
```bash
mkdir -p ~/immich-app
cd ~/immich-app
```
2. Якщо не переносиш старі файли, завантажити офіційні:
```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```
3. У `.env` виставити нові шляхи:
```env
UPLOAD_LOCATION=/mnt/immich-library/uploads
DB_DATA_LOCATION=/mnt/immich-library/postgres
IMMICH_VERSION=release
```
4. Запустити:
```bash
docker compose up -d
```
5. Перевірити статус:
```bash
docker compose ps
docker compose logs -f
```
6. Локально Immich має відкриватися на:
```text
http://IP_ТВОЄЇ_VM:2283
```

### 7. Відновлення з бекапу
1. Після перенесення старих конфігів і запуску контейнерів відновити Immich з бекапу.
2. Перевірити, що локально Immich працює правильно.
3. Переконатися, що uploads і postgres реально лежать на другому диску:
```bash
df -h /mnt/immich-library
du -sh /mnt/immich-library/*
```

### 8. Підключення Tailscale для віддаленого доступу
Найкращий варіант — встановити Tailscale прямо в Ubuntu VM, а не в Docker-контейнер Immich.

1. Встановити Tailscale:
```bash
curl -fsSL https://tailscale.com/install.sh | sh
```
2. Підняти tailnet-підключення:
```bash
sudo tailscale up
```
3. Після авторизації перевірити:
```bash
tailscale status
tailscale ip -4
```
4. На телефон встановити Tailscale і увійти в той самий акаунт.
5. У застосунку Immich як server URL вказати:
```text
http://TAILSCALE_IP_VM:2283
```
або MagicDNS-ім’я, якщо воно ввімкнене.

### 9. Автозапуск Tailscale після перезавантаження
Tailscale у VM має запускатися автоматично, якщо `tailscaled` увімкнений у systemd.

Перевірити:
```bash
systemctl status tailscaled
systemctl is-enabled tailscaled
```
Якщо треба — увімкнути:
```bash
sudo systemctl enable --now tailscaled
```

Також потрібно переконатися, що сама VM має автозапуск у Proxmox, інакше після reboot хоста вона не підніметься.

## References
- Immich local + remote setup notes from chat
- Proxmox VM sizing and storage notes
- rsync migration notes
- Tailscale notes

---
[[Projects/Proxmox/Home|Home]] | [[Projects/Proxmox/Meetings/All Meetings|Meetings]] | [[Projects/Proxmox/Notes/All Notes|Notes]] | [[Projects/Proxmox/References|References]]
