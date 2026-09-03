# Uncomplicated Firewall (UFW)

Uncomplicated Firewall (UFW) is a user-friendly command-line interface program designed to manage firewall rules on Linux systems. It acts as a frontend to the Linux kernel's netfilter firewall, simplifying the complex iptables commands into easily understandable syntax. UFW aims to make firewall management accessible, especially for users unfamiliar with detailed firewall configurations, while still providing enough flexibility for experienced administrators.

## Key Features of UFW Include:

- Simplified rule definition for allowing or denying traffic based on ports, protocols, IP addresses, or application profiles.
- Default secure policies that deny all incoming connections and allow all outgoing connections, providing a secure baseline.
- Support for both IPv4 and IPv6 traffic management.
- Preconfigured application profiles for common services to ease firewall rule creation.
- Easy rule management commands such as enabling/disabling the firewall, viewing rules, deleting specific rules, and resetting all rules.
- UFW integrates with system service management (systemd) for automatic startup and status monitoring.
- Logging capabilities to track firewall activity.
- It is well-suited for host-based firewalls and is the default firewall management tool on many Debian-based Linux distributions such as Ubuntu.

## Installing UFW

Update package lists:
```bash
apt update
```

Install UFW:
```bash
apt install ufw
```

Install ncat (useful for testing rules):
```bash
apt install ncat
```

Check UFW version:
```bash
ufw version
```
