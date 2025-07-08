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
- deploy server and install app with its packages and dependecies
- do upgrade/patch lifetime
- Difficult to maintain if u have 100's of servers
- ansible can be used but even there can be a possiblity of error

Immutable infrastructure
- create custom images with application and its dependencies
- like createing a docker image but here we creaatea os image which can be directldy deployed on VMs on different cloud providers

