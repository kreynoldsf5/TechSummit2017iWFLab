# Module 1: iWF Generally

## iWorkflow Setup

[service creation]: /images/L4-L7_Service_Creation_diagram.jpg "service creation"

http://f5-automation-labs-jt.readthedocs.io/en/latest/module2/lab2.html

talk through how tenant/concepts work together

## Discover BIG-IP Devices

In order for iWorkflow to interact with a BIG-IP device it must be discovered. The device discovery process leverages the existing CMI Device Trust infrastructure on BIG-IP (Like BIG-IQ). Credentials are only exchanged during the initial discovery process -- further communication uses device certificate based trust. 

1. Open the TechSummit2017 Postman Collection. Open the 'Module 1' folder and then the 'Task 1' folder.
2. Open a browser window to your iWF box. Go to the 'Devices' pane.
3. 'Send' the first two requests to generate an authentication token and the increase that token's timeout. The following steps could be done with basic authentication but we're using X-F5-AUTH-TOKEN headers. 
4. Examine the POST payload for the 'Discover BIGIP-A Device'. Send the request. Note the status in the response body. This is a standard 'tasks' based pattern that is commonly used.
5. Use 'Check Discovered Device State' to determine when the device discovery procedure finishes or fails. Follow along in the iWF GUI or ```tailf /var/log/restjavad.0.log```.
6. Craft the appropiate POST payload to discover BIGIP-2. I've included a Postman test to save the uuid in to an environment variable for later. We'll use this device in Module 4.
7. Use the 'Get Discovered Devices' request to look at all device iWF has discovered.


## Create a Tenant
All service deployments are performed by tenants. In fact, iWF administrators cannot perform service deployments. A tenant is an object which is assigned permissions for one or more clouds. Users and Groups may be assigned permissions to the tenant. In this task we'll create a tenant and associate a user through the GUI and the API.

1. Create a tenant in the GUI.
  * Login to iWF as the admin user.
  * Click the 'Clouds and Services' tab.
  * Click the 'Tenant' pane.
  * Click the plus sign.
  * Give the tenant a name. We'll not assign a cloud at this time.
  * Notice a 'Role' was created when you created the tenant.

2. Create a user in the GUI.
  * Under 'Clouds and Services' click the 'Users' pane.
  * Fill out the user dialogue.
  * Click the 'Roles' pane and find the role that was created when you created the tenant.
  * 'Active Users and Groups' is used to add a group or a user _by the full name_.

3. In Postman send the 'Create iWorkflow Tenant' request. Look at the parity of the POST payload for this request as compared to the tenant creation dialogue in the GUI.

4. Send the 'Create Tenant User' request. The 'displayName' JSON value is synonymous with 'Full Name' in the GUI.

5. The 'Assign User to Tenant Role' PATCH request associates the user we just created with the tenant (or more specifically the role associated with the tenant). 
  * The role that was automatically created for the tenant is that tenant's admin role. 
  * Look at the 'resource' JSON object that includes allowed 'resource masks' or look at the role's permissions in the GUI. Look at the pattern of the allowed URIs. This is basis of iWF RBAC.

6. How would you go about making a lesser privlidged role for a tenant? Make such a role for the tenant you created via the GUI earlier in this task.


## Create a Connector
A connector is a target for service deployments. In our case this is a single BIG-IP device but this would typically be a BIG-IP device service cluster. 

1. Study the POST payload for 'Create Local Connector'. Where did the link in the 'deviceReferences' JSON array come from?

2. In the GUI, the connector you just created is under the 'Clouds' pane. There is history here dating back to Big-IQ Cloud days. 

3. Use the followup GET request, 'Get Local Connectors', to find the connectorId or selflink of the returned connector. Note the GUID naming scheme (connector+{{guid}} in this case).

4. Finally, assign the connector to the tenant you created earlier with 'Assign Connector to Tenant'. The  'cloudConnectorReferences' JSON array could be used to assign multiple connectors to a given tenant.


## iWorkflow Proxy Examples
REST, SOAP, and TMSH proxy functionality is available in iWF. Since iWF acts as the central point for L4-L7 service deployments so it makes sense that iWF can function as an API proxy for discovered devices. This can be useful for tasks that cannot be completed through iApp/service template -- like GTM/BIG-IP DNS.

1. Run through the example REST and TMSH requests which will be proxied to BIGIP-1. 
2. Log into BIGIP-1 and verify the test pool is created, updated, deleted, etc.
3. Try the 'Run TMSH Command on BIGIP-1' request. Why does it fail?
  * Go back and look at the POST you made when the device was discovered.
  * Craft a PATCH request to fix this issue.
4. Once correct, send 'Run TMSH Command on BIG-IP1' and check the 'status' in the response.
5. Is there another issue to be corrected?


