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
