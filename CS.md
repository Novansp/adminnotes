# Linux Server Administration Cheat Sheet (Old School)

Focus: **FTP, DNS, DHCP, Permissions, Compression, Networking Commands**

---

# 1. FTP

## Connect as Client
```bash
ftp ftp.psu.ac.th
User: anonymous
Password: (enter anything)
```

## Essential FTP Commands

| Command | Purpose |
|-------|------|
| pwd | remote working directory |
| cd \<dir> | change directory |
| dir / ls | list files |
| get \<file> | download |
| mget \<pattern> | download multiple |
| put \<file> | upload |
| mput \<pattern> | upload multiple |
| hash | show progress |
| quit / bye | exit |

## Transfer Modes

```
bin   → binary files (.exe .zip .pdf .tar.gz)
asc   → text files (.txt)
```

Always use **bin for executables and archives**

---

## Example Download

```bash
ftp> cd pub
ftp> cd 7zip
ftp> bin
ftp> hash
ftp> get 7z1514-x64.exe
ftp> quit
```

---

# FTP Server (vsftpd)

## Install

```bash
sudo apt-get install vsftpd
```

## Service Control

```bash
service vsftpd status
service vsftpd restart
```

## Config File

```
/etc/vsftpd.conf
```

Always restart after editing.

---

## Important Config Options

### Enable Anonymous

```
anonymous_enable=YES
```

### Enable Local Login + Upload

```
local_enable=YES
write_enable=YES
```

### Allow Anonymous Upload

```
write_enable=YES
anon_upload_enable=YES
```

Create folder first:

```bash
cd /srv/ftp
mkdir upload
chown ftp upload
```

### Allow Anonymous mkdir

```
anon_mkdir_write_enable=YES
```

### Timeouts

```
idle_session_timeout=300
data_connection_timeout=120
```

### Banner

```
ftpd_banner=***Welcome***
```

### Jail Users

```
chroot_local_user=YES
allow_writeable_chroot=YES
```

### Limit Connections

```
max_clients=1
```

---

# 2. DNS (bind9)

## Install

```bash
sudo apt-get install bind9
```

## Directory

```
/etc/bind
```

## Service

```bash
service bind9 status
service bind9 restart
```

---

# DNS Query (nslookup)

## Basic

```bash
nslookup www.pn.psu.ac.th
```

## Reverse

```bash
nslookup 172.18.100.97
```

## Interactive

```bash
nslookup
```

Find NS:

```
set type=ns
domain.com
```

Find MX:

```
set type=mx
domain.com
```

---

# Forward Lookup Zone

## Define Zone

Edit:

```
/etc/bind/named.conf.local
```

Example:

```
zone "cim64.com" {
 type master;
 file "/etc/bind/cim64.db";
};
```

---

## Create Zone File

```bash
cd /etc/bind
cp db.local cim64.db
vi cim64.db
```

---

## Records

### A Records

```
www     IN A 192.168.135.9
mail    IN A 192.168.135.10
mail2   IN A 192.168.135.11
cim64.com. IN A 192.168.135.9
```

### CNAME

```
www2 IN CNAME www.cim64.com.
www3 IN CNAME www.cim64.com.
```

### MX

```
cim64.com. IN MX 5 mail.cim64.com.
cim64.com. IN MX 15 mail2.cim64.com.
```

**Important:** FQDN ends with `.`

Restart:

```bash
service bind9 restart
```

---

# Reverse Lookup Zone

Reverse network:

```
192.168.135 → 135.168.192.in-addr.arpa
```

Config:

```
zone "135.168.192.in-addr.arpa" {
 type master;
 file "/etc/bind/cim64.ip";
};
```

Create file:

```bash
cp db.127 cim64.ip
```

PTR record:

```
9 IN PTR www.cim64.com.
```

---

# Test Local DNS

```bash
nslookup
server localhost
www.cim64.com
exit
```

---

# 3. DHCP (isc-dhcp-server)

## Install

```bash
sudo apt-get install isc-dhcp-server
```

## Config

```
/etc/dhcp/dhcpd.conf
```

## Service

```bash
service isc-dhcp-server start
service isc-dhcp-server restart
service isc-dhcp-server status
```

---

# DHCP Config Template

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

Restart service:

```bash
service isc-dhcp-server restart
```

---

# Static DHCP IP

```
host fanta {
 hardware ethernet 7C:05:07:91:67:3B;
 fixed-address 203.154.178.51;
}
```

Linux MAC uses `:` not `-`

---

# 4. Permissions

## Permission Values

| Permission | Value |
|------|------|
| r | 4 |
| w | 2 |
| x | 1 |

Example:

```
chmod 755 file
```

Meaning:

```
7 = rwx
5 = r-x
5 = r-x
```

Symbolic:

```bash
chmod u+x script.sh
```

---

## Ownership

Change owner:

```bash
chown user:group file
```

Example:

```bash
chown root:admin config.txt
```

Change group:

```bash
chgrp webdev index.html
```

---

# 5. User / Group Management

Create group

```bash
groupadd groupname
```

Rename group

```bash
groupmod -n new old
```

Add user to group

```bash
usermod -aG sudo username
```

---

# 6. Compression

## bzip2

Compress:

```bash
bzip2 -k file.txt
```

Output:

```
file.txt.bz2
```

Decompress:

```bash
bunzip2 -k file.txt.bz2
```

---

# tar

## Key Flags

| Flag | Meaning |
|----|----|
| c | create |
| x | extract |
| t | list |
| f | file |
| v | verbose |
| z | gzip |
| j | bzip2 |

---

## tar.gz

Create

```bash
tar cfzv archive.tar.gz folder/
```

View

```bash
tar tfzv archive.tar.gz
```

Extract

```bash
tar xfzv archive.tar.gz
```

---

## tar.bz2

Create

```bash
tar cfjv archive.tar.bz2 folder/
```

View

```bash
tar tfjv archive.tar.bz2
```

Extract

```bash
tar xfjv archive.tar.bz2
```

---

# Old School Networking Commands

| Task | Command |
|----|----|
| View IP | ifconfig |
| View Ports | netstat -tuln |
| DNS Query | nslookup domain |
| Restart service | service name restart |
| Routing table | route -n |

---

# Quick Service Pattern

```
service <daemon> status
service <daemon> restart
```

Examples

```
service vsftpd restart
service bind9 restart
service isc-dhcp-server restart
```

---
