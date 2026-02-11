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

## Feb 11, 2026

## Chapter 5: Graphical interface

### Learning objectives

By the end of this chapter, I should be able to:
- Manage graphical interface sessions
- Perform basic operations using the graphical interface
- Change the graphical desktop to suit my needs

---

### Graphical desktop overview

**Two ways to use Linux:**

1. **Command Line Interface (CLI):**
   - Need to remember programs and commands
   - Often more efficient for repetitive tasks
   - Requires knowledge of command options

2. **Graphical User Interface (GUI):**
   - Quick and easy
   - Navigate through graphical icons and screens
   - Easier if you don't remember all details or do tasks rarely

**Distribution coverage:**  
Managing sessions using GUI for three main families:
- Red Hat (CentOS, Fedora)
- SUSE (openSUSE)
- Debian (Ubuntu, Mint)

**Note:** Course uses GNOME-based variants. If using KDE, XFCE, or other desktops, experience will vary slightly but not fundamentally different.

---

### X Window System

**Historical context:**

- Loading the graphical desktop is one of the final steps in the boot process
- Historically called the **X Windows System** (often just "X")
- Dates back to mid-1980s

**How it works:**

- **Display Manager:** Keeps track of displays and loads the X server
- **X server:** Provides graphical services to applications (X clients)
- Display manager handles graphical logins and starts desktop environment

**Modern replacement:**

- **Wayland:** Newer system gradually superseding X
- Default for Fedora, RHEL, and other recent distributions
- Looks similar to X for users, but quite different under the hood
- Addresses security and other deficiencies in X

---

### Desktop environment components

**Main components:**

1. **Session manager:** Starts and maintains components of graphical session
2. **Window manager:** Controls placement/movement of windows, title-bars, and controls

**Note:** These components are generally used together as a unit to provide a seamless desktop environment.

**Starting the graphical desktop manually:**

- If display manager doesn't start by default, can run `startx` from command line after logging in to text-mode console
- Or manually start display manager (`gdm`, `kdm`, `xdm`, etc.) from command line
  - This differs from `startx` as it projects a sign-in screen

---

### GUI startup and display managers

**Boot process:**

- Display manager starts at end of boot process
- Responsible for:
  - Starting graphics system
  - Logging in the user
  - Starting user's desktop environment
- Can often select from multiple desktop environments at login

**Common display managers:**

- **gdm:** Default display manager for GNOME
- **kdm:** Popular display manager associated with KDE

---

### GNOME desktop environment

**What is GNOME:**

- Popular desktop environment with easy-to-use GUI
- Bundled as default for most Linux distributions:
  - RHEL, Fedora, CentOS
  - SUSE Linux Enterprise
  - Ubuntu, Debian

**Characteristics:**

- Menu-based navigation
- Easy transition for Windows users
- Look and feel can vary across distributions, even when all using GNOME

**Other desktop alternatives:**

- **KDE:** Very important in Linux history, often used with SUSE/openSUSE
- **Unity:** Present on older Ubuntu (still based on GNOME)
- **XFCE**
- **LXDE**

**Course focus:** Restricting mostly to GNOME to keep things less complex.

---

### Customizing the desktop

#### Changing desktop background

**Two ways to change background:**

1. **Image wallpaper:** Choose from defaults or use custom picture
2. **Solid color:** Select color to display instead of image

**How to change:**

- Right-click anywhere on desktop
- Choose "Change Background"

**Theme changes:**

- Can also change desktop theme
- Theme changes look and feel of Linux system
- Defines appearance of application windows

---

#### gnome-tweaks utility

**Problem:**

- Default settings utility is limited in modern GNOME-based distributions
- Many settings users want to modify are not accessible through standard interface
- Quest for simplicity has made it difficult to adapt system to personal tastes

**Solution: gnome-tweaks**

- Standard utility that exposes many more setting options
- Permits easy installation of extensions by external parties
- Not always installed by default, but always available
- Older distributions used name `gnome-tweak-tool`

**How to run:**

