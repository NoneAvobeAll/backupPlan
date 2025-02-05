# **Server Disk Expansion Plan – Expanding `/` (Root Filesystem)**  

## **Objective**  
Expand the root (`/`) filesystem by adding a new disk while ensuring system stability and minimal downtime.  

---

## **Pre-Expansion Activities**  

### **1. Backup Strategy (Critical Step)**  
Ensure data safety before making changes.  

✅ **Application Backup** (Handled by our team)  
✅ **Database Backup** (Handled by our team)  
✅ **OS Backup** (Handled by the client)  

### **2. Verify Current Disk Space & Layout**  
Run the following commands to analyze disk usage and partitioning:  
```bash
df -h
lsblk
fdisk -l
mount | grep "on / "
```
- Confirm the current root filesystem type (ext4, XFS, etc.).  
- Identify whether `/` is on LVM or a standard partition.  

### **3. Check Active Services & Dependencies**  
- Identify critical services running on `/`:  
  ```bash
  systemctl list-units --type=service --state=running
  ```
- Inform stakeholders of any potential downtime.  

---

## **Disk Expansion Execution**  

### **Step 1: Add & Verify the New Disk**  
1. After attaching the new disk, confirm its detection:  
   ```bash
   lsblk
   dmesg | tail
   fdisk -l
   ```
   Assume the new disk is `/dev/sdb` (adjust based on your system).  

### **Step 2: Expanding `/` Based on Current Setup**  

#### **A. If Root (`/`) is on LVM (Recommended Method)**  
1. **Create a Physical Volume (PV) on the New Disk**  
   ```bash
   pvcreate /dev/sdb
   ```
2. **Extend the Volume Group (VG)**
   ```bash
   vgextend vg_root /dev/sdb
   ```
3. **Extend the Logical Volume (LV)**
   ```bash
   lvextend -l +100%FREE /dev/vg_root/lv_root
   ```
4. **Resize the Root Filesystem**  
   - **For ext4:**  
     ```bash
     resize2fs /dev/vg_root/lv_root
     ```
   - **For XFS:**  
     ```bash
     xfs_growfs /
     ```
5. **Verify Expansion**  
   ```bash
   df -h /
   ```

#### **B. If Root (`/`) is on a Standard Partition (Non-LVM)**  
⚠ **This method may require a reboot and booting into a live environment.**  

1. **Check Current Partitioning Scheme**  
   ```bash
   fdisk -l
   ```
2. **Expand the Root Partition**  
   - Use `fdisk`, `parted`, or `growpart`:  
     ```bash
     growpart /dev/sda 1  # Adjust partition number as needed
     ```
3. **Resize the Filesystem**  
   - **For ext4:**  
     ```bash
     resize2fs /dev/sda1
     ```
   - **For XFS:**  
     ```bash
     xfs_growfs /
     ```
4. **Reboot (If Required)**  
   ```bash
   reboot
   ```

---

## **Post-Expansion Verification**  

### **1. Validate Expansion**  
```bash
df -h /
lsblk
```

### **2. Check Application & Database Functionality**  
Restart necessary services and validate operations:  
```bash
systemctl restart <service>
```

### **3. Monitor Logs for Errors**  
```bash
dmesg | tail -50
journalctl -xe
```

---

## **Rollback Plan (In Case of Failure)**  
- If issues arise, boot into a **live environment** and restore from backup.  
- For LVM errors, revert changes using:  
  ```bash
  vgcfgrestore vg_root
  ```
- Run `fsck` if filesystem corruption occurs.  

---

## **Final Steps**  
✅ Update documentation on disk changes.  
✅ Notify stakeholders that expansion is complete.  
✅ Monitor disk performance over the next few days.  

---

### **Expected Outcome**  
- `/` (root) successfully expanded with additional disk space.  
- No service disruptions or data loss.  
