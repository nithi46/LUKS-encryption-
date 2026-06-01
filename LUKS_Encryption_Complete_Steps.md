# Complete LUKS Encryption Setup - Step by Step

## Step 1: Install the cryptsetup tool

```bash
apt-get install cryptsetup parted
```

**Explanation:**
- `cryptsetup` - The main encryption tool
- `parted` - For disk partitioning

---

## Step 2: Check the partitions available

```bash
# Check the virtual disk status
lsblk

# Setup new disk table on /dev/sdb
parted /dev/sdb -i mksdos primary

# Create new partition on /dev/sdb
parted /dev/sdb -s mkpart primary 1 -1

# List all partitions
parted /dev/sdb -l
```

**Expected Result:** You should see `/dev/sdb1` as the new partition created

---

## Step 3: Encrypt the partition

```bash
# Format the partition with LUKS encryption
# This will prompt you to enter a passphrase
cryptsetup -y -v luksFormat /dev/sdb1
```

**What to do:**
- When prompted, enter your passphrase (e.g., `MySecurePassword123`)
- Confirm the passphrase again
- Type `YES` (in uppercase) to confirm encryption

---

## Step 4: Map the partition

```bash
# Open the encrypted partition and create a device mapper
# The mapper name will be: /dev/mapper/crypt
cryptsetup -v luksOpen /dev/sdb1 crypt
```

**Explanation:**
- `/dev/sdb1` - The encrypted partition
- `crypt` - The name of the decrypted device mapper (you can use any name)

---

## Step 5: Format the partition

```bash
# Create a filesystem on the decrypted device
mkfs.ext4 /dev/mapper/crypt

# Create a directory where we will mount the encrypted partition
mkdir -p /mnt/encrypted_CND
```

**Explanation:**
- `mkfs.ext4` - Creates an ext4 filesystem
- `/dev/mapper/crypt` - The decrypted mapped device
- `/mnt/encrypted_CND` - Directory name (you can change "encrypted_CND" to any name)

---

## Step 6: Mount the partition

```bash
# Mount the encrypted device to the directory
mount -v /dev/mapper/crypt /mnt/encrypted_CND

# Remount with additional options if needed
mount -v -o remount /mnt/encrypted_CND
```

**Verification - Check if mounted:**
```bash
lsblk
# or
df -h
```

**Expected Output (like Image 2):**
```
sda            8:0    0  696G  0 disk
├─sda1         8:1    0  686G  0 part /
├─sdb          8:16   0  199M  0 disk
├─sdc1         8:33   0  199M  0 crypt /mnt/encrypted_CND
└─sdd1         8:49   0  195M  0 crypt
root@stack-VirtualBox:~#
```

---

## Step 7: Retrieve LUKS details

```bash
# Dump all LUKS information about the encrypted partition
cryptsetup luksDump /dev/sdb1
```

**This shows:**
- Encryption method
- Key slots
- Passphrase information
- UUID of the partition

---

## Step 8: Add a new LUKS key

### Step 8a: Create a directory to store the key

```bash
# Create the directory
mkdir /etc/luks-keys

# Set permissions (only root can access)
chmod 700 /etc/luks-keys
```

### Step 8b: Generate a random key file

```bash
# Create a random key file (32 bytes = 256 bits)
dd if=/dev/random of=/etc/luks-keys/luks-key bs=32 count=1
```

**Explanation:**
- `if=/dev/random` - Read from random device
- `of=/etc/luks-keys/luks-key` - Output file path
- `bs=32` - Block size of 32 bytes
- `count=1` - Generate only 1 block

### Step 8c: Add the key to LUKS

```bash
# Add the key file to LUKS slot 1
cryptsetup luksAddKey /dev/sdb1 /etc/luks-keys/luks-key -S 0
```

**Explanation:**
- `/dev/sdb1` - The encrypted partition
- `/etc/luks-keys/luks-key` - The key file you created
- `-S 0` - Add to slot 0 (slots are numbered 0-7)

## Complete Command Reference

### Quick Copy-Paste Sequence (One at a time)
# 1. Install packages
apt-get update
apt-get install -y cryptsetup parted

# 2. Check partitions
Lsblk

# 3. Setup disk and partition
parted /dev/sdb -s mklabel msdos
parted /dev/sdb -s mkpart primary 1% 100%

# 4. Encrypt partition
cryptsetup -y -v luksFormat /dev/sdb1
# When prompted: type YES and enter passphrase

# 5. Map encrypted device
cryptsetup -v luksOpen /dev/sdb1 crypt
# When prompted: enter passphrase

# 6. Format and create mount point
mkfs.ext4 /dev/mapper/crypt
mkdir -p /mnt/encrypted_CND

# 7. Mount and configure SELinux
mount -v /dev/mapper/crypt /mnt/encrypted_CND
apt-get install -y policycoreutils
restorecon -WRF /mnt/encrypted_CND
mount -v -o remount /mnt/encrypted_CND

# 8. View encryption details
cryptsetup luksDump /dev/sdb1

# 9. Create key storage and backup key
mkdir /etc/luks-keys
chmod 700 /etc/luks-keys
dd if=/dev/random of=/etc/luks-keys/luks-key bs=32 count=1
cryptsetup luksAddKey /dev/sdb1 /etc/luks-keys/luks-key -S 0

---
## Verification Checklist

# Verify all components
✓ Packages installed
cryptsetup --version
parted --version
✓ Partitions created
lsblk
parted /dev/sdb -l
✓ Device mapped
dmsetup ls
✓ Mounted correctly
mount | grep encrypted_CND
df -h
✓ SELinux context
ls -lZ /mnt/encrypted_CND
✓ LUKS details
cryptsetup luksDump /dev/sdb1
✓ Key file created
ls -la /etc/luks-keys/
stat /etc/luks-keys/luks-key
✓ LUKS slots filled
cryptsetup luksDump /dev/sdb1 | grep "Key Slot"

---


**Expected Output (like Image 3):**
```
root@stack-VirtualBox:~# mkdir /etc/luks-keys
root@stack-VirtualBox:~# dd if=/dev/random of=/etc/luks-keys/luks-key bs=32 count=1
1+0 records in
1+0 records out
32 bytes copied, 0.000457662 s, 69.9 kB/s
root@stack-VirtualBox:~#
```

---