- May need to hit `Alt-F2` and type the name
- May want to add to Favorites list

**Recent change:**

- Some recent distributions moved functionality to new tool: `gnome-extensions-app`

**Example use case:**

- Remap CapsLock key to act as additional Ctrl key
- Saves users who use Ctrl frequently (like emacs users) from pinkie strain

---

#### Changing themes

**What themes control:**

- Visual appearance of applications
  - Buttons, scroll bars, widgets, other graphical components

**How to change:**

- Method varies by distribution
- Many GNOME-based distributions: Run `gnome-tweaks`
- Some newer systems: Use `gnome-extensions-app`

**Getting additional themes:**

- Download and install from GNOME's Wiki website
- Beyond the default selection

---

### Session management

#### Logging in and out

**Note:** Evolution has brought distributions to a stage where it matters little which you choose—they're all rather similar.

---

#### Locking the screen

**Why lock:**

- Prevents others from accessing your session while away
- Does NOT suspend computer
- All applications and processes continue running

**Two ways to lock:**

1. **Graphical method:**
   - Click in upper-right corner of desktop
   - Click lock icon

2. **Keyboard shortcut:**
   - `SUPER-L` (or `SUPER-Escape`)
   - SUPER key = Windows key
   - Can be modified in keyboard settings

**To unlock:**

- Provide password again

---

#### Switching users

**Key concept:**

- Linux is a true multi-user operating system
- Multiple users can be simultaneously logged in

**Requirements:**

- Each person needs own user account and password
- Allows for:
  - Individualized settings
  - Home directories
  - Personal files
- Protects against accidental and malicious corruption

**How it works:**

- Users can take turns using machine
- Keep everyone's sessions alive
- Can be logged in simultaneously through network

---

#### Shutting down and restarting

**When system restart is required:**

- Part of certain major system updates
- Generally only when installing new Linux kernel

**Process:**

- Rather trivial on all current distributions
- Click settings (gear) or power icon
- Follow prompts
- Can also use `shutdown` command from command line (covered later)

**GNOME shutdown steps:**

1. Click Power or Gear icon in upper-right corner
2. Click Power Off, Restart, or Cancel
3. If no action, system shuts down in 60 seconds

**Important:**

- Operations ask for confirmation
- Many applications won't save data properly when terminated this way
- **Always save documents and data before restarting, shutting down, or logging out**

---

#### Suspending

**What is Suspend/Sleep Mode:**

- Saves current system state
- Allows quick session resume
- Uses very little power while sleeping

**How it works:**

- Keeps applications, desktop, etc. in system RAM
- Turns off all other hardware
- Shortens time for full system start-up
- Conserves battery power

**Note:** Modern Linux distributions boot so fast that time saved is often minor.

**How to suspend:**

- Process starts same as shutdown or lock screen
- Click Power icon and hold briefly, then release
- Click double line icon to suspend
- Some distributions (Ubuntu) may show separate Suspend icon

**To wake up:**

- Move mouse or press any keyboard button
- Screen will be locked
- Type password to resume

---

### Basic operations

#### Locating applications

**Where applications are found:**

- Applications menu (upper-left corner)
- Activities menu (upper-left corner)
- Some Ubuntu versions: Dash button (upper-left)
- KDE and some environments: Button in lower-left corner

**Initial install:**

- Usually comes with wide range of applications
- Software archives contain thousands of programs
- Default application usually already installed for most tasks
- Can always install more from repositories

**Example:**

- Firefox is popular default browser
- Alternatives available: Epiphany, Konqueror, Chromium
- Proprietary browsers: Opera, Chrome

**Organization:**

- Applications neatly organized in functional submenus

---

#### Default applications

**What are default applications:**

- Multiple applications can accomplish same task
- Example: Clicking web address in email launches default browser

**How to set:**

1. Enter Settings menu
2. Click "Default Applications" or "Details > Default Applications"
3. Exact list varies by what's installed on system

---

#### File manager (Nautilus)

**What is Nautilus:**

