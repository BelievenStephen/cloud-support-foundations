# Linux Foundation "Intro to Linux" notes

## What this course is aiming to teach

- Build working knowledge of Linux basics: navigating distros, configs/GUI basics, basic CLI operations, and common Linux use cases.
- Course is self-paced, chapters build on each other, knowledge checks are for reinforcement, final exam requires 70%+.

---

## Feb 7, 2026

## Chapter 1: Course intro (key points)

- Use the class forum for peer discussion and troubleshooting help. Support system is for account/enrollment issues.
- Course uses formatting/color conventions to separate commands vs output vs filenames vs links.

---

## Chapter 2: The Linux Foundation + distribution families

### The Linux Foundation (what it is)

- The Linux Foundation is a major hub for open-source collaboration. It hosts/helps many large projects across infrastructure and software ecosystems, not just Linux.

### Three major Linux distribution families (and why they matter)

The course focuses on three distro families because most Linux systems you will run into in the real world map back to one of these.

#### 1) Red Hat family (RHEL, CentOS, Fedora, etc.)

- Fedora is often "upstream" testing for RHEL. CentOS has historically been close to RHEL, and CentOS Stream gets updates earlier than RHEL.
- Common package manager style: RPM-based, uses `dnf`.

#### 2) SUSE family (SLES, openSUSE)

- Similar relationship: enterprise (SLES) and community (openSUSE) are very close.
- Uses RPM-based `zypper` and includes YaST admin tooling.

#### 3) Debian family (Debian, Ubuntu, Mint, etc.)

- Debian is upstream for Ubuntu. Ubuntu is upstream for Mint and others. Debian emphasizes stability and has a huge software repository.
- Uses APT tooling (`apt`, `apt-get`, etc.). Ubuntu is common in cloud deployments.

### Big idea: "distribution-flexible"

- Most Linux concepts and procedures carry across distros. The main differences are usually package managers, software versions, and file locations. Desktop environment used in the course is GNOME.

---

## Chapter 3: Linux philosophy, history, community, and terms

### Context for the course

- Linux changes constantly, so exact screens/features may drift over time.
- Some repetition is intentional to reinforce key skills (example: safe use of `sudo`).
- Course avoids "holy wars" (tool preference debates).
- Man pages are a core habit. Use `man <topic>` when you need details.

### Linux history (high level)

- Linus Torvalds started the Linux kernel in 1991. In 1992 it moved to the GPL, which helped create the global dev community and distributions.
- Today Linux runs a large share of servers, most smartphones via Android, most public cloud workloads, and essentially all top supercomputers.

### Linux philosophy (how it's built/why it works)

- Inspired by UNIX (but not UNIX). Strong focus on open collaboration.
- Key model: "everything is a file" style thinking (processes/devices/sockets often represented like files), plus multiuser, multitasking, networking, and background services (daemons).

### Linux community (where help comes from)

- Help channels include forums, IRC, GitHub/GitLab projects, mailing lists, and events. Linux.com is called out as a major community portal.

### Lab: "What OSS products do you use?"

**Examples listed:**

- Android, Apache, major platforms/services relying on OSS, supercomputers for weather, embedded Linux devices, etc.

### Core terminology (memorize these)

- **Kernel:** layer between hardware and apps.
- **Distribution:** Linux OS bundle (kernel + userland + tools).
- **Boot loader:** starts the OS (e.g., GRUB).
- **Service/daemon:** background process (e.g., `httpd`, `sshd`).
- **Filesystem:** how files are stored/organized (ext4, XFS, etc.).
- **Desktop environment / X Window system:** GUI layer on Linux (GNOME, KDE, etc.).
- **Shell / command line:** text interface and interpreter (bash, zsh, etc.).

---

## Feb 8, 2026

## Chapter 3 (continued): Linux distributions

- A Linux distribution is more than the kernel. It includes the kernel plus tools for:
  - file operations
  - user management
  - package management (install/update software)
- Distros can use different kernel versions. Enterprise distros often pick older kernels for stability and backport improvements.
  - Example idea: RHEL 8 uses an older but stable kernel line, RHEL 9 uses a newer one.
- Distributions bundle "core ingredients" needed for a full system:
  - compilers (C/C++ / clang), debugger (gdb)
  - system libraries apps depend on
  - graphics stack + desktop environment (if a GUI is included)
  - update services + a baseline set of apps

### Commercial vs community support

- Large organizations often choose paid/support-backed distros:
  - Red Hat (RHEL), SUSE, Canonical (Ubuntu)
