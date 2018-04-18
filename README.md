# smstools3_MT7688
This is a compiled version of SMS Server Tools 3 for the Omega2 (MediaTek MT7688). 


## Guide to compiling
This is the guide that i follow to compile the software when there is a update. You need a SD-card connected to your Omega2 to be able to compile the software as gcc taked some space.

1. Follow the guide to mount your SD-card as a overlay: https://docs.onion.io/omega2-docs/boot-from-external-storage.html
1. As gcc is not part of the normal repos you need to add a new repo to your `/etc/opkg/distfeeds.conf`. Add the following line last in that file `src/gz omega2_test http://repo.onion.io/omega2/test/packages`.
    ```
    #src/gz reboot_core http://downloads.lede-project.org/releases/17.01-SNAPSHOT/targets/ramips/mt7688/packages
    #src/gz reboot_base http://downloads.lede-project.org/releases/17.01-SNAPSHOT/packages/mipsel_24kc/base
    #src/gz reboot_onion http://downloads.lede-project.org/releases/17.01-SNAPSHOT/packages/mipsel_24kc/onion
    ## src/gz reboot_luci http://downloads.lede-project.org/releases/17.01-SNAPSHOT/packages/mipsel_24kc/luci
    ## src/gz reboot_packages http://downloads.lede-project.org/releases/17.01-SNAPSHOT/packages/mipsel_24kc/packages
    ## src/gz reboot_routing http://downloads.lede-project.org/releases/17.01-SNAPSHOT/packages/mipsel_24kc/routing
    ## src/gz reboot_telephony http://downloads.lede-project.org/releases/17.01-SNAPSHOT/packages/mipsel_24kc/telephony
    src/gz omega2_core http://repo.onion.io/omega2/packages/core
    src/gz omega2_base http://repo.onion.io/omega2/packages/base
    src/gz omega2_packages http://repo.onion.io/omega2/packages/packages
    src/gz omega2_onion http://repo.onion.io/omega2/packages/onion
    src/gz omega2_test http://repo.onion.io/omega2/test/packages
    ```
1. Now you have installed a repo where they store gcc as it is named test i guess it isn't stable but it worked ok for me. So lets move on to install gcc and make
    ```
    opkg update
    opkg install gcc
    opkg install make
    ```
1. Lets download and unpack the source code from http://smstools3.kekekasvi.com/. At the time if writing it is [smstools3-3.1.21.tar.gz](http://smstools3.kekekasvi.com/packages/smstools3-3.1.21.tar.gz).
    ```
    mkdir -p /root/src/
    cd /root/src/
    wget http://smstools3.kekekasvi.com/packages/smstools3-3.1.21.tar.gz
    tar -zxvf smstools3-3.1.21.tar.gz
    cd smstools3
    ```
1. Before we can build the source we need to make a small fix in the `Makefile`. We are going to use `vim`, if you need help on how to edit files with `vim` please read the guide [File Editing on the Omega](https://docs.onion.io/omega2-maker-kit/file-editing-on-the-omega.html).
    ```
    vi src/Makefile
    ```
    On line 6 to 9 you will find some stuff for Solaris, we don't run Solaris but we are going to use one of them. Uncomment line 9 as below. This will tell `make` to use `gcc` and not `cc` which is not installed.
    ```
    # Uncomment for Solaris
    # CFLAGS += -D SOLARIS
    # This might be also needed for Solaris:
    CC=gcc
    ```
1. Compile the software and install the binaries.
    ```
    make
    make install
    ```
1. Now it is all compiled and installed now we just need to make a few tweaks to the init.d-script and config. Open `/etc/init.d/sms3` and edit line 30 as below
    ```
    NAME=smsd
    PSOPT=""
    ECHO=echo
    ```
    As default smstools whats to keeps its SMS in `/var/spool/sms/`, but as `/var` is a link to `/tmp` which is a temp filesystem so the files we stored there would get deleted on reboot. Open `/etc/smsd.conf` and add the following rows to the first half of it
    ```
    outgoing = /sms/outgoing
    checked = /sms/checked
    incoming = /sms/incoming
    ```
    Now we need to specify which port your modem is on. If you have a `Cellular Expansion Board` for your Omega2 and it is the only expansion you have connected then try `/dev/ttyACM0`.
    ```
    [GSM1]
    device = /dev/ttyACM0
    incoming = yes
    ```
1. Turn the key and start the daemon!
    ```
    /etc/init.d/sms3 start
    ```

