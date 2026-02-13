# SPDX-License-Identifier: GPL-2.0
#
# Makefile for standalone MD RAID personality modules
# Out-of-tree kernel module build
#
# IMPORTANT LIMITATION:
# This builds ONLY the RAID personality modules (raid0, raid1, raid10, raid456, linear).
# The MD core (md-mod) and Device Mapper (dm-mod) are built into your kernel
# (CONFIG_MD=y, CONFIG_BLK_DEV_DM=y) and cannot be rebuilt as out-of-tree modules.
#
# These RAID modules depend on the kernel's built-in MD core.
#

# Kernel build directory
KERNELDIR ?= /lib/modules/$(shell uname -r)/build

# Current directory
PWD := $(shell pwd)

# MD RAID Personality Modules
# Note: md-mod is NOT included - using kernel's built-in MD core
obj-m += raid0.o
obj-m += raid1.o
obj-m += raid10.o
obj-m += raid456.o
obj-m += linear.o

# Composite module source files
raid0-y := drivers/md/raid0.o
raid1-y := drivers/md/raid1.o
raid10-y := drivers/md/raid10.o
raid456-y := drivers/md/raid5.o drivers/md/raid5-cache.o drivers/md/raid5-ppl.o
linear-y := drivers/md/md-linear.o

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
	@echo "MD RAID Personality Modules - Standalone Build System"
	@echo ""
	@echo "IMPORTANT: This builds ONLY RAID personality modules."
	@echo "The MD core (md-mod) is built into your kernel and cannot be rebuilt."
	@echo ""
	@echo "Modules built:"
	@echo "  linear   - Linear (append) mode"
	@echo "  raid0    - RAID-0 (striping)"
	@echo "  raid1    - RAID-1 (mirroring)"
	@echo "  raid10   - RAID-10 (mirrored striping)"
	@echo "  raid456  - RAID-4/5/6 with parity"
	@echo ""
	@echo "Targets:"
	@echo "  all      - Build all RAID personality modules (default)"
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
	@echo ""
	@echo "Kernel Configuration:"
	@echo "  CONFIG_MD=y (built-in) - MD core is in kernel, not module"
	@echo "  CONFIG_BLK_DEV_DM=y (built-in) - DM core is in kernel, not module"
	@echo ""
	@echo "To build md-mod or dm-mod as modules, you must rebuild your kernel"
	@echo "with CONFIG_MD=m and CONFIG_BLK_DEV_DM=m respectively."

.PHONY: all clean install help
