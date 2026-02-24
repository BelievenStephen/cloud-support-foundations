# Runbook: EC2 EBS Attach/Detach (Add Disk, Verify, Clean Up)

## Goal
Attach an additional EBS volume to an EC2 instance, verify it's accessible, and safely clean up afterward.

## When to Use This Runbook
- Need additional disk space (separate from root volume)
- Application requires dedicated storage (databases, logs, application data)
- Testing EBS attach/detach operations
- Learning EBS volume management

## Common Symptoms That May Lead Here
- Application errors: `No space left on device`
- Instance performance degraded, writes failing
- Logs not rotating due to disk space
- Data volume missing after reboot (mount not persistent)
- New EBS volume attached but not visible or mounted in OS

---

## Prerequisites Check

Before attaching a volume, verify:

**1) Instance Availability Zone:**
```bash
# From AWS CLI
aws ec2 describe-instances --instance-ids <INSTANCE_ID> --query 'Reservations[0].Instances[0].Placement.AvailabilityZone'

# OR check in Console
# EC2 → Instances → Select instance → Details tab → Availability Zone
```

**Critical:** EBS volume MUST be in the same AZ as the instance.

**2) Current disk usage (if addressing space issue):**
```bash
# Check filesystem usage
df -h

# Find large directories on root volume
sudo du -xhd1 / | sort -h

# Check specific paths
sudo du -xhd1 /var /home /opt 2>/dev/null | sort -h
```

**3) Existing block devices:**
```bash
lsblk
```

---

## Procedure: Attach New EBS Volume

### Step 1: Create the Volume

**EC2 Console → Volumes → Create volume**

**Configuration:**
- **Volume type:** gp3 (recommended for most use cases)
- **Size:** Based on requirements (minimum 1 GiB for lab)
- **Availability Zone:** MUST match instance AZ exactly
- **Encryption:** Enable if required by policy
- **Tags:** Add Name tag for identification

**Click:** Create volume

**Wait for state:** `available` (usually takes a few seconds)

---

### Step 2: Attach Volume to Instance

**EC2 Console → Volumes → Select volume → Actions → Attach volume**

**Configuration:**
- **Instance:** Select target instance from dropdown
- **Device name:** `/dev/sdf` (or next available, e.g., `/dev/sdg`, `/dev/sdh`)

**Click:** Attach

**Wait for state:** `in-use`

**Note:** Device name in console may differ from actual device name in OS. Use `lsblk` to find actual device.

---

### Step 3: Verify Attachment on Instance

**Check block devices:**
```bash
lsblk
```

**Expected output (example):**
```
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme0n1       259:0    0   8G  0 disk
└─nvme0n1p1   259:1    0   8G  0 part /
nvme1n1       259:2    0  10G  0 disk
```

**Key observations:**
- New device appears (e.g., `nvme1n1` on Nitro instances, `xvdf` on older instances)
- Size matches created volume
- No mountpoint yet (not mounted)

**Device naming note:**
- Console shows `/dev/sdf` but OS may show `/dev/xvdf` or `nvme1n1`
- Use `lsblk` output for actual device name in commands

---

## Optional: Format and Mount Volume

**⚠️ Warning:** Only perform these steps if you intend to use the volume. Skip for learning attach/detach only.

### Step 1: Create Filesystem

**Choose filesystem type:**
- **ext4:** General purpose, widely supported
- **xfs:** Better for large files, default on Amazon Linux

**Format the volume:**
```bash
# For ext4
sudo mkfs -t ext4 /dev/nvme1n1

# OR for xfs
sudo mkfs -t xfs /dev/nvme1n1
```

**⚠️ Critical:** Formatting erases all data. Verify device name is correct.

---

### Step 2: Create Mount Point
```bash
# Create directory for mount point
sudo mkdir -p /mnt/data

# OR choose different path based on use case
sudo mkdir -p /data
```

---

### Step 3: Mount Volume
```bash
sudo mount /dev/nvme1n1 /mnt/data
```

