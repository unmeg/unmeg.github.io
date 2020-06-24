---
title:  "TP-Link MR3020: SITREP 0x01"
layout: post
---
A weekly log for you and me.

## What I said I'd do

Let's call this week one. The goal was easy peasy: Just backdoor the firmware! No big deal.

The plan was to begin with a n00b run: use [firmware mod kit](https://github.com/rampageX/firmware-mod-kit/wiki) to unpack the firmware, add a basic bindshell binary and set the init script to trigger it, then rebuild the firmware and flash it via the web UI. If that worked, I'd move on to writing a kernel module. 

I thought I'd probably get through that pretty quickly, and maybe be able to deploy the kernel module similarly (FMK and so forth). 

I even thought that I might have time this week to — as an academic exercise — recreate the final firmware image manually (i.e. without FMK) using the GPL code on the TPLINK website.

Cute.

## What I did

- I grabbed the GPL code and dipped into the bundled toolchain to compile my C bindshell for MIPS, in casual anticipation of BACKDOORAGE.
- I wanted to test that it worked, so I got distracted by [QEMU+Docker emulation](https://megwhite.com.au/emulating-firmware-docker-qemu/).
- I discovered the firmware was not compatible with FMK (both new and old scripts) while trying to extract it for tampering. I didn't think it would matter too much as I already had the GPL dump, so I could just build everything from scratch with only slightly more effort, right (:smile:)!? No.
    - I spent a bunch of time (spread out over a few days) doctoring build scripts/source so that things would compile on my normal VM (which was not the same environment as the GPL README mentioned). I hit a wall.
    - I then set up a build environment that matched the README and tried again. New errors! Solved them, and ended up stuck at the same place as before.

## Excessive details

For those playing at home, here's where I'm at:

```bash
sudo make MODEL=MR3020V3 apps_build
build wireless tools
#cd /home/lol/TL-MR3020V3/build/../mtk_ApSoC_4320/modules/public/wireless_tools.29 &&  make realclean && make all;\
	cd /home/lol/TL-MR3020V3/build/../mtk_ApSoC_4320/modules/public/wireless_tools.29 && make all;	
make[1]: Entering directory `/home/lol/TL-MR3020V3/mtk_ApSoC_4320/modules/public/wireless_tools.29'
mipsel-linux-gcc -L/home/lol/TL-MR3020V3/mtk_ApSoC_4320/modules/public/wireless_tools.29/../../../../apps/public/os_libs/include//../ -los -lrt -pthread  -Os -W -Wall -Wstrict-prototypes -Wmissing-prototypes -Wshadow -Wpointer-arith -Wcast-qual -Winline -I. -I/home/lol/TL-MR3020V3/mtk_ApSoC_4320/modules/public/wireless_tools.29/../../../../apps/public/os_libs/include/ -DINCLUDE_LAN_WLAN -MMD      -o iwconfig iwconfig.o libiw.so.29 -lm
/opt/buildroot-gcc463/usr/lib/gcc/mipsel-buildroot-linux-uclibc/4.6.3/../../../../mipsel-buildroot-linux-uclibc/bin/ld: skipping incompatible /home/lol/TL-MR3020V3/mtk_ApSoC_4320/modules/public/wireless_tools.29/../../../../apps/public/os_libs/include//..//libos.so when searching for -los
/opt/buildroot-gcc463/usr/lib/gcc/mipsel-buildroot-linux-uclibc/4.6.3/../../../../mipsel-buildroot-linux-uclibc/bin/ld: cannot find -los
libiw.so.29: warning: gethostbyname is obsolescent, use getnameinfo() instead.
collect2: ld returned 1 exit status
make[1]: *** [iwconfig] Error 1
make[1]: Leaving directory `/home/lol/TL-MR3020V3/mtk_ApSoC_4320/modules/public/wireless_tools.29'
make: *** [wirelesstool] Error 2
```

The relevant slice of Makefile:

```bash
ifeq ($(strip $(SUPPLIER)),mtk_ApSoC_4120)
        @echo "build wireless toolsBLOOP"
        cd $(WIRELESSTOOLS) &&  make realclean && make all;\
        cp -f $(WIRELESSTOOLSLIB) $(INSTALL_WIRELESS)/lib;\
        cp -f iwpriv $(INSTALL_WIRELESS)/sbin;\
        cp -f wlNetlinkTool $(INSTALL_WIRELESS)/sbin

        #@echo "build wireless tools"
        #cd $(WIRELESSTOOLS)/../sysstat-9.0.6 &&  make clean && make mpstat;\
        #cp -f mpstat $(INSTALL_WIRELESS)/sbin

        @echo "build ated for QA"
        cd $(WIRELESSTOOLS)/../ated &&  make clean && make all;\
        cp -f ated $(INSTALL_WIRELESS)/sbin

        echo "Build the rt2860apd for 802.1x"
        cd $(WIRELESSTOOLS)/../8021x &&  make clean && make all;\
        cp -f rt2860apd $(INSTALL_WIRELESS)/sbin/;\
        ln -sf rt2860apd $(INSTALL_WIRELESS)/sbin/rtinicapd

        echo "Build wscd for wps while coexist of dual-band"
        cd $(WIRELESSTOOLS)/../wsc_upnp/wsc_upnp && chmod -R +rx ./ && make clean && make all;\
        cp -f wscd $(INSTALL_WIRELESS)/sbin/;\
        ln -sf wscd $(INSTALL_WIRELESS)/sbin/wscd_5G
endif
```

And if we jump into the wireless tools Makefile:

```bash
SRC = os_linux.c os_msgLinux.c os_log.c 
FOBJ = libos.so 
 
OBJS = $(SRC:.c=.o) 
CFLAGS = -D__LINUX_OS_FC__ -fPIC -I./include/ $(DF_FLAGS)

$(FOBJ).so: $(OBJS)
	$(CC) $(CFLAGS) -shared $(OBJS) -o $@
	$(STRIP) $@

sinclude $(SRC:.c=.d) 

%.o: ./src/%.c
	$(CC) $(CFLAGS) -c -o $@ $<

%.d: ./src/%.c
	@$(CC) -MM $(CFLAGS) $< > $@.$$$$; \
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
	rm -f $@.$$$$

.PHONY : clean
clean :
	$(RM) $(FOBJ) $(SRC:.c=.d) $(OBJS) 
lol@ubuntu:~/TL-MR3020V3/apps/public/os_libs$
```

## What is up next

Having reviewed my actual timeline and recalibrated my productivity expectations accordingly, I pledge only to solve my build errors between now and next Wednesday. I'll flash the new firmware, too.

Side quest the first: I'll probably tidy up the extremely generic bindshell (i.e. take the word 'lol' out of every variable) too and chuck it on the internet, because we obviously need more of those.

Side quest the second: I'll pay more attention to documenting things as I go. I have been taking notes in Notion and just realised today that I could export directly from Notion to Markdown, which basically makes me giddy.
