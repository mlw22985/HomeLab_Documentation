Markdown
# Lab 01: VirtualBox & Kali Linux Setup

## Objective
Establish an isolated virtual environment to safely practice Linux terminal navigation and network monitoring tools.

## Setup Details
* **Hypervisor:** [VirtualBox](https://www.virtualbox.org/wiki/Downloads) v7.2.16
* **Guest OS:** [Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines) (64-bit)
* **Network Adapter Mode:** NAT / Host-Only

## Steps Executed
1. Downloaded and verified the Kali Linux ISO image.
2. Allocated 4 vCPUs, 10GB RAM, and a 20GB dynamically allocated virtual disk.
3. Updated Linux package repositories:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
4. Practiced essential Linux terminal commands:
  * Checked IP configurations: `ip a`
  * Navigated filesystems: `pwd`, `ls -la`, `cd /etc/`

## Trouble Shooting
  **Issue:** Couldn't use mouse or keyboard to log in.
  **Solution** Close and Reopen.
