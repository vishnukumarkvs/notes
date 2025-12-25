docker
======

- Its a lightwweight VM (but its not)

- uses the host kernel
- most popular are linux baased images and windows images(windows nased ocntainers can run natively on windows server)
- Containers use kernel features like control groups  (cgroups) and namespaces

cgroups
- limits, accounts for resource usage (cpu, memory, disk,io) for groups of processes
- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/resource_management_guide/ch01
- /sys/fs/cgroup - unifies place - result of cgroup v2
- 