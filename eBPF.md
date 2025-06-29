eBPF
====
https://cilium.isovalent.com/hubfs/Learning-eBPF%20-%20Full%20book.pdf
63

All codes are available here - https://github.com/lizrice/learning-ebpf/tree/main


- extended Berkley Packet Filtering
- Unlike the acronym, it does more than that
- dynamically program kernel for efficient networking, observability, tracing and security
- run custom code in kernel
- apps in user space does syscalls to kernel. Kernel listens to these events

eBPF programms can be attached to different events
- Kprobes
- Uprobes
- Tracepoints
- Network packets
- Linux security module

Programmable kernel in kubernetes
- In k8s -> app -> container -. pod
- Pods run on a host, which share one kerneljjj
- ebPF can instrument the kernel. SO, it can change its behaviour

Small packet filter bpf program

```
ldh [12]
jeq #ETHERTYPE IP, L1, L2
L1: ret #TRUE
L2: ret #0
```

It checks if the packet received is an ip packet or not
- ldh = load 12th byte
- jeq = compares type
- L1 return true if it is
- L2 returns 0


Syscalls
- Any user application does syscall to kernel
- To create a new process, to read a file, write a file etc

 Ebpf probes
 - we have bpf() syscall which can be used by apps
 - eBPF has a feature called kprobes (kernel probes) which is used for tracing
 - Netflix heavily uses eBPF
 - Cilium started using eBPF to skip container datapath
 - Facebook created Katran which is a L4 loadbalancer which uses eBPF.  Every packet from facebook.com goes through this