**Verify mount:**
```bash
df -h | grep /mnt/data
```

**Expected output:**
```
/dev/nvme1n1   10G   33M  10G   1% /mnt/data
```

**Test write access:**
```bash
sudo touch /mnt/data/test.txt
ls -l /mnt/data/
```

---

### Step 4: Make Mount Persistent (Optional)

**Only perform if volume should remain mounted after reboot.**

**Get volume UUID:**
```bash
sudo blkid /dev/nvme1n1
```

**Example output:**
```
/dev/nvme1n1: UUID="abc123-def456-..." TYPE="xfs"
```

**Edit /etc/fstab:**
```bash
sudo nano /etc/fstab
```

**Add entry (use UUID, not device name):**
```
UUID=abc123-def456-...  /mnt/data  xfs  defaults,nofail  0  2
```

**Breakdown:**
- `UUID=...` - Volume identifier (survives device name changes)
- `/mnt/data` - Mount point
- `xfs` - Filesystem type (change to `ext4` if you used ext4)
- `defaults,nofail` - Mount options (`nofail` prevents boot hang if volume missing)
- `0 2` - Dump and fsck order

**Test fstab before reboot:**
```bash
sudo mount -a
```

**If no errors, configuration is correct.**

---

## Detach and Clean Up

### Step 1: Unmount Volume (If Mounted)
```bash
# Unmount
sudo umount /mnt/data

# Verify unmounted
df -h | grep /mnt/data
# Should return nothing
```

**If "device is busy" error:**
```bash
# Find processes using the mount
sudo lsof +f -- /mnt/data

# OR
sudo fuser -mv /mnt/data

# Stop processes or change directory away from mount
cd ~
```

**Remove from /etc/fstab (if added):**
```bash
sudo nano /etc/fstab
# Comment out or delete the volume's entry
```

---

### Step 2: Detach Volume in Console

**EC2 Console → Volumes → Select volume → Actions → Detach volume**

**Confirmation dialog:** Click "Detach"

**Wait for state:** `available` (may take a few seconds)

**⚠️ Never use "Force detach" unless:**
- Normal detach has been stuck for >5 minutes
- You accept risk of data corruption
- You've confirmed volume is unmounted

---

### Step 3: Delete Volume (Cost Control)

**EC2 Console → Volumes → Select volume → Actions → Delete volume**

**Confirmation dialog:** Click "Delete"

**⚠️ Warning:** This is permanent. Ensure you don't need the data or have a snapshot backup.

---

## Verification Checklist

After each step, verify expected state:

| Step | Command/Check | Expected Result |
|------|--------------|-----------------|
| **Volume created** | Console: Volumes → State | `available` |
| **Volume attached** | Console: Volumes → State | `in-use` |
| **Device visible** | `lsblk` | New device appears with correct size |
| **Filesystem created** | `sudo blkid /dev/nvme1n1` | Shows UUID and TYPE |
| **Volume mounted** | `df -h \| grep /mnt/data` | Shows mount with correct size |
| **Write access works** | `sudo touch /mnt/data/test.txt` | File created successfully |
| **Mount persists** | Reboot, then `df -h` | Mount appears automatically |
| **Volume unmounted** | `df -h \| grep /mnt/data` | Returns nothing |
| **Volume detached** | Console: Volumes → State | `available` |
| **Volume deleted** | Console: Volumes list | Volume no longer listed |

---

## Troubleshooting

### Volume Won't Attach

**Error:** "The volume is not in the same availability zone as the instance"

**Cause:** Volume and instance in different AZs

**Fix:**
1. Delete the volume
2. Create new volume in correct AZ (match instance AZ exactly)
3. Retry attach

---

### Device Not Visible After Attach

**Check instance state:**
```bash
# Refresh device list
sudo partprobe

# Rescan for new devices
echo 1 | sudo tee /sys/block/*/device/rescan
```

**Check console:**
- Volume state should be `in-use`
- Attachment info should show instance ID

**Verify with dmesg:**
```bash
sudo dmesg | tail -20
```

