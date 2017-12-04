# Tech Summit 2017 Presentation
__bold__

_italics_

[link](www.google.com)

- UDF Build
  - iWF
  - Big-IP
  - Linux Host running Docker
  - Windows Host
    - Postman
    - atom/sublime/notepad++


##Tasks
###Big-IP Build
- iControlLX (Placeholder -- let's not do this)
###Discover BIG-IP device
###Create Tenant/Role
- build all the items needed
###Create Connector
- get the ordering of this all togeter
###Deploy Integration Service iApp on iWF
###Deploy Existing Service template on iWF
###Create Your own Service Template
- demonstrating simplist possible service Template
- demonstrate tenant editable
- demonstrate service helpers


##iControlLX Basics
- purpose/overview
- RPM packaging tasks
- helloworld.js, memoryworker.js
- more

##iControlLX Tasks
###Docker API
- Provision containers, return payload for pool membership
- input is simply the container name and the number needed
- Swarm API is similiar, but this is simpler
###CloudFare SSL
- REST call for cert/key
- passed into service template at runtime
###triage/controller
- consumes our custom payload, call to the other workers/extensions
- then calls the service template
