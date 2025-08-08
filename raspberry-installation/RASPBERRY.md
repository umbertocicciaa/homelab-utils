# Raspberry Pi Setup Guide

A comprehensive guide for setting up Raspberry Pi with Raspberry Pi OS and configuring SSH for remote access in a homelab environment.

## 📋 Overview

This guide covers the complete process of setting up a Raspberry Pi from scratch, including:

- Flashing Raspberry Pi OS to an SD card
- Initial system configuration
- SSH setup for remote access
- Basic security hardening
- Network configuration
- Essential system updates

## 🛠️ Hardware Requirements

### Essential Components

- **Raspberry Pi 4** (4GB+ RAM recommended for homelab use)
- **MicroSD Card** (32GB+ recommended, Class 10 or better)
- **Power Supply** (Official Raspberry Pi USB-C power adapter)
- **Ethernet Cable** (for initial setup and stable connection)
- **MicroSD Card Reader** (for flashing the OS)

### Optional Components

- **Case with Fan** (for better cooling)
- **Heat Sinks** (for thermal management)
- **HDMI Cable** (for direct monitor access if needed)
- **USB Keyboard/Mouse** (for initial setup without SSH)

## 💻 Software Requirements

### On Your Computer

- **Raspberry Pi Imager** (official tool)
- **SSH Client** (built-in on macOS/Linux, PuTTY on Windows)
- **Text Editor** (for configuration files)

## 🚀 Installation Process

### Step 1: Download Raspberry Pi Imager

Download the official Raspberry Pi Imager from the official website:

