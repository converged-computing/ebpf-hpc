# bcc

I'm testing using bcc on a Google Cloud VM, which may or may not work. Note this is how to inspect what the kernel supports / the VM is booted with:

```bash
grep -i BPF /boot/config-`uname -r`
```
```console
CONFIG_BPF=y
CONFIG_HAVE_EBPF_JIT=y
CONFIG_ARCH_WANT_DEFAULT_BPF_JIT=y
# BPF subsystem
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
CONFIG_BPF_JIT_ALWAYS_ON=y
CONFIG_BPF_JIT_DEFAULT_ON=y
CONFIG_BPF_UNPRIV_DEFAULT_OFF=y
# CONFIG_BPF_PRELOAD is not set
CONFIG_BPF_LSM=y
# end of BPF subsystem
CONFIG_CGROUP_BPF=y
CONFIG_IPV6_SEG6_BPF=y
CONFIG_NETFILTER_BPF_LINK=y
CONFIG_NETFILTER_XT_MATCH_BPF=m
CONFIG_BPFILTER=y
CONFIG_BPFILTER_UMH=m
CONFIG_NET_CLS_BPF=m
CONFIG_NET_ACT_BPF=m
CONFIG_BPF_STREAM_PARSER=y
CONFIG_LWTUNNEL_BPF=y
# HID-BPF support
CONFIG_HID_BPF=y
# end of HID-BPF support
CONFIG_BPF_EVENTS=y
CONFIG_BPF_KPROBE_OVERRIDE=y
CONFIG_TEST_BPF=m
```

Here is how I installed bcc:

```bash
sudo apt-get install -y bpfcc-tools libbpfcc libbpfcc-dev linux-headers-$(uname -r)
```

And I cloned the bcc repository to get to "tools":

```bash
git clone https://github.com/iovisor/bcc /opt/bcc
/opt/bcc/tools
sudo python funclatency.py do_sys_open
```

But I got an error, [this one](https://github.com/iovisor/bcc/issues/3911#issuecomment-1073399703). So I tried building from source:

```bash
sudo apt-get -y install bison build-essential cmake flex git libedit-dev   llvm llvm-dev libclang-dev  zlib1g-dev 
libelf-dev zip

# Recommended for testing
sudo apt-get install -y iperf3 netperf arping

mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr
make
sudo make install
sudo rm /usr/lib/python3/dist-packages/bcc/*
sudo cp -r /opt/bcc/build/src/python/bcc-python3/bcc/* /usr/lib/python3/dist-packages/bcc/
```

Works!

```console
$ sudo python3 funclatency.py 'open*'
Tracing 11 functions for "open*"... Hit Ctrl-C to end.
^C
     nsecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 0        |                                        |
         8 -> 15         : 0        |                                        |
        16 -> 31         : 0        |                                        |
        32 -> 63         : 0        |                                        |
        64 -> 127        : 8998     |*                                       |
       128 -> 255        : 354810   |****************************************|
       256 -> 511        : 129239   |**************                          |
       512 -> 1023       : 25048    |**                                      |
      1024 -> 2047       : 6066     |                                        |
      2048 -> 4095       : 1648     |                                        |
      4096 -> 8191       : 2226     |                                        |
      8192 -> 16383      : 554      |                                        |
     16384 -> 32767      : 939      |                                        |
     32768 -> 65535      : 466      |                                        |
     65536 -> 131071     : 9        |                                        |
    131072 -> 262143     : 1        |                                        |

avg = 395 nsecs, total: 209516331 nsecs, count: 530004

Detaching...
```
