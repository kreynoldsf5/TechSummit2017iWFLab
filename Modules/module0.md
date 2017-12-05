# Module 0: Getting Started

## Environment Overview

This Lab assumes a working knowledge of the following:
- BIG-IP LTM concepts
- BIG-IP iApps
- iControl REST API
- Use of Postman in interact with RESTful APIs
- Minimal understanding of JavaScript

## Environment Layout
The UDF hosted Lab environment consists of two BIG-IP devices (TMOS 12.1.1), one iWorkflow device (iWF 2.3), an ubuntu server (14.01 LTS) running docker, and a Window jump host. All tools needed for the lab can be found on the Windows jump host (putty, winSCP, Postman, etc.). The Lab consists of a Management network, an Application network, and a VIP network. 

|               | Management | Application | VIP   |
| ------------- |:-----------:|:----------:|:-----:|
| iWF.ts2017.local  | 10.1.1.4     | 10.1.100.4 | 10.1.200.4     |
| bip1.ts2017.local | 10.1.1.5     | 10.1.100.5 | 10.1.200.5     |
| docker            | 10.1.1.6     | 10.1.100.6 | 10.1.200.6     |
| Windows           | 10.1.1.7     | 10.1.100.7 | 10.1.200.7     |
| bip2.ts2017.local | 10.2.1.8     | 10.1.100.8 | 10.1.200.8     |

Account information for various hosts is available in the UDF for the Windows jump host and on the desktop of the jump host for the other devices. SSH sessions using ssh keys for Putty and WinSCP have been created (use the 'admin' user). Log into the [UDF](https://udf.f5.com) now. This will require MFA. Search for our lab ("GSTS 2017 iWorkflow and iControl LX Lab") under "Courses".

**Please go ahead and start the deployment now as the F5 devices will take some time (5-6 min) to boot.**

## Preliminary Tasks
### Log into your Windows Jump Host
1. Grab an RDP link from 'Systems -> Details' for the Windows jump host. The 'Administrator' credentials will be in the 'Details' tab.
2. Open Chrome and click on the kreynoldsf5/TechSummit2017iWFLab link in the bookmarks bar. This repo contains all the materials needed for the lab. 
  * Click 'Clone or Download' and then 'Download ZIP'.
  * Extract the archive.
  * In your browser, view the Lab Guides under the ['Modules'](https://github.com/kreynoldsf5/TechSummit2017iWFLab/tree/master/Modules) directory. These files are markdown and will display nicely on Github. 
  * Feel free to clone the repo locally instead of on the Windows jump host if that's easier.
3. Verify Access to your Lab BIG-IPs/iWF devices. 
  * I have saved SSH sessions for Putty and WinSCP on the windows jump host for BIGIP-A and iWF (you won't need SSH access to BIGIP-B). Use the 'admin' user and key based auth should _just work_.
 * There are bookmarks to BIGIP-A and iWF in Chrome. I'll give you the admin credentials during the lab. 

### Postman
1. If you're not familiar with Postman let me know and we'll spend a few minutes covering the tool.
2. Import the Postman Collection from the repo.
  * 'File' -> 'Import' and then drag the 'TechSummit2017.postman_collection.json' file from the 'Postman Collection' directory in the repository.
3. Import the Postman Environment from the repo.
  * 'File' -> 'Import' and then drag the 'TechSummit2017Env.postman_environment.json' file from the 'Postman Collection' directory in the repository.
4. Select the environment you imported from the drop down in the top right of the window. Fill in the Postman environment variables which were not included in the environment collection by clicking the "eye" and then 'Edit'.
  * {{iwf_admin_password}}
  * {{big_admin_password}}
5. Note how Postman performs variable substitution using the {{variable}} nomenclature. You'll see this frequently in request URIs and request body payloads.
6. Look at the 'Tests' tab for requests that assign variable values based on HTTP responses. This is simple JavaScript.

### Docker Pool Members
Start a couple docker pool members based on the F5Devcentral/F5-Demo-App container. 
1. Use the 'Web Shell' link from the UDF 'Access' drop down for the 'docker' system.
2. run ```docker run -d -P f5devcentral/f5-demo-app``` 2 times. Each time the command is run a new container will be created and bound to a host port.
3. run ```docker ps``` and make a note of the assigned ports for these containers.

The results should look something like:
```
root@ip-10-1-1-6:/# docker ps
CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                    NAMES
abe6516aa26c        f5devcentral/f5-demo-app   "npm start"         2 seconds ago       Up 1 second         0.0.0.0:32769->80/tcp    clever_stonebraker
ea807f84a946        f5devcentral/f5-demo-app   "npm start"         4 seconds ago       Up 3 seconds        0.0.0.0:32768->80/tcp    fervent_jang
ad535dc106f0        djenriquez/sherpa          "./main --allow"    6 days ago          Up 5 hours          0.0.0.0:4550->4550/tcp   sherpa
```
In this example the pool members' ports will be 32768 and 32769. If your ports differ, update the Postman environment variables for 'pool_member1_port' and 'pool_member2_port'.

### iControl REST Hints

You can view the API's layout, and interact if you wish, in a browser with an HTML based representation via the following URL:

https://{{iwf_mgmt}}/mgmt/toc

Notice how URIs that would otherwise just be REST endpoints are rendered in HTML when ```/presentation#/``` is appended to the URI. You can also see the JSON body of the resource by clicking 'advanced'.

Unlike BIG-IP, iWF's GUI is _completely_ driven by REST calls. This means if you are trying to figure out the appropriate URI for a resource or the proper JSON payload for a change walk through the operation you are trying to perform in the GUI and capture the HTTP request/response pairs. Chrome Developer tools is available on the Windows jump host.







