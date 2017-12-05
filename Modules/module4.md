# Module 4:  Onboarding Tasks with iControlLX Extensions

PD currently provides two iControl LX extensions designed to be used for onbaoarding tasks.

[BIG-IP Settings](https://devcentral.f5.com/wiki/DevOps.HowToSamples_bigip_settings_reset.ashx)

[BIG-IP Cluster](https://devcentral.f5.com/wiki/DevOps.HowToSamples-create-bigip-cluster.ashx)

These packages can be installed on a BIG-IP device or an iWF device. We'll use our iWF device for this Lab. Due to limitations in the UDF we can't walk through the BIG-IP Clustering extension. These extensions use the 'block' interface which, although functionally similiar, is a more feature filled pattern than the 'tasks' pattern we've seen in our previous examples.

Our target device for this exercise will be the BIGIP-2 device in our environment.

## BIG-IP settings worker
The 'BIG-IP Settings' worker provides a simple declarative interface for base onboarding tasks. The following items can be configured through this worker:

* NTP servers
* DNS servers
* timezone
* hostname and FQDN
* Syslog servers
* licensing
* resetting the device to factory defaults

1. SCP the RPM in '{{repo}}/f5-rest-devops-bigip-settings' to the ```/var/config/rest/downloads``` directory of the iWF device. Installation must be performed from this directory.

2. Send the first request in the Module 4 Postman Collection -'Install BIGIP Settings RPM'. The POST payload for this request is simply the ```operation``` and the ```package``` path.
  * What was the response code to this POST?
  * How does this HTTP response code differ from a 200?

3. The next request will check the status of the installation task based on the task ```Id``` returned in the install POST. Check ```/var/log/restnoded/restnoded.log``` for troubleshooting purposes. Note how the RPM is installed in ```var/config/rest/iapps/f5-rest-bigip-settings/```. This is simply another iControlLX extension that uses 'block templates' as opposed to a 'WORKER_URI_PATH'. 
  * This interface provides a pattern for persistence
  * The iControl LX examples in module 3 executed after the HTTP request and generated the HTTP response. What if the extension's task was long running (ie. longer than an HTTP request timeout)?


4. 'Find block template ID' will query available block templates and return and array of 1 containing the 'big-ip settings' template. The id/GUID is a unique identifier for the block associated with this worker.
  * The filter "$filter=state eq 'TEMPLATE' and name eq 'bigip-settings'" returns just templates with name we are looking for.
  * Use [Query parameters](https://devcentral.f5.com/articles/demystifying-icontrol-rest-part-3-how-to-pass-query-parameters-and-tmsh-options) for iControl where applicable. 


5. Use the 'Retrieve block template' request to return an example POST payload we can use for a deployment (an instance of a block that will be passed to bigIpSettingsConfigProcessor.js -- the worker).

6. The 'Create an instance block' request POSTs the desired settings to the worker. The request contains the following differences from the example block we returned earlier:
  * ```inputProperties``` contains our desired settings
  * the ```state``` is now ```BINDING```
  * ```sourcePackage``` is removed -- this only applies to block templates
  * a ```baseReference``` has been added to reference the block template
  * the ```deviceReference``` has been updated to point to the target device. This could be 'localhost' if modifying the device where the package is installed (not in our case).

7. Correct/Verify the deviceReference link for your BIGIP-2 device. 
  * Hint: if you discovered this device in Module 1 the ```selfLink``` should be {{bigip2_device_selfLink}}.

8. Change a handful of settings in 'inputProperties'. Use the values for the settings you'd use if running tmsh commands. POST the block with 'Deploy Provisioning Task'.

7. Check progress with 'Check Provisioning Task Status'. Once again we need the GUID from our instance block in order to check the task status. The important value is ```state``` which will go from 'BINDING' to the final state. 'BOUND' means the task has succeeded.

8. When the task completes, settings can be updated with a PATCH to the same instance block .
  * You should create *another* instance block for running the worker against a different device. 

### Device Reset
Use the provided 'Reset Device to Factory Defaults' request to blank out a BIG-IP. All setting after 'resetToDefaults' are ignored but are still required in the payload.

By accepting the EULA, providing a base registration key you can license a device through this interface. Development licenses would work if connected to the VPN but this will not work in the UDF.


