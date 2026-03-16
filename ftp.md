```markdown
# Guide to Installing and Using FTP Server (vsftpd) on Ubuntu Server

## 1. What is FTP?

**FTP (File Transfer Protocol)** is a network protocol used to transfer files between a **client computer** and a **server**.

Two primary operations exist:

- **Download** – Retrieving files from the server to the local machine.
- **Upload** – Sending files from the local machine to the server.

### Example FTP URLs

```

ftp://ftp.psu.ac.th
ftp://ftp.gnu.org

```

---

# 2. Basic FTP Usage (Client Side)

FTP servers can be accessed in several ways.

## 2.1 Access via Web Browser or File Explorer

1. Open a browser or file explorer.
2. Enter the FTP URL in the address bar:

```

ftp://ftp.psu.ac.th/pub/

```

3. Press **Enter**.

If authentication is required:

- **Username:** `anonymous`
- **Password:** usually empty or any text

---

## 2.2 Access via Command Line (Windows CMD / Linux Terminal)

Command line access provides greater control and automation.

### Step 1: Open Command Line

Windows:

```

cmd

```

Linux:

```

terminal

```

### Step 2: Connect to FTP Server

```

ftp [Server Name or IP]

```

Example:

```

ftp ftp.psu.ac.th

```

### Step 3: Login

```

Username: anonymous
Password: (press Enter or type anything)

```

---

## Common FTP Commands

| Command | Description |
|------|------|
| `pwd` | Display current directory on server |
| `cd [folder]` | Change directory |
| `dir` or `ls` | List files and folders |
| `bin` | Binary mode (required for files like `.exe`, `.zip`, `.pdf`, `.ova`) |
| `asc` | ASCII mode (for text files) |
| `get [file]` | Download a single file |
| `mget [pattern]` | Download multiple files |
| `put [file]` | Upload a file |
| `mput [pattern]` | Upload multiple files |
| `hash` | Show `#` during transfer progress |
| `quit` | Exit FTP session |

### Example Commands

Download a file:

```

get 7z1514.exe

```

Download multiple files:

```

mget *.exe

```

Upload a file:

```

put example.txt

```

---

# 2.3 Accessing FTP Server from Another Machine

When the FTP server runs on Ubuntu, other devices must connect using the **server’s IP address**.

## Step 1: Find Server IP Address

On the Ubuntu server:

```

ip a

```

or

```

hostname -I

```

Example IP:

```

192.168.1.50

```

---

## Step 2: Connect from Client Machine

### Using Browser

```

ftp://192.168.1.50

```

### Using Command Line

```

ftp 192.168.1.50

```

### Firewall Configuration (if connection fails)

Allow FTP port **21**:

```

sudo ufw allow 21/tcp

```

---

# 3. Installing FTP Server on Ubuntu

The FTP server software used is **vsftpd (Very Secure FTP Daemon)**.

## Installation

Open terminal and run:

```

sudo apt-get install vsftpd

```

## Check Service Status

```

service vsftpd status

```

---

## Important System Information

| Item | Location |
|------|------|
| FTP user account | `ftp` |
| FTP public directory | `/srv/ftp` |
| Configuration file | `/etc/vsftpd.conf` |

---

## Backup Configuration File

Before modifying configuration:

```

cp /etc/vsftpd.conf /root/vsftpd_backup.conf

```

---

# 4. vsftpd Configuration

Edit the configuration file:

```

sudo vi /etc/vsftpd.conf

```

If a line contains `#`, remove it to enable the option.

⚠ **Important Rule**

After modifying the configuration file, restart the service:

```

sudo service vsftpd restart

```

---

# 4.1 Enable Anonymous Access

Allow public users to download files without an account.

```

anonymous_enable=YES

```

---

# 4.2 Allow Local Users to Login

Local Ubuntu users can login using their system credentials.

```

local_enable=YES

```

Example user:

```

tester1
yamaha

```

After login, the user will enter their home directory:

```

/home/tester1

```

To allow uploads, also enable:

```

write_enable=YES

```

---

# 4.3 Enable Write Access

Allow users to:

- Upload files
- Delete files
- Create files

Configuration:

```

write_enable=YES

```

---

# 4.4 Allow Anonymous Upload and Directory Creation

First ensure:

```

write_enable=YES

```

Then enable:

```

anon_upload_enable=YES
anon_mkdir_write_enable=YES

```

### Create Upload Directory

```

cd /srv/ftp
mkdir asus
sudo chown ftp asus

```

---

# 4.5 Chroot Jail (Restrict Users to Home Directory)

Prevents users from accessing other system directories.

```

chroot_local_user=YES
allow_writeable_chroot=YES

```

### Restrict Only Specific Users

Enable:

```

chroot_list_enable=YES
chroot_list_file=/etc/list.txt

```

Add usernames to:

```

/etc/list.txt

```

---

# 4.6 Timeout and Welcome Banner

### Idle Timeout

Disconnect after **300 seconds of inactivity**:

```

idle_session_timeout=300

```

### Data Transfer Timeout

Disconnect if transfer stalls for **120 seconds**:

```

data_connection_timeout=120

```

### Login Banner

```

ftpd_banner=*** Welcome to my FTP Server ***

```

---

# 4.7 Limit Number of Connections

Restrict simultaneous users:

```

max_clients=1

```

### Testing

1. Open two command prompts.
2. Login in the first window.
3. Attempt login in the second window.
4. The second connection will fail until the first user exits using:

```

quit

```

---

# 5. Summary Workflow

### 1. Install FTP Server

```

apt-get install vsftpd

```

### 2. Configure Server

Edit configuration file:

```

/etc/vsftpd.conf

```

Enable options such as:

- anonymous access
- local user login
- upload permissions
- chroot restrictions

### 3. Configure Directory Permissions

Use `chown` for upload folders.

Example:

```

sudo chown ftp foldername

```

### 4. Restart Service

```

service vsftpd restart

```

### 5. Test FTP Server

Use a client machine:

- Web browser
- File explorer
- Command line FTP client

Test:

- file download
- file upload
- authentication behavior
```
