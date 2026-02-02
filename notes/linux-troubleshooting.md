# Linux troubleshooting quick notes

## Logs (what happened and when)
- `journalctl -xe`
- `journalctl -u <service> --no-pager`
- `/var/log/`

## Services (is the service running)
- `systemctl status <service>`
- `systemctl restart <service>`
- `systemctl enable <service>`

## Processes (what is consuming resources)
- `ps aux | head`
- `top`
- `kill <pid>`

## Disk / Memory (is the system out of space or RAM)
- `df -h`
- `du -sh * | sort -h`
- `free -m`
