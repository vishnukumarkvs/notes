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

## Inode
- every file in a filesytem has an inode (index node)
- Contains all file information except file contents and name
- just like a personalid or a passport(without name)

It contains
- inode number
- file size
- permissions
- owner info
- file type
- number of linkis etc

## Soft link
- its like a shortcut in windows
- sometimes known as sysmbolic link
- inode number of a soflink is different from the original file inode
- if we delete original file, softlink will be useless
- softlink file size is very low

## Hardlink
- different name of a same file
- same file size as original
- same inode
- delete original file will not affect hardlinks
- its like a copy of a file
- directories not supported

```
# Create a hard link
ln /home/user/documents/original.txt /home/user/backup_link.txt

# Both files now point to the same data on disk
ls -li original.txt backup_link.txt

# Create a soft link to a file
ln -s /home/user/documents/original.txt /home/user/shortcut.txt

# Create a soft link to a directory
ln -s /home/user/documents /home/user/docs_link

# Create a soft link with relative path
ln -s ../config/settings.conf current_settings.conf
```

-s means softlink

# Chroot
- chroot command is a unix operation that changes the apparant root directory for the current running process and its children
- a program running in such a modified encironment cant access files outside tthe designated directory tree

```
sudo chroot /mnt/newroot /bin/bash
```

- You need to be sudo to run chroot
- new root must contain al necessary  files to run the program in it
- essential binaries should be in that new root
- /bin, /lib.,/etc, /dev, /proc

- used for repairing sytems


# Initramfs

- https://www.linuxfromscratch.org/blfs/view/svn/postlfs/initramfs.html
- The only purpose of an initramfs is to mount the root filesystem. The initramfs is a complete set of directories that you would find on a normal root filesystem. It is bundled into a single cpio archive and compressed with one of several compression algorithms.

- At boot time, the boot loader loads the kernel and the initramfs image into memory and starts the kernel. The kernel checks for the presence of the initramfs and, if found, mounts it as / and runs /init. The init program is typically a shell script. Note that the boot process takes longer, possibly significantly longer, if an initramfs is used.

# Ostree

- transactional upgrades (atomic)
- easy rollback
- replicating content incremently over http
- flexible support for mul;tiple branches and repositories
- delivery and deployment of file sytem trees
- called now to libostree
- ostree is a command line tool where as libostree is a library


### Package vs image vs differential upgrades

Packages
- rpm, apt, yum
- low bandwidth
- easy to use

Image based (full filesytem update)
- can be tested exhausticvely
- can be pwersafe with A/B configurartion
- high bandwidth

Atomic
- Only differrence is downloaded
- transactional nature
- requires a reboot
- one http request for each file we dont already have

### Pull vs Push

- Add a remote server
  - ostree remote add poky http://ip/ostree-repo -no-gpg-verify
  - ostree remote summary poky