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

## Feb 13, 2026

## Chapter 6: System configuration from the graphical interface

### Learning objectives

By the end of this chapter, I should be able to:
- Apply system, display, and date/time settings using System Settings panel
- Track network settings and manage connections using Network Manager
- Install and update software in Linux from graphical interface

**Note:** These tasks will be revisited later from the command line interface.

---

### System Settings

**What it controls:**
- Basic configuration options
- Desktop settings
- Screen resolution
- Network connections
- Date and time

**How to access:**
- Click upper right-hand corner
- Select tools image (screwdriver/wrench or gear icon)
- Exact menu layout varies by distribution and version

**Configuration options:**
- Click appropriate items to configure:
  - Display
  - Keyboard
  - Printers
  - Applications
  - Users (under Details) - set login picture, password, etc.

---

### gnome-tweaks

**Why it's needed:**
- Many personalized configuration settings don't appear in standard settings menus
- Older distributions: `gnome-tweak-tool`
- Launch with `Alt-F2` and type command name
- Some distributions hide this tool, making it hard to discover

**What you can do:**
- Select themes
- Configure extensions (from distribution or internet)
- Control fonts
- Modify keyboard layout
- Set programs that start at login

**Recent change:**
- Latest GNOME versions removed much functionality from gnome-tweaks
- Extensions now configured with `gnome-extensions-app`

---

### Display settings

**How to access:**
- Settings → Displays (or Settings → Devices → Displays)
- OR right-click anywhere on desktop → Display Settings

**What it controls:**
- Desktop appearance settings
- Functions independently of display drivers
- Appearance depends on: number of monitors, distribution, version

**Proprietary drivers:**
- Systems with nVidia/AMD cards may have separate config program
- Separate program may offer more options but is more complicated
- May require root access
- **Recommendation:** Use Displays panel when possible

**X Window configuration:**
- Config file: `/etc/X11/xorg.conf` (if it exists)
- Modern distributions usually only have this for unusual circumstances (uncommon graphic drivers)
- Direct editing is for advanced users

---

### Resolution and multiple screens

**Resolution configuration:**
- System usually auto-detects best resolution
- Can change via Displays panel
- Click Apply to test new resolution
- Must confirm resolution works
- Automatic timeout and revert if no confirmation

**Multiple displays:**
- Usually auto-configured as one big screen
- System makes reasonable guess for layout
- Options:
  - Mirrored mode (same display on all monitors)
  - Individual monitor configuration (resolution per monitor)
  - One big screen or mirrored video

---

### Date and time settings

**Internal timekeeping:**
- Linux always uses **Coordinated Universal Time (UTC)** internally
- Displayed/stored times use system time zone setting
- UTC is more accurate than Greenwich Mean Time (GMT)

**How to adjust:**
- Click time on top panel
- Adjust format for date/time display
- Some distributions allow value changes from top panel
- Detailed settings: System Settings Menu → Date & Time window

---

### Network Time Protocol (NTP)

**What it is:**
- Most popular/reliable protocol for setting local time
- Consults established Internet servers

**Setup:**
- All distributions come with working NTP setup
- References specific time servers (run/relied on by distribution)
- Generally only requires "on" or "off" setting
- No complex setup needed for network time synchronization

---

### Network Manager

**Purpose:**
- Makes network configuration easier and more uniform across distributions
- Alternative to hand-editing network config files

**What it can do:**
- List all available networks (wired and wireless)
- Choose wired, wireless, or mobile broadband network
- Handle passwords
- Set up Virtual Private Networks (VPNs)

**Recommendation:** Generally best to let Network Manager handle connections and settings (except unusual situations).

---

### Wired and wireless connections

**Wired connections:**
- Usually don't require complicated/manual configuration
- Hardware interface and signal automatically detected
- Network Manager sets network settings via **DHCP (Dynamic Host Configuration Protocol)**

**Static configurations:**
- Manual setup easily done through Network Manager
- Can change Ethernet **MAC address** (if hardware supports)
- MAC address = unique hexadecimal number of network card

---

### Configuring wireless connections

**Process (GNOME-based distributions):**

1. Click upper-right corner of top panel
2. Opens settings/network window
3. Click Wi-Fi submenu (if hardware present)
4. Select wireless network to connect to
5. For secure networks: Enter password on first connection
6. Password saved by default for subsequent connections

**Advanced configuration:**
- Click "Wi-Fi Settings" for more options
- Click Gear icon for any connection to configure in detail

---

### Mobile broadband and VPN connections

**Mobile broadband:**
- Set up via Network Manager wizard
- Wizard configures connection details
- Network auto-configured when broadband attached

**VPN support:**
- Network Manager manages VPN connections
- Supported technologies:
  - Native IPSec
  - Cisco OpenConnect (Cisco client or native open source client)
  - Microsoft PPTP
  - OpenVPN

**Note:** VPN support may be separate package from distributor. Install if preferred VPN not supported by default.

---

### Installing and updating software

**Package concept:**
- Each package provides one piece of the system
- Examples: Linux kernel, C compiler, text utilities, network config, browsers, email clients

**Dependencies:**
- Packages often depend on each other
- Example: Email client using SSL/TLS depends on encryption package
- Won't install unless dependency also installed

**Package management levels:**

1. **Lower-level utility:**
   - Handles unpacking package
   - Puts pieces in right places

2. **Higher-level utility (most common):**
   - Downloads and installs packages from Internet
   - Manages dependencies and groups
   - What you'll usually work with

---

### Debian packaging system

**Used by:** Debian, Ubuntu, and related distributions

**Lower-level: dpkg**
- Underlying package manager
- Can install, remove, and build packages
- Does NOT automatically download packages or satisfy dependencies

**Higher-level: APT (Advanced Package Tool)**
- System of utilities built on top of dpkg
- Each Debian family distribution creates own user interface:
  - `apt` and `apt-get`
  - synaptic
  - gnome-software
  - Ubuntu Software Center

**Repository notes:**
- APT repositories generally compatible with each other
- Software they contain generally is NOT compatible
- Most repositories target specific distribution (like Ubuntu)
- Distributors often ship multiple repositories for multiple distributions

---

### RPM packaging system

**Used by:** Red Hat, Fedora, CentOS, SUSE/openSUSE, Oracle Linux, others

**RPM (Red Hat Package Manager):**
- Developed by Red Hat
- Adopted by many distributions

**Higher-level package managers (varies by distribution):**
- **Red Hat family (RHEL/CentOS):** Historically used `yum`, now uses `dnf`
- **Fedora:** Uses `dnf`
- **SUSE family (openSUSE):** Uses `zypper` interface (still RPM-based)

---

### YaST Software Management (openSUSE)

**YaST (Yet another Setup Tool):**
- Similar to other graphical package managers
- RPM-based application
- Can add, remove, or update packages

**How to access:**

**Method 1:**
1. Click Activities
2. Search box: type "YaST"
3. Click YaST icon
4. Click Software Management

**Method 2:**
- Applications → Other-YaST (unusual location)

---

### Chapter 6 summary

**Key concepts covered:**

- **System Settings panel:** Controls basic configuration and desktop settings
- **Date and time:**
  - Linux uses UTC internally
  - Settings adjustable from System Settings window
- **Network Time Protocol:** Most popular/reliable for setting local time via Internet servers
- **Displays panel:** Change resolution and configure multiple screens
- **Network Manager:**
  - Present available wireless networks
  - Choose wireless or mobile broadband
  - Handle passwords
  - Set up VPNs
- **Package management:**
  - dpkg and RPM are most popular systems
  - Debian distributions: dpkg and apt-based utilities
  - RPM: Developed by Red Hat, adopted by openSUSE, CentOS, Oracle Linux, others

---

## Feb 15, 2026

## Chapter 7: Common applications

### Learning objectives

By the end of this chapter, I should be familiar with common Linux applications, including:
- Internet applications (browsers, email programs)
- Office productivity suites (LibreOffice)
- Developer tools (compilers, debuggers, etc.)
- Multimedia applications (audio and video)
- Graphics editors (GIMP and other graphics utilities)

---

### Internet applications overview

**Common network-aware applications:**
- Web browsers
- Email clients
- Streaming media applications
- Internet Relay Chats
- Conferencing software

---

### Web browsers

**Available browsers (graphical and text-based):**
- Firefox
- Google Chrome
- Chromium
- Epiphany (renamed "web")
- Konqueror
- linx, lynx, w3m (text-based)
- Opera

---

### Email applications

**What they do:**
- Send, receive, and read messages over the Internet
- Many users use browsers to access email accounts

**Protocols:**
- **IMAP (Internet Message Access Protocol):** Modern standard
- **POP (Post Office Protocol):** Older protocol
- Both access emails stored on remote mail server

**Features:**
- Display HTML-formatted emails (pictures, hyperlinks)
- Import address books/contact lists
- Import configuration info and emails from other applications

**Available email clients:**
- **Graphical:** Thunderbird, Evolution, Claws Mail
- **Text mode:** Mutt, mail
- **Web-based:** Gmail, Yahoo Mail, Office 365

---

### Other Internet applications

**FileZilla:**
- Intuitive graphical FTP client
- Supports: FTP, SFTP (Secure File Transfer Protocol), FTPS (FTP Secured)
- Used to transfer files to/from FTP servers

**Pidgin:**
- Access multiple messaging networks:
  - GTalk, AIM, ICQ, MSN, IRC

**Hexchat:**
- Access Internet Relay Chat (IRC) networks

---

### Office applications (productivity suites)

**What office suites provide:**
- Collection of closely coupled programs
- Create and edit different file types:
  - Text (articles, books, reports)
  - Spreadsheets
  - Presentations
  - Graphical objects

**LibreOffice:**
- Most widely used open source office suite on Linux
- Started in 2010, evolved from OpenOffice
- Most mature, widely used, intensely developed

**Additional options:**
- Internet-based office suites:
  - Google Docs
  - Microsoft Office 365

---

### LibreOffice components

**Main applications:**

| Component | Purpose |
|-----------|---------|
| Writer | Word processing |
| Calc | Spreadsheets |
| Impress | Presentations |
| Draw | Create and edit graphics and diagrams |

**Compatibility:**
- Can read and write non-native document formats (Microsoft Office)
- Fidelity usually maintained well
- Complicated documents might have imperfect conversions

---

### Development applications

**What's included in Linux distributions:**
- Complete set of tools for developing/maintaining applications and kernel
- Tools are tightly integrated

**Categories:**

**Text editors:**
- Advanced editors customized for programmers
- Examples: vi, emacs

**Compilers:**
- gcc and clang for C and C++
- Support for every computer language
- Including newer languages: Golang, Rust

**Debuggers:**
- gdb
- Graphical front ends to gdb
- Other debugging tools (Valgrind)

**Performance tools:**
- Measuring and monitoring programs
- Graphical interfaces available
- Advanced tools for experienced engineers

**Version control:**
- git (with GitHub and GitLab interfaces)
- Older systems: Apache Subversion

**Integrated Development Environments (IDEs):**
- Eclipse
- Visual Studio Code
- Put all tools together in one interface

**Key advantage:**
- All tools available at no cost on Linux
- Install via standard package management or direct download
- Other operating systems: tools must be obtained separately, often at high cost

---

### Sound players

**Amarok:**
- Mature MP3 player with graphical interface
- Plays audio and video files, streams (online audio)
- Create playlists containing groups of songs
- Database stores music collection information

**Audacity:**
- Record and edit sounds
- Simple interface

**Audacious:**
- Smart audio media player

**Rhythmbox:**
- Supports variety of digital music sources
  - Streaming Internet audio
  - Podcasts
- Search for audio in library
- Smart playlists with automatic update
- Revise playlists based on selection criteria

---

### Movie players

**What they do:**
- Portray input from many sources (local or Internet)

**Available players:**
- VLC
- MPlayer
- Xine
- Totem

---

### Movie editors

**Blender:**
- Create 3D animation and design
- Professional tool using modeling as starting point
- Complex and powerful tools for:
  - Camera capture
  - Recording
  - Editing
  - Enhancing
  - Creating video

**Cinelerra:**
- Capture, compose, and edit audio/video

**FFmpeg:**
- Record, convert, and stream audio/video
- Format converter
- Includes other tools: ffplay, ffserver

---

### GIMP (GNU Image Manipulation Program)

**What it is:**
- Feature-rich image retouching and editing tool
- Similar to Adobe Photoshop
- Available on all Linux distributions

**Supported formats:**
- JPEG (JPG) - Joint Photographic Experts Group
- PNG - Portable Network Graphics
- GIF - Graphics Interchange Format
- TIFF - Tagged Image File Format

