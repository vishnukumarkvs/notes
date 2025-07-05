Ostree
======

# Overlayfs

- Union based file system
- Used in docker and yocto

- Its a union of directories
- They are stacked on each other
- Upper directories and lower directories
- lower directories have read only file
- upper directory , we can modify stuff

- copy up approach. if u edit or delete files in lower layers, it will be copied in upper layer, add more modifications and save in upper layer
- when a LD file is deleted, it creates a white out file which tells overlayfs to not look at it when we search for that particular file


```
# To check if overlay filesystem existts
- cat /proc/filesystems | grep overlay

# need to create al empty folders for the command
mkdir -p /root/overlay-example/{l1,l2,l3,l4,workdir,mount}

create l1.txt etc files in those directories

sudo mount -t overlay overlay -o lowerdir=/root/overlay-example/l1:/root/overlay-example/l2:/root/overlay-example/l3,upperdir=/root/overlay-example/l4,workdir=/root/overlay-example/workdir /root/overlay-example/mount

# Go to mount directory and edit a lower dir file to see a copy in upperdir
cd mount && echo "hfjdslkj" >> l3.txt

# Remove file
/mount: rm l2.txt

This file still exists in l2 folder. But go to l4 and check a whiteout l2.txt. Do stat on it.

deleting an apended or copy up file in upper directory will mess things up. you need to umount and run above command again
```

- Docker uses overlayfs as a default storarage driver
- it uses this to create container layers
- Every instruction in Dockwrfile is a layer
- space optmizatttion is odne via overlayfs as all containers use same layers of a speific image

# Hardlinks
