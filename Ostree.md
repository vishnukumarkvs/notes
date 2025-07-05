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