**Features:**
- Handle any image file format
- Many special purpose plugins and filters
- Extensive image information:
  - Layers
  - Channels
  - Histograms

---

### Other graphics utilities

**Eye of Gnome (eog):**
- Image viewer
- Slide show capability
- Image editing tools: rotate, resize
- Step through images in directory with one click

**Inkscape:**
- Image editor with extensive editing features
- Works with layers and transformations
- Sometimes compared to Adobe Illustrator

**convert:**
- Command line tool (part of ImageMagick)
- Modify image files in many ways
- File format conversion
- Image modification options: blur, resize, despeckle

**Scribus:**
- Creating documents for publishing
- WYSIWYG (What You See Is What You Get) environment
- Numerous editing tools

---

### Chapter 7 summary

**Key concepts covered:**

- **Internet applications:**
  - Wide variety available
  - Graphical and text-based browsers (Firefox, Chrome, w3m, lynx)
  - Email clients (Thunderbird, Evolution, Mutt, mail)
  - Other applications (FileZilla, Pidgin, Hexchat)

- **Office productivity:**
  - LibreOffice for creating and editing documents
  - Components: Writer, Calc, Impress, Draw

- **Development:**
  - Complete suites of development tools
  - Compilers, debuggers, editors, version control
  - IDEs (Eclipse, Visual Studio Code)

- **Multimedia:**
  - Sound players: Amarok, Audacity, Audacious, Rhythmbox
  - Movie players: VLC, MPlayer, Xine, Totem
  - Movie editors: Blender, Cinelerra, FFmpeg

- **Graphics:**
  - GIMP for image editing (similar to Photoshop)
  - Other utilities: eog, Inkscape, convert, Scribus

---

## Chapter 8: Command line operations

### Learning objectives

By the end of this chapter, I should be able to:
- Use command line to perform operations in Linux
- Search for files
- Create and manage files
- Install and update software

---

### Introduction to the command line

**Why command line matters:**
- System administrators spend significant time at command line prompt
- Often automate and troubleshoot tasks in text environment

**Key saying:**
> "Graphical user interfaces make easy tasks easier, while command line interfaces make difficult tasks possible"

**Advantages:**
- No GUI overhead
- Virtually any task can be accomplished
- Can implement scripts for often-used tasks
- Can sign into remote machines anywhere on Internet
- Can initiate graphical applications directly
- Command line interface doesn't vary among distributions (unlike graphical tools)

---

### Terminal emulator programs

**What they do:**
- Emulate standalone terminal within window on desktop
- Behave as if logging into machine at pure text terminal
- Support multiple terminal sessions (tabs or windows)

**Available programs:**
- **gnome-terminal:** Default on GNOME desktop
- **xterm**
- **konsole:** Default on KDE
- **terminator**

---

### Launching terminal windows

**GNOME systems:**

**Method 1:**
- Applications → System Tools → Terminal
- OR Applications → Utilities → Terminal
- (May need to install gnome-shell-extension package)

**Method 2:**
- Right-click on desktop background
- Select "Open in Terminal"
- (May need to install gnome-shell-extension-apps-menu package)

**Method 3:**
- Hit `Alt-F2`
- Type `gnome-terminal`

**Pro tip:** Pin terminal icon to panel or add to Favorites for easy access

---

### Basic command line utilities

**Essential utilities (used constantly):**

| Command | Usage |
|---------|-------|
| `cat` | Type out a file (or combine files) |
| `head` | Show first few lines of a file |
| `tail` | Show last few lines of a file |
| `man` | View documentation |

**Note:** Pipe symbol (`|`) used to have one program take output of another as input

---

### Command line structure

**Three basic elements:**

1. **Command:** Name of program or script
2. **Options:** Modify what command does (start with `-` or `--`)
3. **Arguments:** What the command operates on

**Example:**
```bash
command -option argument
```

**Important notes:**
- Many commands have no options, no arguments, or neither
- Other elements can appear (like environment variables)

---

### sudo

**What it does:**
- Allows users to run programs using security privileges of another user
- Generally used to run as root (superuser)

**Setup:**
- Many distributions (Ubuntu) configure sudo during installation
- Others require manual configuration after installation

---

### Setting up sudo

**If not already configured:**

**Step 1: Switch to root**
```bash
$ su
Password: 
#
```

**Step 2: Create configuration file**
```bash
# echo "student ALL=(ALL) ALL" > /etc/sudoers.d/student
```
(Replace "student" with your username)

**Step 3: Change permissions (some distributions require this)**
```bash
# chmod 440 /etc/sudoers.d/student
```

**Using sudo:**
- Prompted for your own user password (not root password)
- Password cached for specified time interval
- Can configure to not require password (insecure)

---

### Virtual terminals (VT)

**What they are:**
- Console sessions using entire display and keyboard
- Outside of graphical environment
- Multiple active terminals, only one visible at a time
- NOT the same as terminal window in GUI

**Usage:**
- One VT (usually 1 or 7) reserved for graphical environment
- Text logins enabled on unused VTs

**When useful:**
- Problems with graphical desktop
- Switch to text VT to troubleshoot

**How to switch:**
- `CTRL-ALT-F#` (function key for VT number)
- Example: `CTRL-ALT-F6` for VT 6
- If already in VT: `ALT-F#` to switch to another VT

---

### Turning off the graphical desktop

**For systemd-based distributions:**

**Stop GUI:**
```bash
$ sudo systemctl stop gdm
```
OR
```bash
$ sudo telinit 3
```

**Restart GUI:**
```bash
$ sudo systemctl start gdm
```
OR
```bash
$ sudo telinit 5
```

**Note:** Exact method differs among distributions and versions

---

### Logging in and out

**Text terminal login:**
- Prompts for username (`login:`)
- Prompts for password
- Nothing displayed when typing password (prevents others seeing it)

**Remote login:**
- Use Secure Shell (SSH)
- Example: `ssh student@remote-server.com`
- Can use password or cryptographic key

---

### Rebooting and shutting down

**Preferred method: shutdown command**
- Sends warning message
- Prevents further logins
- init process controls shutdown/reboot

**Important:** Always shut down properly to avoid:
- System damage
- Data loss

**Commands:**

| Command | Action | Notes |
|---------|--------|-------|
| `halt` | `shutdown -h` | Halt system |
| `poweroff` | `shutdown -h` | Power off system |
| `reboot` | `shutdown -r` | Reboot system |

**All require superuser (root) access**

**Schedule shutdown with notification:**
```bash
$ sudo shutdown -h 10:00 "Shutting down for scheduled maintenance."
```

---

### Locating applications

**Common installation directories:**
- `/bin`, `/usr/bin`, `/sbin`, `/usr/sbin`
- `/opt`
- `/usr/local/bin`, `/usr/local/sbin`
- User account space: `/home/student/bin`

**Finding programs:**

**which utility:**
```bash
$ which diff
/usr/bin/diff
```

**whereis utility (if which fails):**
```bash
$ whereis diff
diff: /usr/bin/diff /usr/share/man/man1/diff.1.gz
```
- Looks in broader range of directories
- Also locates source and man files

---

### Directory navigation basics

**Default directory:**
- Home directory when logging in
- Check location: `echo $HOME`
- Graphical terminals often open in `$HOME/Desktop`

**Essential navigation commands:**

| Command | Result |
|---------|--------|
| `pwd` | Display present working directory |
| `cd ~` or `cd` | Change to home directory (~ is tilde) |
| `cd ..` | Change to parent directory |
| `cd -` | Change to previous working directory |

---

### Absolute vs relative paths

**Absolute pathname:**
- Begins with root directory (`/`)
- Follows tree until reaching destination
- Always starts with `/`

**Example:**
```bash
$ cd /usr/bin
```

**Relative pathname:**
- Starts from present working directory
- Never starts with `/`
- Uses shortcuts: `.` (current), `..` (parent), `~` (home)

**Example from home directory:**
```bash
$ cd ../../usr/bin
```

**Multiple slashes:**
- Allowed: `////usr//bin`
- System sees as: `/usr/bin`
- All but one slash ignored

---

### Exploring the filesystem

**Useful commands:**

| Command | Usage |
|---------|-------|
| `cd /` | Change to root directory |
| `ls` | List contents of present directory |
| `ls -a` | List all files (including hidden files starting with `.`) |
| `tree` | Display tree view of filesystem |
| `tree -d` | Display only directories (suppress files) |

---

### Hard links

**Creating hard links:**
```bash
$ ln file1 file2
```

**What happens:**
- Two names for same file
- Share same inode number
- Verified with `ls -li`

**Important characteristics:**
- Save space
- Both names point to same data
- Removing one name leaves other intact
- File data deleted only when all links removed

**Cautions:**
- Editing behavior depends on editor
- Some editors break the link
- May lead to subtle errors if recreating file names

---

### Soft (symbolic) links

**Creating symbolic links:**
```bash
$ ln -s file1 file3
```

**Characteristics:**
- Points to original file
- Different inode number
- Takes no extra space (unless name very long)
- Can point to different filesystems/partitions/media
- Can create "dangling link" if target doesn't exist

**Advantages:**
- Easy to modify (point to different places)
- Create shortcuts to long pathnames
- More flexible than hard links

---

### Directory history navigation

**cd with memory:**
```bash
$ cd -
```
- Returns to previous directory

**pushd/popd for multiple directories:**
- `pushd`: Change directory and push starting directory onto list
- `popd`: Return to directories in reverse order
- `dirs`: Display list of directories

---

### Viewing files

**Common commands:**

| Command | Usage |
|---------|-------|
| `cat` | View files that are not very long; no scroll-back |
| `tac` | View file backwards (starts with last line) |
| `less` | View larger files; paging program with scroll-back |
| `tail` | Print last 10 lines (default); use `-n 15` or `-15` for different number |
| `head` | Print first 10 lines (default) |

**less navigation:**
- `/` - Search forward for pattern
- `?` - Search backward for pattern
- More capabilities than older `more` program

**Note:** "less is more"

---

### touch command

**Primary uses:**

**Update timestamps:**
- Sets/updates access, change, and modify times
- Default: resets to current time

**Create empty file:**
```bash
$ touch <filename>
```
- Done to create placeholder for later use

**Set specific timestamp:**
```bash
$ touch -t 12091600 myfile
```
- Sets to 4 p.m., December 9th (12 09 1600)

---

### Creating and removing directories

**mkdir:**
```bash
$ mkdir sampdir                # In current directory
$ mkdir /usr/sampdir           # In /usr
```

**rmdir:**
- Removes only empty directories
- Fails if directory contains files

**Remove directory and all contents:**
```bash
$ rm -rf <directory>
```
⚠️ **Warning:** Extremely dangerous, especially as root. Use with utmost care.

---

### Moving, renaming, and removing files

**mv command (dual purpose):**
- Rename a file
- Move file to another location (possibly changing name)

**rm command:**

| Command | Usage |
|---------|-------|
| `rm` | Remove a file |
| `rm -f` | Forcefully remove a file |
| `rm -i` | Interactively remove (prompts before removal) |

**Pro tip:** Use `rm -i` when uncertain about pattern matching

---

### Removing directories

**rmdir:**
- Works only on empty directories

**rm -rf:**
```bash
$ rm -rf <directory>
```
⚠️ **Warning:** 
- Fast and easy way to remove entire filesystem tree
- Extremely dangerous
- Use with utmost care, especially as root
- "Recursive" means drilling down through all subdirectories

---

### Modifying the command line prompt

**PS1 variable:**
- Controls prompt display

**Example customization:**
```bash
$ PS1="\u@\h \$ "
student@hostname $
```

**Common variables:**
- `\u` - Username
- `\h` - Hostname
- `\$` - `$` for users, `#` for root

**Convention:** Root user typically has `#` prompt

---

### Standard file streams

**Three standard file streams (file descriptors):**

| Name | Symbolic Name | Value | Example |
|------|---------------|-------|---------|
| Standard input | stdin | 0 | keyboard |
| Standard output | stdout | 1 | terminal |
| Standard error | stderr | 2 | log file |

**Default behavior:**
- stdin: Keyboard
- stdout: Terminal
- stderr: Terminal (often redirected to error log)

**File descriptors:**
- Represented by numbers starting at 0
- Additional open files start at 3 and increase

---

### I/O redirection

**Redirect input:**
```bash
$ do_something < input-file
```

**Redirect output:**
```bash
$ do_something > output-file
```

**Redirect both:**
```bash
$ do_something < input-file > output-file
```

**Redirect stderr:**
```bash
$ do_something 2> error-file
```

**Redirect stderr to same place as stdout:**
```bash
$ do_something > all-output-file 2>&1
```