---

### Unmount Fails: "Device is Busy"

**Find what's using the volume:**
```bash
sudo lsof +f -- /mnt/data
sudo fuser -mv /mnt/data
```

**Resolution:**
- Stop processes accessing the volume
- Change directory away from mount point
- Close files open on the volume

---

### Wrong Device Name in Commands

**Symptom:** `mkfs` or `mount` fails with "No such file or directory"

**Cause:** Device name in console differs from actual OS device name

**Fix:** Always use `lsblk` to find actual device name, not console device name

**Examples:**
- Console: `/dev/sdf` → OS: `/dev/xvdf` (older instances)
- Console: `/dev/sdf` → OS: `/dev/nvme1n1` (Nitro instances)

---

### Boot Hangs After Adding to fstab

**Cause:** Bad fstab entry or volume not available at boot

**Prevention:** Always use `nofail` option in fstab:
```
UUID=...  /mnt/data  xfs  defaults,nofail  0  2
```

**Recovery (if boot hangs):**
1. Use EC2 Serial Console or Instance Connect
2. Comment out problematic fstab entry
3. Reboot

---

## Cost Management

### EBS Volume Costs

**Critical cost facts:**
- **Volumes cost money even if instance is stopped**
- **Unattached (`available`) volumes still incur charges**
- **Snapshots cost money for storage (per GB-month)**
- **Charges accrue until volume is deleted**

**Cost optimization:**
1. Delete volumes when no longer needed
2. Check for `available` volumes regularly:
   - Console: EC2 → Volumes → Filter by State: `available`
3. Delete old snapshots not needed for recovery
4. Use gp3 instead of gp2 (lower cost for same performance)

---

### Before Ending Lab Session

**Quick cost check:**
```bash
# From AWS CLI
aws ec2 describe-volumes \
  --filters "Name=status,Values=available" \
  --query 'Volumes[*].[VolumeId,Size,VolumeType,State]' \
  --output table
```

**In Console:**
1. EC2 → Volumes
2. Check for any volumes in `available` state
3. Delete volumes not needed
4. Verify Snapshots page for any created snapshots

---

## Best Practices

**Volume management:**
- Always tag volumes with Name and Purpose
- Use gp3 as default volume type (better price/performance)
- Enable encryption for sensitive data
- Document mount points and fstab entries

**Operational safety:**
- Always unmount before detach
- Use `nofail` in fstab to prevent boot issues
- Test fstab with `sudo mount -a` before reboot
- Never force detach unless absolutely necessary

**Cost control:**
- Delete volumes immediately after lab/testing
- Set up billing alerts for EBS costs
- Regular audit of `available` volumes
- Use snapshots sparingly (they add up quickly)

---

## Quick Reference Commands
```bash
# Check disk usage
df -h
du -xhd1 / | sort -h

# List block devices
lsblk

# Create filesystem
sudo mkfs -t xfs /dev/nvme1n1     # xfs
sudo mkfs -t ext4 /dev/nvme1n1    # ext4

# Mount volume
sudo mount /dev/nvme1n1 /mnt/data

# Get UUID for fstab
sudo blkid /dev/nvme1n1

# Test fstab
sudo mount -a

# Unmount volume
sudo umount /mnt/data

# Find processes using mount
sudo lsof +f -- /mnt/data
sudo fuser -mv /mnt/data
```

---

## Key Takeaways

**Volume attachment:**
- Volume and instance MUST be in same AZ
- Device name in console may differ from OS device name
- Use `lsblk` to find actual device name

**Mount persistence:**
- Mount without fstab entry = lost after reboot
- Always use UUID in fstab, not device name
- Use `nofail` option to prevent boot issues

**Cost awareness:**
- Volumes cost money even when detached or instance stopped
- Delete unused volumes immediately
- Check for `available` volumes regularly

**Safe operations:**
- Always unmount before detach
- Never force detach unless normal detach stuck
- Test fstab changes before reboot
- Verify each step before proceeding to next