- File manager utility used to navigate filesystem
- Called "Files" in the interface
- Can locate files and launch them

**Behavior:**

- Click on file → runs if program, or launches associated application
- Familiar to users of other operating systems

**How to start:**

1. Click file cabinet icon
2. Usually found under Favorites or Accessories
3. Named "Files"

**What it displays:**

- Opens with Home directory
- Left panel shows commonly used directories:
  - Desktop
  - Documents
  - Downloads
  - Pictures
- Magnifying glass icon (top-right) for searching

**Accessing locations:**

- Home directory
- Desktop
- Documents
- Pictures
- Other Locations
- Network locations

---

#### Home directories

**Structure:**

- Every user has home directory
- Usually created under `/home`
- Named according to user (e.g., `/home/student`)

**Default behavior:**

- Files user saves placed in tree starting there
- Default directories created:
  - Documents
  - Desktop
  - Downloads

**When created:**

- During system installation, or
- When new user is added later

---

#### Viewing files

**View formats:**

- **Icons view:** `CTRL-1`
- **List view:** `CTRL-2`
- Switch via icons in top bar or keyboard shortcuts

**Sorting options:**

- By name, size, type, or modification date
- Click View → Arrange Items

**Hidden files:**

- Configuration files starting with dot (.)
- Usually hidden by default
- To show: View → Show Hidden Files or `CTRL-H`

**Customization:**

- Multiple window view options
- Zoom In/Zoom Out under View menu
- Facilitates drag and drop operations

---

#### Searching for files

**Basic search:**

1. Click Search in toolbar
2. Enter keyword in text box
3. System performs recursive search from current directory
4. Finds any file/directory containing keyword

**Keyboard shortcuts:**

- `CTRL-F`: Get to search text box
- `CTRL-F` again: Exit search
- `CTRL-L`: Get Location text box to type directory path

**Command line:**

- Type `nautilus` to open File Manager

**Refining search:**

- Use dropdown menus for additional filters
  - Location
  - File Type
- Click + button for multiple search criteria
- Click Reload to regenerate search

**Example:**

- Find PDF containing "Linux" in home directory:
  1. Navigate to Home
  2. Search "Linux"
  3. Click + for additional criterion
  4. Select File Type
  5. Choose PDF

---

#### Editing files

**Default text editor:**

- **gedit** in GNOME
- Simple yet powerful
- Features:
  - Spell-checking
  - Syntax highlighting
  - File listings
  - Statistics

**How to edit:**

- Double-click file on desktop or in Nautilus
- Opens with default text editor

**Uses:**

- Editing documents
- Quick notes
- Programming

**Note:** More about text editors in later chapter.

---

#### Removing files

**Delete behavior:**

- Deleted files automatically move to trash
- Location: `.local/share/Trash/files/` under user's home directory

**How to delete:**

1. Select files/directories to delete
2. Press `CTRL-Delete` or right-click
3. Select "Move to Trash"

**Delete Permanently option:**

- Bypasses trash folder
- May be visible all time or only in list mode

**To permanently delete:**

**Method 1:**
1. Right-click Trash in left panel
2. Select Empty Trash

**Method 2:**
- Select file/directory
- Press `Shift-Delete`

**Important warning:**

- Never delete Home directory
- Will erase all GNOME configuration files
- May prevent you from logging in
- Many personal system and program configurations stored there

---

### Chapter 5 summary

**Key concepts covered:**

- **GNOME:** Popular desktop environment and GUI running on Linux
- **gdm:** Default display manager for GNOME, presents login screen
- **Session management:**
  - Logging out kills all processes and returns to login screen
  - Linux enables switching between logged-in sessions
  - Suspending puts computer into sleep mode
- **Applications:**
  - Default application installed for each key task
  - Every user has home directory
  - Places menu accesses different parts of computer/network
- **File management:**
  - Nautilus provides three formats to view files
  - Most text editors in Accessories submenu
- **Customization:**
  - Each distribution has own desktop backgrounds
  - GNOME themes change application appearance

---
