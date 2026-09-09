# Secure Remote Access with Tailscale

Secure remote access solution for a self-hosted homelab using **Tailscale**, providing encrypted access to internal systems and services without exposing management ports directly to the public Internet.

## Overview

This project implements secure remote connectivity to my homelab using Tailscale.

Rather than exposing SSH, web management interfaces, or other administrative services through router port forwarding, Tailscale creates an encrypted private network between authorized devices.

The environment allows trusted computers to remotely access homelab systems while keeping administrative services inaccessible from the public Internet.

### Project Goals

- Provide secure remote access to homelab infrastructure
- Eliminate unnecessary Internet-facing management ports
- Use encrypted WireGuard-based communication
- Authenticate devices before granting network access
- Provide secure SSH access to Linux hosts
- Maintain access across changing public IP addresses
- Reduce dependence on traditional VPN server infrastructure
- Demonstrate Zero Trust networking concepts

---

## Architecture

```text
                         Internet
                            │
                            │
                    No Port Forwarding
                            │
                    ┌───────▼────────┐
                    │  Home Router   │
                    │   TP-Link      │
                    │  Archer BE550  │
                    └───────┬────────┘
                            │
                    Internal Network
                            │
              ┌─────────────┴─────────────┐
              │                           │
      ┌───────▼────────┐         ┌────────▼───────┐
      │ Raspberry Pi 5 │         │   Orange Pi    │
      │ Homelab Server │         │ Secondary Host │
      └────────────────┘         └────────────────┘

                   Tailscale Network
                         (Tailnet)
                             │
             ┌───────────────┼───────────────┐
             │               │               │
       ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐
       │  macOS    │   │  Windows  │   │   Linux   │
       │  Client   │   │  Client   │   │  Client   │
       └───────────┘   └───────────┘   └───────────┘
```

Tailscale creates a private encrypted overlay network between authorized devices.

Whenever possible, devices establish direct peer-to-peer connections without routing traffic through a traditional centralized VPN server.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Tailscale | Secure remote-access overlay network |
| WireGuard | Encrypted networking protocol used by Tailscale |
| Linux | Homelab server operating systems |
| Raspberry Pi | Primary homelab infrastructure host |
| Docker | Containerized services |
| SSH | Secure command-line administration |
| DNS | Internal hostname resolution |
| macOS | Remote administration client |
| Windows | Remote administration client |

---

## Security Design

The primary security goal of this project is to provide remote administrative access without unnecessarily exposing services to the Internet.

### Traditional Remote Access

A traditional configuration might expose services such as:

```text
Internet
   │
   ▼
Public IP
   │
   ▼
Port Forwarding
   │
   ├── TCP 22  → SSH Server
   ├── TCP 443 → Management Interface
   └── Other Management Services
```

This increases the externally accessible attack surface.

### Tailscale Approach

With Tailscale:

```text
Internet
   │
   ▼
Encrypted Tailscale Tunnel
   │
   ▼
Authenticated Device
   │
   ▼
Private Homelab Resource
```

Administrative services do not need to be directly exposed through the router.

---

## Installation

### Linux

Tailscale can be installed using the official installation script:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Start and enable the service:

```bash
sudo systemctl enable --now tailscaled
```

Authenticate the device:

```bash
sudo tailscale up
```

The command provides an authentication URL that can be opened in a browser.

---

## Verify Tailscale

Check the current node status:

```bash
tailscale status
```

Display the assigned Tailscale IPv4 address:

```bash
tailscale ip -4
```

Example:

```text
100.x.x.x
```

Tailscale addresses are part of the private Tailnet and are not publicly routable Internet addresses.

---

## Remote SSH Access

Once both devices are connected to the same Tailnet, a Linux host can be reached using its Tailscale address:

```bash
ssh username@100.x.x.x
```

When Tailscale DNS features are enabled, the hostname can also be used:

```bash
ssh username@hostname
```