- Common "RHEL-like" options:
  - CentOS used to be the free RHEL alternative. After 2021, it shifted to CentOS Stream.
  - Newer RHEL-compatible alternatives: AlmaLinux and Rocky Linux
- "Binary compatible" (RHEL family) means packages often install/run across those variants.

### Chapter 3 key takeaways

- Linux is UNIX-like, multi-user, multitasking, with networking and background services ("daemons").
- Common terms: kernel, distribution, boot loader, service, filesystem, X Window system, desktop environment, command line.
- A distro = kernel + tools + package/update system + typical apps.

---

## Chapter 4: Linux basics and system startup (boot process)

### Why this matters

- Understanding boot steps helps troubleshooting (especially when a system will not start cleanly).

### High-level boot flow

Power on → BIOS/UEFI → boot loader (ex: GRUB) → kernel → initramfs → `/sbin/init` → login (text or GUI)

### BIOS/UEFI and the boot loader

- BIOS runs hardware checks (POST) and initializes basic devices.
- Control then passes to the boot loader:
  - On BIOS/MBR systems: boot loader starts from the MBR.
  - On UEFI systems: firmware uses the EFI partition and boot manager entries.
- Boot loader loads:
  - the kernel image
  - the initramfs (initial RAM filesystem with early drivers/tools)

### initramfs

- Used to load drivers and mount the real root filesystem.
- After root filesystem is mounted successfully, initramfs is released from memory and the system runs `/sbin/init`.

### `/sbin/init` and system startup

- `init` becomes the "parent" of most user-space processes and is responsible for:
  - starting services
  - keeping services running
  - clean shutdown/restart behavior
- Modern distros use systemd as the init system.
  - `/sbin/init` typically points to systemd now.

### Why systemd replaced older init methods

- Older SysVinit was sequential and slow. systemd uses more parallel startup.
- systemd uses simpler unit config files instead of large startup scripts.

### Basic systemd service commands (pattern)

**Start/stop/restart:**

```bash
sudo systemctl start|stop|restart <service>
```

**Enable/disable at boot:**

```bash
sudo systemctl enable|disable <service>
```

**Status:**

```bash
sudo systemctl status <service>
```

### Lab exercise

- Check Apache (`httpd`) status, stop/start it, verify with `systemctl status`.

---

## Feb 10, 2026

## Chapter 4 (continued): Linux filesystem basics

### Linux filesystems

**Concept analogy:**  
Libraries organize books and media into sections by subject, audience, type, and frequency of use. A filesystem applies the same concept to storing and organizing data in a human-usable form.

### Different types of filesystems supported by Linux

- **Conventional disk filesystems:** ext3, ext4, XFS, Btrfs, JFS, NTFS, vfat, exfat, etc.
- **Flash storage filesystems:** ubifs, jffs2, yaffs, etc.
- **Database filesystems**
- **Special purpose filesystems:** procfs, sysfs, tmpfs, squashfs, debugfs, fuse, etc.

---

### Partitions and filesystems

**Key definitions:**

- **Partition:** A dedicated subsection of physical storage media. Historically a physically contiguous portion of a hard disk. Today's storage can be more complex, but we still treat a partition as a fixed area.
- **Filesystem:** A method of storing and accessing files.

**Relationship:**  
A partition is a container in which a filesystem resides. In some cases, a filesystem can span multiple partitions using symbolic links.

### Comparison: Windows vs Linux

