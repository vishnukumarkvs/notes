Keycloak
========

singlw page application access token and id token authorizatuoin code better ay is to call keycloak token we have intrispection endpoint 
identity management - raelmd , clisents roles users griup m tdsns for stiorng underlyiong data

- Written in java

- RDBMS - For storing underlying data, can also use postgres
- Infinispan - for clustering and cross-site replication
- Infinispan - in memory distrubuted database

UI
- Admin console
- Account management
- User registrations, Reset passwords during login
- Fully plugagble themes
- User profile for custom attributes

Auth
- 2ndFactor - OTP, Passkeys
- Kerberis auth
- 3rd Party like facebook, google etc
- Step up authentication - Support for addditional authentication for critical areas lilke otp, reentering password 

Others
- LDAP
- REST API's for admin ,account
- tracing , monitoring

Deployment
- Standalone / bare metal java application
- Containers 
- Operator - Kubernetes
- Clustering and HA supoport


## Realm

- Realm is a top level logical container that manages Users, Roles, Groups , Clients and Audthentiication flows
- Its an isolated security domain 
- Each realm do not share users or configurations
- Each relam has sepaarate login portals , roles , clients
- Keycloak comes with master realm which is used to manage other realms

# KCADM
- kcadm.sh is a file which is used to ru;n cli commands on keycloak. This file will be available in container
- filnd /opt -name "kcadm.sh" --> gives the path
- cd $(dirname $find /opt -name "kcadm.sh")) --> goes into that file directory
- 