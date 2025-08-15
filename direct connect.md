direct connect
==============

equinix direct connect

https://youtu.be/QRRmSvu-Ozs?si=UHow99rGIDqathlf

Equinix to aws direct connect

- In equinix portal, create a virtual dx connection. for example in region chicago
- you give aws account id and also specify in which region you waant direct connect in
- you use same region to reduce latency, so chicago again (us-east-2)
- if u use chicago all our services like eks , vpc should be deployed in same region
- once created, you will need select bandwidth also, depends on pricing as well
- once everything is done in equinix portral side, go to your aws console,s ect us east 2 region and select direcdt connect
- see the request from equinix
- accept it
- next create a virtual interface, choose the equix connect