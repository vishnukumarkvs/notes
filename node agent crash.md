node agent crash
================

https://github.com/coroot/coroot-node-agent/issues/376


## Core issue

Think attendance register.

Agent asks kernel every 15s: "how long did each process wait for CPU/disk?"

\Kernel clerk goes to copy the process name-tag (16-letter slot) onto a 32-letter form. Old instruction says "always read 32 letters." Safety guard watches and shouts on letter 17: "you're reading past the tag!" and pulls the emergency brake — whole node reboots.

Most name-tags are short, guard never notices. One node had a long name, guard tripped. Fix = tell clerk to stop at the end of the tag, or stop asking for that form.



Both, in stages.
1. Kernel builds struct taskstats (with ac_comm[32]) in kernel memory as the netlink reply (skb).
2. That reply is sent via netlink to userspace; agent's library casts it to a Go struct.

The crash happens at stage 1, before sending — while copying tsk->comm[16] to the in-kernel ac_comm[32].


### Linux version
amazon linux version: 6.12.79-101.147.amzn2023.x86_64

AL2023 kernel6.12 package build
6.12.79-101.147.amzn2023.x86_64 = Amazon Linux 2023, kernel 6.12 variant (not default 6.1).

Shipped: late-April 2026. Evidence in search



###

Almost — not PID, comm (process name).
- src = tsk->comm[16], dst = ac_comm[32]. Copy fails under FORTIFY.
- coroot-node-age in your Oops is the querier (truncated coroot-node-agent, 17 -> 15+NUL), not necessarily the victim. Victim is whichever target PID fillstats was copying at that moment.
- Yes: short comm (<15 chars, early NUL) stops strnlen early — safe. Only long comms (~15-16 chars, NUL at/near byte 16, word-read tries byte 17) trip it. That's why it's random per node.


