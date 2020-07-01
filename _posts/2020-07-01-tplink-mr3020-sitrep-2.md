---
title:  "TP-Link MR3020: SITREP 0x02"
layout: post
---
A weekly log for you and me.

## What I said I'd do

My main goal this week was to work through the build errors I was getting from the GPL code dump.

I also said I'd tidy up the bind shell a bit.

## What I did

I prevailed, that's what. Admittedly, I prevailed because I set the bar rather low. It's a strategy I'm now quite fond of.

I thought I'd never get a) to the filesystem build, or b) through the filesystem build, so I invite you to gaze upon this moneyshot:

```bash
building FS ...
Parallel mksquashfs: Using 2 processors
Creating 4.0 filesystem on ../rootfs.MR3020V3, block size 131072.
[===========================================================================================================================\] 129/129 100%
Exportable Squashfs 4.0 filesystem, xz compressed, data block size 131072
	compressed data, compressed metadata, compressed fragments, compressed xattrs
	duplicates are removed
Filesystem size 2418.21 Kbytes (2.36 Mbytes)
	29.55% of uncompressed filesystem size (8184.37 Kbytes)
Inode table size 2296 bytes (2.24 Kbytes)
	23.82% of uncompressed inode table size (9638 bytes)
Directory table size 2698 bytes (2.63 Kbytes)
	53.73% of uncompressed directory table size (5021 bytes)
Number of duplicate files found 1
Number of inodes 303
Number of files 89
Number of fragments 30
Number of symbolic links  92
Number of device nodes 89
Number of fifo nodes 0
Number of socket nodes 0
Number of directories 33
Number of ids (unique uids + gids) 1
Number of uids 1
	root (0)
Number of gids 1
	root (0)
test -d /home/lol/Desktop/TL-MR3020V3/build/../targets/image_MR3020V3/ || mkdir -p /home/lol/Desktop/TL-MR3020V3/build/../targets/image_MR3020V3/
/home/lol/Desktop/TL-MR3020V3/build/../host_tools/imageTool/mkimage2 -s 0x800000 -m "0x150000" -e 1 -t 7 -p /home/lol/Desktop/TL-MR3020V3/build/../targets/fs.MR3020V3/../reduced_data_model_plaintext_MR3020V3.xml -b /home/lol/Desktop/TL-MR3020V3/build/../targets/mtk_ApSoC_4320_boot/boot_MR3020V3.bin -k /home/lol/Desktop/TL-MR3020V3/build/../targets/mtk_ApSoC_4320_mt7628_kernel_MR3020V3/linux.7z -f /home/lol/Desktop/TL-MR3020V3/build/../targets/rootfs.MR3020V3 -v /home/lol/Desktop/TL-MR3020V3/build/../targets/mtk_ApSoC_4320_mt7628_kernel_MR3020V3/vmlinux -i /home/lol/Desktop/TL-MR3020V3/build/../targets/image_MR3020V3
YangXv <yangxu@tp-link.com.cn> modified from trx for make image
mtd_type=7
appSize	is 7a0000
bootOffset	is 0
kernelOffset	is 20000
rootfsOffset   is 170000
hwid is CCCC2233445566778899AABBCCDDEEFF
fwid is 00112233445566778899AABBCCDDEEFF
oemid is FFFF2233445566778899AABBCCDDEEFF
ModelName   is TL-MR3020
product_id  is 0x89741003
product_ver is 0x00000001
swRevision  is 0x55aa000b
platformVer is 0xa5000901
addHwVer    is 0x00000001
specilaVer  is 0x00000000

devModelVer is v3
buildDate   is 200630

##target cpu is little endian
bootloader is 0x200
kernel is 0x20400
kernel_len(0x127a1c), rootfsOffset(0x170000),rootfs_len(0x170200)
rootfs is 0x170200
Entry Point: 0X8000C150
Text Addr: 0X80000000
kernelTextAddr(80000000),kernelEntryPoint(8000c150)
```

Specifically:

