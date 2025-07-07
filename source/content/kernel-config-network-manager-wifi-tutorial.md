---
{"publish":true,"title":"Kernel Configuration for Network Manager and WiFi Support in Yocto/Kirkstone","description":"Complete guide to configuring Linux kernel for Network Manager and WiFi support in Yocto Kirkstone builds","created":"2025-01-07","modified":"2025-07-07T21:02:53.138+02:00","tags":["kernel-config","wifi","network-manager","kirkstone","embedded"],"cssclasses":""}
---


# Kernel Configuration for Network Manager and WiFi Support in Yocto/Kirkstone

This tutorial provides a comprehensive guide for configuring the Linux kernel to support Network Manager and WiFi functionality in Yocto Kirkstone builds. Based on real-world implementation experience, this guide covers essential kernel configurations, testing procedures, and troubleshooting steps.

## Overview

When integrating Network Manager with WiFi support in embedded Linux systems, proper kernel configuration is crucial. This guide covers the essential kernel configuration options needed for:

- RFKILL (Radio Frequency Kill Switch) support
- USB WiFi adapter drivers
- Network Manager compatibility
- Proper hardware abstraction

## Essential Kernel Configuration Changes

### 1. RFKILL Configuration

RFKILL provides a standardized interface for controlling radio frequency devices like WiFi and Bluetooth adapters.

```bash
# Enable RFKILL core functionality
CONFIG_RFKILL=y

# Enable LED indication for RFKILL state
CONFIG_RFKILL_LEDS=y

# Enable input device support for RFKILL switches
CONFIG_RFKILL_INPUT=y
```

**Why these are needed:**
- `CONFIG_RFKILL=y`: Core RFKILL subsystem for managing RF devices
- `CONFIG_RFKILL_LEDS=y`: Visual indication of RF state through LEDs
- `CONFIG_RFKILL_INPUT=y`: Support for hardware RF kill switches

### 2. EEPROM Support

```bash
# Enable 93CX6 EEPROM support (used by many WiFi adapters)
CONFIG_EEPROM_93CX6=y
```

Many WiFi adapters store configuration data in 93CX6 EEPROMs, making this support essential.

### 3. USB Network Device Support

```bash
# CDC MBIM support for mobile broadband
CONFIG_USB_NET_CDC_MBIM=y

# QMI WWAN support for Qualcomm-based modems
CONFIG_USB_NET_QMI_WWAN=y
```

These configurations enable support for USB-based network devices, including mobile broadband modems.

### 4. MediaTek WiFi Driver Support

```bash
# MediaTek MT7601U USB WiFi adapter
CONFIG_MT7601U=m

# MediaTek MT76 driver core
CONFIG_MT76_CORE=m
CONFIG_MT76_LEDS=y
CONFIG_MT76_USB=m

# MT76x02 library and USB support
CONFIG_MT76x02_LIB=m
CONFIG_MT76x02_USB=m

# MT76 CONNAC library
CONFIG_MT76_CONNAC_LIB=m

# MT76x0 series support
CONFIG_MT76x0_COMMON=m
CONFIG_MT76x0U=m

# MT76x2 series support
CONFIG_MT76x2_COMMON=m
CONFIG_MT76x2U=m

# MT7615 series support
CONFIG_MT7615_COMMON=m
CONFIG_MT7663_USB_SDIO_COMMON=m
CONFIG_MT7663U=m
```

### 5. Additional USB WiFi Drivers

```bash
# ZyDAS ZD1201 USB WiFi
CONFIG_USB_ZD1201=m

# ZyDAS ZD1211 series
CONFIG_ZD1211RW=m
```

## Additional WiFi Driver Recipes

For comprehensive WiFi adapter support, you may need to add these Yocto recipes:

```
8812au_0.0.0.bb    # Realtek RTL8812AU
8814au_0.0.0.bb    # Realtek RTL8814AU
8821au_0.0.0.bb    # Realtek RTL8821AU
8821cu_0.0.0.bb    # Realtek RTL8821CU
8852bu_0.0.0.bb    # Realtek RTL8852BU
8852cu_0.0.0.bb    # Realtek RTL8852CU
88x2bu_0.0.0.bb    # Realtek RTL88x2BU
```

## Testing Procedures

### RFKILL Functionality Testing

#### Test 1: Device Detection
```bash
# List all RF devices
rfkill list

# Expected output example:
# 0: phy0: Wireless LAN
#     Soft blocked: no
#     Hard blocked: no
```

