Here's a complete **useful-commands.md** file with all your requested commands, properly organized and explained:

```markdown
# 🔧 Useful Raspberry Pi Commands - Master Guide

A curated collection of the most practical commands for Raspberry Pi 4 - networking, Wi-Fi management, SSH, and system configuration.

## 📑 Command Index
- [Network Inspection](#network-inspection)
- [Internet Connectivity](#internet-connectivity)
- [Wi-Fi Management](#wi-fi-management)
- [System Configuration](#system-configuration)
- [SSH & Remote Access](#ssh--remote-access)
- [Quick Reference Card](#quick-reference-card)

---

## 🌐 Network Inspection

### `iwconfig` - Check Wireless Interface
```bash
iwconfig
```
**What it does:** Displays Wi-Fi adapter settings - SSID, frequency, signal quality, bit rate.  
**When to use:** Troubleshooting Wi-Fi connection or checking which network you're connected to.

**Example output:**
```
wlan0     IEEE 802.11  ESSID:"HomeNetwork"  
          Frequency:2.437 GHz (Channel 6)
          Bit Rate=65 Mb/s   Tx-Power=31 dBm
          Link Quality=70/70  Signal level=-40 dBm
```

### `ip neigh | grep "10.42"` - Find Local Devices
```bash
ip neigh | grep "10.42"
```
**What it does:** Shows ARP (Address Resolution Protocol) table - all devices on your local network, filtered by IPs starting with 10.42.  
**When to use:** Finding your Pi's IP or discovering other Pis on a 10.42.x.x subnet (common with USB tethering or some VPNs).

**Example output:**
```
10.42.0.1 dev wlan0 lladdr 00:11:22:33:44:55 REACHABLE
10.42.0.105 dev wlan0 lladdr aa:bb:cc:dd:ee:ff STALE
```

**Variations:**
```bash
# Show all devices on network (no filter)
ip neigh

# Filter for specific subnet
ip neigh | grep "192.168"
ip neigh | grep "10.0"
```

### `nmcli connection show` - List All Network Connections
```bash
nmcli connection show
```
**What it does:** Displays ALL saved network connections (Wi-Fi, Ethernet, VPNs) on your system.  
**When to use:** Before connecting to a Wi-Fi network - see exactly what name to use in the `up` command.

**Example output:**
```
NAME                UUID                                  TYPE      DEVICE
HomeWiFi            abc123...                             wifi      wlan0
OfficeGuest          def456...                             wifi      --
Ethernet             ghi789...                             ethernet  eth0
Starbucks WiFi       jkl012...                             wifi      --
```

**Detailed view:**
```bash
# Show active connection only
nmcli connection show --active

# Show specific connection details
nmcli connection show "HomeWiFi"
```

---

## 🌍 Internet Connectivity

### `ping -c 4 8.8.8.8` - Test Internet Connection
```bash
ping -c 4 8.8.8.8
```
**What it does:** Sends 4 packets to Google's DNS server (8.8.8.8) to test if you have internet access.  
**When to use:** First test when "Wi-Fi is connected but no internet" - checks beyond local network.

**Example output:**
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=15.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=15.1 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=117 time=14.9 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
```

**Interpret results:**
- ✅ **0% packet loss** = Good connection
- ⚠️ **50-100% loss** = Network issues
- ❌ **"Destination Host Unreachable"** = No route to internet

**Other ping targets:**
```bash
ping -c 4 google.com        # Tests DNS + internet
ping -c 4 1.1.1.1           # Cloudflare DNS (alternative)
ping -c 4 raspberrypi.com   # Tests Pi Foundation site
```

---

## 📶 Wi-Fi Management (nmcli - NetworkManager CLI)

### `sudo nmcli dev disconnect wlan0` - Disconnect Wi-Fi
```bash
sudo nmcli dev disconnect wlan0
```
**What it does:** Forces disconnection from current Wi-Fi network.  
**When to use:** Switching networks, troubleshooting stuck connections, or disabling Wi-Fi to use Ethernet.

**Alternative (no sudo needed often):**
```bash
nmcli device disconnect wlan0
```

**Verify disconnection:**
```bash
nmcli device status
# Should show wlan0 as "disconnected" or "unmanaged"
```

### `sudo nmcli device disconnect wlan0` (Yes, same command twice - common for emphasis)
*Note: Some tutorials show this with and without sudo. Usually `nmcli` works without sudo, but `sudo` ensures permission for all scenarios.*

### `sudo nmcli connection up "THE NAME OF . . ."` - Connect to Wi-Fi
```bash
sudo nmcli connection up "Your_Network_Name"
```
**What it does:** Connects to a specific saved Wi-Fi network by its NAME (from `nmcli connection show`).  
**When to use:** After disconnecting, to manually reconnect or switch networks.

**Steps to use properly:**
```bash
# Step 1: List all saved connections
nmcli connection show

# Step 2: Note the EXACT name (case-sensitive!)
# Example: "HomeWiFi", "Starbucks WiFi", "iPhone Hotspot"

# Step 3: Connect using that name
sudo nmcli connection up "HomeWiFi"

# Success output:
# Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
```

**Connect to NEW network (not saved):**
```bash
sudo nmcli device wifi connect "NetworkName" password "YourPassword"
```

