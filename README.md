
## **1. Preparation**

### 1.1. Verify Current Setup
- **Check Disk Layout and LVM Information:**  
  ```bash
  lsblk
  sudo pvs
  sudo vgs
  sudo lvs
  ```
  - Note which volume group (VG) and logical volume (LV) are used for the root filesystem (typically mounted at `/`).

### 1.2. Schedule Downtime/Maintenance Window
- Even though the process is largely online, extending the root LV (and filesystem) may require a maintenance window. Inform all stakeholders accordingly.

---

## **2. Adding the New Disk Partition**

### 2.1. Identify and Prepare the New Disk
- **Detect the New Disk:**  
  Once the new disk is physically attached or allocated via your hypervisor, confirm its presence:
  ```bash
  lsblk
  ```
  Assume the new disk is `/dev/sdX` (replace `sdX` with the actual identifier).

### 2.2. Partition the New Disk
- **Using `fdisk` (or `parted` for GPT):**
  1. Start `fdisk`:
     ```bash
     sudo fdisk /dev/sdX
     ```
  2. Create a new partition:
     - Press **`n`** to create a new partition.
     - Choose the default partition number.
     - Accept the default first and last sector values (or adjust as needed).
  3. Change the partition type to LVM (if required):
     - Press **`t`** then type the code for Linux LVM (typically **`8e`**).
  4. Write the changes:
     - Press **`w`** to save and exit.

- **Verify the new partition:**  
  ```bash
  lsblk
  ```
  You should now see something like `/dev/sdX1` as the new partition.

---

## **3. Integrating the New Partition into the Root Filesystem**

### 3.1. Create a Physical Volume (PV)
- **Initialize the new partition for LVM:**
  ```bash
  sudo pvcreate /dev/sdX1
  ```
- **Verify that the physical volume was created successfully:**
  ```bash
  sudo pvs
  ```

### 3.2. Extend the Volume Group (VG)
- **Add the new physical volume to your existing volume group:**  
  Identify the VG that holds your root LV (for example, `vg_root`).
  ```bash
  sudo vgextend vg_root /dev/sdX1
  ```
- **Verify the volume group extension:**
  ```bash
  sudo vgs
  ```

### 3.3. Extend the Logical Volume (LV) for Root
- **Identify the root LV:**  
  Typically it might be something like `/dev/vg_root/root` or `/dev/mapper/vg_root-root`.
- **Extend the LV to use the newly available space:**  
  To use 100% of the free space, run:
  ```bash
  sudo lvextend -l +100%FREE /dev/vg_root/root
  ```
  > **Tip:** You can also extend by a specific size (e.g., `-L +10G`) if you prefer.

### 3.4. Resize the Filesystem
- **Determine the Filesystem Type:**  
  Most root filesystems are either **ext4** or **XFS**.
- **For ext4 Filesystem:**
  ```bash
  sudo resize2fs /dev/vg_root/root
  ```
- **For XFS Filesystem:**
  ```bash
  sudo xfs_growfs /
  ```
  > **Note:** The XFS resize command uses the mount point (here `/`) rather than the device path.

---

## **4. Post-Expansion Verification**

### 4.1. Verify the Filesystem Size
- **Check the filesystem capacity:**
  ```bash
  df -h /
  ```
  Confirm that the new free space is now available on the root filesystem.

### 4.2. System Health and Log Check
- **Review system logs for any errors during the expansion:**
  ```bash
  sudo dmesg | tail
  sudo journalctl -xe
  ```
- **Optionally run a filesystem check:**  
  For ext4, you might schedule an `fsck` on the next reboot if desired.

### 4.3. Monitor Application and System Performance
- Ensure that the system continues to run normally.
- Monitor resource usage to catch any unexpected issues after the expansion.

---

## **5. Documentation & Communication**

### 5.1. Update System Documentation
- Record the following:
  - New disk and partition details (e.g., `/dev/sdX1`).
  - Updated volume group and logical volume sizes.
  - Filesystem type and resize commands used.
  - Any issues encountered during the process.

### 5.2. Stakeholder Notification
- Inform all relevant parties (clients, application owners, DB administrators) that the root partition expansion is complete.
- Provide a summary report and confirm that the system is fully operational.

---

## **6. Rollback and Contingency**

- **Rollback Plan:**  
  In case any critical issue is encountered:
  - Ensure that your backups (Application, DB, OS) are readily available.
  - Have a plan in place to restore the system from the OS backup (client-side) or from the application/DB backups.
  - Document and test rollback procedures in a non-production environment if possible.

---