**bash shorthand:**
```bash
$ do_something >& all-output-file
```

---

### Pipes

**UNIX/Linux philosophy:**
- Many simple, short programs cooperate
- Produce complex results
- Rather than one complex program with many options

**Pipe syntax:**
```bash
$ command1 | command2 | command3
```

**Benefits:**
- Commands don't wait for previous to complete
- Process data in input streams immediately
- Better utilizes multiple CPU/cores
- No need for temporary files
- Saves disk space and I/O operations

---

### locate utility

**What it does:**
- Searches using pre-constructed database
- Matches entries containing specified string
- Can produce very long lists

**Filter with grep:**
```bash
$ locate zip | grep bin
```
- Lists files/directories with both "zip" and "bin"

**Database:**
- Created by `updatedb` utility
- Most systems run automatically once daily
- Update manually: `sudo updatedb`

---

### Wildcards

**Wildcard characters:**

| Wildcard | Result |
|----------|--------|
| `?` | Matches any single character |
| `*` | Matches any string of characters |
| `[set]` | Matches any character in set (e.g., `[adf]`) |
| `[!set]` | Matches any character NOT in set |

**Examples:**
```bash
$ ls ba?.out         # Three-letter filename starting with "ba"
$ ls *.out           # All files ending with .out
```

---

### find command

**What it does:**
- Recurses down filesystem tree
- Locates files matching specified conditions
- Default: searches current directory

**Common options:**

| Option | Purpose |
|--------|---------|
| `-name` | Files with pattern in name |
| `-iname` | Case-insensitive name search |
| `-type` | Restrict to file type (d=directory, l=link, f=regular file) |

**Examples:**
```bash
$ find /usr -name gcc                # All files/dirs named gcc
$ find /usr -type d -name gcc        # Only directories named gcc
$ find /usr -type f -name gcc        # Only regular files named gcc
```

---

### Advanced find options

**Execute commands on found files:**
```bash
$ find -name "*.swp" -exec rm {} ';'
```
- `{}` placeholder for found filenames
- Must end with `';'` or `\;`

**Interactive execution:**
```bash
$ find -name "*.swp" -ok rm {} ';'
```
- Prompts for permission before executing
- Good for testing potentially dangerous commands

---

### Finding files by time and size

**Time-based searches:**
```bash
$ find / -ctime 3        # Inode metadata change time
$ find / -atime 3        # Access/last read time
$ find / -mtime 3        # Modified/last written time
```

**Time values:**
- `n` - Exactly n days
- `+n` - Greater than n days
- `-n` - Less than n days

**Minute-based:** `-cmin`, `-amin`, `-mmin`

**Size-based searches:**
```bash
$ find / -size 0         # Empty files (0 blocks)
$ find / -size +10M      # Greater than 10 MB
```

**Size units:**
- Default: 512-byte blocks
- `c` - bytes
- `k` - kilobytes
- `M` - megabytes
- `G` - gigabytes

---

### Package management systems

**Two main families:**
1. **Debian-based:** dpkg/apt
2. **RPM-based:** rpm/dnf/zypper

**Key concept:**
- Incompatible with each other
- Provide same essential features
- Satisfy same needs

**Two levels of tools:**

**Low-level:**
- dpkg (Debian) or rpm (Red Hat)
- Handles unpacking packages
- Runs scripts
- Gets software installed

**High-level:**
- apt (Debian), dnf (Red Hat), zypper (SUSE)
- Works with groups of packages
- Downloads from vendor
- Resolves dependencies

**Important:** Dependency resolution is key feature of high-level tools

---

### Package management commands

**Common operations:**

| Operation | RPM | DEB |
|-----------|-----|-----|
| Install package | `rpm -i foo.rpm` | `dpkg --install foo.deb` |
| Install with dependencies | `dnf install foo` | `apt install foo` |
| Remove package | `rpm -e foo.rpm` | `dpkg --remove foo.deb` |
| Remove with dependencies | `dnf remove foo` | `apt autoremove foo` |
| Update package | `rpm -U foo.rpm` | `dpkg --install foo.deb` |
| Update with dependencies | `dnf update foo` | `apt install foo` |
| Update entire system | `dnf update` | `apt dist-upgrade` |
| Show installed packages | `rpm -qa` or `dnf list installed` | `dpkg --list` |
| Get package info | `rpm -qil foo` | `dpkg --listfiles foo` |
| Search for package | `dnf list "foo"` | `apt-cache search foo` |
| Show all available | `dnf list` | `apt-cache dumpavail foo` |
| Find package for file | `rpm -qf file` | `dpkg --search file` |

---

### Chapter 8 summary

**Key concepts covered:**

- **Virtual terminals (VT):**
  - Console sessions outside graphical environment
  - Switch with `CTRL-ALT-F#`

- **Terminal emulators:**
  - Emulate terminal within window on desktop

- **Login/passwords:**
  - Nothing printed when typing password
  - Can log in via text terminal or remotely

- **Shutdown:**
  - Preferred method: `shutdown` command

- **Pathnames:**
  - Absolute: starts with `/`, follows tree from root
  - Relative: starts from present working directory

- **Links:**
  - Hard links: multiple names for same inode
  - Soft links: pointers to files/directories

- **Command shortcuts:**
  - `cd -`: returns to previous directory

- **File utilities:**
  - `locate`: database search for filenames
  - `find`: recursive search with extensive options
  - `touch`: set file times, create empty files

- **Package management:**
  - apt: Debian-based systems
  - dnf: RPM-based Red Hat family
  - zypper: RPM-based SUSE/openSUSE

---

## Feb 18, 2026

## Chapter 9: Finding Linux documentation

### Learning objectives

By the end of this chapter, I should be able to:
- Use different sources of documentation
- Use the man pages
- Access the GNU Info System
- Use the help command and --help option
- Use other documentation sources

---

### Linux documentation sources overview

**Why documentation matters:**
- Won't always know proper use of programs/utilities
- Need to know: what command, what options, what results to expect
- Will need to consult help documentation regularly

**Important sources:**
- The man pages (manual pages)
- GNU Info
- The help command and --help option
- Other documentation sources (distribution-specific handbooks, wikis)

**Note:** Distributors consolidate documentation and present it in comprehensive, easy-to-use manner

---

### The man pages

**What they are:**
- Most often-used source of Linux documentation
- Provide in-depth documentation about:
  - Programs and utilities
  - Configuration files
  - Programming APIs for system calls
  - Library routines
  - The kernel
- Present on all Linux distributions
- Always available

**History:**
- Introduced in early UNIX versions (beginning of 1970s)
- "man" = abbreviation for "manual"

**Usage:**
```bash
man <topic>
```

**Other formats:**
- Often converted to PDF documents and web pages
- Many graphical help interfaces include man pages
- Linux man pages available online

---

### Using the man program

**What it does:**
- Searches, formats, and displays information from man page system
- Output piped through pager program (like `less`) for one page at a time
- Information formatted for good visual display

**Important options:**

| Option | Function | Equivalent |
|--------|----------|------------|
| `man -f <topic>` | List all pages on topic | `whatis` |
| `man -k <topic>` | List all pages discussing topic | `apropos` |

**Section order:**
- Default order specified in `/etc/man_db.conf`
- Roughly in ascending numerical order by section
- Given topic may have multiple pages

---

### GNU Info System

**What it is:**
- GNU project's standard documentation format
- Preferred alternative to man pages
- Free-form format supporting linked subsections

**Key characteristics:**
- Topics connected using links (predates World Wide Web)
- Information viewable through:
  - Command line interface
  - Graphical help utility
  - Printed format
  - Online

**Comparison to man:**
- Similar functionality in many ways
- Often provides more complete information
- Example: `man ls` vs `info ls` (count the lines)
- Interface may seem outdated but is important to learn

---

### Using info from command line

**Basic usage:**

**View index of available topics:**
```bash
info
```

**View specific topic:**
```bash
info <topic name>
```

**Navigation:**
- Regular movement keys work (arrows, Page Up, Page Down)
- Browse through topic list

**Useful keys:**

| Key | Function |
|-----|----------|
| `q` | Quit |
| `h` | Help |
| `Enter` | Select menu item |

---

### Info page structure

**Nodes:**
- Topic viewed in info page = "node"
- Essentially sections and subsections
- Can move between nodes or view sequentially
- Each node may contain menus and linked subtopics (items)

**Navigation between nodes:**

| Key | Function |
|-----|----------|
| `n` | Go to next node |
| `p` | Go to previous node |
| `u` | Move one node up in index |

**Items (links):**
- Function like browser links
- Identified by asterisk (`*`) at beginning of item name
- Named items (outside menu) identified with double-colons (`::`)
- Can refer to other nodes within file or to other files

---

### The --help option

**What it provides:**
- Short description of commands
- Quick reference
- Faster than man or info pages

**Usage:**
```bash
<command> --help
```
OR
```bash
<command> -h
```

**Example:**
```bash
$ man --help
```

---

### The help command

**For bash built-in commands:**
- Some popular commands (like `echo` and `cd`) run as bash built-ins
- More efficient (faster execution, fewer resources)
- Built-in versions run instead of binaries in `/bin` or `/usr/bin`

**Note:** Can be some (usually small) differences between built-in and standalone versions

**View synopsis of built-in commands:**
```bash
help
```

**Function:** Similar to `-h` and `--help` for standalone programs

---

### Other documentation sources

**Desktop help systems:**
- All Linux desktop systems have graphical help application
- Usually displayed as question-mark icon or life preserver image
- Found in menu system
- Contains:
  - Custom help for desktop
  - Help for some applications
  - Graphically-rendered info and man pages

**Launch from terminal:**

**GNOME:**
```bash
gnome-help
```
OR
```bash
yelp
```

**KDE:**
```bash
khelpcenter
```

---

### Package documentation

**Location:** `/usr/share/doc`

**What it contains:**
- Documentation from upstream source code
- Information about how distribution packaged and set up software
- Grouped in subdirectories named after each package

---

### Online resources

**Recommended book:**
- "The Linux Command Line" by William Shotts
- Free, downloadable command line compendium
- Under Creative Commons license
- Well-reviewed by course users

**Distribution-specific documentation:**
- Each distribution has user-generated forums and wiki sections

**Examples:**
- Ubuntu Documentation
- CentOS Documentation
- openSUSE Documentation
- Gentoo Documentation
- Fedora Documentation

**Additional sources:**
- Online search sites
- Blog posts
- Forum and mailing list posts
- News articles

---

### Chapter 9 summary

**Key concepts covered:**

- **Main documentation sources:**
  - man pages
  - GNU Info
  - help options and command
  - Rich variety of online sources

- **man utility:**
  - Searches, formats, and displays man pages
  - Provides in-depth documentation about:
    - Programs and topics
    - Configuration files
    - System calls
    - Library routines
    - The kernel

- **GNU Info System:**
  - Created by GNU project as standard documentation
  - Robust and accessible via:
    - Command line
    - Web
    - Graphical tools using `info`

- **Quick help:**
  - Short descriptions with `-h` or `--help` argument
  - `help` command displays synopsis of built-in commands

- **Other resources:**
  - Many help resources on system and Internet
  - Distribution-specific documentation
  - Package documentation in `/usr/share/doc`

---

## Feb 20, 2026

## Chapter 10: Processes

### Learning objectives

By the end of this chapter, I should be able to:
- Describe what a process is and distinguish between types of processes
- Enumerate process attributes
- Manage processes using ps and top
- Understand the use of load averages and other process metrics
- Manipulate processes by putting them in background and restoring them to foreground
- Use at, cron, and sleep to schedule processes in the future or pause them

---

### What is a process?

**Definition:**
- An instance of one or more related tasks (threads) executing on computer
- NOT the same as a program or command
- Single command may start several processes simultaneously

**Key characteristics:**
- Some processes are independent, others are related
- Failure of one process may or may not affect others
- Processes use system resources:
  - Memory
  - CPU cycles
  - Peripheral devices (network cards, hard drives, printers, displays)

**Operating system responsibility:**
- Allocate proper share of resources to each process
- Ensure overall optimized system utilization

---

### Process types

**Terminal window as example:**
- Command shell is a process that runs as long as needed
- Allows users to execute programs and access resources interactively
- Can also run programs in background (detached from shell)

**Process types by task:**

