# adminnotes


Linux Server Administration: "Old School" Revision GuideThis guide covers traditional, battle-tested Linux server configurations, focusing on Ubuntu systems using legacy daemons and classic SysV/net-tools commands commonly found in older syllabi. It includes both theory for written exams and practical commands for lab exams.1. FTP (File Transfer Protocol)FTP is a client-server protocol used to transfer files between computers.Download: Getting data from another machine to your own machine.Upload: Sending data from your machine to another machine.Standard Daemon: vsftpdCommon Servers: ftp.psu.ac.th, ftp.gnu.orgPart A: Acting as an FTP Client (Command Line)In the lab, you will use the command-line ftp tool to connect to remote servers.Connecting & Logging In:D:\download> ftp ftp.psu.ac.th
User: anonymous
Password: (press enter or type anything)



Essential Client Commands:pwd: Print working directory on the remote server.cd <folder>: Change directory on the remote server.dir or ls: List files and folders.Transfer Modes (Crucial Exam Concept):bin (Binary mode): MUST be used for binary files like .exe, .com, .zip, .rar, .docx, .pdf, .tar.gz.asc (ASCII mode): Used for text files like .txt.hash: Displays hash marks (#) on the screen during download to show progress.get <filename>: Download a single file.mget <pattern>: Download multiple files (e.g., mget *, mget a*, mget *.exe).put <filename>: Upload a single file.mput <pattern>: Upload multiple files.quit or bye: Exit the FTP session.Lab Example: Downloading a Binary Fileftp> cd pub
ftp> cd 7zip
ftp> dir
ftp> bin         <-- Set to binary mode for .exe
ftp> hash        <-- Show progress
ftp> get 7z1514-x64.exe
ftp> quit



Part B: Setting up the FTP Server (vsftpd)Installation: sudo apt-get install vsftpdStatus/Restart: sudo service vsftpd status / sudo service vsftpd restartDefault Storage: When installed, the system creates an ftp user account. The files are stored in the home directory of this ftp account, which is typically /srv/ftp.Configuration File: /etc/vsftpd.confGolden Rule: Every time you edit vsftpd.conf, you MUST run service vsftpd restart.Key vsftpd.conf Configurations (Know these for the exam!):Enable Anonymous Access:anonymous_enable=YES



Enable Local User Access & Uploads:local_enable=YES
write_enable=YES



Allow Anonymous Users to Upload:Prerequisite: You cannot upload directly to the root of /srv/ftp. You must create a subfolder and give the ftp user ownership.Commands: cd /srv/ftp -> mkdir asus -> chown ftp asusConfig:write_enable=YES
anon_upload_enable=YES



Allow Anonymous Users to Create Folders (mkdir):write_enable=YES
anon_mkdir_write_enable=YES



Session & Data Timeouts:idle_session_timeout=300
data_connection_timeout=120



Set a Welcome Banner:ftpd_banner=***Welcome to 871-316.***



Jail Local Users (Prevent cd to other system directories outside their home):chroot_local_user=YES
allow_writeable_chroot=YES



To limit only specific users, add: chroot_list_enable=YES and chroot_list_file=/etc/list.txtLimit Connections (e.g., limit to 1 connection total):max_clients=1



Lab Test: Open two Windows Command Prompts. Connect with anonymous on the first. If you try to connect on the second, it will fail until you type quit on the first window.2. DNS (Domain Name System)DNS is the system used to translate human-readable domain names (like google.com) into IP addresses (like 142.250.4.100), or vice versa. The system relies entirely on IP addresses, but humans cannot easily memorize them.Standard Daemon: bind9Installation: sudo apt-get install bind9Configuration Directory: /etc/bindCheck Status / Restart: service bind9 status / service bind9 restartPart A: DNS Querying with nslookup (Name Server Lookup)nslookup is the standard command-line tool used to search/query DNS records.Basic Lookup (Name to IP): nslookup www.pn.psu.ac.thReverse Lookup (IP to Name): nslookup 172.18.100.97Interactive Mode (set type):When you type nslookup and press Enter, you enter an interactive prompt (>).Find Name Servers (NS): > set type=ns then type the domain (e.g., pn.psu.ac.th)Find Mail Servers (MX): > set type=mx then type the domain (e.g., email.psu.ac.th)Part B: Setting up a Forward Lookup Zone (Name to IP)This creates the records that translate your custom domain (e.g., cim64.com) to IPs.1. Define the Zone in Config:Edit /etc/bind/named.conf.local to tell bind9 about your new domain.2. Create the Zone File (.db):Navigate to /etc/bind. Copy the default template (db.local) to create your new file.cd /etc/bind
cp db.local cim64.db
vi cim64.db


3. Add Records to the Zone File:Add your custom A (Address), CNAME (Alias/Canonical Name), and MX (Mail Exchange) records.; A Records (Map names to IP)
www     IN A 192.168.135.9
mail    IN A 192.168.135.10
mail2   IN A 192.168.135.11
cim64.com. IN A 192.168.135.9   ; Allows reaching site without 'www'

; CNAME Records (Aliases/Fake names pointing to the Real name)
www2    IN CNAME [www.cim64.com](https://www.cim64.com).
www3    IN CNAME [www.cim64.com](https://www.cim64.com).

; MX Records (Mail Routing - Lower number means higher priority)
cim64.com. IN MX 5 mail.cim64.com.
cim64.com. IN MX 15 mail2.cim64.com.


Note: Don't forget the trailing dots (.) on fully qualified domain names! Always service bind9 restart after editing.Part C: Setting up a Reverse Lookup Zone (IP to Name)This allows systems to look up an IP address to find out what domain name belongs to it.1. Define the Reverse Zone:Edit /etc/bind/named.conf.local. Reverse the first three octets of your network IP (e.g., 192.168.135 becomes 135.168.192) and append .in-addr.arpa.zone "135.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/cim64.ip";
};


2. Create the Reverse Zone File (.ip):Copy the local loopback template (db.127).cd /etc/bind
cp db.127 cim64.ip
vi cim64.ip


3. Add PTR (Pointer) Records:Use the last octet of the IP address (e.g., 9 for 192.168.135.9) and point it to the domain.9   IN PTR  [www.cim64.com](https://www.cim64.com).


💻 Lab Practice: Testing Your Local DNS Servernslookup
> server localhost       <-- Tells nslookup to use YOUR newly built server
> [www.cim64.com](https://www.cim64.com)
> exit


3. DHCP (Dynamic Host Configuration Protocol)DHCP automatically assigns IP addresses and network configurations (DNS, subnet mask, gateway) to client machines.Standard Daemon: isc-dhcp-serverInstallation: sudo apt-get install isc-dhcp-serverConfiguration File: /etc/dhcp/dhcpd.confService Management: service isc-dhcp-server start | stop | restart | statusPart A: Navigating to the Configuration FileYour professor highlighted a few ways to edit the config file. Make sure you are comfortable with these paths in the lab:Direct edit: vi /etc/dhcp/dhcpd.confNavigating first: cd /etc/dhcp, then pwd, then vi dhcpd.confPart B: Key Configuration Keywords (Know these!)You will need to construct the dhcpd.conf file using these exact parameters:domain-name: The DNS domain (e.g., "pn.psu.ac.th", "facebook.com"). Must be in quotes.option domain-name-servers: The IP address of your DNS server(s).default-lease-time: The standard duration (in seconds) a client keeps an IP.max-lease-time: The maximum duration (in seconds) a client can keep an IP.subnet: The network address.netmask: The Subnet Mask (divider of the network, e.g., 255.255.255.0).range: The pool/span of IP addresses to hand out to clients.option routers: The Default Gateway (the path in/out of the network).💻 Lab Practice: Configuring Subnets (Based on Notes)Scenario: Set up a DHCP server for anwa.com.DNS: 203.154.177.5, 203.154.177.6Lease Time: 5 minutes (300 seconds)Max Lease: 10 minutes (600 seconds)Network 1: 203.154.178.0 / Mask 255.255.255.192 / IPs 10-20 / GW 203.154.178.1Network 2: 203.154.179.0 / Mask 255.255.255.224 / IPs 30-50 / GW 203.154.179.1The Config Block (/etc/dhcp/dhcpd.conf):option domain-name "anwa.com";
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

(Note: Always remember to run service isc-dhcp-server restart after saving!)Part C: Static IP Reservation (Fixed-Address)Sometimes you need a specific machine (like a printer or a boss's computer) to always get the same IP address. You do this by mapping its MAC Address to an IP.Example: Assign IP 203.154.178.51 to MAC 7C:05:07:91:67:3B (Computer named 'fanta').Add this block to your dhcpd.conf:host fanta {
    hardware ethernet 7C:05:07:91:67:3B;
    fixed-address 203.154.178.51;
}

(Note: In Linux config files, MAC addresses use colons :, whereas Windows often uses hyphens -. Make sure to convert them if given a hyphenated MAC in a prompt!)4. Essential Linux Commands for SysadminsPermissions & OwnershipIn Linux, every file and directory has an Owner, a Group, and Others. Permissions are calculated as Read (r=4), Write (w=2), and Execute (x=1).chmod (Change Mode): Changes file permissions.Octal method: chmod 755 script.sh7 (Owner): 4+2+1 (Read/Write/Execute)5 (Group): 4+1 (Read/Execute)5 (Others): 4+1 (Read/Execute)Symbolic method: chmod u+x script.sh (adds execute permission for the user/owner).chown (Change Owner): Changes the user and/or group ownership of a file.chown root:admin config.txt (Changes owner to root and group to admin).chgrp (Change Group): Changes just the group ownership of a file.chgrp webdev index.htmlGroup & User Managementgroupmod: Modifies an existing group on the system (e.g., changing its name or Group ID/GID).groupmod -n new_name old_namegroupadd: Creates a brand new group.usermod: Modifies a user account. Commonly used to add users to groups.usermod -aG sudo username (Appends the user to the sudo group).File Archiving & CompressionIn Linux, file archiving (grouping files together) and compression (making them smaller) are often handled by utilities like bzip2, bunzip2, and tar.1. bzip2 & bunzip2 (Single File Compression)These commands are used to compress and decompress individual files.Compress: bzip2 -k <filename> (e.g., bzip2 -k psu1.txt)Important Note: The -k flag tells the system to keep the original file. If you don't use -k, the original psu1.txt is deleted and replaced with psu1.txt.bz2!Decompress: bunzip2 -k <filename> (e.g., bunzip2 -k psu1.txt.bz2)Again, -k keeps the compressed .bz2 file intact while extracting the original.2. tar (Tape Archive)tar is used to group entire folders into a single file and can apply compression simultaneously.Key Options to Memorize:c = createx = extract (คลาย)t = view/list contentsf = file (always placed right before the filename)v = verbose (shows you what files are being processed)z = compress/decompress using gzip (.tar.gz)j = compress/decompress using bzip2 (.tar.bz2)Working with .tar.gz (Gzip):Compress: tar cfzv data.tar.gz data/ (Compresses the 'data' folder into 'data.tar.gz')View Contents: tar tfzv data.tar.gzExtract: tar xfzv data.tar.gzWorking with .tar.bz2 (Bzip2):Compress: tar cfjv data.tar.bz2 data/View Contents: tar tfjv data.tar.bz2 (Tip: Add | more if the list is too long to read: tar tfjv httpd.tar.bz2 | more)Extract: tar xfjv data.tar.bz2"Old School" Networking Commands (net-tools & SysVinit)Older exams often test deprecated commands instead of their modern iproute2 or systemd equivalents.Action"Old School" Command (Know these!)Modern EquivalentView IP Addressesifconfigip aView Open Portsnetstat -tulnss -tulnDNS Lookupnslookup example.comdig example.comManage Servicesservice vsftpd restartsystemctl restart vsftpdView Routing Tableroute -nip route
