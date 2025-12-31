# Security Onion 2.4.90-2 Lab Setup on Proxmox

Complete documentation for deploying Security Onion as a VM on Proxmox with port mirroring for network traffic analysis.

## 📁 Contents

- **SecurityOnion_Lab_Writeup.md** - Complete step-by-step lab documentation (940 lines)
- **QUICK_REFERENCE.md** - Command cheat sheet for common tasks and troubleshooting
- **images/** - Screenshots and network diagrams (5 images included)

## 🚀 Quick Start

1. **Read the main writeup**: Start with `SecurityOnion_Lab_Writeup.md` for complete setup instructions
2. **Follow step-by-step**: The guide includes detailed explanations, visual guides, and troubleshooting
3. **Use quick reference**: Keep `QUICK_REFERENCE.md` handy for command lookups during setup

## 📋 What's Included

### Main Documentation (`SecurityOnion_Lab_Writeup.md`)
- Complete hardware requirements and specifications
- Network architecture with topology diagram
- Proxmox VM configuration (CPU, memory, network)
- Proxmox host network setup (bridges, promiscuous mode, offloading)
- Security Onion installation walkthrough
- Network interface configuration
- Comprehensive troubleshooting guide (8 common issues)
- Verification and testing procedures
- Maintenance and operations guide
- Complete configuration file examples

### Quick Reference (`QUICK_REFERENCE.md`)
- Proxmox host commands (network config, offloading, traffic verification)
- Security Onion VM commands (service management, network config, updates)
- Testing commands (generating test alerts)
- Critical configuration checklist
- Troubleshooting quick fixes
- File locations and web access URLs

### Images (`images/`)
- `home-network-diagram.png` - Complete network topology
- `Port-Mirroring.png` - NETGEAR switch port mirroring configuration
- `Port-Stats.png` - Switch port statistics verification
- `SecOnion-Alerts.png` - Security Onion alerts dashboard
- `SecOnion-PortMirror-TCPdumpCAP.png` - tcpdump output showing traffic capture

## ✅ Prerequisites

- **Proxmox VE 8.x** installed and running
- **Managed switch** with port mirroring capability (NETGEAR GS308EPP used in this lab)
- **Security Onion 2.4.90-2 ISO** downloaded from [securityonion.net](https://securityonion.net)
- **Minimum 16GB RAM** for VM (32GB+ recommended for production)
- **Two network interfaces** on Proxmox host (one for management, one for sniffing)
- **SSD storage** (200GB minimum, 500GB+ recommended)

## 🎯 Key Features Documented

- ✅ Port mirroring configuration on managed switch
- ✅ Proxmox bridge setup (Hub Mode with `bridge-ageing 0`)
- ✅ Checksum offloading fixes (VirtIO → Intel E1000)
- ✅ Network interface configuration (management + sniffing)
- ✅ Security Onion installation (EVAL mode for 16GB RAM)
- ✅ Alert generation and testing
- ✅ Log rotation and maintenance
- ✅ Complete troubleshooting workflow

## 📖 How to Use This Documentation

1. **First-time setup**: Read `SecurityOnion_Lab_Writeup.md` from start to finish
2. **During setup**: Keep `QUICK_REFERENCE.md` open for quick command lookups
3. **Troubleshooting**: Refer to the troubleshooting section in the main writeup
4. **Daily operations**: Use QUICK_REFERENCE for common maintenance commands

## 🔧 Common Use Cases

- **Initial Setup**: Follow the main writeup step-by-step
- **Quick Command Lookup**: Use QUICK_REFERENCE.md
- **Troubleshooting**: Check the troubleshooting section in the writeup
- **Verification**: Use the verification section to confirm everything works
- **Maintenance**: Refer to the maintenance section for updates and log management

## 📝 Notes

- The writeup includes detailed visual guides, so images are helpful but not required
- All configuration examples are based on a home lab environment
- IP addresses and network ranges can be adjusted for your environment
- The guide documents real troubleshooting scenarios encountered during setup

## 🆘 Support

- **Troubleshooting**: See the "Troubleshooting Guide" section in `SecurityOnion_Lab_Writeup.md`
- **Quick Fixes**: Check `QUICK_REFERENCE.md` for common issues
- **Configuration Files**: See the "Appendix" section in the main writeup for complete configs

---

**Ready to start?** Open `SecurityOnion_Lab_Writeup.md` and begin with the "Project Overview" section.

