
# 🔐 Guide to Resetting Password in Ubuntu

This guide provides two practical and tested methods for resetting the password in Ubuntu. These methods are intended for users who do not have access to the current password but still have access to the bootloader or GRUB.

---

## 📘 Method 1: Using **Recovery Mode**

### 🔧 Step-by-Step Instructions:

1. **Restart your system.**
2. During boot, press the **Shift** key (for BIOS systems) or **Esc** key (for UEFI systems) repeatedly to bring up the **GRUB menu**.
3. From the list, select **Advanced options for Ubuntu**.
4. Choose a kernel entry that includes `(recovery mode)` and press Enter.
5. In the recovery menu, select:
   ```
   root - Drop to root shell prompt
   ```
6. You should now have access to the root shell. To remount the filesystem as writable, run:
   ```bash
   mount -o remount,rw /
   ```
7. To change the user password, run:
   ```bash
   passwd your_username
   ```
   > Replace `your_username` with your actual username.

8. After setting the new password, reboot the system:
   ```bash
   reboot
   ```

---

## 📘 Method 2: Editing GRUB Boot Entry Directly

If Recovery Mode is unavailable or the above method doesn't work, you can directly modify the GRUB boot entry.

### 🔧 Step-by-Step Instructions:

1. **Restart your system.**
2. In the GRUB menu, highlight the default Ubuntu boot entry, but **do not press Enter**.
3. Press the `e` key to enter the boot edit screen.
4. Locate the line that begins with `linux`, for example:
   ```bash
   linux /boot/vmlinuz-xxx root=UUID=xxxx ro quiet splash ---
   ```
5. Replace `ro quiet splash` with:
   ```bash
   rw init=/bin/bash
   ```
6. Press `Ctrl + X` or `F10` to boot with the modified command.
7. Once in the shell, change the password:
   ```bash
   passwd your_username
   ```
   To list user directories if you're unsure:
   ```bash
   ls /home
   ```

8. After resetting the password, reboot the system:
   ```bash
   exec /sbin/init
   ```
   Or, if that fails:
   ```bash
   reboot -f
   ```

---

## ⚠️ Important Security Notes

- These methods only work if you have physical access and can reach the GRUB bootloader.
- If your system uses **disk encryption (LUKS)** or **Secure Boot**, additional configuration may be required in BIOS/UEFI.
- This guide applies only to **local user accounts**, not for systems using LDAP or Active Directory.

---

> © 2025 – Guide by ChatGPT, suitable for GitHub or personal documentation.
