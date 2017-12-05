# Module 3: iControl LX

<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/image002.png" alt="Drawing" style="width: 200px;"/>

## Extension creation
iControlLX extensions need to be created and packaged in an RPM format prior to being installed. Packages can be created on one BIG-IP or iWF device and installed on another. If your packages require dependencies, you can install them with NPM in your working directory with ```npm install package``` and then copy the ```node_modules``` directory along with your JavaScript to the F5 device that will build the package. 

Several sample iControlLX extensions are provided in the repository:
  * SkeletonWorker.js - event framework outline example
  * HelloWorld.js - obligatory first extension
  * MemoryWorker.js - simple example where a worker holds state
  * my-app-interface.js - monolithic example from [Nicolas Menant](https://github.com/nmenant/iControl_LX_Lab) which retrieves destination IP from IPAM, deploys a service template.

### Task1 - build HelloWorld.js extension
1. Make the directory from which the extension will be built on your iWF device. SSH to your iWF device and drop to bash.
  * ```mkdir -p /var/config/rest/iapps/Helloworld/nodejs```
2.  SCP the JavaScript files to the build directory
  * Use winSCP on the windows jump host or SCP from elsewhere to cp {{repo}}/iControlLX_extensions/Helloworld.js to the build directory
3. Monitor the relevant iWF log while the RPM build procedure is performed
   * ```tailf /var/log/restnoded/restnoded.log```
   
4. Open the module 3 Postman collection, 'Task 1: Build RPM' folder.
5. Set/verify the Postman environment variable {{extension_name}} is set to the build directory name (just the directory name -- not the full path) you created earlier.
6. Send the 'Create RPM on iWF' request. This procedure follows the 'tasks' pattern (the initial ```status``` will be "CREATED").
7.  Send the 'Check RPM Creation Process' request. Notice the URL ends with the ```id``` from the 'Create' task. Evaluate the ```step``` and ```status``` values from this request.

8. On your iWF machine, look in the directory specified in ```packageDirectory``` in the last response to verify your RPM was created (if the status was 'FINISHED' it should be there). The iControlLX extension is now built and packaged. This RPM can be installed on this device or another BIG-IP/iWF device. 

### Task 2: Extension installation
 iControlLX extensions must be installed from ```/var/config/rest/downloads/```. Not coincidently, this is the same directory used by the iControl [file upload endpoint](https://devcentral.f5.com/wiki/iControl.File_transfer_resource_APIs.ashx). You can POST to https://{{F5-Device}}/mgmt/shared/file-transfer/uploads/{{filename}} with the appropriate body and headers (chunking dependent) to upload a file via iControl REST. For this lab, since we're installing on the same machine where we're building, just cp the RPM by hand.

1. Copy the RPM from ```packageDirectory``` to this directory.
  * ```cp /var/config/rest/iapps/RPMS/Helloworld-0.1-001.noarch.rpm /var/config/rest/downloads/```
2. In Postman open the 'Task 2: Install RPM' folder of this module directory. Inspect the POST body of the 'Install RPM' request. Look at your Postman environment variables to see the value of {{rpm_name}}. (Look back at the 'Test' tab from the 'Create RPM on iWF' request to see how this variable was built). 
  * If you need to uninstall a package use the requests in the 'Delete RPM' folder.
4. Open 'Helloworld.js' in your text editor. Find the ```WORKER_URI_PATH```. 
5. Open your web browser to https://{{iwf_mgmt}}/mgmt/{{WORKER_URI_PATH}} . What is the result?

### Task 3: Build/Install the MemoryWorker.js extension
1. Using the steps in Task 1 and Task 2 above, build and install the memoryworker.js iControl extension. 
  * Copy the extension from the repo 'extensions' folder to the build directory on the iWF device.
  * Update the {{extension_name}} Postman environment variable.
  * Build the RPM.
  * Copy the RPM to the installation directory.
  * Install the RPM.
2. Open ```memoryworker.js``` in your text editor. 
  * Look at the contents of the ```onPost``` event. What do you think the POST payload for this worker needs to look like?
  * Look at the contents of the ```onGet``` event. What is the response to a GET going to look like?
3. from the 'Task 3' folder, send the example POST, 'Memoryworker example POST', to the extension's ```WORKER_URI_PATH```.
4. Send the example GET with 'Memoryworker Example GET', and note the response.
5. Change the persisted data by updating the previous POST request. What HTTP methods are available? Why doesn't the version increment? 

### Task 4: iControlLX RBAC 
In the previous exercises, we made requests to iControlLX entension URIs using the iWF admin token. Commonly, tenants will need to make requests to these URIs (as only tenants perform L4-L7 service deployments).
1. Send the 'View Tenant Permissions' request. Look over the ```resourceMask``` and ```resourceMethod``` values in the ```resources``` array. You can also view this list in the GUI under 'Access Control -> Roles -> {{tenant role}}'
  * What's missing from this list? 
  * What permissions will be needed for a tenant to use the iControlLX workers we've deployed so far?

Depending on our RBAC strategy, we can add the needed permissions in one of two ways:
  * Create a seperate role for iControlLX permissions and assign that role to our tenant
  * Add the permissions we need to our existing Tenant role. This can only be done via the API.

2. Look over the POST payload for 'Create dedicated iCLX Role'. The {{iwf_tenant_user}} will be added to this role. What permissions are granted by this request?
3. Send the 'Test GET permissions' request. Notice that X-F5-Auth-Token header is the _tenant_ token. Evaluate the response.
4. Send the 'Test POST permissions' request. Why didn't this work?
5. Delete this role as we'll be using the alternative method to assign permissions. Send the 'Delete dedicated iCLX Role' request. 
6. The other method is to grant the necessary permissions to the tenant (admin) role -- the role that was created when the tenant was created. This cannot be edited in the GUI but we can PATCH the resource list via the API.
  * Look at the POST payload for 'Update Tenant Role Permissions'. We're allowing this tenant all HTTP verbs for anything under ```/shared/ts2017/``` -- the WORKER_URI_PATH where we've been registering our workers. Notice the other set of permissions for ```/shared/ts2017/*/*``` as we'll need them later in this module.
  * Send the request.
7. Test the GET and POST permisssions again.


### Task 5: my-app-interface example

This iControlLX extension was created as a POC by [Nicolas Menant](https://github.com/nmenant/iControl_LX_Lab). Its a monolithic example in which the virtual server destination address for a L4-L7 deployment is retrieved from an IPAM provider at the time of deployment. The IPAM provider in this example is a BIG-IP with an iRule that responds to a GET with an IP address.

I encourage you to open the iControl LX worker in Notepad++ on the Windows jump host or on your local machine. As we create service deployments with POSTs, make changes to deployment with PATCH, and remove deployments with DELETE, follow along in the cooresponding ```ipam_extension.prototype.on{{VERB}}``` events in the extension.


1. Build/Install the iControlLX extension. Follow the workflow from the last two tasks.
  * Copy the extension from the repo 'extensions' folder to the iWF device
  * Build the RPM
  * Copy the RPM to the installation directory
  * Install the RPM
2. The permissions we added to the tenant group in the last task will allow access to this ```WORKER_URI_PATH```.
3. We need to create custom service templates where we can specify the port for the pool members (this is not included in the default f5-http-lb service template). Our new service templates will inherit values from the service templates we imported from the 'iWorkflow_json_payloads' archive with the ```var``` ```pool__port``` marked as tenant editable. 
   * Send the request to 'Create http service template for my-app-interface'.
   * Send the request to 'Create tcp service template for my-app-interface'.
   * Note the subtle difference in the tenant editable fields of these templates -- the latter (tcp) includes ```ConnectionLimit```.
4. Verify the POST payload for 'Create HTTP Service' inside the 'HTTP Service Deployment' folder. Make sure you are tailing ```/var/log/restnoded/restnoded.log``` before you send this request. Send the request and verify that the L4-L7 deployment was created.
  * Note that logging is rudimentary for this extension
5. Send the 'GET HTTP Service (worker)' request. Notice the payload returned mimics that of one we'd send to the iControlLX extension and *not* the payload of the service deployment itself.
  * Look at the ```onGet``` event of the extension to see how this is built.
  * CRUD Functionality for standard HTTP verbs is necessary -- especially if this was the only interface/endpoint being exposed to a tenant (ie. the tenant had no permissions other that to this iControl LX extension).
  * Revisit the request where we altered permissions for this tenant to allow requests to ```/shared/ts2017/*```. Do you see why we also had to add permissions for ```/shared/ts2017/*/*```?
6. Send the 'GET HTTP Service (deployment)' request. See the difference in response payload between this request and the last? What destination IP did your service deployment get?
7. Test your service deployment in a browser. 
  * appending "/stats" to the L4-L7 deployment endpoint should now show some status data.
    * ```health.summary``` boolean
    * ```health.stats.total-member-cnt```
    * ```health.stats.active-member-cnt```
    * Connection/traffic information will not be immediately available 
8. Verify PUT functionality by sending the 'Update HTTP Server' request.
  * Verify the state of the pool member we disabled (in either the GUI or the 'GET HTTP Service (deployment)' request.
8. From the 'TCP Template Service Deployment' folder, send the 'Create TCP Service' request.
9. Since this iControlLX Extension reads the template from a JSON value in the POST payload (```template```), note the differences in the required ```server-data``` for these requests as opposed to the HTTP example. Remember the inclusion of ```ConnectionLimit``` when this service template was created?
10. Send 'Update TCP Service' request.
11. The iControlLX worker also implements ```onDelete``` (as it should). Look at the payload for 'Delete Service' then send the request.
12. It is standard practice to implement an ```/example``` URI for your worker. This will give the consumer some idea what the POST payload will need to look like when calling the endpoint. The iControlLX extensions which use the '/iapps/block' interface (so far just PD developed/provided) provide a comprehensive template. We'll be looking at one of these in the next module.



