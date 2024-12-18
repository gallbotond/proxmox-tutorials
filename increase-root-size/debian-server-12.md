# How to Resize Disks on Debian-12 Server

This tutorial demonstrates how to resize a disk on a Debian-based system, specifically focusing on Logical Volume Manager (LVM). The process involves resizing partitions, physical volumes, and logical volumes to increase available space on your file system.

---

## **Step 1: Verify Current Disk Layout and Volume Sizes**

First, check your current partition layout, physical volumes, and logical volumes to understand the state of your storage.

### Commands:
```bash
pvs  # Displays physical volume information
df -h  # Displays file system disk space usage
```

Example Output of `pvs`:
```
PV         VG          Fmt  Attr PSize  PFree
/dev/sda5  debian12-vg lvm2 a--  <3.52g    0
```

This shows that `/dev/sda5` is part of the volume group `debian12-vg` with no free space available.

---

## **Step 2: Install Required Tools**

If not already installed, ensure `parted` is available to manage partitions.

### Command:
```bash
apt install parted
```

---

## **Step 3: Resize the Partition**

Use `parted` to resize the partition associated with the physical volume.

### Commands:
```bash
parted /dev/sda
```

Inside `parted`, execute:
1. **Print the Partition Layout**:
   ```bash
   print
   ```
2. **Resize the Extended Partition** (in this example, partition 2):
   ```bash
   resizepart 2 -1s
   ```
   `-1s` expands the partition to use all available disk space.
3. **Resize the Logical Partition** (in this example, partition 5):
   ```bash
   resizepart 5 -1s
   ```
   Similarly, this resizes the logical partition to the maximum available size.

Exit `parted`:
```bash
quit
```

Verify the changes:
```bash
pvs
```

---

## **Step 4: Resize the Physical Volume**

Resize the physical volume (PV) to use the newly extended partition space.

### Command:
```bash
pvresize /dev/sda5
```

Verify the updated size:
```bash
pvs
```

Example Output:
```
PV         VG          Fmt  Attr PSize  PFree
/dev/sda5  debian12-vg lvm2 a--  <5.52g  2.00g
```

---

## **Step 5: Resize the Logical Volume**

Expand the logical volume (LV) to use the free space in the volume group.

### Command:
```bash
lvresize -r -l +100%FREE /dev/mapper/debian12--vg-root
```

- `-r`: Automatically resizes the file system after resizing the logical volume.
- `-l +100%FREE`: Expands the logical volume to use all available free space.

Verify the updated file system:
```bash
df -h
```

Example Output:
```
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/debian12--vg-root  4.5G  1.6G  2.7G  38% /
```

---

## **Summary**

The disk has been successfully resized, and the new space is now available on the file system. The steps involved are:

1. Check current disk layout with `pvs` and `df -h`.
2. Install and use `parted` to resize the partition.
3. Use `pvresize` to extend the physical volume.
4. Use `lvresize` to extend the logical volume and file system.

### Important Notes:
- Ensure you have a backup before resizing partitions and volumes, as errors can lead to data loss.
- Replace `/dev/sda`, `/dev/sda5`, and `/dev/mapper/debian12--vg-root` with the appropriate device names for your setup.

Feel free to adapt this tutorial to your specific needs!