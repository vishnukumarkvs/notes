Packer
======

Packer is a tool to create os images from tremplates

- AMI - is a blueprint to deploy EC2 instances
- Packer is used to customize your own images
- Its used to crreate golden image

Custom image
- upgrade packages
- install dependencies
- copy source code
- start services

Stages
- Builder
  - used to create image
  - depends on the platform
  - specify base image
  - It will launch a vm with base mage
- Provisioner
  - customize vm like add extra packages using commands
  - creates a snapshot or new ami from it
- Post  processor
  - used to upload artifacts, compress and repackage files/images

Mutable Infrastructure
- deploy server and isntall app with its packages and dependecies
- do upgrade/patch lifetime

