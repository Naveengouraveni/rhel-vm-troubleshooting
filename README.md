# RHEL 10 VM Troubleshooting & Password Recovery

A hands-on Linux system administration case study documenting the recovery and troubleshooting of a Red Hat Enterprise Linux 10 virtual machine running in VMware Workstation.

> **Project type:** Infrastructure / Linux System Administration  
> **Environment:** RHEL 10.2 x86_64 on VMware Workstation  
> **Primary scenario:** Recovering access to a local Linux account after a forgotten password  
> **Focus areas:** UEFI boot, rescue mode, filesystem mounting, `chroot`, password recovery, SELinux, system logs and post-recovery validation

---

## 1. Problem Statement

The RHEL virtual machine could not be accessed because the password for the local `naveen` account was unavailable. The objective was to recover access without reinstalling the operating system and then verify that the system remained operational.

The troubleshooting process also included an initial failed `chroot` attempt from the normal operating environment. This was used to establish why the RHEL rescue environment is required for this recovery workflow.

---

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Hypervisor | VMware Workstation |
| Guest OS | Red Hat Enterprise Linux 10.2, 64-bit |
| Firmware | UEFI |
| Secure Boot | Disabled / unavailable in the captured configuration |
| Memory | 4 GB |
| vCPU | 4 |
| Virtual Disk | 60 GB NVMe |
| Network | NAT |
| Recovery Media | RHEL 10.2 DVD ISO |

---

## 3. Troubleshooting Approach

```text
Forgotten Linux password
        |
        v
Normal RHEL login unavailable
        |
        v
Initial chroot attempt from normal system
        |
        v
FAIL: /mnt/sysroot does not exist
        |
        v
Boot VMware firmware / UEFI
        |
        v
Boot RHEL installation media
        |
        v
RHEL Rescue Environment
        |
        v
Installed system mounted at /mnt/sysroot
        |
        v
chroot /mnt/sysroot
        |
        v
Reset local account password
        |
        v
Exit + reboot
        |
        v
SELinux autorelabel during subsequent boot
        |
        v
Successful login + system health validation
```

---

## 4. Evidence & Screenshots

The screenshots are organized to show the troubleshooting sequence rather than simply listing commands.

### 4.1 VMware Environment

**VM configuration** — 4 GB RAM, 4 vCPUs, 60 GB virtual disk and NAT networking.

![VM configuration](screenshots/01-vm-configuration.png)

**RHEL ISO attached to the virtual CD/DVD drive.**

![RHEL ISO attached](screenshots/02-rhel-iso-attached.png)

### 4.2 Boot and Recovery

**VMware firmware boot option used to access the UEFI boot manager.**

![Power on to firmware](screenshots/03-power-on-to-firmware.png)

**UEFI Boot Manager showing the RHEL entry and virtual SATA CD-ROM drive.**

![UEFI Boot Manager](screenshots/04-uefi-boot-manager.png)

**RHEL rescue environment selection.**

![RHEL rescue environment](screenshots/05-rescue-environment.png)

**Rescue shell after the installed system was mounted under `/mnt/sysroot`.**

![RHEL rescue shell](screenshots/06-rescue-shell.png)

### 4.3 Troubleshooting Evidence

An initial attempt to run `chroot /mnt/sysroot` from the normal RHEL session failed because `/mnt/sysroot` was not present in the normal boot environment.

![Failed chroot attempt](screenshots/00-failed-chroot-normal-session.png)

This was an important diagnostic observation: `/mnt/sysroot` is created by the rescue workflow when the installed operating system is mounted.

### 4.4 Password Recovery

![Password reset](screenshots/07-password-reset.png)

### 4.5 Recovery Verification

![Successful recovery](screenshots/08-successful-recovery.png)

### 4.6 Post-Recovery Health Checks

![Post recovery health checks](screenshots/09-post-recovery-health-checks.png)

---

## 5. Recovery Procedure

### Step 1 — Attach the RHEL ISO

The RHEL 10.2 DVD ISO was attached to the VMware virtual CD/DVD drive and configured to connect at power-on.