| Component | Windows | Linux |
|-----------|---------|-------|
| Partition | Disk1 | `/dev/sda1` |
| Filesystem type | NTFS/VFAT | EXT3/EXT4/XFS/BTRFS... |
| Mounting parameters | DriveLetter | MountPoint |
| Base folder (where OS is stored) | `C:\` | `/` |

---

### The Filesystem Hierarchy Standard (FHS)

- Linux systems organize files according to the **Filesystem Hierarchy Standard (FHS)**, maintained by the Linux Foundation.
- **Purpose:** Ensures users, admins, and developers can move between distributions without re-learning system organization.

**Key differences from Windows:**

- Linux uses `/` to separate paths (not `\`)
- No drive letters
- Multiple drives/partitions are mounted as directories in the single filesystem

**Removable media locations:**

- **Modern systems:** `/run/media/yourusername/disklabel`
- **Older distributions:** `/media`

**Example:**  
If username is `student` and USB drive is labeled `FEDORA`:
- Mount point: `/run/media/student/FEDORA`
- File path: `/run/media/student/FEDORA/README.txt`

---

### More about the Filesystem Hierarchy Standard

**Case sensitivity:**  
All Linux filesystem names are case-sensitive.
- `/boot`, `/Boot`, and `/BOOT` are three different directories.

**Organization pattern:**

- Core utilities needed for system operation are in root directories
- Other programs are in directories under `/usr` (think "user")
- Compare subdirectories under `/usr` with those directly under `/` to see the organization pattern

---

## Chapter 4 (continued): Linux distribution installation

### Choosing a Linux distribution

Determining which distribution to deploy requires thoughtful planning. Many embedded Linux systems use custom-crafted contents rather than Android or Yocto.

### Questions to ask when choosing a distribution

- What is the main function of the system (server or desktop)?
- What types of packages are important? (web server, word processing, etc.)
- How much storage space is required, and how much is available?
  - Example: Embedded devices usually have constrained space
- How often are packages updated?
- How long is the support cycle for each release?
  - Example: LTS releases have long-term support
- Do you need kernel customization from the vendor or a third party?
- What hardware are you running on? (x86, RISC-V, ARM, PPC, etc.)
- Do you need long-term stability? Or can you accept/need a cutting-edge system with latest software?

---

### Linux installation steps

#### 1. Planning

**Partition layout:**

- Best decided at installation time; difficult to change later
- Linux handles multiple partitions by mounting them at specific points in the filesystem
- Most installers provide reasonable defaults:
  - All space on one big partition + smaller swap partition, OR
  - Separate partitions for space-sensitive areas like `/home` and `/var`
- Override defaults if you have special needs or want to use more than one disk

---

#### 2. Software choices

**What's included:**

- All installations include bare minimum software for running Linux
- Options for adding categories:
  - Common applications (Firefox, LibreOffice)
  - Developer tools (vi, emacs text editors)
  - Popular services (Apache web server, MySQL database)
  - Desktop environment (GNOME, KDE) if using GUI

**Modern approach:**

- Quick basic install first
- Make software choices once system is running
- (Older method required many choices during installation, which was intimidating and time-consuming)

---

#### 3. Initial security features

**Basic setup:**

- Set password for superuser (root)
- Set up initial user account

**Distribution-specific approaches:**

- Some distros (Fedora, Ubuntu): Only initial user is set up
  - Direct root login not configured
  - Root access requires logging in as normal user, then using `sudo`
- Some install advanced security frameworks:
  - Red Hat-based (Fedora, CentOS): Use SELinux by default
  - Ubuntu: Uses AppArmor

---

#### 4. Install source

**Installation media options:**

- Removable media (USB drives, CDs, DVDs)
- Network boot: Boot small image, download rest over network
- Can perform install without any local media

**Automatic installation:**

- Uses configuration files to specify options
- File types by distribution family:
  - **Red Hat-based:** Kickstart file
  - **SUSE-based:** AutoYAST profile
  - **Debian-based:** Preseed file

---

#### 5. The installation process

**General flow:**

1. Boot from installation media
2. Installer asks questions about system setup
   - Questions skipped if using automatic installation file
3. Installation is performed
4. Computer reboots into newly-installed system
5. Additional configuration questions asked

**Updates during installation:**

- Most installers can download/install updates during installation (requires internet)
- Otherwise, system uses normal update mechanism after installation

---

#### 6. ⚠️ Important warning

**Caution:**  
The demonstration installations show how to install Linux directly on your machine, **erasing everything**. Following these procedures will erase all current data.

**Alternate installation methods** (from "Preparing Your Computer for Linux Training" document):

1. **Re-partition hard disk** for dual boot (side-by-side with current OS)
   - Sometimes complicated
   - Do when confidence is high and you understand the steps

2. **Use hypervisor program** (VMWare, Oracle VirtualBox)
   - Install Linux as a virtual machine
   - Safe method

3. **Boot from Live CD/USB**
   - Don't write to hard disk at all
   - Safe method

---

### Chapter 4 summary

**Key concepts covered:**

- A **partition** is a logical part of the disk
- A **filesystem** is a method of storing/finding files on a hard disk
- **Benefit of partitions:** When failure or mistake occurs, only data in affected partition is damaged; other partitions likely survive
- **Boot process steps:**
  1. BIOS triggers boot loader
  2. Boot loader starts Linux kernel
  3. initramfs filesystem is invoked
  4. init program completes startup process
- **Choosing a distribution:** Match your specific system needs to the capabilities of different distributions

---