| Process Type | Description | Examples |
|--------------|-------------|----------|
| **Interactive Processes** | Started by user (command line or GUI) | bash, firefox, top, Slack, LibreOffice |
| **Batch Processes** | Automatic processes scheduled from terminal then disconnected. Queued and work on FIFO basis | updatedb, ldconfig |
| **Daemons** | Server processes that run continuously. Launched during startup, wait for service requests | httpd, sshd, libvirtd, cupsd |
| **Threads** | Lightweight processes under main process. Share memory/resources but scheduled individually. Individual thread can end without terminating whole process | dconf-service, gnome-terminal-server |
| **Kernel Threads** | Kernel tasks users don't start or terminate. Little user control | kthreadd, migration, ksoftirqd |

---

### Process scheduling and states

**Scheduler function:**
- Critical kernel function
- Constantly shifts processes on and off CPU
- Shares time according to:
  - Relative priority
  - Time needed
  - Time already granted

**Process states:**

**Running state:**
- Currently executing instructions on CPU, OR
- Waiting to be granted time slice to execute
- All running processes reside on "run queue"
- Multi-CPU systems: separate run queue per CPU/core
- Note: "Running" can be misleading - process may be swapped out waiting its turn

**Sleep state:**
- Process waiting for something to happen before resuming
- Example: waiting for user to type something
- Process sits in "wait queue"

**Zombie state:**
- Child process completed but parent hasn't asked about its state
- Not really alive but still shows up in process list

---

### Process and thread IDs

**System tracking:**
- Operating system keeps track of processes
- Each assigned unique Process ID (PID) number

**PID characteristics:**
- Used to track:
  - Process state
  - CPU usage
  - Memory use
  - Resource locations in memory
  - Other characteristics
- Usually assigned in ascending order as processes are born
- PID 1 = init process (system initialization)

**ID types:**

| ID Type | Description |
|---------|-------------|
| **Process ID (PID)** | Unique process ID number |
| **Parent Process ID (PPID)** | Process that started this process. If parent dies, PPID refers to adoptive parent (kthreadd on modern kernels, PPID=2) |
| **Thread ID (TID)** | Thread ID number. Same as PID for single-threaded processes. Multi-threaded: each thread shares same PID but has unique TID |

---

### Terminating a process

**When needed:**
- Application stops working properly
- Need to eliminate it

**Kill command:**
```bash
kill -SIGKILL <pid>
```
OR
```bash
kill -9 <pid>
```

**Important restrictions:**
- Can only kill your own processes
- Other users' processes are off-limits (unless you are root)
- Name "kill" is historical/misleading - can send any signal, not just termination

---

### User and group IDs

**Multi-user access:**
- Many users can access system simultaneously
- Each user can run multiple processes

**User identification:**
- **Real User ID (RUID):** Identifies user who starts the process
- **Effective UID (EUID):** Determines access rights for users
- EUID may or may not be same as RUID

**Group identification:**
- Users organized into enumerated groups
- **Real Group ID (RGID):** Identifies the group
- **Effective Group ID (EGID):** Determines group access rights
- Each user can be member of one or more groups

**Common usage:**
- Usually just refer to User ID (UID) and Group ID (GID)

---

### Process priorities

**Why priorities matter:**
- Many processes running simultaneously
- CPU can only accommodate one task at a time
- Some processes more important than others
- Higher priority processes get preferential CPU access

**Nice values:**
- Priority set by specifying "nice value" or "niceness"
- Lower nice value = higher priority
- Low values for important processes
- High values for processes that can wait longer
- High nice value allows other processes to execute first

**Nice value range:**
- `-20` = highest priority
- `+19` = lowest priority
- Convention: nicer the process, lower the priority (dates back to earliest UNIX)

**Real-time priority:**
- Can assign to time-sensitive tasks
- Examples: controlling machines, collecting incoming data
- Very high priority
- NOT same as "hard real-time" (different concept about completion time windows)

---

### Load averages

**What load average measures:**
- Average of load number for given period
- Takes into account processes that are:
  - Actively running on CPU
  - Runnable but waiting on run queue for CPU
  - Sleeping (waiting for resource, typically I/O)

**Linux-specific:**
- Differs from other UNIX-like systems
- Includes sleeping processes
- Only includes "uninterruptible sleepers" (cannot be awakened easily)

**View load average:**
```bash
w
top
uptime
```

---

### Interpreting load averages

**Three numbers displayed:**
- Example: 0.45, 0.17, 0.12

**For single-CPU system:**
- **0.45** (first number): Last minute, system 45% utilized on average
- **0.17** (second number): Last 5 minutes, 17% utilization
- **0.12** (third number): Last 15 minutes, 12% utilization

**Understanding 1.00:**
- For single-CPU: 1.00 = 100% utilized
- Value over 1.00 = over-utilized (more processes needing CPU than available)

**Multi-CPU systems:**
- Divide load average by number of CPUs
- Quad-CPU system: 1 minute load of 4.00 = 100% (4.00/4) utilization

**What to watch for:**
- Short-term increases usually not a problem
- High peak likely burst of activity
- High peak in 5 and 15 minute averages may be cause for concern

---

### Background and foreground processes

**Job definition:**
- Command launched from terminal window

**Foreground jobs:**
- Run directly from shell
- When one foreground job running, others wait for shell access
- Fine when jobs complete quickly
- Problematic for long-running jobs (hours)

**Background jobs:**
- Run in background
- Free shell for other tasks
- Executed at lower priority
- Allows smooth execution of interactive tasks
- Can type other commands while background job runs

**Running jobs:**
- Default: all jobs execute in foreground
- Run in background: suffix `&` to command
```bash
  updatedb &
```

**Control keys:**
- `CTRL-Z` - Suspend foreground job (put in background)
- `CTRL-C` - Terminate job
- `bg` command - Run suspended process in background
- `fg` command - Run background process in foreground

---

### Managing jobs

**jobs utility:**
```bash
jobs
```
- Displays all jobs running in background
- Shows: job ID, state, command name

**jobs with PID:**
```bash
jobs -l
```
- Same information as `jobs`
- Adds PID of background jobs

**Important note:**
- Background jobs connected to terminal window
- If you log off, jobs started from that window won't show

---

### The ps command (System V style)

**What ps provides:**
- Information about currently running processes
- Keyed by PID

**Alternative monitoring:**
- `top` - periodic updates
- Other variants: `htop`, `atop`, `btop`
- Graphical: `gnome-system-monitor`, `ksysguard`

**Common ps options:**

| Command | Purpose |
|---------|---------|
| `ps` | All processes under current shell |
| `ps -u <username>` | Processes for specified username |
| `ps -ef` | All processes in system (full detail) |
| `ps -eLf` | One line per thread (process can contain multiple threads) |

---

### The ps command (BSD style)

**BSD option style:**
- Options specified without preceding dashes
- Stems from BSD variety of UNIX

**Common BSD commands:**

| Command | Purpose |
|---------|---------|
| `ps aux` | All processes of all users |
| `ps axo` | Specify which attributes to view |

---

### The process tree

**pstree command:**
```bash
pstree
```

**What it shows:**
- Processes in tree diagram
- Relationship between process and parent
- Other processes it created
- Repeated entries not displayed
- Threads displayed in curly braces

---

### top command

**What it provides:**
- Constant real-time updates (every 2 seconds by default)
- Exit by typing `q`

**Advantages over static ps:**
- Monitors system performance live over time
- Better than running ps at regular intervals

**Key features:**
- Highlights processes consuming most CPU cycles
- Highlights memory usage
- Use appropriate commands from within top

---

### First line of top output

**Information displayed:**
- How long system has been up
- How many users logged on
- Load average

**Load average interpretation:**
- Determines how busy system is
- 1.00 per CPU = fully subscribed but not overloaded
- Above 1.00 = processes competing for CPU time
- Very high = possible problem (runaway process)

---

### Second line of top output

**Process counts:**
- Total number of processes
- Number of running processes
- Sleeping processes
- Stopped processes
- Zombie processes

**What to check:**
- Compare running processes with load average
- Helps determine if system reached capacity
- Check if particular user running too many processes
- Examine stopped processes for correct operation

---

### Third line of top output

**CPU time division:**
- Shows how CPU time divided between users (us) and kernel (sy)
- Displays percentage for each

**Other CPU metrics:**
- **ni** - User jobs at lower priority (niceness)
- **id** - Idle mode (should be low if load average high)
- **wa** - Jobs waiting for I/O
- **hi** - Hardware interrupts
- **si** - Software interrupts
- **st** - Steal time (used with virtual machines)

---

### Fourth and fifth lines of top output

**Memory usage categories:**

**Line 4: Physical memory (RAM)**
- Total memory
- Used memory
- Free space

**Line 5: Swap space**
- Total swap
- Used swap
- Free swap

**Important monitoring:**
- Critical to monitor memory usage for good performance
- Physical memory exhausted → system uses swap space
- Swap = temporary storage on hard drive (extended memory pool)
- Disk access much slower than memory access
- Negatively affects system performance

**Solutions for frequent swapping:**
- Add more swap space
- Consider adding more physical memory

---

### Process list in top output

**Information per process (by default, ordered by highest CPU usage):**
- **PID** - Process Identification Number
- **USER** - Process owner
- **PR** - Priority
- **NI** - Nice value
- **VIRT** - Virtual memory
- **RES** - Physical memory
- **SHR** - Shared memory
- **S** - Status
- **%CPU** - Percentage of CPU used
- **%MEM** - Percentage of memory used
- **TIME+** - Execution time
- **COMMAND** - Command

---

### Interactive keys with top

**While top is running:**
- Enter single-letter commands to change behavior
- Can view top-ranked processes by CPU or memory
- Can alter priorities of running processes
- Can stop/kill processes

**Common interactive commands:**

| Command | Function |
|---------|----------|
| `h` or `?` | Display available interactive keys |
| `t` | Display or hide summary information (rows 2 and 3) |
| `m` | Display or hide memory information (rows 4 and 5) |
| `l` | Show information for each CPU (not just totals) |
| `d` | Change display update interval |
| `A` | Sort process list by top resource consumers |
| `r` | Renice (change priority of) specific process |
| `k` | Kill specific process |
| `f` | Enter top configuration screen |
| `o` | Interactively select new sort order |

**Note:** Most keys are toggles (hit second time to revert)

**Alternatives to top:**
- `atop`, `btop`, `htop`
- Each has its fans
- Offer prettier displays and additional capabilities

---

### Scheduling future processes with at

**Use case:**
- Need to perform task on specific future day
- Won't be at machine that day

**Solution: at utility**
- Execute any non-interactive command at specified time

**Example usage:**
```bash
at 10:00 AM tomorrow
```

---

### cron

**What it is:**
- Time-based scheduling utility program
- Launches routine background jobs at specific times/days
- Ongoing basis

**Configuration:**
- Driven by `/etc/crontab` (cron table)
- Contains shell commands to run at scheduled times
- System-wide crontab files
- Individual user-based crontab files

**Crontab structure:**
- Each line = one job
- CRON expression + shell command

**Edit crontab:**
```bash
crontab -e
```

**Crontab fields:**

| Field | Description | Values |
|-------|-------------|--------|
| MIN | Minutes | 0-59 |
| HOUR | Hour | 0-23 |
| DOM | Day of Month | 1-31 |
| MON | Month | 1-12 |
| DOW | Day of Week | 0-6 (0=Sunday) |
| CMD | Command | Any command to execute |

**Examples:**

**Run every minute:**
```
* * * * * /usr/local/bin/execute/this/script.sh
```

**Run at specific time/date:**
```
30 08 10 06 * /home/sysadmin/full-backup
```
(8:30 a.m. on June 10, any day of week)

---

### anacron

**Why it exists:**
- cron implicitly assumed machine always running
- If machine powered off, scheduled jobs wouldn't run
- Modern Linux distributions moved to anacron

**How it works:**
- Runs necessary jobs in controlled, staggered manner
- When system is up and running
- Still uses cron infrastructure for submitting jobs
- Defers running until opportune times when system alive

**Configuration:**
- Key file: `/etc/anacrontab`

**Job scheduling:**
- Daily, weekly, monthly basis
- Runs when system actually available

---

### sleep command

**Use cases:**
- Command or job must be delayed or suspended
- Example: application needs to save report but backup system busy
- System process runs periodically, then lurks until needed again

**What it does:**
- Suspends execution for at least specified period
- After time passes (or interrupting signal received), execution resumes

**Syntax:**
```bash
sleep NUMBER[SUFFIX]
```

**Suffixes:**
- `s` - seconds (default)
- `m` - minutes
- `h` - hours
- `d` - days

**Difference from at:**
- `sleep` - delays execution for specific period
- `at` - starts execution at specific designated later time

---

### Chapter 10 summary

**Key concepts covered:**

- **Processes:**
  - Used to perform various tasks on system
  - Can be single-threaded or multi-threaded
  - Different types: interactive and non-interactive