### Step 2 — Enter UEFI Boot Manager

The VM was powered on to firmware and the virtual CD/DVD device was selected as the recovery boot source.

### Step 3 — Enter the RHEL Rescue Environment

From the RHEL installation environment, the rescue workflow was selected. The **Continue** option was used so the installer could locate and mount the installed RHEL system.

The rescue environment reported that the installed system had been mounted at:

```bash
/mnt/sysroot
```

### Step 4 — Enter the Installed System

From the rescue shell:

```bash
chroot /mnt/sysroot
```

This changes the apparent root directory for the shell from the rescue environment to the installed RHEL system.

### Step 5 — Reset the Local Account Password

```bash
passwd naveen
```

The password should be entered interactively and must never be committed to GitHub or included in screenshots.

### Step 6 — Exit and Reboot

```bash
exit
reboot
```

The RHEL rescue screen captured during this lab explicitly warned that the rescue shell would trigger an SELinux autorelabel on the subsequent boot. Therefore, this project does **not** treat `touch /.autorelabel` as a mandatory additional command for this specific RHEL 10 workflow.

---

## 6. Why the Initial `chroot` Attempt Failed

The following command was initially attempted from the normal RHEL session:

```bash
chroot /mnt/sysroot
```

It returned:

```text
chroot: cannot change root directory to '/mnt/sysroot': No such file or directory
```

The reason is that `/mnt/sysroot` is not normally the location of the installed system during a standard boot. It is a mount point established by the RHEL rescue environment.

This distinction is important when troubleshooting Linux recovery procedures: the same command can be valid in one execution context and invalid in another.

---

## 7. Post-Recovery Validation

After regaining access, the system should be validated rather than assuming that a successful login means the incident is completely resolved.

### Identity

```bash
whoami
```

### SELinux state

```bash
getenforce
```

### Disk capacity

```bash
df -h
```

### Memory

```bash
free -h
```

### Failed systemd units

```bash
systemctl --failed
```

### Boot-time errors

```bash
journalctl -p 3 -xb
```

### Network interfaces

```bash
ip a
```

### Connectivity test

```bash
ping -c 3 google.com
```

See [`commands/health-checks.md`](commands/health-checks.md) for the validation checklist.

---

## 8. Troubleshooting Lessons

### 1. Execution context matters

`chroot /mnt/sysroot` is meaningful in the RHEL rescue environment because that environment mounts the installed system there. Running the same command from the normal system produced a predictable failure.

### 2. Rescue media can recover an installed Linux system without reinstalling it

The recovery process uses the installation environment as a controlled administrative context while leaving the existing installation and data in place.

### 3. SELinux must be considered during recovery

RHEL's rescue workflow explicitly warns about SELinux autorelabeling after rescue-shell changes. Recovery procedures should account for security contexts rather than treating SELinux as an unrelated component.

### 4. Recovery should end with verification

Authentication recovery is only one part of the task. Identity, SELinux state, disk capacity, memory, systemd failures, logs and networking provide a basic post-recovery health picture.

### 5. Failed attempts are useful evidence

The failed normal-session `chroot` attempt was retained as part of the case study because it demonstrates diagnosis and correction rather than presenting only the successful path.

---

## 9. Security Notes

- Never commit passwords, password hashes, authentication tokens or private keys.
- Do not publish personal Windows paths if they reveal unnecessary account information.
- Crop screenshots when they contain unrelated personal data.
- Use a disposable lab VM for recovery practice whenever possible.
- Password recovery should only be performed on systems you own or are authorized to administer.

---

## 10. Project Outcome

The exercise demonstrates a complete infrastructure troubleshooting workflow:

- VMware virtual machine inspection
- UEFI boot management
- RHEL installation media usage
- Rescue environment operation
- Linux filesystem mounting concepts
- `chroot` and execution context
- Local account password recovery
- SELinux awareness
- systemd and journal inspection
- Network and resource validation

This project is intended as a practical demonstration of Linux administration and troubleshooting skills rather than as a generic command reference.
