# SPDX-License-Identifier: GPL-2.0
#
# Makefile for MD RAID with P2PDMA support (md-p2p)
# Out-of-tree kernel module build
#
# This builds MD core and RAID personality modules with P2PDMA (Peer-to-Peer DMA) support.
# P2PDMA allows direct DMA between PCIe devices without CPU involvement.
#
# Modules use -p2p suffix to avoid conflicts with kernel's built-in MD modules.
#

# Kernel build directory
KERNELDIR ?= /lib/modules/$(shell uname -r)/build

# Current directory
PWD := $(shell pwd)

# MD P2P Core Module
obj-m += md-p2p.o
md-p2p-y := drivers/md/md.o drivers/md/md-bitmap.o

# MD RAID Personality Modules with P2PDMA support
obj-m += raid0-p2p.o
obj-m += raid1-p2p.o
obj-m += raid10-p2p.o
obj-m += raid456-p2p.o
obj-m += linear-p2p.o

# Composite module source files
raid0-p2p-y := drivers/md/raid0.o
raid1-p2p-y := drivers/md/raid1.o
raid10-p2p-y := drivers/md/raid10.o
raid456-p2p-y := drivers/md/raid5.o drivers/md/raid5-cache.o drivers/md/raid5-ppl.o
linear-p2p-y := drivers/md/md-linear.o

# Build all modules
all:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules

# Clean build artifacts
clean:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) clean
	rm -f Module.symvers modules.order

# Install modules
install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

# Help target
help:
	@echo "MD RAID with P2PDMA Support (md-p2p) - Build System"
	@echo ""
	@echo "This builds MD core and RAID personality modules with P2PDMA support."
	@echo "P2PDMA (Peer-to-Peer DMA) enables direct DMA between PCIe devices."
	@echo ""
	@echo "Modules built:"
	@echo "  md-p2p      - MD core with P2PDMA support"
	@echo "  linear-p2p  - Linear (append) mode with P2PDMA"
	@echo "  raid0-p2p   - RAID-0 (striping) with P2PDMA"
	@echo "  raid1-p2p   - RAID-1 (mirroring) with P2PDMA"
	@echo "  raid10-p2p  - RAID-10 (mirrored striping) with P2PDMA"
	@echo "  raid456-p2p - RAID-4/5/6 with parity and P2PDMA"
	@echo ""
	@echo "Targets:"
	@echo "  all      - Build all modules (default)"
	@echo "  clean    - Remove all build artifacts"
	@echo "  install  - Install modules to system (requires root)"
	@echo "  help     - Show this help message"
	@echo ""
	@echo "Variables:"
	@echo "  KERNELDIR - Kernel build directory (default: /lib/modules/\$$(uname -r)/build)"
	@echo ""
	@echo "Example usage:"
	@echo "  make                          # Build with current kernel"
	@echo "  make KERNELDIR=/path/to/kernel # Build with specific kernel"
	@echo "  sudo make install             # Install modules"
	@echo "  modprobe md-p2p               # Load MD core"
	@echo "  modprobe raid0-p2p            # Load RAID-0 personality"
	@echo ""
	@echo "Note: Module names use -p2p suffix to avoid conflicts with built-in MD."

.PHONY: all clean install help

