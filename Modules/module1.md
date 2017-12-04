# Module 1: iWorkflow General

## iWorkflow Object Model

<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/L4-L7_Service_Creation_diagram.jpg" alt="Drawing" style="width: 100px;"/>

### iWF Administrator Tasks:
* Discover BIG-IP devices
* Assign device(s) to 'connectors' (sometimes referred to as 'clouds')
* Create 'Tenants'
* Give Tenants permissions on one or more connector
* Create Groups or Users 
* Give Groups or Users permissions to one or more tenants
* Create 'Service Templates' 
* Allow Service Templates to be used by one or more tenant

### iWF Tenants
* Deploy 'L4-L7 Services' from service templates to allowed connectors
  * Tenants only see connectors -- not BIG-IP devices
* The collection of service templates available to a tenant is often referred to as the "Service Catalog".
  
_iWF Administrators do not have the ability to delete L4-L7 services. This *must* be done by the tenant._



## Task 1: Discover BIG-IP Devices

In order for iWorkflow to interact with a BIG-IP device it must be discovered. The device discovery process leverages the existing CMI Device Trust infrastructure of BIG-IP (Like BIG-IQ). Credentials are only exchanged during the initial discovery process -- later communication between devices uses device certificate based trust. 

2. Open a browser window to your iWF box. Log in as 'admin'. Go to the 'Devices' pane.
3. Open Postman and open the collection you previously imported.
4. Send the first two requests under 'Token Auth' -> 'Admin' folder. These requests will generate an authentication token and increase the token timeout. The following steps could be done with basic authentication but its good practice to use the X-F5-AUTH-TOKEN header. 
4. Examine the POST payload for the 'Discover BIGIP-A Device' request. Send the request. Note the status in the response body. This is a standard 'tasks' based pattern that is commonly used.
5. Use 'Check Discovered Device State' to determine when the device discovery procedure finishes or fails. Follow along in the iWF GUI or ```tailf /var/log/restjavad.0.log``` to verify the device has been discovered.
6. Craft the appropiate POST payload to discover BIGIP-2. I've included a Postman test to save the uuid to an environment variable for later. We'll use this device in Module 4.
7. Use the 'Get Discovered Devices' request to look at all device iWF has discovered.


## Task 2: Create a Tenant
All service deployments are performed by tenants. A tenant is an object which is assigned permissions for one or more connectors. Users and Groups may be assigned permissions to a tenant. In this task we'll create a tenant and associate a user through the GUI and the API.

1. Create a tenant in the GUI.
  * Login to iWF as the admin user.
  * Click the 'Clouds and Services' tab.
  * Click the 'Tenant' pane.
  * Click the plus sign.
  * Give the tenant a name. We'll not assign a cloud at this time.
  * Notice a 'Role' was created when you created the tenant.

2. Create a user in the GUI.
  * Under 'Clouds and Services' click the 'Users' pane.
  * Fill out the user dialogue. Leave 'Auth Provider' to local for this lab. Hit 'Add' when done.
  * Click the 'Roles' pane and find the role that was created when you created the tenant.
  * 'Active Users and Groups' is used to add a group or a user _by the full name_.

3. In Postman send the 'Create iWorkflow Tenant' request. Look at the parity of the POST payload for this request as compared to the tenant creation dialogue in the GUI.

4. Send the 'Create Tenant User' request. The 'displayName' JSON value is synonymous with 'Full Name' in the GUI.

5. The 'Assign User to Tenant Role' PATCH request associates the user we just created with the tenant (or more specifically the role associated with the tenant). 
  * The role that was automatically created for the tenant is that tenant's admin role. It has full permissions *under that tenant*. 
  * Look at the 'resource' JSON object that includes allowed 'resource masks' or look at the role's permissions in the GUI. Look at the pattern of the allowed URIs. This is basis of iWF RBAC.

6. How would you go about making a lesser privlidged role for a tenant? Make such a role for the tenant you created via the GUI earlier in this task.


## Task 3: Create a Connector
A connector is a target for service deployments made up of BIG-IP(s). In our case this will be a single BIG-IP device but this would typically be a BIG-IP device service cluster. 

1. Study the POST payload for 'Create Local Connector'. Where did the link in the ```deviceReferences``` JSON array come from?

2. In the GUI, the connector you just created is under the 'Clouds' pane. There is history here dating back to Big-IQ Cloud days. 

3. Use the followup GET request, 'Get Local Connectors', to find the ```connectorId``` and the ```deviceGroupReference``````link``` of the returned connector. Note the GUID naming scheme (connector+{{guid}} in this case).

4. Finally, assign the connector to the tenant you created earlier with 'Assign Connector to Tenant'. The 'cloudConnectorReferences' JSON array could be used to assign multiple connectors to a given tenant.


## Task 4: iWorkflow Proxy Examples
REST, SOAP, and TMSH proxy functionality is available in iWF. iWF acts as the central point for L4-L7 service deployments so it makes sense that iWF can function as an API proxy for discovered devices. This can be useful as a single API gateway to all of a customer's BIG-IP devices (with iWF RBAC as opposed to permissions needed for iControl REST). These endpoints could also be used for tasks that cannot be completed through an iApp/service template -- like GTM/BIG-IP DNS changes.

1. Run through the example REST and TMSH requests which will be proxied to BIGIP-1. 
2. Log into BIGIP-1 and verify the test pool is created, updated, deleted, etc.
3. Try the 'Run TMSH Command on BIGIP-1' request. Why does it fail?
  * Go back and look at the POST you made when the device was discovered.
  * Craft a PATCH request to fix this issue.
4. Once correct, send 'Run TMSH Command on BIG-IP1' and check the 'status' in the response.
5. Is there another issue to be corrected?


