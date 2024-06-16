# Flux eBPF

This is for flux alongside eBPF. We will also do other tutorials and learning here.
 
## Usage

```bash
limactl start --network=lima:user-v2 --name=flux-ebpf ./flux-ebpf.yaml
```

If you are coming back to it from later:

```bash
limactl start flux-ebpf
```

It says it doesn't reach running status, but I don't see any errors in the logs, and the shell works:

```bash
limactl shell flux-ebpf
export PATH=/opt/conda/bin:$PATH
```

And then try flux.

```bash
export PATH=/opt/conda/bin:$PATH
$ flux start --test-size=4
$ flux run hostname
lima-flux-lima
```

### ebpf examples

At this point we can grab the examples in this repository!

### Testing ebpf

I'm watching [this talk](https://www.youtube.com/watch?v=uBqRv8bDroc) and want to experiment with running some of the ebf tools.

#### Count Number of System Calls

In the VM note that we are running as our user, and I am going to be using `sudo` to run these as root. First, let's try tracing a raw syscall [sys_enter](https://www.kernel.org/doc/Documentation/trace/events.txt) and counting programs that run it.

```bash
sudo su
export LD_LIBRARY_PATH=/usr/lib64
bpftrace -e 'tracepoint:raw_syscalls:sys_enter {@[comm] = count(); }'
```
```console
Attaching 1 probe...
^C

@[packagekitd]: 1
@[in:imklog]: 2
@[slirp4netns]: 4
@[rs:main Q:Reg]: 4
@[cron]: 6
@[buildkitd]: 7
@[sshd]: 9
@[systemd-resolve]: 15
@[systemd-timesyn]: 15
@[systemd]: 19
@[sudo]: 20
@[multipathd]: 24
@[irqbalance]: 61
@[bpftrace]: 90
@[systemd-journal]: 111
@[lima-guestagent]: 117
@[containerd]: 158
```

In the above we are counting every system call happening in the VM!

#### Count Number of eBPF System Calls

Now let's do the same, but just look at bpf system calls:

```bash
strace -e bpf bpftrace -e 'tracepoint:raw_syscalls:sys_enter {@[comm] = count(); }'
```
```console
bpf(BPF_PROG_LOAD, {prog_type=BPF_PROG_TYPE_SOCKET_FILTER, insn_cnt=2, insns=0x7ffdbb706e20, license="GPL", log_level=0, log_size=0, log_buf=NULL, kern_version=KERNEL_VERSION(0, 0, 0), prog_flags=0, prog_name="", prog_ifindex=0, expected_attach_type=BPF_CGROUP_INET_INGRESS, prog_btf_fd=0, func_info_rec_size=0, func_info=NULL, func_info_cnt=0, line_info_rec_size=0, line_info=NULL, line_info_cnt=0, attach_btf_id=0, attach_prog_fd=0}, 116) = 3
bpf(BPF_MAP_CREATE, {map_type=BPF_MAP_TYPE_RINGBUF, key_size=0, value_size=0, max_entries=4096, map_flags=0, inner_map_fd=0, map_name="", map_ifindex=0, btf_fd=0, btf_key_type_id=0, btf_value_type_id=0, btf_vmlinux_value_type_id=0, map_extra=0}, 72) = 3
Attaching 1 probe...
bpf(BPF_MAP_UPDATE_ELEM, {map_fd=5, key=0x5630b027a530, value=0x5630b027a528, flags=BPF_ANY}, 32) = 0
```

The above (truncated) are bpf programs and maps.  We use the same call but with a different parameter to manipulate programs a map. A...

 - *eBPF programs*: are written in a restricted C that is compiled (using clang) into the eBPF bytecode. This restricted C doesn't have loops, global variables, variadic functions, floating point numbers, and passing structures as function arguments.
 - *eBPF helper functions*: are used by these programs to interact with the system (e.g., print debugging message, get the user, etc.)
 - *maps* are how we get information between our program and user space.

The programs are event driven, or driven by events in the kernel running. We "hook" into a call. This might be:

 - system calls 
 - function entry/exit
 - kernel tracepoints
 - network events (e.g,. packet arrival)

If a predefined hook doesn't exist for a need, we can create a kernel probe (kprobe) or user probe (uprobe) to attach eBPF programs almost anywhere in programs or applications.

#### Seeing eBPF Program Load

```bash
strace -e bpf,perf_event_open,ioctl bpftrace -e 'tracepoint:raw_syscalls:sys_enter {@[comm] = count(); }'
```
```
# Return file descriptor 9
bpf(BPF_PROG_LOAD, {prog_type=BPF_PROG_TYPE_TRACEPOINT, insn_cnt=29, insns=0x55abc0cf8600, license="GPL", log_level=0, log_size=0, log_buf=NULL, kern_version=KERNEL_VERSION(5, 15, 122), prog_flags=0, prog_name="sys_enter", prog_ifindex=0, expected_attach_type=BPF_CGROUP_INET_INGRESS, prog_btf_fd=8, func_info_rec_size=0, func_info=NULL, func_info_cnt=0, line_info_rec_size=0, line_info=NULL, line_info_cnt=0, attach_btf_id=0, attach_prog_fd=0, fd_array=NULL}, 128) = 9

# perf event open comes back descriptor of 7
perf_event_open({type=PERF_TYPE_TRACEPOINT, size=0 /* PERF_ATTR_SIZE_??? */, config=348, sample_period=1, sample_type=0, read_format=0, precise_ip=0 /* arbitrary skid */, ...}, -1, 0, -1, PERF_FLAG_FD_CLOEXEC) = 7

# associate 7 with 9
"This is the bpf program I want you to run when we hit that tracepoint"
ioctl(7, PERF_EVENT_IOC_SET_BPF, 9)     = 0
ioctl(7, PERF_EVENT_IOC_ENABLE, 0)      = 0
```
#### libbpfgo

Let's test out the go wrapper.

```bash
git clone https://github.com/lizrice/libbpfgo-beginners
```

And watch (and do) the [tutorial from here](https://youtu.be/uBqRv8bDroc?si=NMtq4T6MT9nJpdYY&t=898). Note that I stopped in the VM because I don't have access to performance counters, e.g.,

```
# perf list hw

List of pre-defined events (to be used in -e):
```

TODO need a way to test writing these programs, or look up others I can use.

## Clean Up

You can stop:

```bash
limactl stop flux-ebpf
```

or just nuke it!

```bash
limactl delete flux-ebpf
```
