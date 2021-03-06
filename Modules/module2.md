# Module 2:  Service Templates

An iWF 'Service Template' serves as a layer of abstraction on top of BIG-IP Application Service Object (ASO) deployments. The service template defines what values of an iApp deployment are editable by a tenant, default values (which may or may not be editable), and the destination connector for the deployment. Interaction with this interface is fronted by fine grained RBAC.


Service templates are built on top of iApp templates. An iApp template can be uploaded to iWF in several ways. In the GUI you can provide the TMPL files through pasting or uploaded. APL needs to be provided via JSON and can be uploaded, pasted, or retrieved from an existing BIG-IP. Alternatively, the entire iApp template can be provided in a single upload via JSON. The necessary JSON for the iApp used in this lab is packaged in the Github repository for the [App Services Integration iApp](https://github.com/F5Networks/f5-application-services-integration-iApp/releases). This archive includes JSON for the iApp, usable common service templates based on this iApp, and example POST bodies for L4-L7 service deployments. The 'iWorkflow JSON Payloads' zip archive has been extracted in the Lab repository.

### Task 1: Upload iApp Template

A request is provided in the Postman folder 'Task 1: App Services Integration iApp' folder under this module. In the alternative, you could upload the template via the GUI by navigating to 'Clouds and Service -> iApps Templates -> "+"'. For the 'Import Method' select 'Use pre existing JSON' and 'import JSON'. Under the 'Choose File' dialogue select ```{{repo}}/iWorkflow_json_payloads_v2.0-5.004/import-json/iWorkflow_appsvcs_integration_v2.0.004.json```.

### Task 2: Upload pre-existing service templates

The 'iWorkflow_json_payloads' archive provides several pre-existing service templates which reference the App Services Integration iApp.

1. Send the request to 'Create the f5-http-lb Service Template'.
2. Send the request to 'Create the f5-fastl4-tcp-lb Service Template'.

### Task 3: Tenant View

Log into the iWF GUI as the tenant that you previously created. Click around and notice what tabs and panes are available to the tenant. Now let's look at this view in the API.

Obtain a tenant token to use for API requests from the tenant. Send both the requests at the top of the collection under 'Token Auth' -> 'Tenant' to generate a token and extend the expiry.

1. Send the 'List Tenant L4-L7 Service Templates' request. Find the service templates we created in the last step.
2. Send the 'List Deployed Services" requests (both 'Virtual Servers & Pool Members' as well as 'Nodes'). We have not yet deployed a L4-L7 service so the ```items``` array is blank (```[]```).
3. Send 'List Deployed Services: L4-L7 Services' to verify that we currently have no deployed services.
4. Send the 'Deploy Temporary L4-L7 Service'. The POST payload contains all needed values .
5. Resend the requests for 'List deployed Services: '. Notice how the tenant can now see all objects (Virtuals, Nodes, Pool Members, and Service Deployments) for this tenant.
6. Send the request for 'View Tenant L4-L7 Service Stats'. Search for 'health.summary' JSON objects. 
  * Is your Application Service healthy? 
  * If not, can you fix it with a PATCH?
7. Go ahead and remove the deployment by sending the 'Delete Temporary L4-L7 Service' request.

### Task 4: Build a Service Template 

1. In the GUI logged in as 'admin', navigate to 'Clouds and Services -> Service Templates -> "+"'. Select the App Services Integration iApp as the template and select the appropiate version. Name your template. In this exercise we are going to create a minimalist service template for demonstration purposes.
<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/min1.png" alt="Drawing" style="width: 200px;"/>
2. Fill out the service tier information.
<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/min2.png" alt="Drawing" style="width: 200px;"/>
3. Expand the 'Virtual Server Listener & Pool Configuration' section. Make the 'Virtual Server: Address' tenant editable.
<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/min3.png" alt="Drawing" style="width: 200px;"/>
4. Create static entries for pool members according to your docker containers.
<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/min4.png" alt="Drawing" style="width: 200px;"/>
5. Set the 'state' to enabled.
<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/min5.png" alt="Drawing" style="width: 200px;"/>
6. Finally, expand the 'Virtual Server Configuration' section. Assign a 'Virtual Server: name' and a 'Virtual Server: Clientside L4 Protocol Profile'.
<img src="https://github.com/kreynoldsf5/TechSummit2017iWFLab/blob/master/Modules/images/min6.png" alt="Drawing" style="width: 200px;"/>
  
  * The first setting is prevent a small re-entrancy bug with the App Services Integration iApp (which occurs when the virtual server has no name).
  
  * The second setting is required in order for the iApp to deploy. Like in the GUI the iApp will assume that the same protocol profile should be used on the serverside as well.
  
7. What fields are tenant editable? What fields have default values but are not editable? Can you predict what this will look like to a tenant in the GUI when deploying? Can you predict what the POST payload for a L4-L7 service deployment will look like?

8. Just for fun, capture the POST payload when the service template is created using Chrome Dev Tools. 
  * Paste the payload into your favorite text editor. Find the ```vars``` section. 
  * Send the 'GET base iApp template' request in the 'Task 4' folder. Find the ```vars``` section. 
  * Compare these sections.
 
If you had issues creating the minimal template i've provided a 'Create Minimal Template' request in the 'Task 4' folder.

### Task 5: Deploying a Service Template
The JSON from our example 'minimal' template was lengthly as it defined all necessary variable defaults for the underlying iApp. The 'minimalist' aspect is the POST needed from the tenant to instantiate a deployment from the service template.

1. Log into the iWF GUI under the tenant account. Click on 'L4-L7 Service -> "+"'. Select the 'MinimalTemplate' from the 'L4-L7 Service Template' dropdown. This is the tenant's view of the template -- just the ability to specify a name, what connector to be deployed against, and to configure the virtual server address.
2. Fill out the form. Before hitting 'Save' capture the payload (with Chrome Dev Tools) in the tenant POST. 
```javascript
{"tenantTemplateReference":{"link":"https://localhost/mgmt/cm/cloud/tenant/templates/iapp/minimalTemplate"},
"properties":[{"id":"cloudConnectorReference","value":"https://localhost/mgmt/cm/cloud/connectors/local/89800a56-13d3-4a0c-96b7-c2331880705f"}],
"tables":[],
"vars":[{"name":"pool__addr","value":"10.1.200.56"}],
"name":"minService"}
```
After declaring the service ```name```, the template to be used, and which connector to deploy against, the payload specifies the *only* tenant editable value - a ```var``` called ```pool__addr``` and a specified ```value```.

### Task 6: Build Your Own
We've established a service deployment is simply a POST which specifies the iApp template, the connector, the name, and all necessary tenant editable values. Tenant editable values along with specified defaults from the service template are used to build the POST payload necessary for the ASO deployment on the target BIG-IP. Let's examine the syntax of the ```overrides``` section of the Service Template. Here is a small section from our template:

```javascript
  "overrides": {
    "vars": [
            {
                "name": "iapp__strictUpdates",
                "description": "iApp: Strict Updates",
                "displayName": "strictUpdates",
                "isRequired": false,
                "provider": "enabled"
            }
            ...
            {
                "name": "pool__addr",
                "description": "Virtual Server: Address",
                "displayName": "addr",
                "isRequired": true,
                "defaultValue": "",
                "providerType": "NODE",
                "serverTier": "default"
            },
            ]
          ...
```

Let's decipher this by differentiating between the JSON values from the iApp and the values added by the service template.

#### JSON values from the iApp:
```name``` is the variable name in the iApp.

```description``` is displayed in the GUI. 

```displayName``` is the field name in the GUI. 

```isRequired``` is a self explanatory boolean. 

#### JSON values added for the service template:
```provider``` is the defined default value.

```providerType``` is used for data input validation.

```serverTier``` dictates whether value is mapped to those few static service tier values.

```defaultValue``` determines whether the value is tenant editable.

The App Services Integration iApp is _well_ documented. The [Reference Guide](https://devcentral.f5.com/wiki/iApp.AppSvcsiApp_refguide.ashx) contains a cononical list of every variable used in the iApp. 

1. Build your own service template and then deploy a L4-L7 service. 

The easiest way would be to build the template through the GUI, perform a deployment as a tenant, and capture the JSON payload. Once you've captured the L4-L7 payload, edit the POST in the 'Task 6' folder to perform another service deployment. Be sure to change the name and the virtual server destination address.

The hard (fun) way is to build a template in your text editor. 
  - Start with the contents of the 'f5-http-lb' service template. Place this in your text editor.
  - change the value of ```templateName```
  - Find some values in the template (or in the App Services Integration iApp reference guide) that you want to alter/include in your template. 
  - Add the relevant service template JSON values (```provider```, ```providerType```, or ```defaultValue```) depending on what you want to change. For example, 


