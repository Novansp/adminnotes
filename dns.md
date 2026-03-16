```markdown
# Comprehensive Guide: Setting Up a DNS Server on Ubuntu

This document explains the principles of the Domain Name System (DNS) and provides a complete tutorial for configuring a DNS server using **BIND9** on Ubuntu Server.

---

# Part 1: How DNS Works (Client & Server)

## What is DNS?

DNS (Domain Name System) functions as the **“phonebook of the internet.”**

- Computers communicate using **IP addresses** (e.g., `142.250.4.100`).
- Humans prefer **domain names** (e.g., `google.com`).

DNS translates **human-readable domain names into IP addresses**, allowing computers to locate services on networks.

---

## The Client–Server Relationship

### The Client (The Asker)

When a user enters:

```

[www.cim64.com](http://www.cim64.com)

```

into a web browser:

1. The computer acts as a **DNS resolver (client)**.
2. It sends a **DNS query** to its configured DNS server.
3. The query asks:

```

"What is the IP address for this domain?"

```

---

### The Server (The Answerer)

The DNS server receives the request and responds in two possible ways:

**Authoritative Response**

- If the server hosts the domain database, it directly returns the IP address.

**Recursive Response**

- If the server does not know the answer:
  - It queries other DNS servers on the internet.
  - Retrieves the correct IP address.
  - Stores it temporarily in its **cache**.
  - Sends the result back to the client.

---

## Key Tool: `nslookup`

`nslookup` (Name Server Lookup) is a command-line tool used to query DNS servers.

Common uses:

### Find IP Address of a Domain

```

nslookup google.com

```

### Reverse Lookup (IP → Domain)

```

nslookup 142.250.4.100

```

### Find Name Servers

```

nslookup
set type=ns
google.com

```

### Find Mail Servers

```

nslookup
set type=mx
google.com

```

---

# Part 2: Installing and Configuring BIND9 on Ubuntu (Server-Side)

This example sets up a **local DNS domain**:

```

Domain: cim64.com
Subnet: 192.168.135.x
Server IP: 192.168.135.5

```

---

# Step 1: Install the DNS Server (BIND9)

BIND9 is the most widely used DNS software on Linux systems.

Update repositories:

```

sudo apt-get update

```

Install BIND9:

```

sudo apt-get install bind9

```

All configuration files are stored in:

```

/etc/bind

```

---

# Step 2: Configure the Forward Lookup Zone (Name → IP)

A **Forward Lookup Zone** resolves domain names to IP addresses.

---

## Declare the Zone

Edit the local configuration file:

```

cd /etc/bind
sudo nano named.conf.local

```

Add the following:

```

zone "cim64.com" {
type master;
file "/etc/bind/cim64.db";
};

```

---

## Create the Zone Database File

Copy the default template:

```

sudo cp db.local cim64.db

```

Edit the new file:

```

sudo nano cim64.db

```

---

## Add DNS Records

```

; Bind data file for cim64.com
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
2         ; Serial
604800         ; Refresh
86400         ; Retry
2419200         ; Expire
604800 )       ; Negative Cache TTL
;

@       IN      NS      localhost.

; Root Domain Record
cim64.com.  IN  A       192.168.135.9

; A Records (Hostname → IP)
www         IN  A       192.168.135.9
mail        IN  A       192.168.135.10
mail2       IN  A       192.168.135.11

; CNAME Records (Aliases)
www2        IN  CNAME   [www.cim64.com](http://www.cim64.com).
www3        IN  CNAME   [www.cim64.com](http://www.cim64.com).

; MX Records (Mail Servers)
cim64.com.  IN  MX 5    mail.cim64.com.
cim64.com.  IN  MX 15   mail2.cim64.com.

```

---

# Step 3: Configure the Reverse Lookup Zone (IP → Name)

A **Reverse Lookup Zone** converts IP addresses back to domain names.

---

## Declare the Reverse Zone

Edit the configuration again:

```

sudo nano named.conf.local

```

Add:

```

zone "135.168.192.in-addr.arpa" {
type master;
file "/etc/bind/cim64.ip";
};

```

---

## Create the Reverse Zone File

Copy the template:

```

sudo cp db.127 cim64.ip

```

Edit the file:

```

sudo nano cim64.ip

```

---

## Add PTR Records

```

; Bind reverse data file for 192.168.135.x subnet
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
1         ; Serial
604800         ; Refresh
86400         ; Retry
2419200         ; Expire
604800 )       ; Negative Cache TTL
;

@       IN      NS      localhost.

; PTR Record (9 represents 192.168.135.9)
9       IN      PTR     [www.cim64.com](http://www.cim64.com).

```

---

# Step 4: Server Management and Testing

After modifying configuration files, always verify and reload the DNS service.

---

## Check Configuration Syntax

```

sudo named-checkconf

```

Check zone file:

```

sudo named-checkzone cim64.com /etc/bind/cim64.db

```

---

## Restart BIND9

```

sudo systemctl restart bind9

```

Check service status:

```

sudo systemctl status bind9

```

---

# Part 3: Using the DNS Server (Client-Side)

Assume the DNS server IP is:

```

192.168.135.5

```

Client devices must be configured to use this server for DNS resolution.

---

# 1. Temporary Client Test

Test the server from another machine on the same network.

```

nslookup [www.cim64.com](http://www.cim64.com) 192.168.135.5

```

Expected result:

```

192.168.135.9

```

If this appears, the DNS server is functioning correctly.

---

# 2. Configuring an Ubuntu / Linux Client

Open the Netplan configuration:

```

sudo nano /etc/netplan/01-netcfg.yaml

```

Example configuration:

```

network:
version: 2
ethernets:
eth0:
dhcp4: true
nameservers:
addresses: [192.168.135.5, 8.8.8.8]

```

Apply the configuration:

```

sudo netplan apply

```

`8.8.8.8` acts as a fallback DNS server.

---

# 3. Configuring a Windows Client

1. Open **Settings**
2. Go to **Network & Internet**
3. Select **Ethernet** or **Wi-Fi**
4. Click **Change adapter options**
5. Right-click the network connection → **Properties**
6. Select **Internet Protocol Version 4 (TCP/IPv4)**
7. Click **Properties**
8. Choose:

```

Use the following DNS server addresses

```

Enter:

```

Preferred DNS server: 192.168.135.5

```

Click **OK**.

---

# Verification

After configuration, test with:

```

ping [www.cim64.com](http://www.cim64.com)

```

If DNS is working correctly, the domain should resolve to:

```

192.168.135.9

```

---

# Summary

This guide demonstrated how to:

- Understand DNS client–server interactions
- Install **BIND9** on Ubuntu
- Configure **Forward Lookup Zones**
- Configure **Reverse Lookup Zones**
- Validate and restart the DNS server
- Configure Linux and Windows clients to use the DNS server

A properly configured DNS server enables **local domain resolution**, improves **network management**, and allows **centralized control of hostname mappings**.
```
