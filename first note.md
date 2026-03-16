# Linux Server Administration: "Old School" Revision Guide

This guide covers traditional, battle-tested Linux server configurations, focusing on Ubuntu systems using legacy daemons and classic SysV/net-tools commands commonly found in older syllabi. It includes both **theory for written exams** and **practical commands for lab exams**.

---

# 1. FTP (File Transfer Protocol)

FTP is a **client-server protocol** used to transfer files between computers.

- **Download** – Getting data from another machine to your machine  
- **Upload** – Sending data from your machine to another machine  

**Standard Daemon:** vsftpd  
**Common Servers:** ftp.psu.ac.th, ftp.gnu.org

---

## Part A: Acting as an FTP Client (Command Line)

In the lab you will use the command-line FTP tool.

### Connecting & Logging In

```bash
D:\download> ftp ftp.psu.ac.th
User: anonymous
Password: (press enter or type anything)
```

### Essential Client Commands

| Command | Function |
|-------|--------|
| pwd | Print working directory on remote server |
| cd \<folder> | Change directory |
| dir / ls | List files |
| get \<file> | Download file |
| mget \<pattern> | Download multiple files |
| put \<file> | Upload file |
| mput \<pattern> | Upload multiple files |
| quit / bye | Exit FTP |

---

### Transfer Modes (Important for Exams)

**Binary Mode**

Used for binary files.

Examples:
- .exe
- .zip
- .rar
- .docx
- .pdf
- .tar.gz

```bash
bin
```

**ASCII Mode**

Used for text files.

Examples:
- .txt

```bash
asc
```

---

### Additional Commands

```bash
hash
```

Displays `#` progress indicators during downloads.

---

### Lab Example: Download Binary File

```bash
ftp> cd pub
ftp> cd 7zip
ftp> dir
ftp> bin
ftp> hash
ftp> get 7z1514-x64.exe
ftp> quit
```

---

## Part B: Setting up FTP Server (vsftpd)

### Installation

```bash
sudo apt-get install vsftpd
```

### Service Management

```bash
sudo service vsftpd status
sudo service vsftpd restart
```

### Default Storage

Files are stored in:

```
/srv/ftp
```

### Configuration File

```
/etc/vsftpd.conf
```

**Golden Rule:** Always restart the service after editing.

```bash
service vsftpd restart
```

---

### Key Configuration Options

#### Enable Anonymous Access

```
anonymous_enable=YES
```

#### Enable Local Users & Uploads

```
local_enable=YES
write_enable=YES
```

---

### Allow Anonymous Uploads

You cannot upload directly to `/srv/ftp`.

Create a folder first:

```bash
cd /srv/ftp
mkdir asus
chown ftp asus
```

Config:

```
write_enable=YES
anon_upload_enable=YES
```

---

### Allow Anonymous Folder Creation

```
write_enable=YES
anon_mkdir_write_enable=YES
```

---

### Session Timeouts

```
idle_session_timeout=300
data_connection_timeout=120
```

---

### Welcome Banner

```
ftpd_banner=***Welcome to 871-316.***
```

---

### Jail Local Users

Prevents users from leaving their home directory.

```
chroot_local_user=YES
allow_writeable_chroot=YES
```

Limit only certain users:

```
chroot_list_enable=YES
chroot_list_file=/etc/list.txt
```

---

### Limit Connections

```
max_clients=1
```

**Lab Test:**  
Open two FTP sessions. The second should fail until the first exits.

---

# 2. DNS (Domain Name System)

DNS translates **domain names → IP addresses**.

Example:

```
google.com → 142.250.4.100
```

**Standard Daemon:** bind9

---

## Installation

```bash
sudo apt-get install bind9
```

### Configuration Directory

```
/etc/bind
```

### Service Commands

```bash
service bind9 status
service bind9 restart
```

---

## Part A: DNS Queries using nslookup

### Basic Lookup

```bash
nslookup www.pn.psu.ac.th
```

### Reverse Lookup

```bash
nslookup 172.18.100.97
```

---

### Interactive Mode

```bash
nslookup
```

#### Find Name Servers

```
> set type=ns
> pn.psu.ac.th
```

#### Find Mail Servers

```
> set type=mx
> email.psu.ac.th
```

---

## Part B: Forward Lookup Zone

### Step 1: Define Zone

Edit:

```
/etc/bind/named.conf.local
```

---

### Step 2: Create Zone File

```bash
cd /etc/bind
cp db.local cim64.db
vi cim64.db
```

---

### Step 3: Add Records

#### A Records

```
www     IN A 192.168.135.9
mail    IN A 192.168.135.10
mail2   IN A 192.168.135.11
cim64.com. IN A 192.168.135.9
```

#### CNAME Records

```
www2 IN CNAME www.cim64.com.
www3 IN CNAME www.cim64.com.
```

#### MX Records

```
cim64.com. IN MX 5 mail.cim64.com.
cim64.com. IN MX 15 mail2.cim64.com.
```

Important: Fully qualified domain names require **trailing dots**.

