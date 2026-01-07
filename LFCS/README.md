# Linux Foundation Certified System Administrator

_This exam is an online, proctored and performance-based exam._

## Exam details

* Duration: 2 hours
* You'll get access to two exam simulators before the exam to practice via Killer Shell! Check your Killer Shell dashboard!

## Resources

* Killer Coda scenarios: <https://killercoda.com/pawelpiwosz/course/linuxFundamentals>

## Creating a local Ubuntu server for practice

_Note that this can be done on any Linux distribution supporting `libvirt` and `virt-install`, but there's a million different ways of starting up a small lab environment!_

1. Create a new SSH key pair for cloud-init user 'ubuntu':

```bash
ssh-keygen -t ed25519 -C "ubuntu" -f ~/.ssh/ubuntu
cat ~/.ssh/ubuntu.pub
```

_Don't forget to paste the public key content to `user-data` file provided here in `cloud-init/user-data`!_

1. Download the Ubuntu Cloud image, i downloaded 24.04 LTS from here: <https://cloud-images.ubuntu.com/noble/>

```bash

1. Start the VM with `virt-install`:

```bash
virt-install --name ubuntu01 \
  --memory 2048 \
  --os-variant detect=on,name=ubuntunoble \
  --disk size=10,backing_store="$(pwd)/noble-server-cloudimg-amd64.img",bus=virtio \
  --cloud-init user-data="$(pwd)/user-data",meta-data="$(pwd)/meta-data",network-config="$(pwd)/network-config" \
  --network network=default,model=virtio \
  --graphics none
```

## Topics

<details>
  <summary>Operations Deployment (25%)</summary>

* Configure kernel parameters, persistent and non-persistent
* Diagnose, identify, manage, and troubleshoot processes and services
* Manage or schedule jobs for executing commands
* Search for, install, validate, and maintain software packages or repositories
* Recover from hardware, operating system, or filesystem failures
* Manage Virtual Machines (libvirt)
* Configure container engines, create and manage containers
* Create and enforce MAC using SELinux

## Configure kenrnel parameters, persistent and non-persistent

```bash
sysctl net.ipv4.ip_forward                          # Check current value
sysctl -n net.ipv4.ip_forward                       # Check current value (numeric only)
sysctl -w net.ipv4.ip_forward=1                     # Set non-persistent kernel parameter
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf  # Set persistent kernel parameter
sysctl -p                                           # Apply changes from sysctl.conf
```

## Diagnose, identify, manage, and troubleshoot processes and services

```bash
ps -aux
pgrep -a cron
nice -n 10 <command>
systemctl status <service_name>
systemctl start <service_name>
systemctl stop <service_name>
systemctl restart <service_name>
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service --state=disabled
systemctl cat ssh.service
```

## Manage or schedule jobs for executing commands

```bash
crontab -l
crontab -e
systemctl list-timers --all
```

## Search for, install, validate, and maintain software packages or repositories

```bash
apt update
apt upgrade
apt install <package_name>
dpkg -l | grep <package_name>
dpkg -i <package_file>.deb
```

## Recover from hardware, operating system, or filesystem failures

```bash
journalctl -p err # severity level er
journalctl -b     # logs from current boot
```

</details>

<details>
  <summary>Networking (25%)</summary>

* Configure IPv4 and IPv6 networking and hostname resolution
* Set and synchronize system time using time servers
* Monitor and troubleshoot networking
* Configure the OpenSSH server and client
* Configure packet filtering, port redirection, and NAT
* Configure static routing
* Configure bridge and bonding devices
* Implement reverse proxies and load balancers

</details>

<details>
  <summary>Storage (20%)</summary>

* Configure and manage LVM storage
* Manage and configure the virtual file system
* Create, manage, and troubleshoot filesystems
* Use remote filesystems and network block devices
* Configure and manage swap space
* Configure filesystem automounters
* Monitor storage performance

</details>

<details>
  <summary>Essential Commands (20%)</summary>

* Basic Git Operations
* Create, configure, and troubleshoot services
* Monitor and troubleshoot system performance and services
* Determine application and service specific constraints
* Troubleshoot diskspace issues
* Work with SSL certificates

</details>

<details>
  <summary>Users and Groups (10%)</summary>

* Create and manage local user and group accounts
* Manage personal and system-wide environment profiles
* Configure user resource limits
* Configure and manage ACLs
* Configure the system to use LDAP user and group accounts

</details>
