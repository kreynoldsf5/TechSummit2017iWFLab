# Module 2:  Service Templates

An iApp template can be uploaded to iWF in several ways. In the GUI you can provide the TMPL files through pasting or uploaded. APL needs to be provided via JSON and can be uploaded, pasted, or retrieved from an existing BIG-IP. The entire iApp template can provided in a single upload via JSON. The necessary JSON is packaged in the Github repository for the Integration Service iApp (https://github.com/F5Networks/f5-application-services-integration-iApp/releases). The 'iWorkflow JSON Payloads' zip archive is provided in the Lab repository.

### Task 1: Upload iApp Template

A request is provided in the Postman folder 'Task 1: Upload Integration Services iApp' folder under this module. In the alternative, feel free to upload the template via the GUI by navigating to 'Clouds and Service -> iApps Templates -> "+". For the 'Import Method' select 'Use pre existing JSON' and 'import JSON'. Under the 'Choose File' dialogue select ```{{repo}/iWorkflow_json_payloads_v2.0-5.004/import-json/iWorkflow_appsvcs_integration_v2.0.004.json```.

### Task 2: Upload pre-existing service templates

The iWorkflow_json_payloads archive provides several pre-existing service templates which reference the Integration Services iApp.

1. Send the request to 'Create the f5-http-lb Service Template'.
2. Send the request to 'Create the f5-fastl4-tcp-lb Service Template'.

### Task 3: Tenant View

Log into the iWF GUI as the tenant that you previously created. Click around and notice what tabs and panes are available to the tenant. Now let's look at this view in the API.

1. Send the 'List Tenant L4-L7 Service Templates' request. Find the service templates we created in the last step.
2. Send the 'List Deployed Services" requests (both 'Virtual Servers & Pool Members' as well as 'Nodes'). We have not yet deployed a L4-L7 service so the ```items``` array is blank (```[]```).
3. Send 'List Deployed Services: L4-L7 Services' to verify that we currently have no deployed services.
4. Send the 'Deploy Temporary L4-L7 Service'. Don't worry about the POST payload for this service deployment at this time.
5. Resend the requests for 'List deployed Services:*'. Notice how the tenant can now see all objects (Virtuals, Nodes, Pool Members, and Service Deployments) for this tenant.
6. Send the request for 'View Tenant L4-L7 Service Stats'. Scroll down (or find) health.summary JSON objects. 
  * Is your Application Service healthy? 
  * If not, can you fix it with a PATCH?
7. Go ahead and remove the deployment by sending the 'Delete Temporary L4-L7 Service' request.

### Task 4: Build a Service Template 


1. In the GUI logged in as 'admin', navigate to 'Clouds and Services -> Service Templates -> "+"'. Select the Integration Service iApp as the template and select the appropiate version. Name your template. In this exercise we are going to create a minimalist service template for demonstration purposes.
LINK
2. Fill out the service tier information.
3. Expand the 'Virtual Server Listener & Pool Configuration' section.
4. Create static entries for pool members according to your docker containers.
LINK
  * be sure to include 'state
LINK
<Several pictures here>
5. 

Note the fields which are tenant editable.

Note the fields where defaults are displayed but are not editable.



### Task 5: Deploying a Service Template
The JSON from our example 'minimalist' template was lengthly as it defined all necessary defaults for the underlying iApp. The 'minimalist' component is what is needed from the tenant to instantiate the service template.
1. Log into the iWF GUI under the tenant account. Click on 'L4-L7 Service -> "+"'. Select the 'Minimalist-template' from the 'L4-L7 Service Template' dropdown. This is the tenant's view of the template -- just the ability to specify a name, what connector to be deployed against, and to configure the virtual server address.
2. Fill out the form and hit 'Save' capture the payload in the tenant POST. 
```javascript
{"tenantTemplateReference":{"link":"https://localhost/mgmt/cm/cloud/tenant/templates/iapp/minimalTemplate"},
"properties":[{"id":"cloudConnectorReference","value":"https://localhost/mgmt/cm/cloud/connectors/local/89800a56-13d3-4a0c-96b7-c2331880705f"}],
"tables":[],
"vars":[{"name":"pool__addr","value":"10.1.200.56"}],
"name":"minService"}
```
So after declaring the service ```name```, the template to be used, and which connector to deploy against, the payload specifies the *only* tenant editable value - a ```var``` called ```pool__addr``` and a specified ```value```.

### Task 6: Build Your Own
You've seen that a service deployment is simply a POST which specifies the template, the connector, the name, and all necessary tenant editable values. Tenant editable values along with specified defaults from the service template are used to builf the POST payload necessary for the ASO on the target BIG-IP. Let's examine the syntax of the ```overrides``` section of the Service Template. Here is a small section from a service template that uses AFM:

```javascript
  "overrides": {
    "vars": [
      {
        "name": "afm__allowed_addr",
        "description": "What IP or network addresses should be allowed to access your application?",
        "displayName": "allowed_addr",
        "isRequired": true,
        "provider": ""
      },
      {
        "name": "afm__policy",
        "description": "Do you want to use BIG-IP AFM to protect your application?",
        "displayName": "policy",
        "isRequired": false,
        "provider": "/#do_not_use#"
      },
      {
        "name": "afm__restrict_by_addr",
        "description": "Do you want to restrict access to your application by network or IP address?",
        "displayName": "restrict_by_addr",
        "isRequired": false,
        "provider": "/#do_not_use#"
      },
      ...
```
* ```name``` is the variable name in the iApp.
* ```description``` is displayed in the iWF GUI. 
* ```displayName``` is the field name in the iWF GUI. 
* ```isRequired``` is a self explanatory boolean. 

Whether a field is tenant editable is derived from the ```defaultValue``` field. 


The value of ```provider``` is added to the field when displayed and functions as the field's default. If you wanted to add a value to a var that was not tenant edi ```"serverTier": "default"``` can be used to place a field in  








### Task 5: a Service Template
Blurb on Service Tier Options


a. installation of iApp 
    1. Discussion of the App Services Integration iApp
b. GUI creation of service Template
c. grab the POST payload for the service template creation, make your own template via Postman
d. Permissions on service template creation
e. Concept of "service Workers" (cert example)




## Tenant view vs. Administrator view