- **Process identification:**
  - Every process has unique identifier (PID)
  - Enables OS to keep track

- **Process priority:**
  - Nice value (niceness) sets priority

- **Process monitoring:**
  - `ps` provides info about currently running processes
  - `top` gives constant real-time updates:
    - Overall system performance
    - Information about running processes

- **Load average:**
  - Indicates amount of utilization at particular times

- **Job processing:**
  - Linux supports background and foreground processing

- **Scheduling:**
  - `at` executes non-interactive commands at specified time
  - `cron` schedules tasks at regular intervals

---

## Feb 23, 2026

## Chapter 11: File operations

### Learning objectives

By the end of this chapter, I should be able to:
- Explore the filesystem and its hierarchy
- Explain the filesystem layout and the purpose of important directories
- List common filesystem types used in Linux
- Understand disk partitions and mounting and checking filesystems
- Use NFS
- Compare files and identify different file types
- Back up and compress data

---

### Introduction to filesystems

**"Everything is a file" philosophy:**
- Common adage in Linux and UNIX-like systems
- Interaction with devices (sound cards, printers) uses same I/O operations as regular files
- Simplifies system interactions
- One reason text editors are so important

**Filesystem structure:**
- Structured like an inverted tree
- Starts at root directory (`/`)
- Also called trunk
- Root directory ≠ root user

**Path structure:**
- Elements separated by forward slashes (`/`)
- Example: `/usr/bin/emacs`
- Last element is actual file name

---

### Filesystem varieties

**Native Linux filesystems:**
- ext3
- ext4
- squashfs
- btrfs

**Filesystems from other operating systems:**
- Windows: ntfs, vfat, exfat
- SGI: xfs
- IBM: jfs
- MacOS: hfs, hfs+
- Many older legacy filesystems (FAT)

**Multiple filesystem types:**
- Often used on same machine
- Based on considerations:
  - File size
  - Modification frequency
  - Hardware type
  - Required access speed

**Advanced journaling filesystems:**
- ext4, xfs, btrfs, jfs
- State-of-the-art features
- High performance
- Not easy to corrupt accidentally

**Network/distributed filesystems:**
- All or part of filesystem on external machines
- NFS (Network File System)
- Ceph, Lustre, OpenAFS

---

### Linux partitions

**Basic concept:**
- Most filesystems occupy a disk partition
- Partitions organize disk contents by kind and use

**Common partition scheme:**
- System programs on separate partition (root `/`)
- User files on different partition (`/home`)
- Temporary files may have dedicated partitions

**Advantages of partition isolation:**
- When partition fills up, system may still operate normally
- Data corruption/breach can be confined to smaller area
- Problems isolated by type and variability

---

### Mount points

**Definition:**
- Directory where filesystem is grafted onto filesystem tree
- May or may not be empty
- May need to create directory if doesn't exist

**Important warning:**
⚠️ Mounting filesystem on non-empty directory covers up former contents
- Contents not accessible until filesystem unmounted
- Mount points usually empty directories

---

### Mounting and unmounting

**Mount command:**
```bash
sudo mount /dev/sda5 /home
```
- Attaches filesystem to filesystem tree
- Basic arguments: device node and mount point
- Can also specify by disk label or UUID

**Unmount command:**
```bash
sudo umount /home
```
- Note: Command is `umount`, not `unmount`

**Permissions:**
- Only root user can run these commands
- Unless system configured otherwise

**Automatic mounting:**
- Edit `/etc/fstab` for mount at boot
- File contains configuration of pre-configured filesystems
- `man fstab` for details

**View mounted filesystems:**
```bash
mount           # Show all currently mounted
df -Th          # Display with type and usage statistics
```

**Note on tmpfs:**
- Not real physical filesystems
- Parts of system memory represented as filesystems
- Take advantage of certain programming features

---

### NFS and network filesystems

**Purpose:**
- Share data across physical systems
- Systems may be in same location or anywhere on Internet

**Network filesystem characteristics:**
- May have all data on one machine
- Or spread across multiple network nodes
- Can group different local filesystem types

**Common use case:**
- System administrators mount remote users' home directories on server
- Users get access to same files/configuration across multiple clients
- Can log into different computers with same files/resources

**NFS (Network Filesystem):**
- Most common network filesystem
- Long history
- First developed by Sun Microsystems

**CIFS/SAMBA:**
- Another common implementation
- Has Microsoft roots

---

### NFS on the server

**Starting NFS service:**
```bash
sudo systemctl start nfs
```
- Note: Some systems (RHEL/CentOS, Fedora) call it `nfs-server`

**Configuration file: /etc/exports**
- Contains directories and permissions host will share
- Text file

**Example entry:**
```
/projects *.example.com(rw)
```
- Allows `/projects` to be mounted via NFS
- Read and write (rw) permissions
- Shared with hosts in example.com domain

**File permissions basics:**
- Every file has three possible permissions:
  - Read (r)
  - Write (w)
  - Execute (x)

**Apply changes:**
```bash
exportfs -av                    # Notify Linux about shared directories
sudo systemctl restart nfs      # Heavier option, halts NFS briefly
sudo systemctl enable nfs       # Start NFS at boot
```

---

### NFS on the client

**Automatic mount at boot:**
- Modify `/etc/fstab`

**Example fstab entry:**
```
servername:/projects /mnt/nfs/projects nfs defaults 0 0
```

**One-time mount (without reboot):**
```bash
sudo mount servername:/projects /mnt/nfs/projects
```

**Important notes:**
- Mount without fstab change won't persist after restart
- Consider using `nofail` option in fstab if NFS server might not be live at boot

---

### User home directories

**Structure:**
- Each user has home directory
- Usually under `/home`
- Named according to user (e.g., `/home/student`)

**Root user:**
- `/root` directory = home directory of root user
- Not the same as root directory (`/`)

**Multi-user systems:**
- `/home` infrastructure may be:
  - Mounted as separate filesystem on own partition
  - Exported remotely on network through NFS

**Directory organization:**
- Can group users by department/function
- Create subdirectories under `/home`

**Example school organization:**
```
/home/faculty/
/home/staff/
/home/students/
```

---

### The /bin and /sbin directories

**`/bin` directory:**
- Contains executable binaries
- Essential commands for:
  - Booting system
  - Single-user mode
  - All system users
- Examples: cat, cp, ls, mv, ps, rm

**`/sbin` directory:**
- Essential binaries for system administration
- Examples: fsck, ip

**View programs:**
```bash
ls /bin /sbin
```

**`/usr/bin` and `/usr/sbin`:**
- Non-essential commands (theoretically)
- Not needed to boot or operate in single-user mode
- Historically `/usr` could be separate filesystem
- Distinction now mostly obsolete

**Modern distributions:**
- `/usr/bin` and `/bin` symbolically linked together
- `/usr/sbin` and `/sbin` symbolically linked together
- Really just two directories, not four

---

### The /proc filesystem

**Pseudo-filesystem:**
- No actual permanent presence on disk
- Contains virtual files (exist only in memory)

**Purpose:**
- Permits viewing constantly changing kernel data
- Files/directories mimic kernel structures and configuration
- Runtime system information, not real files

**Important entries:**
```
/proc/cpuinfo
/proc/interrupts
/proc/meminfo
/proc/mounts
/proc/partitions
/proc/version
```

**Subdirectories:**
- `/proc/<Process-ID-#>` - Directory for every running process
- `/proc/sys` - Virtual directory with system/hardware information

**Key advantage:**
- Information gathered only as needed
- Never needs storage on disk

---

### The /dev directory

**Device nodes:**
- Type of pseudo-file for hardware and software devices
- Exception: network devices

**Characteristics:**
- Empty on disk partition when not mounted
- Entries created by udev system
- Creates and manages device nodes dynamically when devices found

**Examples:**
- `/dev/sda1` - First partition on first hard disk
- `/dev/lp1` - Second printer
- `/dev/random` - Source of random numbers

---

### The /var directory

**Purpose:**
- Contains files expected to change in size and content as system runs
- "var" stands for variable

**Common subdirectories:**
- `/var/log` - System log files
- `/var/lib` - Packages and database files
- `/var/spool` - Print queues
- `/var/tmp` - Temporary files

**Common practice:**
- Put `/var` on own filesystem
- Accommodates file growth
- Exploding file sizes don't fatally affect system

**Network services:**
- `/var/ftp` - FTP service
- `/var/www` - HTTP web service

---

### The /etc directory

**Purpose:**
- Home for system configuration files
- Contains no binary programs
- Some executable scripts present

**Examples:**
- `/etc/resolv.conf` - DNS configuration (hostname to IP address mappings)
- `passwd`, `shadow`, `group` - User account management

**Distribution differences:**
- Historically different infrastructure per distribution
- Red Hat/SUSE used `/etc/sysconfig`
- systemd brought more uniformity

**Important note:**
- `/etc` for system-wide configuration
- Only superuser can modify files
- User-specific configuration always in home directory

---

### The /boot directory

**Purpose:**
- Contains essential files for booting system

**Files per kernel (four files each):**

1. **vmlinuz** - Compressed Linux kernel (required for booting)
2. **initramfs** - Initial RAM filesystem (required for booting)
   - Sometimes called initrd
3. **config** - Kernel configuration file (debugging/bookkeeping only)
4. **System.map** - Kernel symbol table (debugging only)

**File naming:**
- Each file has kernel version appended to name

**Boot loader files:**
- GRUB (Grand Unified Bootloader)
- `/boot/grub/grub.conf` or `/boot/grub2/grub2.cfg`

---

### The /lib and /lib64 directories

**`/lib` directory:**
- Contains libraries (common code shared by applications)
- Libraries for essential programs in `/bin` and `/sbin`
- Filenames start with `ld` or `lib`
- Example: `/lib/libncurses.so.5.9`

**Library types:**
- Dynamically loaded libraries
- Also known as: shared libraries or Shared Objects (SO)

**`/lib64` directory:**
- Some distributions have separate directory
- Contains 64-bit libraries
- `/lib` contains 32-bit versions

**Kernel modules:**
- Located in `/lib/modules/<kernel-version-number>`
- Kernel code (often device drivers)
- Can be loaded/unloaded without restarting system

**Modern note:**
- Like `/bin` and `/sbin`, directories often point to those under `/usr`

---

### Removable media directories

**Purpose:**
- Make removable media accessible through regular filesystem
- Must be mounted at convenient location

**Media types:**
- USB drives
- CDs
- DVDs

**Historical location: `/media`**

**Modern location: `/run`**
- Modern distributions use `/run` directory
- Example: USB drive labeled "myusbdrive" for user "student"
  - Mount point: `/run/media/student/myusbdrive`

**The `/mnt` directory:**
- Used since early UNIX days
- For temporarily mounting filesystems
- Common uses:
  - Removable media
  - Network filesystems (not normally mounted)
  - Temporary partitions
  - Loopback filesystems (files pretending to be partitions)

---

### Additional directories under root

**Additional root directories:**

| Directory | Usage |
|-----------|-------|
| `/opt` | Optional application software packages |
| `/sys` | Virtual pseudo-filesystem with system/hardware information. Can alter system parameters and debug. |
| `/srv` | Site-specific data served up by system. Seldom used. |
| `/tmp` | Temporary files. On some distributions: erased across reboot, may be ramdisk in memory. |
| `/usr` | Multi-user applications, utilities, and data |

---

### The /usr directory tree

**Purpose:**
- Contains theoretically non-essential programs and scripts
- Not needed to initially boot system

**Subdirectories:**

| Directory | Usage |
|-----------|-------|
| `/usr/include` | Header files used to compile applications |
| `/usr/lib` | Libraries for programs in `/usr/bin` and `/usr/sbin` |
| `/usr/lib64` | 64-bit libraries for 64-bit programs |
| `/usr/sbin` | Non-essential system binaries, daemons, scripts |
| `/usr/share` | Shared data used by applications (architecture-independent) |
| `/usr/src` | Source code (usually for Linux kernel) |
| `/usr/local` | Data/programs specific to local machine (includes bin, sbin, lib, share, include, etc.) |
| `/usr/bin` | Primary directory of executable programs and scripts |

---

### Comparing files with diff

**Purpose:**
- Compare files and directories

**Common diff options:**

| Option | Usage |
|--------|-------|
| `-c` | Three lines of context before/after differing content |
| `-r` | Recursively compare subdirectories |
| `-i` | Ignore case of letters |
| `-w` | Ignore differences in spaces and tabs (whitespace) |
| `-q` | Quiet: only report if files differ (no listing of differences) |

**Basic usage:**
```bash
diff [options] <filename1> <filename2>
```