eBPF for networking (https://www.youtube.com/watch?v=99jUcLt3rSk)
- Looks like a c code
- clang (LLVM) to convert code to object file (like foo.o)
- BPF loader to load the code
- Sent to BPF verifier in kernelspace
- JIT to convert to machine code
- BPF maps to directly interact in userspace

- eBPF cannot crash kernel
- is as fast as kernel module
- stable API guarentees
- eBPF can work on any kernel version


TC = Traffic Control
OVS = Open vSwitch
netfilter (iptables, ipvs, nftables)

eBPF creates its own datapath - from eBPF program - thats what cilium does

eBPF became its own kernel subsystem to ease continual growth patch management
Patches are applied to bpf and bpf-next kernel trees on git.kernel.org
Pull requests go to Linus Torvals via David S Miller (networking maintainer)

bpftool , libbpf tools for introspection, debugging and usersapce API for applications
bpfiletr to translate iptables rulestes into BPF via user mode driver
bpftrace for performance monitoring
cilium replaces iptables based kube-proxy through BPF
eBPF to LSM Linux Security Module
cilium added XDP based service load balancing capabilities
Facebook added support for BPF based TCP congeston control modules

 
strace -c echo "hello" -> To view system calls

- Accepting code to kernel and making it to production is challenging and may take years
- Other way is to use kernel modules. It can be used to extend kernel behaviour. But this is full-on kernel programming. It may crash whole kernel if code isnt good
- eBPF can run code in kernel and its safe because of eBPF verifier. It ensures the eBPF program loaded is safe to run and it wont crash machine or lock it up in a hard loop
- eBPF can be dynamically loaded. It is attached to an event. Ex: It can be attcahed to syscall which opens files and it will be triggered whenever this sycall occurs. It doesn’t matter whether that process was already running when the program was loaded
- eBPF doesnt require a reboot of kernel
- Istio is a sidecar model where sidecar is injected based on annotations. A malicious actor can easily deploy an app like crypto mining and you may even not able to know it as it doesnt have istio sidecar
- eBPF can know about it as it runs on host kernel


```
VishnuKvs/Workspace/Rough ❯ cat hello.py
#!/usr/bin/python

from bcc import BPF

program = r"""
int hello(void *ctx){
    bpf_trace_printk("Hello World");
    return 0;
}
"""

b = BPF(text=program)
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")

b.trace_print()
```

- sudo python3 hello.py
- Run in a linux vm like ubuntu
- The last line is infinite loop
- Whenever there is a syscall event which is `execve` , this hello() will be executed
- You can test with running `strace -c echo "Hello"`, which will use execve.

- eBPF needs sudo permissions to execute
- CAP_PERFMON and CAP_BPF are both required to load tracing programs.
- CAP_NET_ADMIN and CAP_BPF are both required for loading networking programs.
- https://mdaverde.com/posts/cap-bpf/

- Kernel ebpf programs sends output to this location - /sys/kernel/debug/tracing/trace_pipe
- Better way for userspace programs to capture output of ebpf programs is by using BPF maps
- Maps can be used to share data among multiple eBPF programs or to communicate between a user space application and eBPF code running in the kernel.
- maps are key value stores. Some are hash tables, perf, ring buffers, arrays etc
- Check kernel version (BPF_MAP_TYPE_BLOOM_FILTER was introduced in kernel version 5.16)
- Some eBPF map types hold information about specific types of objects. For example,  sockmaps and devmaps hold information about sockets and network devices and are used by network-related eBPF programs to redirect traffic

#### BPF Hash Maps
```
#!/usr/bin/python3  
from bcc import BPF
from time import sleep

program = r"""
BPF_HASH(counter_table);

int hello(void *ctx) {
   u64 uid;
   u64 counter = 0;
   u64 *p;

   uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;
   p = counter_table.lookup(&uid);
   if (p != 0) {
      counter = *p;
   }
   counter++;
   counter_table.update(&uid, &counter);
   return 0;
}
"""

b = BPF(text=program)
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")

# Attach to a tracepoint that gets hit for all syscalls 
# b.attach_raw_tracepoint(tp="sys_enter", fn_name="hello")

while True:
    sleep(2)
    s = ""
    for k,v in b["counter_table"].items():
        s += f"ID {k.value}: {v.value}\t"
    print(s)
```

All codes are available here - https://github.com/lizrice/learning-ebpf/tree/main

- eBPF can call inline functions - which are copy of functions inside function
- eBPF cant support functions calls like a JUMP instruction (not sure of current scenario)
- Other way is to do tail calls


### Anatomy of eBPF program

- ebPF bytecode instructions are run by eBPF virtual machine
- ebpf interpreter is now has been largely replaced by JIT compiler
- ebpf bytecode instructions run on ebpf registers
- ebpf VM uses 10 general purpose registers numbered 0 to 9

- eBPF bytecode is represnted by series  of bpf_insn structures
- Every bpf_insn has an opcode
- opcode defines what instruction is to perform, like adding a value to register or jumping to different instruction of program
- Some operations might include 2 registars

#### eBPF program for Network Interface
- ebpf can inspect, modify data of a packet as well

```
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

int counter = 0;

SEC("XDP")
int hello(void *ctx){
  bpf_printk("Hello World %d", counter);
  counter++;
  return XDP_PASS;
}

char LICENSE[] SEC("license") = "Dual BSD/GPL";
```

- You need to mention license which will be verified by eBPF verifier
- eBPF code should be GPL compatible
- The above code is XDP type which is for network packets
- It just prints hello world and increases counter whenever a packet is received
- This is an example of an eBPF program that attaches to the XDP hook point on a network interface.
- Some network cards support XDP programs so it can be executed at network card itself without reaching CPU. Its highly effective for DDOS protection
- LLVM clang can compile C code to bytecode if we specify target bpf

Tools to inspect ebpf object filr : hello.bpf.o
- file <filename> -> gives info on the file
- llvm-objdump -S hello.bpf.o -> Gives eBPF instructions

To load a program into kernel, we can do it programatically or use a tool called `bpftool`
- bpftool prog load hello.bpf.o /sys/fs/bpf/hello
- ls /sys/fs/bpf
- List all programs -> bpftool prog list
```
: xdp name hello tag d35b94b4c0c10efb gpl
 loaded_at 2022-08-02T17:39:47+0000 uid 0
 xlated 96B jited 148B memlock 4096B map_ids 165,166
 btf_id 254
```
- bpftool prog show id 540 --pretty
- bpftool prog show name hello
- bpftool prog dump xlated name hello
- bpftool prog dump jited id 540

To attach a program to XDP event
- bpftool net attach xdp id 540 dev eth0

To list network-attached eBPF programs
- bpfftool net list
- ip link
- bpftool prog tracelog

- bpftool map list -> to view maps loaded into kernel

To detach a program
- bpftool net detach xdp dev eth0 -> All gets detached
- bpftool net detach xdp dev eth0 id PROGRAM_ID

To unload program
- rm /sys/fs/bpf/hello
- No reverse load

### bpf() System Call

- bpf() system call is used by user programs to load eBPF programs into kernel
- eBPF programs wont use syscalls for maps. It uses helper functions
  
```
#!/usr/bin/python3  
# -*- coding: utf-8 -*-
from bcc import BPF
import ctypes as ct

program = r"""
struct user_msg_t {
   char message[13];
};

BPF_HASH(config, u32, struct user_msg_t);

BPF_PERF_OUTPUT(output); 

struct data_t {     
   int pid;
   int uid;
   char command[16];
   char message[12];
};

int hello(void *ctx) {
   struct data_t data = {}; 
   struct user_msg_t *p;
   char message[12] = "Hello World";

   data.pid = bpf_get_current_pid_tgid() >> 32;
   data.uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;

   bpf_get_current_comm(&data.command, sizeof(data.command));

   p = config.lookup(&data.uid);
   if (p != 0) {
      bpf_probe_read_kernel(&data.message, sizeof(data.message), p->message);       
   } else {
      bpf_probe_read_kernel(&data.message, sizeof(data.message), message); 
   }

   output.perf_submit(ctx, &data, sizeof(data)); 
 
   return 0;
}
"""

b = BPF(text=program)
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")
b["config"][ct.c_int(0)] = ct.create_string_buffer(b"Hey root!")
b["config"][ct.c_int(501)] = ct.create_string_buffer(b"Hi user 501!")
 
def print_event(cpu, data, size):  
   data = b["output"].event(data)
   print(f"{data.pid} {data.uid} {data.command.decode()} {data.message.decode()}")
 
b["output"].open_perf_buffer(print_event) 
while True:   
   b.perf_buffer_poll()
```

- C program loads first into ebpf machine
- b variable in python interacts from usersapce to loaded ebpf program using maps
- config is ahash map we created at the top
- whenever a event is triggered , the c funtion triggeres and executes
- run the program and do ls,sudo ls in other terminal

