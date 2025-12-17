# Introduction to Linux Network Connectivity

## Networking Fundamentals

### Definitions and Purpose

Network connectivity in Linux allows a system to communicate with other devices; locally, across bridges, or over the internet. Proper configuration ensures that systems can transmit and receive data reliably and securely, supporting both local and wide-area networking.

### Protocol Stack

Linux networking is based on the TCP/IP model:

* **Link Layer**; Physical or virtual interfaces such as Ethernet, Wi‑Fi, and loopback.
* **Internet Layer**; IP addressing (IPv4/IPv6) and routing decisions.
* **Transport Layer**; TCP/UDP session management.
* **Application Layer**; Services such as DNS, HTTP, and SSH.

Understanding these layers and their interactions is essential for network configuration, performance optimization, and troubleshooting.

---

## Network Interfaces and Addressing

### Network Interfaces

Each Linux system exposes one or more *network interfaces* (e.g., `eth0`, `wlan0`, `lo`). These interfaces represent logical or physical connections to networks. Virtual interfaces can include bridges, VLANs, and loopback interfaces.

### IP Addressing

Interfaces require IP addresses to participate in IP networks. IP addresses can be:

* **Static**; manually assigned for fixed addressing.
* **Dynamic**; automatically assigned via DHCP for flexible network participation.

Subnets are defined using **netmasks** (IPv4) or **prefix lengths** (IPv6), which determine the boundaries between network and host addresses.

---

## Tools for Network Configuration

### The iproute2 Suite

`iproute2` is the modern utility suite for Linux networking, providing commands to configure interfaces, assign addresses, manage routing, and control traffic. It communicates directly with the kernel networking stack using netlink.

Key commands include:

* `ip link`; inspect and manage network interfaces.
* `ip addr`; assign and display IP addresses.
* `ip route`; view and modify routing tables.
* `ss`; inspect socket states and connections.

### Legacy Tools

* `ifconfig`; historically used for interface configuration; now largely deprecated.
* `netstat`; used to display open sockets and routing tables; partially replaced by `ss`.

Focusing on `iproute2` provides a consistent and comprehensive approach to network configuration.

---

## Configuration Workflow

### Viewing Interfaces

Network interfaces and their addresses can be inspected using:

```bash
ip link show
ip addr show
```

### Assigning an IP Address

A static IP address can be assigned to an interface:

```bash
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip link set eth0 up
```

### Routing Configuration

Routing determines the path packets take to reach other networks. A default gateway is set with:

```bash
sudo ip route add default via 192.168.1.1
```

---

## Name Resolution (DNS)

DNS converts human-readable domain names into IP addresses. DNS servers are configured in `/etc/resolv.conf` or through network management services. Correct DNS configuration is essential for network services and applications.

---

## Troubleshooting Connectivity

Connectivity can be tested and diagnosed using several tools:

* **Ping**: checks basic IP reachability.

  ```bash
  ping -c 4 8.8.8.8
  ```
* **Traceroute**: maps the route packets take to a destination.
* **ss**: displays socket and port statistics.
* **tcpdump**: captures live packet data for detailed analysis.

Understanding the output of these tools helps identify misconfigurations, connectivity issues, or network bottlenecks.

---

## Security and Traffic Control

### Firewalls

Linux includes firewall subsystems such as `nftables` or legacy `iptables` to manage traffic policies. Rulesets can specify allowed and denied traffic at the kernel level.

### Network Services

Services like SSH, NTP, and DNS must be configured and properly restricted. Firewalls and network policies help enforce traffic control and secure network communication.

---

## Advanced Concepts

### Routing Internals

Linux uses routing tables to determine the next hop for outgoing packets. Multiple routing tables and policy-based routing can be implemented for complex network topologies.

### Virtual Networking

Virtual networking constructs (bridges, VLANs, and network namespaces) allow creation of sophisticated network architectures. These features enable segmentation, virtualization, and containerized environments.

---