**For binary files:**
- Use `cmp` instead of `diff`

**Graphical interfaces:**
- diffuse
- vimdiff
- meld

---

### Using diff3 and patch

**diff3 command:**
- Compare three files at once
- Uses one file as reference basis for other two

**Syntax:**
```bash
diff3 MY-FILE COMMON-FILE YOUR-FILE
```

**Use case:**
- You and co-worker modified same file independently
- diff3 shows differences based on common starting file

**patch command:**
- Apply modifications distributed as patches
- Patch file contains deltas (changes) to update file

**Creating patch:**
```bash
diff -Nur originalfile newfile > patchfile
```

**Benefits:**
- More concise than distributing entire file
- Efficient for small changes (one line in 1000-line file = few-line patch)

**Applying patch:**

**Method 1 (more common):**
```bash
patch -p1 < patchfile
```
- Often used for entire directory tree

**Method 2:**
```bash
patch originalfile patchfile
```
- Used for single file

**Options:**
- See `man patch` for details on `-p1` and other options

---

### Using the file utility

**Linux file extension behavior:**
- File extension doesn't categorize nature by default
- Different from other operating systems
- `file.txt` not necessarily a text file
- Filename more meaningful to user than system

**Application behavior:**
- Applications examine file contents directly
- Don't rely on extension
- Different from Windows (`.exe` = executable)

**file utility:**
```bash
file <filename>
```

**What it does:**
- Examines file contents and characteristics
- Determines actual file type
- Identifies:
  - Plain text
  - Shared libraries
  - Executable programs
  - Scripts
  - Other types

---

### Backing up data

**Basic backup methods:**
- Simple copying with `cp`
- More robust `rsync`

**Both can:**
- Synchronize entire directory trees

**rsync advantages:**
- More efficient than cp
- Checks if file being copied already exists
- Avoids unnecessary copy if no change in size or modification time
- Saves time
- Copies only parts of files that actually changed

**cp limitations:**
- Only copy files on local machine
- Exception: NFS-mounted filesystems

**rsync capabilities:**
- Copy between machines
- Location format: `target:path`
- Target format: `someone@host`
- `someone@` optional (if remote user same as local)

**Recursive copying:**
- Use `-r` option
- Recursively walks directory tree
- Only differences transmitted over network
- Very efficient

---

### Using rsync

**Example backup command:**
```bash
rsync -r project-X archive-machine:archives/project-X
```

**Important warnings:**
⚠️ rsync can be very destructive
- Accidental misuse can harm data and programs
- Inadvertent copying where not wanted
- Specify correct options and paths carefully

**Testing first:**
- Highly recommended: use `--dry-run` option
- Ensures results match expectations before actual run

**Basic syntax:**
```bash
rsync sourcefile destinationfile
```
- Either file can be local or networked
- Contents of sourcefile copied to destinationfile

**Recommended option combination:**
```bash
rsync --progress -avrxH --delete sourcedir destdir
```

---

### Compressing data

**Why compress:**
- Save disk space
- Reduce transmission time over networks

**Linux compression methods:**

| Command | Characteristics |
|---------|----------------|
| `gzip` | Most frequently used Linux compression utility |
| `bzip2` | Produces significantly smaller files than gzip |
| `xz` | Most space-efficient compression utility in Linux |
| `zip` | Required to examine/decompress archives from other operating systems |

**Efficiency tradeoffs:**
- More efficient compression = longer time
- Decompression time doesn't vary as much

**tar utility:**
- Often used to group files in archive
- Then compress whole archive at once

---

### Compressing data using gzip

**Characteristics:**
- Historically most widely used Linux compression
- Compresses well
- Very fast

**Common usage:**

| Command | Usage |
|---------|-------|
| `gzip *` | Compresses all files in current directory. Each file compressed and renamed with `.gz` extension. |
| `gzip -r projectX` | Compresses all files in projectX directory and all subdirectories. |
| `gunzip foo` | Decompresses foo from foo.gz file. (`gunzip` = `gzip -d`) |

---

### Compressing data using bzip2

**Characteristics:**
- Similar syntax to gzip
- Different compression algorithm
- Produces significantly smaller files
- Takes longer time
- More likely used for larger files

**Common usage:**

| Command | Usage |
|---------|-------|
| `bzip2 *` | Compresses all files in current directory. Replaces each with file renamed with `.bz2` extension. |
| `bunzip2 *.bz2` | Decompresses all files with `.bz2` extension. (`bunzip2` = `bzip2 -d`) |

**Current status:**
- Recently deprecated due to lack of maintenance
- xz has superior compression ratios and active maintenance
- Should not be used for compressing new files
- Still needed to decompress existing `.bz2` files

---

### Compressing data using xz

**Characteristics:**
- Most space-efficient compression frequently used in Linux
- Choice for distributing and storing Linux kernel archives
- Trades slower compression for higher compression ratio
- Gradually becoming dominant compression method
- Especially for large files downloaded from Internet

**Common usage:**

| Command | Usage |
|---------|-------|
| `xz *` | Compresses all files in current directory. Replaces each with `.xz` extension. |
| `xz foo` | Compresses foo into foo.xz (default compression level -6). Removes foo if successful. |
| `xz -dk bar.xz` | Decompresses bar.xz into bar. Does not remove bar.xz even if successful. |
| `xz -dcf a.txt b.txt.xz > abcd.txt` | Decompresses mix of compressed and uncompressed files to stdout in single command. |
| `xz -d *.xz` | Decompresses files compressed using xz. |

**File extension:** `.xz`

---

### Handling files using zip

**Current status:**
- Rarely used to compress files in Linux
- Needed to examine/decompress archives from other systems
- Only used when:
  - Files from Windows users/environment
  - Internet downloads
- Legacy program
- Neither fast nor efficient

**Common usage:**

| Command | Usage |
|---------|-------|
| `zip backup *` | Compresses all files in current directory into backup.zip |
| `zip -r backup.zip ~` | Archives login directory and all files/directories under it into backup.zip |
| `unzip backup.zip` | Extracts all files in backup.zip to current directory |

---

### Archiving and compressing with tar

**History:**
- "tar" stood for "tape archive"
- Originally archived files to magnetic tape

**What it does:**
- Create or extract files from archive (tarball)
- Optionally compress while creating
- Decompress while extracting

**Common usage:**

| Command | Usage |
|---------|-------|
| `tar xvf mydir.tar` | Extract all files in mydir.tar into mydir directory |
| `tar zcvf mydir.tar.gz mydir` | Create archive and compress with gzip |
| `tar jcvf mydir.tar.bz2 mydir` | Create archive and compress with bz2 |
| `tar Jcvf mydir.tar.xz mydir` | Create archive and compress with xz |
| `tar xvf mydir.tar.gz` | Extract all files. No need to tell tar it's gzip format. |

**Dash usage:**
- Dash (`-`) before options often done but usually unnecessary
- Example: `tar -xvf mydir.tar` works same as `tar xvf mydir.tar`

**Separate stages (slower, wastes space):**
```bash
tar cvf mydir.tar mydir ; gzip mydir.tar
gunzip mydir.tar.gz ; tar xvf mydir.tar
```
- Creates unneeded intermediary `.tar` file
- Not recommended

---

### Compression efficiency comparison

**Demonstration factors:**
- Compression factors
- CPU time
- Archive sizes

**Key finding:**
- Higher compression factors = longer CPU time
- Producing smaller archives takes longer
- Tradeoff between time and space

---

### Disk-to-disk copying (dd)

**Purpose:**
- Make copies of raw disk space

**Example: Backup Master Boot Record (MBR):**
```bash
dd if=/dev/sda of=sda.mbr bs=512 count=1
```
- First 512-byte sector on disk
- Contains partition table

**⚠️ EXTREME WARNING:**
```bash
dd if=/dev/sda of=/dev/sdb
```
- **Deletes everything on second disk**
- Creates exact copy of first disk on second
- **Do NOT experiment with this command**
- Can erase entire hard disk

**Name origin:**
- "dd" meaning often debated
- Most popular theory: "data definition" (roots in early IBM)
- Jokes: "disk destroyer", "delete data"

---

### Chapter 11 summary

**Key concepts covered:**

- **Filesystems:**
  - "Everything is a file" philosophy
  - Tree structure starting at root (`/`)
  - Multiple filesystem types supported
  - Journaling filesystems: ext4, xfs, btrfs, jfs

- **Partitions and mounting:**
  - Filesystems occupy disk partitions
  - Mount points graft filesystems onto tree
  - `/etc/fstab` for automatic mounting
  - `df -Th` shows mounted filesystems

- **NFS:**
  - Network filesystem for sharing data
  - Server configuration in `/etc/exports`
  - Client configuration in `/etc/fstab`

- **Important directories:**
  - `/bin`, `/sbin` - Essential binaries
  - `/etc` - System configuration
  - `/var` - Variable files (logs, caches)
  - `/home` - User home directories
  - `/boot` - Boot files
  - `/proc` - Virtual kernel data
  - `/dev` - Device nodes
  - `/lib` - Shared libraries
  - `/usr` - User applications and utilities

- **File comparison:**
  - `diff` - Compare files and directories
  - `diff3` - Compare three files
  - `patch` - Apply changes from patch files
  - `file` - Determine file type

- **Backup and compression:**
  - `rsync` - Efficient file synchronization
  - `gzip` - Fast, widely used
  - `bzip2` - Deprecated, smaller than gzip
  - `xz` - Most efficient, modern choice
  - `zip` - Legacy, for cross-platform compatibility
  - `tar` - Archiving with optional compression
  - `dd` - Raw disk copying (dangerous)

---

## Feb 25, 2026

## Chapter 12: Text editors

### Learning objectives

By the end of this chapter, I should be familiar with:
- How to create and edit files using available Linux text editors
- nano, a simple text-based editor
- gedit, a simple graphical editor
- vi and emacs, two advanced editors with both text-based and graphical interfaces

---

### Overview of text editors in Linux

**Why text editors matter:**
- Need to manually edit text files for many tasks:
  - Composing email offline
  - Writing scripts (bash or other interpreters)
  - Altering system/application configuration files
  - Developing source code (C, Python, Java, etc.)

**Why not use word processors:**
- Graphical utilities can be more laborious than text editors
- More limited in capability
- Word processors add invisible formatting information
- Formatting can render system configuration files unusable
- Knowing text editors is essential skill for Linux

**Text editor choices:**
- nano (simple)
- gedit (simple graphical)
- vi (advanced)
- emacs (advanced)
- Visual Studio Code (modern IDE, not covered in detail)

**Note on Visual Studio Code:**
- Full-featured integrated development environment
- Not lightweight
- Many new Linux users already familiar from other OS
- Installation varies by distribution
- Usually requires additional package repositories

---

### Creating files without using an editor

**Why this matters:**
- Sometimes want to create short file quickly
- Useful when used from within scripts
- Can create longer files this way

**Method 1: Using echo repeatedly**
```bash
echo line one > myfile
echo line two >> myfile
echo line three >> myfile
```

**Important:**
- Single `>` sends output to file (overwrites existing file)
- Double `>>` appends to existing file

**Method 2: Using cat with redirection**
```bash
cat << EOF > myfile
> line one
> line two
> line three
> EOF
```

**Notes:**
- String to show beginning/end need not be `EOF`
- Could be `STOP` or any string not used in content
- Both techniques produce same file content
- Extremely useful when employed by scripts

---

### nano

**Characteristics:**
- Easy to use
- Requires very little effort to learn
- Text terminal-based editor

**Starting nano:**
```bash
nano <filename>
```
- If file doesn't exist, it will be created

**Interface:**
- Two-line shortcut bar at bottom lists available commands
- All help displayed on screen

**Common commands:**

| Command | Function |
|---------|----------|
| `CTRL-G` | Display help screen |
| `CTRL-O` | Write to file |
| `CTRL-X` | Exit file |
| `CTRL-R` | Insert contents from another file |
| `CTRL-C` | Show cursor position |

---

### gedit

**Characteristics:**
- Pronounced "g-edit"
- Simple-to-use graphical editor
- Only runs within graphical desktop environment
- Part of GNOME desktop system
- Similar to Windows Notepad but far more capable
- Very configurable
- Wealth of plugins available

**Starting gedit:**
- Find in desktop's menu system, OR
```bash
gedit <filename>
```
- If file doesn't exist, it will be created

**Interface:**
- Straightforward, doesn't require much training
- Composed of familiar elements

**KDE alternative:**
- kwrite (associated with KDE)
- kate (variant also supported by KDE)

---

### Visual Studio Code

**Characteristics:**
- Simple-to-use graphical editor
- Much more than text editor
- Full-featured integrated development environment

