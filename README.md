# autosleep_devdemo

Follow these steps to set up the autosleep service:

-- Prepare manifest: https://github.com/cloudfoundry-community/autosleep/blob/develop/doc/publish.md#prepare-your-manifest
* You will need a CC API user with cloudcontroller.admin scope
* Ignore the paragraph beginning "Autosleep service needs properties to work." This is a holdover from a previous version.
* Other than the CC API user info (cf.client.username and cf.client.password), all values are created from the manifest, so you can set them however you want.
* You will need to first create the database service and include the service name in the services section (ie mysql)
    
-- Deploy autosleep app
* 'cf push -f manifest.yml'
* If you are pushing the autowake-app, you will need to create the wildcare route.
  * cf create-route <autosleep-space> mydomain.org -n '*'
  * cf map-route autowakeup-app mydomain.org --hostname '*'

-- Publish on the market place: https://github.com/cloudfoundry-community/autosleep/blob/develop/doc/publish.md#publish-on-the-market-place
* you will need to expose the plan into all orgs (this is another area we might want to look into automating)
    
-- Create autosleep service instance: https://github.com/cloudfoundry-community/autosleep#create-your-autosleep-service-instance
* Look through the advanced configuration parameters.  For testing, you would likely want to at least change the idle-duration.  
* For my two minute test, I created the service with:
  * cf create-service autosleep default my-autosleep -c '{"idle-duration": "PT0H02M"}'
    * The service will need to be created in each space.
 
 
 Note about ignoring the healthcheck: This might take a little adjustment.  
The autosleep app determines if an app is inactive by looking at the start and stop events and the timestamps on the app logs.
Some issues I discovered:
* One app was consistently failing the healthcheck for some reason.  This resulted in additional logs being emitted, which meant the app was not turned off.
* An app did not have a /health endpoint, but did have a 404 re-direct, which resulted in additional logging, which resulted in the app not being turned off.
   
If an app does not seem to be responding to the autosleep service, please send me the recent app logs.  We might need to add additional items other than /health to the ignore function.
This will be easier when it is a configurable option, but for testing purposes, please just forward to me.


Example command sequence for deploying in org autosleep-test, space autosleep. Test apps in org autosleep-test, space test-apps.

* cf target -o autosleep-test -s autosleep
* cf create-service p-mysql 100mb mysql
* cf push
* cf create-service-broker autosleep autosleepsecurity autosleepsecurity <autosleep-app url>
* cf enable-service-access autosleep
* cf target -s test-apps
* cf create-service autosleep default my-autosleep -c '{"idle-duration": "PT0H02M"}'
