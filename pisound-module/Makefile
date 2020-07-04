# pisound Linux kernel module.
# Copyright (C) 2016  Vilniaus Blokas UAB, http://blokas.io/pisound
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; version 2 of the
# License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
#

ifeq ($(DEBUG), yes)
	ccflags-y += -DDEBUG
else
	ccflags-y += -DNDEBUG
endif

ifneq ($(KERNELRELEASE),)
	obj-m := snd_soc_pisound.o
	snd_soc_pisound-y := pisound.o
	dtbo-y += pisound.dtbo
	DTC_FLAGS ?= -@
	always := $(dtbo-y)
else
	KERNELDIR ?= /lib/modules/$(shell uname -r)/build
	PWD :=$(shell pwd)
	TARGET_PATH := kernel/sound/soc/rockchip
	INBOXDRIVER := $(shell find $(subst build,$(TARGET_PATH),$(KERNELDIR)) -name snd_soc_pisound.ko.* -type f)
#	RULEFILE = 50-usb-realtek-net.rules
#	RULEDIR = /etc/udev/rules.d/



.PHONY: modules
modules:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules

.PHONY: all
all: clean modules install

.PHONY: clean
clean:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) clean

.PHONY: install
install:
ifneq ($(shell lsmod | grep snd_soc_pisound),)
	rmmod snd_soc_pisound
endif
ifneq ($(INBOXDRIVER),)
	rm -f $(INBOXDRIVER)
endif
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install
	depmod -a  $(KERNELRELEASE)
	cp -v pisound.dtbo /boot/overlays/
	echo " " | sudo tee -a /boot/hw_intfc.conf
	echo "#pisound Need set: intfc:uart4=off intfc:spi1=on" | sudo tee -a /boot/hw_intfc.conf
	echo intfc:dtoverlay=pisound | sudo tee -a /boot/hw_intfc.conf
	sed -i 's/intfc:dtoverlay=spi1-flash/#intfc:dtoverlay=spi1-flash/g' /boot/hw_intfc.conf
endif