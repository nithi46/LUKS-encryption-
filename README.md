# LUKS Encryption Setup Guide

> A comprehensive step-by-step guide to perform LUKS (Linux Unified Key Setup) encryption on Linux systems for securing sensitive data.

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Encryption](https://img.shields.io/badge/Encryption-LUKS-blue?style=for-the-badge)
![Course](https://img.shields.io/badge/Course-Advanced%20Linux%20Administration-green?style=for-the-badge)

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Verification](#verification)
- [Reference Table](#reference-table)
- [Troubleshooting](#troubleshooting)

---

## Overview

This guide demonstrates how to implement LUKS encryption on a Linux system to protect sensitive data stored on disk partitions. It covers the complete process from installing necessary tools to adding encryption keys and mounting encrypted volumes.

**What You'll Learn:**
- Installing cryptsetup and parted packages
- Creating and encrypting disk partitions
- Mapping encrypted volumes
- Mounting encrypted filesystems
- Managing LUKS encryption keys

---

## Prerequisites

- Root or sudo access
- A Linux system (Ubuntu/Debian based)
- A secondary disk or partition to encrypt (e.g., `/dev/sdb`)
- Basic command-line knowledge

---

## Verification

### Check Mounted Partitions

```bash
# Display all block devices and mount points
lsblk

# Display mounted filesystems with sizes
df -h
```

### Check LUKS Slots

```bash
# View all LUKS key slots
cryptsetup luksDump /dev/sdb1

# View key slots summary
cryptsetup luksDump /dev/sdb1 | grep "Slot"
```

### Verify Key File

```bash
# Check key directory contents
ls -la /etc/luks-keys/

# Verify key file size (should be 32 bytes)
stat /etc/luks-keys/luks-key
```

### Test Access

```bash
# Navigate to encrypted mount point
cd /mnt/encrypted_CND

# Create test file
touch test.txt

# List contents
ls -la
```

---

## Reference Table

| Component | Actual Value | Purpose |
|-----------|--------------|---------|
| **Package 1** | `cryptsetup` | LUKS encryption tool |
| **Package 2** | `parted` | Disk partitioning tool |
| **Disk** | `/dev/sdb` | Secondary virtual disk |
| **Partition** | `/dev/sdb1` | Encrypted partition |
| **Mapper Name** | `crypt` | Device mapper identifier |
| **Mount Point** | `/mnt/encrypted_CND` | Filesystem mount location |
| **Key Directory** | `/etc/luks-keys` | Secure key storage location |
| **Key File** | `luks-key` | Generated key filename |
| **Key Size** | `32 bytes` | 256-bit encryption key |
| **Slot Number** | `0` | LUKS slot 0 (0-7 available) |

---

## Troubleshooting

### Issue: "Device /dev/sdb1 not found"

**Solution:**
```bash
# Check available devices
lsblk

# Create partition if missing
parted /dev/sdb -s mkpart primary 1 -1
```

### Issue: "No key available with this passphrase"

**Solution:**
```bash
# Try unlocking with key file instead
cryptsetup luksOpen /dev/sdb1 crypt --key-file=/etc/luks-keys/luks-key
```

### Issue: "Mount point does not exist"

**Solution:**
```bash
# Create mount directory
mkdir -p /mnt/encrypted_CND

# Then mount again
mount -v /dev/mapper/crypt /mnt/encrypted_CND
```

### Issue: "Device mapper already exists"

**Solution:**
```bash
# Remove existing mapper
cryptsetup luksClose crypt

# Then reopen
cryptsetup luksOpen /dev/sdb1 crypt
```

---

## Security Best Practices

✅ **DO:**
- Use strong passphrases (mix of uppercase, lowercase, numbers, special characters)
- Store key files in secure locations with restricted permissions
- Regularly backup LUKS headers
- Use different passphrases for different slots
- Document your encryption setup

❌ **DON'T:**
- Share passphrases or key files
- Use weak or simple passphrases
- Store keys in world-readable locations
- Keep unencrypted backups of sensitive data
- Forget your passphrase (it cannot be recovered)

---

## Additional Resources

- [LUKS Documentation](https://gitlab.com/cryptsetup/cryptsetup)
- [Linux Disk Encryption Guide](https://wiki.archlinux.org/title/Dm-crypt)
- [Cryptsetup Manual](https://man7.org/linux/man-pages/man8/cryptsetup.8.html)

---

## License

This guide is provided for educational purposes as part of Advanced Linux System Administration Course - Sprint 4: Hardening Linux Systems.

---

## Author Notes

This is a practical implementation guide for LUKS encryption. Always test in a non-production environment first.

**Last Updated:** 2026

---

**Made with ❤️ for Linux Security**
