# Week 1 Summary (Feb 2–Feb 8, 2026)

## Goal for Week 1

Build strong fundamentals in Linux, networking, and CLI troubleshooting. Practice common "support" failure modes and document fixes with runbooks.

---

## What I Completed

### Hands-on Drills (labs/)

- **SSH Break/Fix (Drill 01)**  
  Basic SSH troubleshooting and recovery steps.

- **DNS Failure (Drill 02)**  
  DNS failure simulation and how to prove "DNS vs network vs service".

- **Firewall timeout drill (Drill 03)**  
  Practiced telling the difference between "refused" vs "timed out" and what each suggests (service down vs traffic dropped/blocked).

- **Routing / gateway failure (Drill 04)**  
  Broke the default route to simulate "network down". Verified that failures happen even when using raw IPs.

- **Disk-full failure (Drill 05)**  
  Filled disk space in a safe location and observed common errors like "No space left on device". Cleaned up and verified recovery.

- **Linux Basics (Lab 01)**  
  Core Linux navigation and basic commands practice.

### Runbooks (runbooks/)

- **cant-ssh.md**  
  A focused flow for "SSH not working", which is a common Cloud Support ticket type.

- **dns-issues.md**  
  Quick way to separate DNS failure vs network failure vs destination/site failure.

- **network-no-connectivity.md** (routing/connectivity focused)  
  Checks for "no connectivity" issues, including default route and basic network validation.

- **disk-full.md**  
  Steps to confirm disk usage, identify large directories/files, and recover by cleanup.

### Notes and Course Work (notes/)

- **Cisco NetAcad: Networking Basics**
  - Module 3: Wireless and Mobile Networks
  - Module 4: Build a Home Network

- **Linux Foundation: Intro to Linux**
  - Chapter 3 (Linux concepts + distributions)
  - Started Chapter 4 (boot process + system startup overview)

- **MIT Missing Semester**
  - Intro to the Shell
  - Command-line Environment (arguments, streams, env vars, return codes, signals)

- **CLI Troubleshooting Cheatsheet**  
  Quick command reference for common failure symptoms.

- **Git basics**  
  Git workflow basics for daily commits.

- **Linux troubleshooting**  
  Linux-first checks and common commands, which directly support drills/runbooks.

- **Networking troubleshooting**  
  Networking-first checks and command patterns, which directly support drills/runbooks.

---

## 5 Key Learnings (Week 1)

1. **Different failures "feel" different**  
   - *DNS issue:* names fail, but IP pings may work.
   - *Routing/gateway issue:* even IP traffic fails because there's no path out.
   - *Firewall drop:* traffic often "times out" instead of being refused.

2. **Start troubleshooting with the fastest "layer checks"**  
   - Network: `ip route`, then `ping 1.1.1.1`, then `dig`, then `curl`.
   - Disk: `df -h` first, then `du -sh` to find what's big.

3. **A default route matters a lot**  
   - If the default gateway is wrong or missing, the machine can still talk locally, but not reach the internet.

4. **Disk full breaks normal behavior**  
   - Basic writes fail, logs can't write, and package operations can fail. Cleaning up restores stability quickly.

5. **Runbooks are a real Cloud Support skill**  
   - A good runbook is "how I would troubleshoot on-call": symptoms, quick checks, fixes, and verification.

---

## Artifacts I Now Have in My Repo

- A growing set of **drills (labs)** showing I can simulate issues and recover.
- A matching set of **runbooks** showing I can communicate troubleshooting steps clearly.
- **Course notes** proving steady progress in networking + Linux + CLI skills.

---

## What I Plan to Improve in Week 2

- Do more Linux CLI reps.
- Add another troubleshooting drill and a matching runbook.
- Keep building the notes + runbooks so they stay practical and interview-ready.
