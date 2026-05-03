nodeagent
=========

securityContext:
  capabilities:
    add: ["BPF", "PERFMON", "SYS_PTRACE", "NET_ADMIN"]
    drop: ["ALL"]
  # Remove privileged: true
  
  
remove moutpoint /sys/kernel/debug

linux kernel should be >5.8