- I did, in fact, get the build process working. I had to sift through a lot of Makefiles and source to understand what was going on, and what was supposed to be going on. It was great fun and I feel like I learned heaps in the process.
- I also made a script that manages all the dependencies, file changes, manual compilation steps and subsequent build process for this firmware. I haven't tested the script on any other boxes yet so I have decided to post it later, once I have made sure it functions for others. But hopefully it saves someone a bit of time at some point.
- I updated the bind shell. It doesn't die with the socket connection now so it can run indefinitely (?) on the router. This seemed easier than daemonizing it properly and/or rebooting it every time I want to reconnect. I don't know whether it's preferable to post the code here, so it's available in-line as you browse, [or to just put a link to the file on GitHub](https://github.com/unmeg/hax/blob/master/shells/multi_conn_shell.c) , so I'll do both.

    ```c
    #include <stdio.h>
    #include <string.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <arpa/inet.h>
    #include <sys/socket.h>
    #include <sys/types.h>
    #include <netinet/in.h>

    /*
    mipshell-pfieffer by mw \0/
    */

    void error(char *err_msg)
    {
        perror(err_msg);
        exit(1);
    }

    int main(){
        char hello_buf[] = "henlo!\n";
        int host_sockid, client_sockid;
        int port = 9999;
        struct sockaddr_in host_addr, cli_addr;

        socklen_t size_host_addr = sizeof(host_addr);
        socklen_t size_cli_addr = sizeof(cli_addr);

        bzero((char *) &host_addr, size_host_addr);
        bzero((char *) &cli_addr, size_cli_addr);

        if(fork() == 0){
            // create the socket
            host_sockid=socket(PF_INET, SOCK_STREAM, 0); // tcp

            // set up the sockaddr_in struct so we can bind the socket to it
            host_addr.sin_family = AF_INET;
            host_addr.sin_port = htons(port);
            host_addr.sin_addr.s_addr = htonl(INADDR_ANY); // ip

            // bind, listen, accept:

            if(bind(host_sockid, (struct sockaddr*) &host_addr, size_host_addr) < 0){
                error("BIND ERROR");
            }

            if(listen(host_sockid, 5) < 0){ // magic number 5 is just a random choice i made for backlog
                error("LISTEN ERROR");
            }

            while(1){
            client_sockid = accept(host_sockid, (struct sockaddr*) &cli_addr, &size_cli_addr);
            if(client_sockid < 0){
                error("ACCEPT ERROR");
            }

            if(fork() == 0){
                close(host_sockid);
                write(client_sockid, hello_buf, strlen(hello_buf));

                // clone STDIN STDOUT STDERR
                dup2(client_sockid, 0);
                dup2(client_sockid, 1);
                dup2(client_sockid, 2);

                // pick your poison.
                char* arg[] = {"/bin/busybox","sh",NULL};
                execve("/bin/busybox", arg, NULL);
                // execve("/bin/bash", NULL, NULL);

                return 0;
            } else { // not child
                close(client_sockid);
                } // end else
            } // end while
        } // end if child outer
        return 0;
    } 
    ```

## Excessive details

### os_libs

There were a few issues with /apps/public/os_libs, and they manifested as issues building the wireless apps/modules. Eventually, I realised that I had to:

- Remove a reference in os_msg.h to dm_paramLen.h (which was symlinked to a file that didn't exist). I'm still not sure what this file was meant to contain, or do, and I didn't find any other files with that name on the internerd. Commenting out the include appeared to fix things, but mark this down as a potential bug.
- Fix the Makefile to use the correct compiler/stripper (it was not building the .so for MIPS and was ambiguously erroring on the stripping step, which messed with things too). This was by far the thing that took me longest to identify.
- Actually add the library compilation to the build process.

Reading through all the connected documents to understand what needed to happen for libos probably took up the majority of my debugging time. Somehow that made it hilarious when I finally figured out the few, tiny things that needed tweaking.

### fakeroot

After the libos issue, I was pretty happy when I saw this error: 

```bash
fakeroot: preload library not found, aborting.
```

"Great," I thought, "I love this error. It's clear, it's perfect, it's straightforward. It can't find the library. I am sure I can pass the library path somewhere!"

You can, of course. Fakeroot is a script with hardcoded paths. The Makefile for the firmware edited that script to hit the correct directory, but it was pointing at libfakeroot.a and not libfakeroot-0.so. Quick, hacky fix.

### libtool

I didn't note this down (FOOL) but I remember that alongside the fakeroot error, I was dealing with a libtool error. It was actually quite interesting — if I ran the build_fs make stage a few times in a row, I'd get changing errors each time. It mostly flipped between libtool and fakeroot, so it was a simultaneous investigation and solve. So I don't know — others may or may not need to solve this. I never figured out how critical it was. The fix was easy though.

I read [about the shell bug](https://lists.gnu.org/archive/html/bug-libtool/2007-12/msg00016.html) and I got rid of it:

```bash
sudo bash -c "ln -s /bin/bash /bin/sh"
```

### Misc

There were a bunch of packages (byacc, bison, libusb, texinfo, flex, and so on) that I grabbed along the way (mostly before I committed to taking better notes) and some ad-hoc edits I did here and there (again, mostly before I committed to taking notes). Hopefully when I test out my build script later I will be able to better account for all of those little things for lookers-on!

## What next?

I'm not planning much for this project over the next week. I have a lot of other stuff on.

If I can get a spare moment, my focus will be on testing the validity of my perverted firmware. I haven't flashed it yet because I wanted to poke at the router in as-is condition for a bit first. I've kind of ignored it so far.

Anyway, once I run the firmware in an emulator, I will add the bindshell to the filesystem and edit the initialisation script to boot it up. Then I'll flash and take bets on bricking.

I'm also pretty keen to write up a skeleton kernel module.

I have a lot of ODEs to (remember how to) do though, so I make no promises. Math before hax.