This provides remote administration without requiring TCP port `22` to be forwarded through the Internet-facing router.

---

## Connectivity Testing

### Check Tailnet Membership

```bash
tailscale status
```

### Test Connectivity

```bash
tailscale ping hostname
```

or:

```bash
ping 100.x.x.x
```

### Test SSH

```bash
ssh username@hostname
```

### Check Publicly Listening Services

On a Linux host:

```bash
sudo ss -tulpn
```

This helps verify which services are listening locally.

The router can then be checked to confirm that administrative ports such as SSH are **not forwarded from the Internet**.

---

## DNS

Tailscale can provide DNS-based hostname resolution between devices on the Tailnet.

Instead of remembering addresses such as:

```text
100.x.x.x
```

hosts can be accessed using names such as:

```bash
ssh raspbi-claw
```

This simplifies administration while maintaining encrypted connectivity.

My homelab also maintains its own internal DNS infrastructure using:

- AdGuard Home
- Unbound
- Redundant DNS hosts

This provides local DNS filtering and recursive DNS resolution independently of the remote-access solution.

---

## Homelab Integration

Tailscale provides remote administrative access to systems hosting services such as:

- AdGuard Home
- Unbound
- Caddy
- Portainer
- Dozzle
- Homepage
- Open WebUI
- NetBox
- n8n
- Monitoring tools
- Other Docker-based services

Sensitive administrative interfaces can remain restricted to trusted networks rather than being exposed directly to the public Internet.

---

## Why Tailscale?

I selected Tailscale because it provides several advantages for a home infrastructure environment.

### No Traditional VPN Server Required

Traditional VPN deployments often require:

- VPN server configuration
- Public IP configuration
- Router port forwarding
- Firewall configuration
- Certificate or key distribution

Tailscale simplifies this by creating authenticated encrypted connections between devices.

### Dynamic Public IP Compatibility

Remote connectivity does not depend on manually connecting to the home's current public IP address.

This is particularly useful for residential Internet connections where the ISP may change the assigned public address.

### Reduced Attack Surface

Management services do not need to be exposed directly through router port forwarding.

### Encryption

Tailscale uses WireGuard to encrypt traffic between participating devices.

### Device-Based Access

Only authenticated devices that are members of the Tailnet can participate in the private network.

---

## Zero Trust Concepts

This project demonstrates several concepts associated with Zero Trust networking.

Instead of assuming that a device should be trusted simply because it exists on a particular network, access is based on authenticated identities and devices.

The general model becomes:

```text
Authenticate
     │
     ▼
Authorize
     │
     ▼
Establish Encrypted Connection
     │
     ▼
Access Resource
```

This differs from a traditional perimeter-based model where systems inside the LAN are automatically considered trusted.

---

## Firewall and Router Configuration

One of the objectives of this implementation is to minimize inbound Internet exposure.

The router does **not** need to forward administrative ports such as:

```text
22    SSH
5900  VNC
9443  Portainer
```

Remote administration takes place through the Tailscale network instead.

Publicly accessible services should be evaluated separately and exposed only when specifically required.

---

## Useful Commands

### Show connected Tailnet devices

```bash
tailscale status
```

### Display IPv4 address

```bash
tailscale ip -4
```

### Display IPv6 address

```bash
tailscale ip -6
```

### Test another Tailscale host

```bash
tailscale ping <hostname>
```

### Reauthenticate or change configuration

```bash
sudo tailscale up
```

### Disconnect the device

```bash
sudo tailscale down
```

### Check daemon status

```bash
systemctl status tailscaled
```

### View service logs

```bash
journalctl -u tailscaled
```

---

## Verification

The implementation was tested by performing the following checks:

- Confirmed each authorized system appeared in the Tailnet
- Verified devices received private Tailscale IP addresses
- Tested encrypted communication between devices
- Verified remote SSH connectivity
- Tested hostname-based access
- Confirmed remote access worked outside the home network
- Confirmed SSH did not require Internet-facing port forwarding
- Verified administrative services remained inaccessible directly from the public Internet