Restart DNS after editing.

```bash
service bind9 restart
```

---

## Part C: Reverse Lookup Zone

Used for **IP → Domain name**.

Example network:

```
192.168.135.0
```

Reverse format:

```
135.168.192.in-addr.arpa
```

Example config:

```
zone "135.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/cim64.ip";
};
```

---

### Step 2: Create Reverse Zone File

```bash
cd /etc/bind
cp db.127 cim64.ip
vi cim64.ip
```

---

### Step 3: PTR Record

Example:

```
9 IN PTR www.cim64.com.
```

---

### Testing Local DNS

```bash
nslookup
> server localhost
> www.cim64.com
> exit
```

---

# 3. DHCP (Dynamic Host Configuration Protocol)

DHCP automatically assigns:

- IP Address
- DNS Server
- Subnet Mask
- Default Gateway

**Standard Daemon:** isc-dhcp-server

---

## Installation

```bash
sudo apt-get install isc-dhcp-server
```

### Configuration File

```
/etc/dhcp/dhcpd.conf
```

### Service Commands

```bash
service isc-dhcp-server start
service isc-dhcp-server stop
service isc-dhcp-server restart
service isc-dhcp-server status
```

---

## Key Configuration Keywords

| Parameter | Meaning |
|----------|--------|
| domain-name | DNS domain |
| option domain-name-servers | DNS server IP |
| default-lease-time | Default IP lease time |
| max-lease-time | Maximum lease time |
| subnet | Network address |
| netmask | Subnet mask |
| range | DHCP pool |
| option routers | Default gateway |

---

## Example Configuration

Scenario:

- Domain: anwa.com  
- DNS: 203.154.177.5, 203.154.177.6  
- Lease: 300 seconds  
- Max Lease: 600 seconds  

```
option domain-name "anwa.com";
option domain-name-servers 203.154.177.5, 203.154.177.6;
default-lease-time 300;
max-lease-time 600;

subnet 203.154.178.0 netmask 255.255.255.192 {
    range 203.154.178.10 203.154.178.20;
    option routers 203.154.178.1;
}

subnet 203.154.179.0 netmask 255.255.255.224 {
    range 203.154.179.30 203.154.179.50;
    option routers 203.154.179.1;
}
```

Restart service after editing:

```bash
service isc-dhcp-server restart
```

---

## Static IP Reservation

Used when a device must always receive the same IP.

Example:

IP: 203.154.178.51  
MAC: 7C:05:07:91:67:3B

```
host fanta {
    hardware ethernet 7C:05:07:91:67:3B;
    fixed-address 203.154.178.51;
}
```

Note: Linux uses `:` in MAC addresses.

---

# 4. Essential Linux Commands for Sysadmins

## Permissions & Ownership

Linux files have:

- Owner
- Group
- Others

Permission values:

| Permission | Value |
|-----------|------|
| Read | 4 |
| Write | 2 |
| Execute | 1 |

---

### chmod

Change file permissions.

Example:

```bash
chmod 755 script.sh
```

Meaning:

```
7 = rwx
5 = r-x
5 = r-x
```

Symbolic example:

```bash
chmod u+x script.sh
```

---

### chown

Change owner and group.

```bash
chown root:admin config.txt
```

---

### chgrp

Change group only.

```bash
chgrp webdev index.html
```

---

# Group & User Management

### groupmod

Rename group.

```bash
groupmod -n new_name old_name
```

### groupadd

Create group.

### usermod

Modify user.

Add user to sudo group:

```bash
usermod -aG sudo username
```

---

# File Archiving & Compression

## bzip2

Compress file.

```bash
bzip2 -k psu1.txt
```

Creates:

```
psu1.txt.bz2
```

`-k` keeps original file.

---

## bunzip2

Decompress.

```bash
bunzip2 -k psu1.txt.bz2
```

---

# tar (Tape Archive)

Used to archive folders.

### Key Options

| Option | Meaning |
|------|------|
| c | create |
| x | extract |
| t | list |
| f | filename |
| v | verbose |
| z | gzip |
| j | bzip2 |

---

## tar.gz Example

### Compress

```bash
tar cfzv data.tar.gz data/
```

### View

```bash
tar tfzv data.tar.gz
```

### Extract

```bash
tar xfzv data.tar.gz
```

---

## tar.bz2 Example

### Compress

```bash
tar cfjv data.tar.bz2 data/
```

### View

```bash
tar tfjv data.tar.bz2
```

Large output:

```bash
tar tfjv httpd.tar.bz2 | more
```

### Extract

```bash
tar xfjv data.tar.bz2
```

---

# "Old School" Networking Commands

Older exams often use **deprecated commands**.

| Action | Old Command | Modern Equivalent |
|------|-------------|----------------|
| View IP | ifconfig | ip a |
| Open Ports | netstat -tuln | ss -tuln |
| DNS Lookup | nslookup example.com | dig example.com |
| Manage Service | service vsftpd restart | systemctl restart vsftpd |
| Routing Table | route -n | ip route |

---