### `nmcli connection show` - View Connections (covered above)

---

## ⚙️ System Configuration

### `sudo raspi-config` - Raspberry Pi Configuration Tool
```bash
sudo raspi-config
```
**⚠️ Note:** The command is `raspi-config` (NOT `rspi-config` - common typo!)

**What it does:** Opens the official Raspberry Pi configuration menu - adjust system settings without editing config files manually.

**Main menu options:**
```
1 System Options    ─  Change password, boot options, hostname
2 Display Options   ─  Resolution, overscan, screen blanking  
3 Interface Options ─  Enable/disable: Camera, SSH, SPI, I2C, Serial
4 Performance       ─  Overclock, GPU memory, fan control
5 Localisation      ─  Keyboard layout, timezone, Wi-Fi country
6 Advanced Options  ─  Expand filesystem, GL driver, audio config
8 Update            ─  Update raspi-config tool
```

**Most common uses:**
```bash
# First setup - ALWAYS run this:
sudo raspi-config
# Then: 1 System Options → S1 Change Password → Set secure password

# Enable SSH for remote access:
sudo raspi-config
# Then: 3 Interface Options → I2 SSH → Yes

# Expand filesystem to use full SD card:
sudo raspi-config
# Then: 6 Advanced Options → A1 Expand Filesystem
```

**Keyboard shortcut inside raspi-config:** Use arrow keys, Tab to navigate, Enter to select.

---

## 📡 Additional Useful Commands

### Check Wi-Fi signal strength
```bash
# Method 1: Using iwconfig
iwconfig wlan0 | grep -i quality

# Method 2: Using nmcli
nmcli device wifi list

# Method 3: Detailed signal info
cat /proc/net/wireless
```

### Complete network restart
```bash
# Restart NetworkManager service
sudo systemctl restart NetworkManager

# Or disable/enable Wi-Fi interface
sudo nmcli radio wifi off
sudo nmcli radio wifi on
```

### Show all IP addresses
```bash
hostname -I          # Just IPs (clean output)
ip a                # Detailed interface info
ifconfig            # Traditional (may need: sudo apt install net-tools)
```

### Forget/deleted saved Wi-Fi network
```bash
# Delete connection from saved list
sudo nmcli connection delete "NetworkName"

# Verify it's gone
nmcli connection show
```

---

## 🚪 SSH & Remote Access

### Connect via SSH (from another computer)
```bash
ssh pi@<IP_ADDRESS>
# Example: ssh pi@192.168.1.100
# Password: (the one you set with raspi-config)
```

### Copy files via SCP
```bash
# Copy FROM your computer TO Pi
scp /path/to/local/file.txt pi@192.168.1.100:/home/pi/

# Copy FROM Pi TO your computer  
scp pi@192.168.1.100:/home/pi/file.txt /path/to/local/

# Copy entire folder
scp -r /local/folder/ pi@192.168.1.100:/home/pi/
```

### Enable SSH without monitor (headless setup)
```bash
# On boot partition of SD card, create empty file named 'ssh'
# Then create 'wpa_supplicant.conf' with Wi-Fi credentials

# On first boot, SSH will be automatically enabled
```

---

## 📋 Quick Reference Card

| Task | Command |
|------|---------|
| **Show Wi-Fi status** | `iwconfig` |
| **List saved networks** | `nmcli connection show` |
| **Disconnect Wi-Fi** | `sudo nmcli dev disconnect wlan0` |
| **Connect to Wi-Fi** | `sudo nmcli connection up "NETWORK_NAME"` |
| **Test internet** | `ping -c 4 8.8.8.8` |
| **Find devices on 10.42.x.x** | `ip neigh \| grep "10.42"` |
| **Pi config menu** | `sudo raspi-config` |
| **Show IP address** | `hostname -I` |
| **Restart network** | `sudo systemctl restart NetworkManager` |
| **Scan for Wi-Fi networks** | `sudo nmcli device wifi list` |

---

## 💡 Pro Tips

1. **Tab completion:** Type part of command then press `Tab` - saves typing and prevents typos
2. **Command history:** Press `↑` (up arrow) to repeat previous commands
3. **Clear screen:** `clear` or `Ctrl+L`
4. **Cancel command:** `Ctrl+C` (when ping is running, etc.)
5. **Run command with sudo:** `sudo !!` (!! repeats last command)

### Common typos to avoid:
- ❌ `rspi-config` → ✅ `raspi-config`
- ❌ `nmcli devive` → ✅ `nmcli device`
- ❌ `iwcofnig` → ✅ `iwconfig`

---

## 🔗 Related Documentation
- [Network Setup Guide](../network-ssh-setup.md)
- [Raspberry Pi OS Details](../operating-systems/raspberry-pi-os.md)
- [Main README](../README.md)

---

**Found a missing command?** [Open an issue](https://github.com/yourusername/raspberry-pi-guide/issues) or submit a pull request!

*Last updated: May 2026*
```

This file includes:
- ✅ All your requested commands with detailed explanations
- ✅ Corrected `raspi-config` (fixed the typo `rspi-config`)
- ✅ Real-world examples and output samples
- ✅ When/why to use each command
- ✅ Quick reference card at the end
- ✅ Pro tips for Raspberry Pi beginners
