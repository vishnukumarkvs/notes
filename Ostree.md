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
- when a LD file is deleted, it creates a white out file which tells overlayfs to not look at it when we search for a p