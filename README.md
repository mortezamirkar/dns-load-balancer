# Local DNS Load Balancer

This project provides a simple and efficient way to set up a local DNS load balancer using `dnsdist` and Docker. It allows you to distribute DNS queries across multiple servers, improve performance with caching, and customize how DNS requests are handled.

## Table of Contents
- [Local DNS Load Balancer](#local-dns-load-balancer)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [What is DNS and a DNS Server?](#what-is-dns-and-a-dns-server)
    - [What is a DNS Load Balancer?](#what-is-a-dns-load-balancer)
  - [Installation](#installation)
  - [Configuration](#configuration)
    - [Adding DNS Servers](#adding-dns-servers)
    - [Setting a Server Policy](#setting-a-server-policy)
    - [Restricting Access](#restricting-access)
    - [Enabling Caching](#enabling-caching)
  - [Changing the DNS Server](#changing-the-dns-server)
    - [On a Single Device (e.g., Your Computer)](#on-a-single-device-eg-your-computer)
    - [On a Network (e.g., Router)](#on-a-network-eg-router)
    - [Testing the Change](#testing-the-change)
  - [Usage](#usage)
  - [Troubleshooting](#troubleshooting)
  - [License](#license)

## Introduction

### What is DNS and a DNS Server?

DNS stands for **Domain Name System**. It acts like the internet's phonebook, translating human-friendly domain names (e.g., `google.com`) into IP addresses (e.g., `142.250.147.100`) that computers use to communicate. Without DNS, you'd need to memorize IP addresses for every website you visit, which is impractical—especially since IP addresses can change due to factors like location or server updates. When you enter a domain name in your browser, your computer queries a DNS server, which responds with the correct IP address, enabling the connection.

### What is a DNS Load Balancer?

A **DNS load balancer** distributes DNS queries across multiple DNS servers based on a chosen policy. This improves speed, reliability, and redundancy. For example, it can send your queries to the fastest available server or rotate them across servers randomly. 

This project also supports **caching**, where resolved domain names and their IP addresses are stored temporarily. If you request the same domain again, the load balancer retrieves the answer from its cache instead of querying a server, reducing response time significantly.

## Installation

Follow these steps to set up the DNS load balancer on your system:

1. **Prerequisites:**
   - Install **Docker** and **Docker Compose** on your operating system. Refer to the official guides if needed:
     - [Docker Installation](https://docs.docker.com/engine/install/)
     - [Docker Compose Installation](https://docs.docker.com/compose/install/)

2. **Clone the Repository:**
   ```bash
   git clone https://github.com/mortezamirkar/dns-load-balancer.git
   ```

3. **Navigate to the Directory:**
   ```bash
   cd dns-load-balancer
   ```

4. **Start the Load Balancer:**
   ```bash
   docker-compose up -d
   ```
   This launches the DNS load balancer in the background using Docker Compose.

## Configuration

The load balancer is configured via the `dnsdist.conf` file in the repository. Below are the key options you can customize:

### Adding DNS Servers

Specify the DNS servers you want to use by adding their IP addresses:

```lua
newServer("178.22.122.100")
newServer("10.202.10.202")
newServer("10.202.10.102")
newServer("78.157.42.100")
newServer("78.157.42.101")
newServer("10.202.10.10")
newServer("10.202.10.11")
newServer("185.51.200.2")
newServer("185.55.225.25")
```

### Setting a Server Policy

Choose how the load balancer selects a server for queries. For example, to use the fastest available server:

```lua
setServerPolicy(firstAvailable)
```

Other options include:
- `roundrobin`: Rotates queries across servers.
- `leastOutstanding`: Sends queries to the server with the fewest pending requests.
See the [dnsdist documentation](https://dnsdist.org/reference/config.html#server-policies) for more policies.


### Restricting Access

Limit which clients can use the load balancer (optional):

```lua
setLocal("0.0.0.0")
```

This binds the service to all interfaces. Modify it (e.g., `setLocal("127.0.0.1")`) to restrict to specific IPs.

### Enabling Caching

Enable caching to store DNS responses and speed up repeated queries:

```lua
pc = newPacketCache(10000, {
    maxTTL = 900,               -- Maximum TTL: 15 minutes
    minTTL = 30,                -- Minimum TTL: 30 seconds
    temporaryFailureTTL = 10,   -- TTL for failure responses: 10 seconds
    staleTTL = 0,               -- Disable stale responses
    dontAge = false             -- Allow TTL to decrease over time
})
getPool(""):setCache(pc)
```

This sets a cache with up to 10,000 entries.

## Changing the DNS Server

To use this load balancer as your DNS server, you need to configure your system or network to point to it. Here’s how:

### On a Single Device (e.g., Your Computer)

1. **Find the Load Balancer’s IP:**
   - If running locally, use `127.0.0.1`.
   - If on a remote server, use that server’s IP address.

2. **Update DNS Settings:**
   - **Windows:**
     - Open `Control Panel > Network and Sharing Center > Change adapter settings`.
     - Right-click your network, select `Properties`.
     - Select `Internet Protocol Version 4 (TCP/IPv4)` and click `Properties`.
     - Choose "Use the following DNS server addresses" and enter the load balancer’s IP (e.g., `127.0.0.1`).
   - **Mac:**
     - Go to `System Preferences > Network`.
     - Select your network, click `Advanced > DNS`.
     - Add the load balancer’s IP (e.g., `127.0.0.1`) and remove others if desired.
   - **Linux:**
     - Edit `/etc/resolv.conf` (may vary by distro):
       ```bash
       sudo nano /etc/resolv.conf
       ```
       Add:
       ```
       nameserver 127.0.0.1
       ```
       Save and exit.

### On a Network (e.g., Router)

1. Log into your router’s admin panel (usually via `192.168.1.1` or similar).
2. Find the DNS settings (often under "LAN" or "DHCP Settings").
3. Set the primary DNS server to the IP address of the machine running the load balancer.
4. Save and restart the router if required.

### Testing the Change

Run this command to verify:
```bash
nslookup google.com
```
If configured correctly, you’ll see the response coming from your load balancer’s IP.

## Usage

Once running and configured, the load balancer handles DNS queries by:
- Distributing them across the servers listed in `dnsdist.conf`.
- Applying the chosen policy (e.g., `firstAvailable`).
- Using the cache for faster responses to repeated queries.

Point your devices or network to the load balancer’s IP to start using it.

## Troubleshooting

- **Check Logs:**
  ```bash
  docker logs dns-server
  ```
  Look for errors in the output.

- **Verify Configuration:**
  Ensure `dnsdist.conf` is valid:
  ```bash
  docker exec dns-server dnsdist --check-config
  ```

- **Test Connectivity:**
  Confirm the listed DNS servers are reachable from your system.

## License

This project is licensed under the [MIT License](LICENSE). *(Update this based on your actual license.)*