#### Test 2: Software Block/Unblock
```bash
# Block WiFi
rfkill block wifi

# Verify blocking
rfkill list

# Unblock WiFi
rfkill unblock wifi

# Verify unblocking
rfkill list
```

#### Test 3: LED Indication (Hardware Dependent)
If your hardware has RFKILL LEDs:
1. Toggle RFKILL state using `rfkill block/unblock`
2. Observe LED state changes
3. LED should reflect the current RF state

#### Test 4: Hardware Switch Testing
If your device has a physical RFKILL switch:
```bash
# Monitor kernel messages
dmesg -w

# Toggle the hardware switch
# Expected: Kernel messages showing RFKILL events
```

### Network Manager Integration Testing

#### Test 1: Service Status
```bash
# Check Network Manager status
systemctl status NetworkManager

# Expected: Active (running) status
```

#### Test 2: WiFi Interface Detection
```bash
# List network interfaces
nmcli device status

# Expected: WiFi interface listed (e.g., wlan0)
```

#### Test 3: WiFi Scanning
```bash
# Scan for available networks
nmcli device wifi list

# Expected: List of available WiFi networks
```

#### Test 4: Connection Testing
```bash
# Connect to a WiFi network
nmcli device wifi connect "SSID" password "PASSWORD"

# Verify connection
nmcli connection show --active
```

## Troubleshooting Common Issues

### Issue 1: WiFi Interface Not Detected

**Symptoms:**
- `nmcli device status` doesn't show WiFi interface
- `ip link` doesn't show wlan interface

**Solutions:**
1. Check if the WiFi driver is loaded:
   ```bash
   lsmod | grep -i wifi
   lsmod | grep mt76  # For MediaTek adapters
   ```

2. Check USB device detection:
   ```bash
   lsusb
   dmesg | grep -i usb
   ```

3. Verify kernel configuration includes the specific driver for your adapter

### Issue 2: RFKILL Blocking WiFi

**Symptoms:**
- WiFi interface exists but cannot scan or connect
- `rfkill list` shows "Hard blocked: yes" or "Soft blocked: yes"

**Solutions:**
```bash
# Unblock all RF devices
rfkill unblock all

# Or specifically unblock WiFi
rfkill unblock wifi
```

### Issue 3: Network Manager Not Managing WiFi

**Symptoms:**
- WiFi interface exists but Network Manager ignores it

**Solutions:**
1. Check Network Manager configuration:
   ```bash
   cat /etc/NetworkManager/NetworkManager.conf
   ```

2. Ensure WiFi is not disabled in Network Manager:
   ```bash
   nmcli radio wifi on
   ```

3. Restart Network Manager:
   ```bash
   systemctl restart NetworkManager
   ```

## Integration with Yocto Build

### Adding to Your Layer

1. **Kernel Configuration:**
   Add the kernel configurations to your kernel bbappend file:
   ```
   # In your-layer/recipes-kernel/linux/linux-%.bbappend
   FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
   SRC_URI += "file://wifi-support.cfg"
   ```

2. **WiFi Driver Recipes:**
   Place the additional WiFi driver recipes in:
   ```
   your-layer/recipes-kernel/wifi-drivers/
   ```

3. **Image Recipe:**
   Include Network Manager and WiFi tools in your image:
   ```bitbake
   IMAGE_INSTALL_append = " \
       networkmanager \
       networkmanager-nmcli \
       wireless-tools \
       wpa-supplicant \
       iw \
   "
   ```

## Best Practices

1. **Modular Approach:** Build WiFi drivers as modules (`=m`) rather than built-in (`=y`) for flexibility
2. **Testing Strategy:** Test each configuration change incrementally
3. **Hardware Validation:** Always verify hardware compatibility before kernel configuration
4. **Documentation:** Maintain a record of working configurations for different hardware platforms

## Conclusion

Proper kernel configuration is essential for Network Manager and WiFi functionality in embedded Linux systems. This guide provides the foundation for successful integration, but remember that specific hardware may require additional drivers or configuration adjustments.

The key to success is systematic testing and validation of each configuration component, ensuring that both the kernel-level support and user-space tools work together seamlessly.

## References

- [Linux Wireless Documentation](https://wireless.wiki.kernel.org/)
- [Network Manager Documentation](https://networkmanager.dev/)
- [Yocto Project Kernel Development Manual](https://docs.yoctoproject.org/kernel-dev/)
- [RFKILL Documentation](https://www.kernel.org/doc/Documentation/rfkill.txt)
