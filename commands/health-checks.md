# RHEL Post-Recovery Health Checks

Use this checklist after recovering access to the RHEL VM. The objective is to verify identity, security state, resources, services, logs and network connectivity.

## 1. Confirm the logged-in account

```bash
whoami
```

Expected: the intended local account, for example `naveen`.

## 2. Check SELinux

```bash
getenforce
```

Record whether SELinux is `Enforcing`, `Permissive` or `Disabled`.

## 3. Check filesystem usage

```bash
df -h
```

Look for filesystems approaching capacity. Full filesystems can cause application and service failures.

## 4. Check memory

```bash
free -h
```

Review available memory and swap usage.

## 5. Check failed systemd units

```bash
systemctl --failed
```

Investigate any failed units rather than assuming the system is healthy.

## 6. Review boot-time errors

```bash
journalctl -p 3 -xb
```

This filters the current boot journal for error-priority messages.

## 7. Inspect network interfaces

```bash
ip a
```

Confirm that the expected interface exists and has an appropriate address.

## 8. Test network connectivity

```bash
ping -c 3 google.com
```

A successful ping tests basic IP connectivity plus DNS resolution. A failed result should be investigated rather than treated as proof of a complete network outage.

## Validation record

For a real troubleshooting report, record the date, command, relevant output and interpretation. Avoid copying secrets or sensitive host information into a public repository.
