# Troubleshooting Notes

## Incident

A local RHEL account could not be accessed because its password was forgotten. The goal was to restore access without reinstalling the operating system.

## Initial observation

From the normal RHEL environment, the following was attempted:

```bash
chroot /mnt/sysroot
```

Result:

```text
chroot: cannot change root directory to '/mnt/sysroot': No such file or directory
```

### Interpretation

The command was being executed in the wrong context. `/mnt/sysroot` is established by the RHEL rescue environment after it locates and mounts the installed operating system.

## Recovery path

1. Power off the VM.
2. Attach/connect the RHEL DVD ISO to the virtual CD/DVD device.
3. Enter the VMware firmware/UEFI boot manager.
4. Boot from the virtual SATA CD/DVD device.
5. Enter the RHEL rescue workflow.
6. Select **Continue** so the installed system is mounted.
7. Enter the rescue shell.
8. Use `chroot /mnt/sysroot` to operate inside the installed system.
9. Reset the local account password with `passwd`.
10. Exit the rescue environment and reboot.
11. Validate authentication and system health.

## SELinux observation

The captured RHEL rescue shell displayed a warning that the rescue shell would trigger an SELinux autorelabel on the subsequent boot. This is why the documented procedure does not add a generic `touch /.autorelabel` step unless the specific recovery situation requires it.

## Diagnostic principle

The important troubleshooting lesson is not simply memorizing `chroot`. It is understanding **where the command is being executed and what filesystem layout exists in that environment**.

A command should be interpreted in the context of:

- current root filesystem
- mounted filesystems
- boot environment
- available devices
- security context
- privileges

## Evidence strategy

Screenshots are retained where they demonstrate an important state transition or diagnostic observation. Duplicate screens and images containing unnecessary personal information should be excluded or cropped before publication.
