node agent crash
================

https://github.com/coroot/coroot-node-agent/issues/376


Core issue

Think attendance register.

Agent asks kernel every 15s: "how long did each process wait for CPU/disk?"

\Kernel clerk goes to copy the process name-tag (16-letter slot) onto a 32-letter form. Old instruction says "always read 32 letters." Safety guard watches and shouts on letter 17: "you're reading past the tag!" and pulls the emergency brake — whole node reboots.

Most name-tags are short, guard never notices. One node had a long name, guard tripped. Fix = tell clerk to stop at the end of the tag, or stop asking for that form.



