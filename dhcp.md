````markdown
# Comprehensive Guide: DHCP Server and Client on Ubuntu

This guide explains how to configure a **DHCP (Dynamic Host Configuration Protocol) environment on Ubuntu**, including both **server-side configuration** and **client-side configuration**. DHCP automatically assigns IP addresses and network parameters to devices connected to a network.

---

# PART 1: SERVER-SIDE CONFIGURATION

DHCP dynamically assigns IP addresses and other communication parameters to client devices on a network.

## 1. Installation

On Ubuntu, the DHCP server package is **isc-dhcp-server**.

```bash
sudo apt-get update
sudo apt-get install isc-dhcp-server
````

---

## 2. Defining the Network Interface

The DHCP server must know which **network interface** to listen on.

### Find the Interface Name

```bash
ip a
```

Typical examples: `eth0`, `enp0s3`, `ens33`.

### Configure the Interface

Edit the DHCP server default configuration file:

```bash
sudo vi /etc/default/isc-dhcp-server
```

Find the following line and specify your interface:

```bash
INTERFACESv4="eth0"
```

---

## 3. Understanding the Configuration Keywords

The primary DHCP configuration file is:

```
/etc/dhcp/dhcpd.conf
```

Important configuration parameters:

| Keyword                      | Description                                     |
| ---------------------------- | ----------------------------------------------- |
| `option domain-name`         | Defines the network domain name                 |
| `option domain-name-servers` | Specifies DNS servers used by clients           |
| `default-lease-time`         | Default duration (seconds) a client keeps an IP |
| `max-lease-time`             | Maximum allowed lease time                      |
| `subnet`                     | Network ID being configured                     |
| `netmask`                    | Subnet mask for the network                     |
| `range`                      | IP address pool assigned to clients             |
| `option routers`             | Default gateway for clients                     |

---

## 4. Implementation Example

Example configuration with **two subnets and one static IP reservation**.

Open the DHCP configuration file:

```bash
sudo vi /etc/dhcp/dhcpd.conf
```

Add the following configuration:

```bash
# Global Settings
option domain-name "anwa.com";
option domain-name-servers 203.154.177.5, 203.154.177.6;

default-lease-time 300; # 5 minutes
max-lease-time 600;     # 10 minutes

# Network A Configuration
subnet 203.154.178.0 netmask 255.255.255.192 {
  range 203.154.178.10 203.154.178.20;
  option routers 203.154.178.1;
}

# Network B Configuration
subnet 203.154.179.0 netmask 255.255.255.224 {
  range 203.154.179.30 203.154.179.50;
  option routers 203.154.179.1;
}

# Specific IP Reservation (Static assignment)
host fanta {
  hardware ethernet 7c:05:07:91:67:3b;
  fixed-address 203.154.178.51;
}
```

---

## 5. Managing the DHCP Service

Restart the service after making configuration changes.

### Restart DHCP Server

```bash
sudo service isc-dhcp-server restart
```

### Check Service Status

```bash
sudo service isc-dhcp-server status
```

### Test Configuration Syntax

```bash
sudo dhcpd -t
```

This checks for syntax errors in the configuration file.

---

# PART 2: CLIENT-SIDE CONFIGURATION

Client machines must be configured to **automatically obtain an IP address from the DHCP server**.

Modern Ubuntu systems use **Netplan** for network configuration.

---

## 1. Configuring Netplan for DHCP

List Netplan configuration files:

```bash
ls /etc/netplan/
```

Example file names:

* `01-netcfg.yaml`
* `50-cloud-init.yaml`

Open the configuration file:

```bash
sudo vi /etc/netplan/01-netcfg.yaml
```

Example configuration enabling DHCP:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
```

Apply the new network configuration:

```bash
sudo netplan apply
```

---

## 2. Manual Client Commands (Testing)

Sometimes manual commands are needed to **release and request IP addresses**.

### Release Current IP Address

```bash
sudo dhclient -r
```

This informs the server that the client is releasing its IP.

### Request a New IP Address

```bash
sudo dhclient
```

The client broadcasts a DHCP request to obtain a new IP address.

### Verify Assigned IP Address

```bash
ip a
```

Check the interface to confirm that the client received an IP within the configured DHCP range.

---

# PART 3: CHECKING LOGS

If the client fails to receive an IP address, examine the DHCP server logs.

```bash
sudo tail -f /var/log/syslog | grep dhcpd
```

Expected DHCP message sequence:

1. **DHCPDISCOVER** – Client searches for a DHCP server
2. **DHCPOFFER** – Server offers an available IP address
3. **DHCPREQUEST** – Client requests the offered IP
4. **DHCPACK** – Server confirms and assigns the IP

Successful completion of these messages indicates that the DHCP handshake process is functioning correctly.

---

```
```