- **Website**: [rpi.org/software](https://www.raspberrypi.org/software/)
- **Platforms**: Windows, macOS, Linux

```bash
# On macOS with Homebrew
brew install --cask raspberry-pi-imager

# On Ubuntu/Debian
sudo apt install rpi-imager

# Or download directly from the website
```

### Step 2: Prepare the SD Card

1. **Insert the microSD card** into your computer's card reader
1. **Launch Raspberry Pi Imager**
1. **Select the OS**: Choose "Raspberry Pi OS (64-bit)" for best performance

### Step 3: Configure Advanced Options

Before flashing, click the **gear icon** (⚙️) to access advanced options:

#### Enable SSH

- ✅ **Enable SSH**
- Choose: **Use password authentication** or **Allow public-key authentication only**
- Set a strong password for the `pi` user

#### Configure User Account

- ✅ **Set username and password**
- **Username**: `pi` (or your preferred username)
- **Password**: Create a strong password

#### Configure WiFi (Optional)

- ✅ **Configure wireless LAN**
- **SSID**: Your WiFi network name
- **Password**: Your WiFi password
- **Country**: Select your country code

#### Set Locale Settings

- ✅ **Set locale settings**
- **Time zone**: Your local timezone
- **Keyboard layout**: Your keyboard layout

### Step 4: Flash the SD Card

1. **Select the SD card** as the target
1. **Click "Write"** to begin flashing
1. **Wait for completion** (this may take 5-10 minutes)
1. **Safely eject** the SD card when done

### Step 5: First Boot

1. **Insert the SD card** into your Raspberry Pi
1. **Connect ethernet cable** (recommended for initial setup)
1. **Connect power** to boot the Pi
1. **Wait 2-3 minutes** for the initial boot and configuration

## 🔧 Initial Configuration

### Step 6: Find Your Raspberry Pi's IP Address

#### Method 1: Router Admin Panel

- Access your router's admin interface
- Look for connected devices
- Find the device named "raspberry" or with hostname you set

#### Method 2: Network Scan

```bash
# On macOS/Linux
nmap -sn 192.168.1.0/24

# Look for devices with "Raspberry Pi Foundation" in the output
```

#### Method 3: Using arp (if you know the MAC address)

```bash
arp -a | grep -i "b8:27:eb\|dc:a6:32\|e4:5f:01"
```

### Step 7: Connect via SSH

Once you have the IP address, connect via SSH:

```bash
# Replace IP_ADDRESS with your Pi's actual IP
ssh pi@IP_ADDRESS

# Example:
ssh pi@192.168.1.100
```

**First-time connection**:

- You'll see a security warning about the host's authenticity
- Type `yes` to continue
- Enter the password you set during imaging

## 🔐 Security Configuration

### Step 8: Change Default Password (if not done during imaging)

```bash
# Change the password for the current user
passwd

# Follow the prompts to set a new strong password
```

### Step 9: Update the System

```bash
# Update package lists
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Clean up
sudo apt autoremove -y
```

### Step 10: Configure SSH Security

#### Edit SSH Configuration

```bash
sudo nano /etc/ssh/sshd_config
```

**Recommended security settings**:

```bash
# Change default SSH port (optional but recommended)
Port 2222

# Disable root login
PermitRootLogin no

# Disable password authentication (after setting up SSH keys)
# PasswordAuthentication no

# Limit login attempts
MaxAuthTries 3

# Set login grace time
LoginGraceTime 60
```

#### Restart SSH Service

```bash
sudo systemctl restart ssh
```

### Step 11: Set Up SSH Key Authentication (Recommended)

#### On Your Local Computer

```bash
# Generate SSH key pair (if you don't have one)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Copy public key to Raspberry Pi
ssh-copy-id -p 2222 pi@IP_ADDRESS
```

#### Test Key Authentication

```bash
# Connect using the new port and key
ssh -p 2222 pi@IP_ADDRESS
```

#### Disable Password Authentication (Optional)

Once key authentication works:

```bash
sudo nano /etc/ssh/sshd_config
```

Set:

```bash
PasswordAuthentication no
```

```bash
sudo systemctl restart ssh
```

## 🌐 Network Configuration

### Step 12: Set Static IP Address (Recommended)

#### Edit DHCP Client Configuration

```bash
sudo nano /etc/dhcpcd.conf
```

#### Add Static IP Configuration

```bash
# Add at the end of the file
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8

# For WiFi (if using WiFi)
interface wlan0
static ip_address=192.168.1.101/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
```

#### Restart Networking

```bash
sudo systemctl restart dhcpcd
```

### Step 13: Configure Hostname

```bash
# Set a descriptive hostname
sudo hostnamectl set-hostname rpi-homelab-01

# Edit hosts file
sudo nano /etc/hosts
```

Update the hosts file:

```bash
127.0.0.1       localhost
127.0.1.1       rpi-homelab-01

# The following lines are desirable for IPv6 capable hosts
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
```

## 🔧 System Optimization

### Step 14: Configure Memory Split

For headless operation (no desktop), optimize memory allocation:

```bash
# Open config
sudo nano /boot/firmware/config.txt

# Add or modify
gpu_mem=16
```

### Step 15: Enable Additional Interfaces (if needed)

```bash
# Run configuration tool
sudo raspi-config
```

Navigate to **Interface Options** and enable:

- **SSH** (should already be enabled)
- **I2C** (for sensors)
- **SPI** (for displays/sensors)
- **Camera** (if using camera module)

### Step 16: Install Essential Packages

```bash
# Install useful packages for homelab
sudo apt install -y \
    htop \
    neofetch \
    git \
    curl \
    wget \
    vim \
    tmux \
    ufw \
    fail2ban
```

## 🛡️ Additional Security Hardening

### Step 17: Configure Firewall

```bash
# Enable UFW firewall
sudo ufw enable

# Allow SSH on custom port
sudo ufw allow 2222/tcp

# Check status
sudo ufw status
```

### Step 18: Configure Fail2Ban

```bash
# Create local jail configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit configuration
sudo nano /etc/fail2ban/jail.local
```

Find the SSH section and modify:

```bash
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

```bash
# Start and enable fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## ✅ Verification and Testing

### Step 19: System Health Check

```bash
# Check system information
neofetch

# Check memory usage
free -h

# Check disk usage
df -h

# Check temperature
vcgencmd measure_temp

# Check running services
systemctl --type=service --state=running
```

### Step 20: Network Connectivity Test

```bash
# Test internet connectivity
ping -c 4 google.com

# Test local network
ping -c 4 192.168.1.1

# Check listening ports
sudo netstat -tlnp
```

## 📋 Post-Installation Checklist

- [ ] System boots successfully
- [ ] SSH access works with key authentication
- [ ] Static IP is configured and working
- [ ] System is updated to latest packages
- [ ] Firewall is configured and active
- [ ] Fail2ban is protecting SSH
- [ ] System temperature is within normal range (<70°C)
- [ ] All required services are running

## 🔄 Maintenance Tasks

### Regular Updates

```bash
# Weekly update routine
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

### System Monitoring

```bash
# Check system logs
sudo journalctl -xe

# Monitor system resources
htop

# Check fail2ban status
sudo fail2ban-client status sshd
```

## 🚨 Troubleshooting

### Common Issues

#### Cannot Connect via SSH

1. **Check if SSH is enabled**:

   ```bash
   sudo systemctl status ssh
   ```

1. **Verify IP address**:

   ```bash
   hostname -I
   ```

1. **Check firewall rules**:

   ```bash
   sudo ufw status
   ```

#### Forgotten IP Address

1. **Connect monitor and keyboard directly**
1. **Check router's device list**
1. **Use network scanning tools**

#### Performance Issues

1. **Check temperature**:

   ```bash
   vcgencmd measure_temp
   ```

1. **Monitor resources**:

   ```bash
   htop
   ```

1. **Check for throttling**:

   ```bash
   vcgencmd get_throttled
   ```

## 📚 Next Steps

After completing this setup, your Raspberry Pi is ready for:

- [K3s Kubernetes Installation](../k3s-installation/K3S-INSTALLATION.md)
- Docker container deployment
- Home automation services
- Network monitoring tools
- Media servers

## 📖 Additional Resources

- [Raspberry Pi Official Documentation](https://www.raspberrypi.org/documentation/)
- [Raspberry Pi OS Documentation](https://www.raspberrypi.org/documentation/computers/os.html)
- [SSH Security Best Practices](https://www.ssh.com/academy/ssh/sshd_config)

## 📝 Notes

- Always use strong passwords and SSH keys
- Keep your system updated regularly
- Monitor system temperature, especially under load
- Consider using a UPS for critical homelab applications
- Regular backups of important configurations

---

**Last Updated**: August 2025