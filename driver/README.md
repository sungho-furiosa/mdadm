# MD RAID Personality Modules - Standalone Build

Linux kernel MD (Multiple Devices / RAID) personality modules extracted from kernel 6.14.0 for out-of-tree compilation.

## ⚠️ Important Limitation

**This project builds ONLY the RAID personality modules** (raid0, raid1, raid10, raid456, linear).

**Why not MD core (md-mod) or Device Mapper (dm-mod)?**

Your kernel has these compiled as **built-in** (not modules):
- `CONFIG_MD=y` - MD core is built into vmlinux
- `CONFIG_BLK_DEV_DM=y` - Device Mapper core is built into vmlinux

Out-of-tree modules **cannot replace** built-in kernel components. Attempting to build md-mod or dm-mod results in "exported twice" symbol conflicts because those symbols already exist in the kernel.

**What this means:**
- ✅ You CAN build and load RAID personality modules (they use the kernel's built-in MD core)
- ❌ You CANNOT build md-mod or dm-mod as out-of-tree modules
- ❌ You CANNOT modify MD/DM core functionality without rebuilding your entire kernel

**To build md-mod/dm-mod as modules:** You must recompile your kernel with `CONFIG_MD=m` and `CONFIG_BLK_DEV_DM=m`.

## Contents

- **drivers/md/** - MD RAID driver source code
  - RAID personality modules (raid0, raid1, raid10, raid456/raid5/raid6, linear)
  - MD core source (md.c, md-bitmap.c) - for reference only, cannot be built
  - Device Mapper source - for reference only, cannot be built
  - bcache, persistent-data, dm-vdo subdirectories - for reference only
  
- **include/** - Required kernel headers (for reference)
  - linux/device-mapper.h, linux/raid/*.h
  - linux/dm-*.h headers
  - uapi headers for userspace interfaces

**Note:** Only RAID personality modules can be built. All other sources are included for reference and potential use if you rebuild your kernel with modular MD/DM support.

## Prerequisites

- Linux kernel headers installed
- GCC and make
- Kernel build tools

```bash
# Ubuntu/Debian
sudo apt-get install linux-headers-$(uname -r) build-essential

# RHEL/CentOS/Fedora
sudo yum install kernel-devel gcc make
```

## Building

Build all modules with the current running kernel:

```bash
make
```

Build for a specific kernel version:

```bash
make KERNELDIR=/lib/modules/<version>/build
```

## Installation

Install modules to the system (requires root):

```bash
sudo make install
```

## Loading Modules

After installation, load modules as needed:

```bash
# RAID levels (MD core is already loaded - built into kernel)
sudo modprobe raid0
sudo modprobe raid1
sudo modprobe raid10
sudo modprobe raid456
sudo modprobe linear

# Check loaded modules
lsmod | grep raid
```

**Note:** You do NOT need to load `md-mod` - it's already built into your kernel.

## Cleaning

Remove all build artifacts:

```bash
make clean
```

## Available Modules

### MD RAID Personality Modules (Buildable)
- **linear** - Linear (append) mode
- **raid0** - RAID-0 (striping)
- **raid1** - RAID-1 (mirroring)
- **raid10** - RAID-10 (mirrored striping)
- **raid456** - RAID-4/5/6 with parity

### Not Buildable (Built into Kernel)
- **md-mod** - MD core module (CONFIG_MD=y)
- **dm-mod** - Device Mapper core (CONFIG_BLK_DEV_DM=y)
- All Device Mapper target modules (dm-crypt, dm-snapshot, etc.)

## Project Structure

```
driver/
├── Makefile              # Main build file
├── README.md            # This file
├── drivers/
│   └── md/              # All driver source code
│       ├── *.c, *.h     # Core driver files
│       ├── bcache/      # Bcache subsystem
│       ├── persistent-data/  # Persistent data structures
│       └── dm-vdo/      # VDO deduplication
└── include/
    ├── linux/           # Kernel headers
    │   └── raid/        # RAID-specific headers
    └── uapi/linux/      # Userspace API headers
        └── raid/
```

## Notes

- This is an out-of-tree build - modules are compiled against installed kernel headers
- **Only RAID personality modules can be built** - MD core and DM are built into your kernel
- Module versions must match your running kernel for proper loading
- These modules depend on the kernel's built-in MD core (`CONFIG_MD=y`)
- To modify MD core or Device Mapper, you must rebuild your kernel with modular support
- For production use, ensure kernel compatibility and thoroughly test

### Why This Limitation Exists

When kernel components are compiled as **built-in** (`=y`), they become part of the monolithic kernel image (vmlinux). Their symbols are exported globally and cannot be overridden by loadable modules.

Out-of-tree modules can only:
1. **Add new functionality** (new drivers, new features)
2. **Replace modular components** (things built with `=m`)
3. **Extend built-in components** (like these RAID personalities extending the built-in MD core)

Out-of-tree modules cannot:
- Replace or override built-in kernel code
- Re-export symbols already in vmlinux
- Modify core kernel subsystems compiled as built-in

## Source

Extracted from: linux-source-6.14.0_6.14.0-37.37_all.deb

## License

GPL-2.0 (same as Linux kernel)
