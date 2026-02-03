# Linux Basics Lab 01

## Goal
Practice basic Linux + networking troubleshooting commands over SSH.

## What I set up
- Ubuntu Server VM in UTM (ARM64)
- SSH enabled and accessible from macOS Terminal
- All commands were run on the VM, from an SSH session in macOS Terminal.
- VM hostname: `stephen-lab`
- VM IP (NAT): `192.168.64.2`
- Gateway: `192.168.64.1`

## Commands I ran
- `whoami`
- `hostname`
- `journalctl -xe | tail -n 50`
- `systemctl status ssh --no-pager`
- `ps aux | head`
- `df -h`
- `free -m`
- `ip route`
- `ss -tulpn`
- `curl -I https://example.com`
- `dig example.com`
- `traceroute example.com`

---

## Outputs + what they mean

### whoami

**Command:**
```bash
whoami
```

**Output:**
```
stephen
```

**Meaning:**  
Confirms which user you are currently logged in as.

---

### hostname

**Command:**
```bash
hostname
```

**Output:**
```
stephen-lab
```

**Meaning:**  
Confirms the server's hostname.

---

### journalctl -xe | tail -n 50

**Command:**
```bash
journalctl -xe | tail -n 50
```

**Output (example pattern from my run):**
```
... CRON[...] session opened for user root ...
... Starting sysstat-collect.service ...
... Deactivated successfully ...
... Finished sysstat-collect.service ...
```

**Meaning:**  
Shows recent system logs. CRON + sysstat entries are normal background activity if there are no errors.

---

### systemctl status ssh --no-pager

**Command:**
```bash
systemctl status ssh --no-pager
```

**Output (key lines from my run):**
```
Loaded: ... enabled
Active: active (running)
Server listening on 0.0.0.0 port 22.
Accepted password for stephen from 192.168.64.1 ...
```

**Meaning:**  
- SSH service is enabled and running.
- It is listening on port 22.
- It shows successful logins from the host side of the NAT network.

---

### ps aux | head

**Command:**
```bash
ps aux | head
```

**Meaning:**  
Quick look at running processes. Useful to confirm systemd is PID 1 and the system is functioning normally.

---

### df -h

**Command:**
```bash
df -h
```

**Output (key lines from my run):**
```
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  ...  / 
/dev/vda2                           2.0G ...  /boot
/dev/vda1                           1.1G ...  /boot/efi
```

**Meaning:**  
Shows disk usage by filesystem and mount point. Confirms root, boot, and EFI partitions are present and not full.

---

### free -m

**Command:**
```bash
free -m
```

**Output (from my run, example):**
```
Mem:  3902 total, 285 used, 3471 free, ... 3617 available
Swap: 3901 total, 0 used
```

**Meaning:**  
Shows RAM and swap usage. Confirms the VM has enough free memory and swap is not under pressure.

---

### ip route

**Command:**
```bash
ip route
```

**Output (from my run):**
```
default via 192.168.64.1 dev enp0s1 src 192.168.64.2
192.168.64.0/24 dev enp0s1 scope link src 192.168.64.2
```

**Meaning:**  
- The default route is how the VM reaches the internet.
- 192.168.64.1 is the NAT gateway.
- 192.168.64.2 is the VM's IP.

---

### ss -tulpn

**Command:**
```bash
ss -tulpn
```

**Output (key lines from my run):**
```
tcp LISTEN ... 0.0.0.0:22 ...
tcp LISTEN ... [::]:22 ...
udp UNCONN ... 127.0.0.53:53 ...
```

**Meaning:**  
- Confirms SSH is listening on port 22 (IPv4 + IPv6).
- 127.0.0.53:53 is the local DNS stub resolver (normal on Ubuntu).

---

### curl -I https://example.com

**Command:**
```bash
curl -I https://example.com
```

**Output (example from my run):**
```
HTTP/2 200
...
server: cloudflare
```

**Meaning:**  
Quick test that DNS + outbound HTTPS work.

---

### dig example.com

**Command:**
```bash
dig example.com
```

**Output (example from my run):**
```
status: NOERROR
example.com.  IN  A  104.18.26.120
example.com.  IN  A  104.18.27.120
SERVER: 127.0.0.53#53
```

**Meaning:**  
Confirms DNS resolution works and shows which resolver is being used.

---

### traceroute example.com

**Command:**
```bash
traceroute example.com
```

**Meaning:**  
Shows the network path (hops) from the VM to the destination. `*` hops are normal if some routers do not reply.

---

## Key observations (short)

- **hostname** returned: `stephen-lab`
- **Default route gateway:** `192.168.64.1` (UTM NAT)
- **VM IP:** `192.168.64.2`
- **SSH listening** on port 22 (`ss -tulpn` showed `0.0.0.0:22`)
- **curl -I https://example.com** succeeded (HTTP status returned)

---

## Issues I hit

SSH connection initially closed when SSH service was not running.

**Fixed by enabling and starting SSH:**
```bash
sudo systemctl enable --now ssh
```

---

## What I learned

- Confirm identity and host quickly with `whoami` and `hostname`.
- Use `systemctl status` to verify a service is running.
- Use `ss -tulpn` to verify ports are actually listening.
- Use `ip route` to confirm gateway and default route.
- Use `curl -I`, `dig`, and `traceroute` for fast network troubleshooting.