**Note:** Not covered in detail in this course

---

### Advanced editors: vi and emacs

**Why they matter:**
- Developers/administrators on UNIX-like systems almost always use one of these
- Present or easily available on all distributions
- Completely compatible across operating systems

**Key characteristics:**
- Basic text-based form (runs in non-graphical environment)
- Graphical interface forms with extended capabilities
- Steep learning curves for new users
- Extremely efficient once learned

**"Holy war":**
- Fights over which is better can be intense
- Many more users of vi than emacs
- Both here to stay

---

### Introduction to vi

**What's actually installed:**
- Usually `vim` (Vi IMproved)
- Aliased to name `vi`
- Name pronounced "vee-eye"

**Why learn vi:**
- Even if you don't want to use it, gain some familiarity
- Standard tool on virtually all Linux distributions
- May be times when no other editor available

**Graphical versions:**
- GNOME: gvim (graphical vi)
- KDE: kvim
- May be easier to use at first

**Key advantage:**
- All commands entered through keyboard
- Don't need to move hands to pointer device
- Exception: can use mouse in graphical versions

---

### vimtutor

**What it is:**
```bash
vimtutor
```
- Launches short but comprehensive tutorial
- Provides introduction with seven lessons
- Enough material to make you proficient vi user
- Covers large number of commands

**After basic lessons:**
- Look up new tricks to incorporate
- Always more optimal ways with less typing

---

### Modes in vi

**Three modes:**

| Mode | Feature |
|------|---------|
| **Command** | Default mode when vi starts. Each key is editor command. Keyboard strokes interpreted as commands to modify file contents. |
| **Insert** | Type `i` to switch from Command mode. Used to enter (insert) text into file. Indicated by "INSERT" at bottom of screen. Press `Esc` to return to Command mode. |
| **Line** | Type `:` to switch from Command mode. Each key is external command (writing file, exiting). Line editing commands inherited from older editors. Most no longer used, but some very powerful. Press `Esc` to return to Command mode. |

**Critical:**
- Vital to not lose track of which mode you're in
- Keys and commands behave differently in different modes

---

### Working with files in vi

**Start, exit, read, and write files:**

**Note:** `ENTER` key must be pressed after all these commands

| Command | Usage |
|---------|-------|
| `vi myfile` | Start editor and edit myfile |
| `vi -r myfile` | Start and edit myfile in recovery mode (from system crash) |
| `:r file2` | Read in file2 and insert at current position |
| `:w` | Write to file |
| `:w myfile` | Write out to myfile |
| `:w! file2` | Overwrite file2 |
| `:x` or `:wq` | Exit and write out modified file |
| `:q` | Quit |
| `:q!` | Quit even though modifications not saved |

---

### Changing cursor positions in vi

**Line mode commands (following `:`) require `ENTER` after command**

| Key | Usage |
|-----|-------|
| arrow keys | Move up, down, left, right |
| `j` or `<ret>` | Move one line down |
| `k` | Move one line up |
| `h` or Backspace | Move one character left |
| `l` or Space | Move one character right |
| `0` | Move to beginning of line |
| `$` | Move to end of line |
| `w` | Move to beginning of next word |
| `b` | Move back to beginning of preceding word |
| `:0` or `1G` | Move to beginning of file |
| `:n` or `nG` | Move to line n |
| `:$` or `G` | Move to last line in file |
| `CTRL-F` or Page Down | Move forward one page |
| `CTRL-B` or Page Up | Move backward one page |
| `^l` | Refresh and center screen |

---

### Searching for text in vi

**`ENTER` key should be pressed after typing search pattern**

**Commands:**

| Command | Usage |
|---------|-------|
| `/pattern` | Search forward for pattern |
| `?pattern` | Search backward for pattern |

**Navigation keys:**

| Key | Usage |
|-----|-------|
| `n` | Move to next occurrence of search pattern |
| `N` | Move to previous occurrence of search pattern |

---

### Working with text in vi

**Changing, adding, and deleting text:**

| Key | Usage |
|-----|-------|
| `a` | Append text after cursor; stop upon Escape |
| `A` | Append text at end of current line; stop upon Escape |
| `i` | Insert text before cursor; stop upon Escape |
| `I` | Insert text at beginning of current line; stop upon Escape |
| `o` | Start new line below current line, insert text; stop upon Escape |
| `O` | Start new line above current line, insert text; stop upon Escape |
| `r` | Replace character at current position |
| `R` | Replace text starting with current position; stop upon Escape |
| `x` | Delete character at current position |
| `Nx` | Delete N characters, starting at current position |
| `dw` | Delete word at current position |
| `D` | Delete rest of current line |
| `dd` | Delete current line |
| `Ndd` or `dNd` | Delete N lines |
| `u` | Undo previous operation |
| `yy` | Yank (copy) current line and put in buffer |
| `Nyy` or `yNy` | Yank (copy) N lines and put in buffer |
| `p` | Paste yanked line(s) from buffer at current position |

---

### vi command reference

**Complete command reference:**

See attached reference guide for comprehensive vi commands:
- Starting, exiting, reading, and writing files
- Changing cursor positions
- Searching for text
- Working with text (changing, adding, deleting)

---

### Using external commands in vi

**Open external command shell:**
```
sh command
```
- When you exit shell, resume editing session

**Execute command from within vi:**
```
! command
```
- Command follows exclamation point
- Best suited for non-interactive commands

**Example:**
```
:! wc %
```
- Runs `wc` (word count) on file
- `%` represents file currently being edited

---

### Introduction to emacs

**Characteristics:**
- Popular competitor for vi
- Does NOT work with modes (unlike vi)
- Highly customizable
- Large number of features
- Initially designed for console
- Soon adapted for GUI

**Additional capabilities:**
- More than simple text editing
- Can be used for email, debugging, etc.

**Command structure:**
- Uses CTRL and Meta (Alt or Esc) keys for special commands
- No separate modes like vi

---

### Working with emacs

**Starting, exiting, reading, and writing files:**

| Key | Usage |
|-----|-------|
| `emacs myfile` | Start emacs and edit myfile |
| `CTRL-x i` | Insert prompted file at current position |
| `CTRL-x s` | Save all files |
| `CTRL-x CTRL-w` | Write to file giving new name when prompted |
| `CTRL-x CTRL-s` | Save current file |
| `CTRL-x CTRL-c` | Exit after being prompted to save modified files |

**emacs tutorial:**
- Available any time within emacs
- Type `CTRL-h` (for help) then `t` (for tutorial)
- Good place to start learning basic commands

---

### Changing cursor positions in emacs

**Keys and key combinations:**

| Key | Usage |
|-----|-------|
| arrow keys | Up, down, left, right |
| `CTRL-n` | One line down |
| `CTRL-p` | One line up |
| `CTRL-f` | One character forward/right |
| `CTRL-b` | One character back/left |
| `CTRL-a` | Move to beginning of line |
| `CTRL-e` | Move to end of line |
| `Meta-f` | Move to beginning of next word |
| `Meta-b` | Move back to beginning of preceding word |
| `Meta-<` | Move to beginning of file |
| `Meta-g-g-n` | Move to line n (can also use `Esc-x Goto-line n`) |
| `Meta->` | Move to end of file |
| `CTRL-v` or Page Down | Move forward one page |
| `Meta-v` or Page Up | Move backward one page |
| `CTRL-l` | Refresh and center screen |

---

### Searching for text in emacs

**Key combinations:**

| Key | Usage |
|-----|-------|
| `CTRL-s` | Search forward for prompted pattern, or for next pattern |
| `CTRL-r` | Search backward for prompted pattern, or for next pattern |

---

### Working with text in emacs

**Changing, adding, and deleting text:**

| Key | Usage |
|-----|-------|
| `CTRL-o` | Insert blank line |
| `CTRL-d` | Delete character at current position |
| `CTRL-k` | Delete rest of current line |
| `CTRL-_` | Undo previous operation |
| `CTRL-space` or `CTRL-@` | Mark beginning of selected region (end at cursor position) |
| `CTRL-w` | Delete current marked text and write to buffer |
| `CTRL-y` | Insert at current cursor location whatever was most recently deleted |

---

### emacs command reference

**Complete command reference:**

See attached reference guide for comprehensive emacs commands:
- Working with emacs (starting, saving, exiting)
- Changing cursor positions
- Searching for text
- Working with text (changing, adding, deleting)

---

### Chapter 12 summary

**Key concepts covered:**

- **Text editors vs word processors:**
  - Text editors used for system config files, scripts, source code
  - Word processors add formatting that can break config files

- **Simple editors:**
  - **nano:** Easy-to-use text-based editor with on-screen prompts
  - **gedit:** Graphical editor similar to Windows Notepad

- **Advanced editors:**
  - **vi:** Available on all Linux systems, very widely used
  - **emacs:** Popular alternative to vi on all Linux systems
  - Both support graphical and text mode interfaces

- **Tutorials:**
  - vi: Type `vimtutor` at command line
  - emacs: Type `CTRL-h` then `t` from within emacs

- **Editor modes:**
  - **vi:** Three modes (Command, Insert, Line)
  - **emacs:** One mode, requires special keys (Control, Escape)

- **Learning curve:**
  - Both use various keystroke combinations
  - Learning curve can be long
  - Once mastered, extremely efficient

---

## Feb 27, 2026

## Chapter 13: User environment

### Learning objectives

By the end of this chapter, I should be able to:
- Use and configure user accounts and user groups
- Use and set environment variables
- Use the previous shell command history
- Use keyboard shortcuts
- Use and define aliases
- Use and set file permissions and ownership

---

### Identifying the current user

**Linux as multi-user system:**
- More than one user can log on at same time

**Identify current user:**
```bash
whoami
```

**List currently logged-on users:**
```bash
who
```

**More detailed information:**
```bash
who -a
```

---

### User startup files

**What they do:**
- Command shell (generally bash) uses startup files to configure user environment
- Global settings: `/etc` directory (for all users)
- User-specific: home directory (can include/override global settings)

**Common startup file tasks:**
- Customize the prompt
- Define command line shortcuts and aliases
- Set default text editor
- Set path for finding executable programs

---

### Order of startup files

**Login shell (first login to Linux):**

**Step 1:** `/etc/profile` read and evaluated first

**Step 2:** Search for these files in order (first found is used, rest ignored):
1. `~/.bash_profile`
2. `~/.bash_login`
3. `~/.profile`

**Note:** `~/` denotes user's home directory

**New shell/terminal window (not full login):**
- Only `~/.bashrc` file read and evaluated
- Not read during login shell
- Most distributions/users include `~/.bashrc` from within one of three user-owned startup files

**Common practice:**
- Users mostly edit `~/.bashrc`
- Invoked every time new command line shell starts
- Or when program launched from terminal window
- Other files only read when user first logs onto system

**Recent distributions:**
- Sometimes don't have `.bash_profile` and/or `.bash_login`
- Some do little more than include `.bashrc`

---

### Creating aliases

**What aliases do:**
- Create customized commands
- Modify behavior of existing commands

**Where to place:**
- Usually in `~/.bashrc` file
- Available to any command shells you create

**Remove alias:**
```bash
unalias <alias_name>
```

**List current aliases:**
```bash
alias
```

**Important syntax rules:**
- No spaces on either side of equal sign
- Alias definition needs quotes if it contains spaces

---

### Basics of users and groups

**User IDs (uid):**
- All Linux users assigned unique user ID
- Just an integer
- Normal users start with uid of 1000 or greater

**Groups:**
- Collections of accounts with shared permissions
- Used to establish users with common interests
- For access rights, privileges, security considerations

**How access works:**
- Access rights to files/devices granted based on:
  - User
  - Group they belong to

**Group membership:**
- Controlled through `/etc/group` file
- Shows list of groups and members
- Every user belongs to default (primary) group
- When user logs in, group membership set for primary group
- All members have same level of access and privilege
- Permissions can be modified at group level

**Group IDs (gid):**
- Users have one or more group IDs
- Including default one (same as user ID)
- Numbers associated with names through:
  - `/etc/passwd`
  - `/etc/group`

**Example entries:**
- `/etc/passwd`: `john:x:1002:1002:John Garfield:/home/john:/bin/bash`
- `/etc/group`: `john:x:1002`

---

### Adding and removing users

**Graphical interfaces:**
- Distributions have straightforward GUIs for user/group management

**Command line:**
- Often useful for scripts
- Only root user can add/remove users and groups

**Add new user:**
```bash
sudo useradd bjmoose
```

