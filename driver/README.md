# MD RAID with P2PDMA Support (md-p2p)

Linux kernel MD (Multiple Devices / RAID) driver with **P2PDMA (Peer-to-Peer DMA) support**, extracted from kernel 6.14.0 for out-of-tree compilation.

> **Part of mdadm project**: This driver is integrated into the [mdadm](https://github.com/sungho-furiosa/mdadm) repository for unified RAID management and development.

## 🚀 What is md-p2p?

**md-p2p** is a modified version of the Linux MD RAID driver that enables **PCI Peer-to-Peer DMA** (P2PDMA) support. P2PDMA allows direct memory transfers between PCIe devices without CPU involvement, significantly improving performance for high-speed storage scenarios.

### Key Features
- ✅ **Full MD core with P2PDMA support** (`md-p2p.ko`)
- ✅ **All RAID personalities with P2PDMA** (raid0-p2p, raid1-p2p, raid10-p2p, raid456-p2p, linear-p2p)
- ✅ **Direct PCIe-to-PCIe transfers** - Bypass CPU for RAID operations between NVMe devices
- ✅ **No kernel conflicts** - Uses `-p2p` suffix to coexist with built-in MD modules

### Use Cases
- **High-performance NVMe RAID** - Direct data transfers between NVMe SSDs
- **GPU-Direct Storage** - RAID arrays accessible by GPUs via P2PDMA
- **Low-latency RAID** - Reduced CPU overhead for storage-intensive workloads
- **Research & Development** - Testing P2PDMA optimizations in RAID subsystems

## 📦 Modules Built

All modules use the `-p2p` suffix to avoid conflicts with kernel's built-in MD:

| Module | Description |
|--------|-------------|
| `md-p2p.ko` | MD core with P2PDMA support |
| `linear-p2p.ko` | Linear (append) mode with P2PDMA |
| `raid0-p2p.ko` | RAID-0 (striping) with P2PDMA |
| `raid1-p2p.ko` | RAID-1 (mirroring) with P2PDMA |
| `raid10-p2p.ko` | RAID-10 (mirrored striping) with P2PDMA |
| `raid456-p2p.ko` | RAID-4/5/6 with parity and P2PDMA |

## 📋 Prerequisites

### System Requirements
- Linux kernel 6.14.0 or compatible
- Kernel with `CONFIG_PCI_P2PDMA=y` enabled
- PCIe devices supporting P2PDMA (e.g., NVMe controllers with CMB)
- PCIe topology supporting peer-to-peer transfers (devices behind same PCIe bridge)

### Build Dependencies
```bash
# Ubuntu/Debian
sudo apt-get install linux-headers-$(uname -r) build-essential

# RHEL/CentOS/Fedora
sudo yum install kernel-devel gcc make
```

### Verify P2PDMA Support
```bash
# Check kernel config
grep CONFIG_PCI_P2PDMA /boot/config-$(uname -r)
# Should show: CONFIG_PCI_P2PDMA=y

# Check for P2PDMA-capable devices
lspci -vv | grep -i "p2p\|peer"
```

## 🔨 Building

### Standalone Build

Build all modules from this directory:

```bash
cd driver/
make
```

### Integrated Build (from mdadm root)

Build driver modules from the top-level mdadm directory:

```bash
# Build only kernel driver modules
make driver

# Build userspace mdadm + driver modules
make all-with-driver

# Clean driver build artifacts
make driver-clean

# Install driver modules to system
sudo make driver-install
```

### Kernel-Specific Build

Build for a specific kernel version:

```bash
make KERNELDIR=/lib/modules/<version>/build
```

View build options:

```bash
make help
```

## 📥 Installation

Install modules to the system (requires root):

```bash
sudo make install
```

This installs modules to `/lib/modules/$(uname -r)/extra/`.

## 🚀 Loading Modules

Load MD core first, then RAID personalities:

```bash
# Load MD P2P core
sudo modprobe md-p2p

# Load desired RAID personality
sudo modprobe raid0-p2p    # For RAID-0
sudo modprobe raid1-p2p    # For RAID-1
sudo modprobe raid10-p2p   # For RAID-10
sudo modprobe raid456-p2p  # For RAID-4/5/6
sudo modprobe linear-p2p   # For linear mode

# Verify modules are loaded
lsmod | grep p2p
```

### Creating RAID Arrays

Use `mdadm` as usual - the P2PDMA functionality is transparent:

```bash
# Create RAID-0 array with P2PDMA support
sudo mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/nvme0n1 /dev/nvme1n1

# Create RAID-1 array
sudo mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/nvme2n1 /dev/nvme3n1
```

**Note:** P2PDMA is automatically used when:
1. Both devices support P2PDMA
2. Devices are behind the same PCIe root complex
3. Kernel has `CONFIG_PCI_P2PDMA=y`

## 🧹 Cleaning

Remove all build artifacts:

```bash
make clean
```

## 📂 Project Structure

```
driver/
├── Makefile              # Main build file with P2PDMA support
├── README.md            # This file
├── .gitignore           # Excludes build artifacts
├── drivers/
│   └── md/              # All driver source code
│       ├── md.c         # MD core (P2PDMA-enabled)
│       ├── md-bitmap.c  # Bitmap management
│       ├── raid0.c      # RAID-0 implementation
│       ├── raid1.c      # RAID-1 implementation
│       ├── raid5.c      # RAID-4/5/6 implementation
│       └── ...          # Other MD/DM sources (for reference)
└── include/
    ├── linux/           # Kernel headers
    │   ├── pci-p2pdma.h # P2PDMA API headers
    │   └── raid/        # RAID-specific headers
    └── uapi/linux/      # Userspace API headers
```

## 🔍 Contents

- **drivers/md/** - MD RAID driver source code
  - MD core (md.c, md-bitmap.c) - **NOW BUILDABLE with P2PDMA**
  - RAID personalities (raid0, raid1, raid10, raid456, linear) - All with P2PDMA
  - Device Mapper source - for reference only (built-in to kernel)
  - bcache, persistent-data, dm-vdo subdirectories - for reference

- **include/** - Required kernel headers (for reference)
  - linux/pci-p2pdma.h - P2PDMA API
  - linux/device-mapper.h, linux/raid/*.h
  - uapi headers for userspace interfaces

## ⚙️ P2PDMA Technical Details

### How P2PDMA Works in md-p2p

1. **Device Discovery**: MD detects P2PDMA-capable devices via `pci_p2pdma_distance()`
2. **Path Validation**: Checks if devices can perform peer-to-peer transfers
3. **Buffer Allocation**: Uses `pci_alloc_p2pmem()` for P2PDMA-enabled buffers
4. **Direct Transfers**: DMA operations bypass CPU, going directly between devices
5. **Fallback**: Automatically falls back to normal DMA if P2PDMA unavailable

### P2PDMA Requirements

For P2PDMA to work, devices must:
- Be connected to the same PCIe root complex
- Support P2PDMA in hardware (e.g., NVMe with Controller Memory Buffer)
- Not be separated by non-transparent bridges or complex PCIe switches

### Checking P2PDMA Status

```bash
# Check if devices can do P2P transfers
cat /sys/block/md0/md/array_state

# View P2PDMA statistics (if available)
cat /sys/class/block/md0/device/p2pdma_*
```

## 🆚 Differences from Built-in MD

| Feature | Built-in MD (`md-mod`) | md-p2p |
|---------|----------------------|--------|
| P2PDMA Support | ❌ No | ✅ Yes |
| Module Name | `md-mod` | `md-p2p` |
| RAID Names | `raid0`, `raid1`, ... | `raid0-p2p`, `raid1-p2p`, ... |
| Symbol Conflicts | N/A (built-in) | ✅ Avoided with `-p2p` suffix |
| Can Coexist | N/A | ✅ Yes (different module names) |

## 🚨 Important Notes

### Why Module Renaming?

The kernel has MD core built-in (`CONFIG_MD=y`). Out-of-tree modules **cannot replace** built-in components due to symbol conflicts. By using `-p2p` suffix:
- ✅ No symbol conflicts with built-in `md-mod`
- ✅ Can load alongside or instead of built-in MD
- ✅ Clear indication of P2PDMA capability

### When to Use md-p2p vs Built-in MD

**Use md-p2p when:**
- You need P2PDMA support for high-performance NVMe RAID
- You want direct PCIe-to-PCIe transfers
- You're testing P2PDMA optimizations
- You have NVMe devices with CMB or similar P2PDMA-capable hardware

**Use built-in MD when:**
- Standard RAID is sufficient
- P2PDMA is not required
- Maximum compatibility is needed

### Limitations

- ⚠️ **Experimental**: P2PDMA support is relatively new in Linux kernel
- ⚠️ **Hardware-dependent**: Requires P2PDMA-capable devices and PCIe topology
- ⚠️ **Kernel version**: Built for kernel 6.14.0 - may need adjustments for other versions
- ⚠️ **Testing required**: Thoroughly test before production use

### Out-of-tree Build Constraints

This is an out-of-tree build, which means:
- ✅ Can add new functionality (P2PDMA support)
- ✅ Can be loaded alongside built-in modules (with different names)
- ✅ Can modify MD core behavior
- ❌ Must match kernel version for symbol compatibility
- ❌ Requires kernel headers at build time

## 🐛 Troubleshooting

### Module Won't Load
```bash
# Check kernel ring buffer for errors
dmesg | tail -50

# Verify kernel version matches
modinfo md-p2p

# Check dependencies
modprobe -D md-p2p
```

### P2PDMA Not Working
```bash
# Verify P2PDMA support in kernel
grep P2PDMA /boot/config-$(uname -r)

# Check PCIe topology
lspci -tv

# Verify devices support P2PDMA
lspci -vv | grep -A 20 "NVMe"
```

### Symbol Conflicts
If you see "symbol exported twice" errors:
- Ensure you're using modules with `-p2p` suffix
- Check that both built-in and modular MD aren't conflicting
- Use `lsmod | grep md` to see which MD modules are loaded

## 📚 Additional Resources

- [Linux P2PDMA Documentation](https://www.kernel.org/doc/html/latest/driver-api/pci/p2pdma.html)
- [MD RAID Documentation](https://raid.wiki.kernel.org/)
- [mdadm Manual](https://linux.die.net/man/8/mdadm)
- [NVMe P2PDMA](https://nvmexpress.org/)

## 📄 Source

Extracted from: `linux-source-6.14.0_6.14.0-37.37_all.deb`

Modified to add P2PDMA support with `-p2p` module naming.

## 📜 License

GPL-2.0 (same as Linux kernel)

## 🤝 Contributing

This is a research/development project. For production use:
1. Thoroughly test in your environment
2. Verify P2PDMA hardware compatibility
3. Monitor kernel compatibility on updates
4. Consider upstreaming P2PDMA improvements to mainline kernel