---

## Security Considerations

Tailscale significantly reduces the need for publicly exposed management services, but it does not replace normal system security practices.

Additional controls used or recommended include:

- SSH key authentication
- Strong account passwords
- Multi-factor authentication
- Automatic security updates
- Least-privilege access
- Host firewalls
- Container isolation
- Network segmentation
- Restricted administrative interfaces
- Regular log review
- Backup and recovery procedures

---

## Troubleshooting

### Device Does Not Appear in Tailnet

Check the Tailscale service:

```bash
systemctl status tailscaled
```

Then check:

```bash
tailscale status
```

---

### Restart Tailscale

```bash
sudo systemctl restart tailscaled
```

---

### View Logs

```bash
sudo journalctl -u tailscaled
```

---

### Check TUN Interface

On Linux:

```bash
ls -l /dev/net/tun
```

Tailscale normally uses the Linux TUN interface for networking.

Some specialized ARM Linux kernels may not include TUN support. This should be verified before deploying Tailscale on single-board computers using vendor-specific kernels.

---

## Challenges Encountered

One challenge encountered while testing Tailscale on ARM-based single-board computers was kernel feature availability.

For example, some vendor-provided Linux kernels may not compile the TUN driver:

```text
CONFIG_TUN
```

Attempting to load the module may therefore return an error similar to:

```text
modprobe: FATAL: Module tun not found
```

This demonstrates an important infrastructure lesson: application compatibility can depend not only on the Linux distribution but also on the features compiled into the underlying kernel.

Potential solutions include:

- Using a kernel with TUN support enabled
- Installing an alternative supported kernel
- Using an appropriate Tailscale userspace networking configuration
- Hosting the required Tailscale functionality on another Linux system

---

## Future Improvements

Planned improvements for this project include:

- Implement Tailscale ACLs / grants
- Apply least-privilege access policies
- Evaluate Tailscale SSH
- Configure subnet routing where appropriate
- Evaluate exit-node functionality
- Integrate additional homelab hosts
- Improve centralized logging
- Document device authorization and removal procedures
- Add network topology diagrams
- Add screenshots demonstrating successful remote connectivity

---

## Skills Demonstrated

This project demonstrates practical experience with:

- Zero Trust networking
- Secure remote access
- VPN technologies
- WireGuard
- Linux administration
- SSH
- Network troubleshooting
- DNS
- TCP/IP networking
- Firewall concepts
- Identity-based access control
- Network security
- ARM Linux
- Infrastructure documentation
- Homelab architecture

---

## Repository Structure

```text
secure-remote-access-tailscale/
├── README.md
├── LICENSE
├── docs/
│   ├── architecture.md
│   ├── installation.md
│   ├── security.md
│   └── troubleshooting.md
├── diagrams/
│   └── network-topology.png
└── screenshots/
    ├── tailscale-status.png
    └── remote-ssh-test.png
```

Sensitive information should always be sanitized before being committed to the repository.

Do **not** upload:

- Authentication keys
- API keys
- Private keys
- Tailscale auth keys
- Passwords
- Personal authentication URLs
- Public IP addresses when unnecessary
- Sensitive internal addressing
- Session tokens
- Unredacted administration screenshots

---

## Lessons Learned

This project reinforced the importance of minimizing externally exposed services.

Instead of solving remote administration by opening additional firewall ports, secure overlay networking can provide authenticated and encrypted access while reducing the externally accessible attack surface.

The project also provided practical experience troubleshooting Linux kernel capabilities on ARM-based systems and demonstrated how operating system, kernel, networking, and application layers interact when deploying secure infrastructure.

---

## License

This project is licensed under the MIT License.

See [LICENSE](LICENSE) for details.

## Author

**Andrew Ciomperlik**

Homelab, networking, cybersecurity, Linux, and cloud infrastructure portfolio project.