**Default behavior:**
- Sets home directory to `/home/bjmoose`
- Populates with basic files (copied from `/etc/skel`)
- Adds line to `/etc/passwd`: `bjmoose:x:1002:1002::/home/bjmoose:/bin/bash`
- Sets default shell to `/bin/bash`

**Remove user:**
```bash
sudo userdel bjmoose
```
- Leaves `/home/bjmoose` directory intact
- Might be useful for temporary inactivation

**Remove user and home directory:**
```bash
sudo userdel -r bjmoose
```

**Get user information:**
```bash
id                    # Current user
id bjmoose           # Specific user
```

**Example output:**
```
uid=1002(bjmoose) gid=1002(bjmoose) groups=106(fuse),1002(bjmoose)
```

---

### Adding and removing groups

**Add new group:**
```bash
sudo /usr/sbin/groupadd anewgroup
```

**Remove group:**
```bash
sudo /usr/sbin/groupdel anewgroup
```

**Add user to existing group:**

**Step 1: Check current groups**
```bash
groups rjsquirrel
```
Output: `rjsquirrel : rjsquirrel`

**Step 2: Add new group**
```bash
sudo /usr/sbin/usermod -a -G anewgroup rjsquirrel
```

**Step 3: Verify**
```bash
groups rjsquirrel
```
Output: `rjsquirrel: rjsquirrel anewgroup`

**Important:**
- Use `-a` option for append
- Avoids removing already existing groups
- Utilities update `/etc/group` as necessary

**Change group properties:**
```bash
groupmod -g <new_gid>        # Change Group ID
groupmod -n <new_name>       # Change name
```

**Remove user from group:**
- Trickier process
- `-G` option must give complete list of groups

**Example (removes all groups except rjsquirrel):**
```bash
sudo /usr/sbin/usermod -G rjsquirrel rjsquirrel
groups rjsquirrel
```
Output: `rjsquirrel : rjsquirrel`

---

### The root account

**Characteristics:**
- Very powerful
- Full access to system
- Called "administrator account" in other OS
- Often called "superuser account" in Linux

**Important warnings:**
- Must be extremely cautious before granting full root access
- Rarely, if ever, justified
- External attacks often try to elevate to root account

**Alternative: sudo**
- Assign more limited privileges to user accounts
- Only on temporary basis
- Only for specific subset of commands

---

### su and sudo

**su (switch or substitute user):**
- Launches new shell running as another user
- Must type password of user you're becoming
- Most often used to become root
- New shell allows elevated privileges until exited

**Warning about su:**
- Almost always bad practice to use `su` to become root
- Dangerous for security and stability
- Can result in:
  - Deletion of vital files
  - Security breaches

**sudo (preferred method):**
- Less dangerous than su
- Must be enabled on per-user basis by default
- Some distributions (Ubuntu) enable by default for at least one user
- Or give as installation option

---

### Elevating to root account

**Become superuser for series of commands:**
```bash
su
```
- Prompted for root password

**Execute just one command with root privilege:**
```bash
sudo <command>
```
- Return to normal unprivileged user when complete

**sudo configuration:**
- Files stored in `/etc/sudoers`
- And in `/etc/sudoers.d/` directory
- By default, `sudoers.d` directory is empty

---

### Environment variables

**What they are:**
- Quantities with specific values
- Used by command shell (bash) or other utilities/applications

**Sources:**
- Some preset by system (can usually be overridden)
- Others set by user (command line or within scripts)

**Definition:**
- Character string containing information used by one or more applications

**View environment variables:**
```bash
set
env
export
```

**Note:** `set` may print many more lines than other two methods

---

### Setting environment variables

**Scope:**
- By default, variables created in script only available to current shell
- Child processes (sub-shells) don't have access
- Use `export` command to allow child processes to see values

**Common tasks:**

| Task | Command |
|------|---------|
| Show value of specific variable | `echo $SHELL` |
| Export new variable value | `export VARIABLE=value` or `VARIABLE=value; export VARIABLE` |
| Add variable permanently | Edit `~/.bashrc` and add line `export VARIABLE=value`, then `source ~/.bashrc` or `. ~/.bashrc` or start new shell with `bash` |

**One-shot environment variables:**
```bash
SDIRS="s_0*" KROOT=/lib/modules/$(uname -r)/build make modules_install
```
- Feeds values to command for that execution only

---

### The HOME variable

**What it represents:**
- Home (or login) directory of user

**Usage:**
```bash
cd              # Changes to $HOME (no arguments)
cd ~            # Same as cd $HOME
```

**Note:** Tilde (`~`) often used as abbreviation for `$HOME`

**Example:**

| Command | Explanation |
|---------|-------------|
| `echo $HOME` | Show value of HOME (e.g., `/home/student`) |
| `cd /bin` | Change directory to `/bin` |
| `pwd` | Show current directory (displays `/bin`) |
| `cd` | Change directory with no argument |
| `pwd` | Shows `/home/student` (back to HOME) |

---

### The PATH variable

**What it is:**
- Ordered list of directories (the path)
- Scanned when command given to find appropriate program/script
- Each directory separated by colons (`:`)

**Null directory:**
- Empty directory name or `./` indicates current directory
- Examples:
  - `:path1:path2` - null directory before first colon
  - `path1::path2` - null directory between path1 and path2

**Prefix private bin directory to path:**
```bash
export PATH=$HOME/bin:$PATH
echo $PATH
```
Output: `/home/student/bin:/usr/local/bin:/usr/bin:/bin/usr`

---

### The SHELL variable

**What it contains:**
- Points to user's default command shell
- Program handling whatever you type in command window
- Usually bash
- Contains full pathname to shell

**Example:**
```bash
echo $SHELL
```
Output: `/bin/bash`

---

### The PS1 variable

**What it is:**
- Prompt Statement (PS)
- Customizes prompt string in terminal windows
- Primary prompt variable controlling command line prompt appearance

**Special characters for PS1:**

| Character | Meaning |
|-----------|---------|
| `\u` | User name |
| `\h` | Host name |
| `\w` | Current working directory |
| `\!` | History number of this command |
| `\d` | Date |

**Must be surrounded by single quotes when used**

**Example customization:**
```bash
echo $PS1                           # Show current prompt
export PS1='\u@\h:\w$ '            # Set new prompt
```
New prompt: `student@example.com:~$`

**Revert changes:**
```bash
export PS1='$ '
```

**Better practice (save and restore):**
```bash
OLD_PS1=$PS1                       # Save old prompt
# ... make changes ...
PS1=$OLD_PS1                       # Restore old prompt
```

---

### Recalling previous commands

**History buffer:**
- bash keeps track of previously entered commands
- Stored in history buffer

**Navigate history:**
- Use Up and Down cursor keys

**View command history:**
```bash
history
```
- Displays list with most recent command last

**History storage:**
- Stored in `~/.bash_history`
- With multiple terminals open, commands not saved until session terminates

---

### History environment variables

**Variables controlling history:**

| Variable | Purpose |
|----------|---------|
| `HISTFILE` | Location of history file |
| `HISTFILESIZE` | Maximum number of lines in history file (default 500) |
| `HISTSIZE` | Maximum number of commands in history file |
| `HISTCONTROL` | How commands are stored |
| `HISTIGNORE` | Which command lines can be unsaved |

**For details:** `man bash`

---

### Finding and using previous commands

**Keyboard shortcuts for history:**

| Key | Usage |
|-----|-------|
| Up/Down arrow keys | Browse through previously executed commands |
| `!!` ("bang-bang") | Execute previous command |
| `CTRL-R` | Search previously used commands (reverse intelligent search) |

**CTRL-R search:**
- Start typing, search goes back in reverse order
- Finds first command matching letters typed
- Type more letters to make match more specific

**Example:**
```
$ ^R                                    (Press CTRL-R)
(reverse-i-search)'s': sleep 1000      (Searched for 's')
$ sleep 1000                            (Press Enter to execute)
```

---

### Executing previous commands

**History substitution syntax:**

| Syntax | Task |
|--------|------|
| `!` | Start history substitution |
| `!$` | Refer to last argument in line |
| `!n` | Refer to nth command line |
| `!string` | Refer to most recent command starting with string |

**Example for `!$`:**
- Command: `ls -l /bin /etc /var`
- `!$` refers to `/var` (last argument)

**More examples:**
```bash
$ history
1. echo $SHELL
2. echo $HOME
3. echo $PS1
4. ls -a
5. ls -l /etc/ passwd
6. sleep 1000
7. history

$ !1                    # Execute command #1
echo $SHELL
/bin/bash

$ !sl                   # Execute command beginning with "sl"
sleep 1000
```

---

### Keyboard shortcuts

**Note:** Case of hotkey doesn't matter (CTRL-a = CTRL-A)

| Keyboard Shortcut | Task |
|-------------------|------|
| `CTRL-L` | Clear screen |
| `CTRL-S` | Temporarily halt output to terminal window |
| `CTRL-Q` | Resume output to terminal window |
| `CTRL-D` | Exit current shell |
| `CTRL-Z` | Put current process into suspended background, return to prompt |
| `CTRL-C` | Kill current process |
| `CTRL-H` | Works same as backspace |
| `CTRL-A` | Go to beginning of line |
| `CTRL-W` | Delete word before cursor |
| `CTRL-U` | Delete from beginning of line to cursor position |
| `CTRL-E` | Go to end of line |
| `Tab` | Auto-complete files, directories, and binaries |

**Note:** Some applications override these (e.g., emacs has other ideas for CTRL-S and CTRL-U)

---

### File ownership

**Ownership in Linux:**
- Every file associated with user (owner)
- Every file associated with group (subset of all users)
- Group has interest in file and certain rights/permissions

**Three permissions:**
- Read (r)
- Write (w)
- Execute (x)

**Utility programs for ownership and permissions:**

| Command | Usage |
|---------|-------|
| `chown` | Change user ownership of file or directory |
| `chgrp` | Change group ownership |
| `chmod` | Change permissions (can be done separately for owner, group, and others) |

---

### File permission modes and chmod

**Three kinds of permissions:**
- Read (r), Write (w), Execute (x)
- Generally represented as `rwx`

**Three groups of owners:**
- User/owner (u)
- Group (g)
- Others (o)

**Result: Three groups of three permissions:**
```
rwx: rwx: rwx
u:   g:   o
```

**Example using symbolic notation:**
```bash
ls -l somefile
-rw-rw-r-- 1 student student 1601 Mar 9 15:04 somefile

chmod uo+x,g-w somefile

ls -l somefile
-rwxr--r-x 1 student student 1601 Mar 9 15:04 somefile
```
- `u` = user (owner)
- `o` = other (world)
- `g` = group

**Numeric notation (shorthand):**
- Single digit specifies all three permission bits for each entity
- Digit is sum of:
  - 4 if read permission desired
  - 2 if write permission desired
  - 1 if execute permission desired

**Examples:**
- 7 = read/write/execute (4+2+1)
- 6 = read/write (4+2)
- 5 = read/execute (4+1)

**Using numeric notation:**
```bash
chmod 755 somefile

ls -l somefile
-rwxr-xr-x 1 student student 1601 Mar 9 15:04 somefile
```
- Three digits required (one for user, group, other)

---

### Example of chown

**Create files:**
```bash
touch file1 file2
```

**Change owner:**
```bash
sudo chown root file2       # Change owner to root (requires sudo)
```

**Change owner and group together:**
```bash
sudo chown root:root file2
```

**Note:** Only superuser can remove files owned by root

---

### Example of chgrp

**Change group ownership:**
```bash
chgrp <groupname> <filename>
```

---

### Chapter 13 summary

**Key concepts covered:**

- **Multi-user system:**
  - Linux is multi-user operating system
  - `who` - find currently logged on users
  - `whoami` - find current user ID

- **Root account:**
  - Full access to system
  - Never sensible to grant full root access to user
  - Use `sudo` for temporary root privileges on regular accounts

- **Shell startup files:**
  - bash uses multiple startup files for user environment
  - Each file affects interactive environment differently
  - `/etc/profile` provides global settings

- **Startup file advantages:**
  - Customize user's prompt
  - Set terminal type
  - Set command-line shortcuts and aliases
  - Set default text editor

- **Environment variables:**
  - Character string containing data used by applications
  - Built-in shell variables can be customized

- **Command history:**
  - `history` command recalls previous commands
  - Can be edited and recycled

- **Keyboard shortcuts:**
  - Various shortcuts available at command prompt
  - Replace long actual commands

- **Aliases:**
  - Customize commands by creating aliases
  - Add to `~/.bashrc` to make available for other shells

- **File permissions and ownership:**
  - `chmod permissions filename` - change permissions
  - `chown owner filename` - change ownership
  - `chgrp group filename` - change group ownership

---
