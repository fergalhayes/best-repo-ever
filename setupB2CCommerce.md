# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**  
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Setting Up Salesforce B2C Commerce
###### This guide captures the configuration and installation activities necessary to enable the service-cloud-connector's integration with Salesforce B2C Commerce.

### What You Need to Get Started

Before moving forward with the installation instructions, verify that your cross-cloud environment meets all installation prerequisites.  

#### Installation Prerequisites

- A Salesforce B2C Commerce sandbox running a storefront based on [SiteGenesis](https://github.com/SalesforceCommerceCloud/sitegenesis) OR [Storefront Reference Architecture](https://github.com/SalesforceCommerceCloud/storefront-reference-architecture).
- A Service Cloud sandbox mirroring your production org with [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5) and [Lightning](https://www.salesforce.com/campaign/lightning/) enabled.

#### Disqualifying Configurations

- A Salesforce B2C Storefront running on the Pipelines.
- A Service Cloud environment that runs on Salesforce Classic.
- A Service Cloud environment that does not support [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5).

> All installation prerequisites must be met in order for the service-cloud-connector to successfully deliver its complete feature-set.

### Configuring a Salesforce B2C Commerce Environment
This section of the document describes how to configure a Salesforce B2C Commerce environment to enable the service-cloud-connector's feature-set.

Be sure to select the SiteGenesis/SFRA-based site that will leverage the connector via the Site Selector found in the Business Manager header.

> In order to move forward with the configuration activities, ensure that you have access to a Business Manager account with Administrative privileges.

#### Import the Service Cloud Connector's Site Preferences
The service-cloud-connector introduces three site preferences to manage the operation of the service-cloud-connector within the Salesforce B2C Commerce environment.  This section of the document provides guidance on how to upload, import, and configure the site preferences.

##### Upload the System Object Definitions Import File
The following instructions describe how to upload the site preferences definition file so that it can be imported.

1. In Business Manager, select **Administration > Site Development > Import & Export**.
1. Click **Upload** in the 'Import & Export Files' section.
1. Click **Choose File** in the 'Upload Import Files' section.
1. Locate and select the [system-object](../b2c/sfcc/sites/site_template/meta/system-objecttype-extensions.xml) type definitions in the sfcc [meta](../b2c/sfcc/sites/site_template/meta) directory.
1. With the import file selected, click **Upload** to upload the import file.

##### Import the System Object Definitions
The following instructions describe how to import the system object definitions containing the service-cloud-connector's site preferences.

> The [system-object](../b2c/sfcc/sites/site_template/meta/system-objecttype-extensions.xml) import file must be uploaded to Business Manager as a pre-requisite to the import process.

1. In Business Manager, select **Administration > Site Development > Import & Export**.
1. Click **Import** in the 'Meta Data' section.
1. From the System Type Extension Import page, select the file named [system-objecttype-extensions.xml](../b2c/sfcc/sites/site_template/meta/system-objecttype-extensions.xml) and click **Next**.
1. The import process attempts to validate the [system-objecttype-extensions.xml](../b2c/sfcc/sites/site_template/meta/system-objecttype-extensions.xml) file.  Confirm that the file is validated without any errors, and click **Next** to continue.
1. Allow the import file to complete its processing, and confirm that the import is successful.

	> The 'Status' section shows the import progress of the file being processed.  A successful import has a process status of Success.

##### Configure the Imported Site Preferences
The following instructions describe how to import and configure the service-cloud-connector's site preferences.

1. In Business Manager, select **Merchant Tools > Site Preferences > Custom Preferences**.
1. Select the 'ServiceCloudConnector' site preference group displayed on the 'Custom Site Preference Groups' page.
1. Update the site preference using the following values:

	| Preference Name | Value | Description |
	| --------------- | ------| ------ |
	| Is async mode for Service Cloud? | No |  When set, the Service Cloud Connector drives cross-cloud data synchronization via its scheduled jobs; otherwise, synchronization is processed in real-time with the scheduled jobs operating as a fallback. |

1. Click **Save** at the top of the Custom Preferences display to save your changes.

	Confirm that the ServiceCloudConnector's custom preferences have been saved.  Once the preferences have been saved, continue with the import of the [custom object](../b2c/sfcc/sites/site_template/meta/custom-objecttype-definitions.xml).

#### Import the Service Cloud Connector's Custom Object
The service-cloud-connector introduces a custom object used to store the generated [Service Cloud AuthToken](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1) retrieved from Service Cloud during the authentication process.  This section provides guidance on how to import the [custom object definition](../b2c/sfcc/sites/site_template/meta/custom-objecttype-definitions.xml).

##### Upload the Custom Object Definition Import File
The following instructions describe how to upload the custom object definition file so that it can be imported.

1. In Business Manager, select **Administration > Site Development > Import & Export**.
1. Click **Upload** in the 'Import & Export Files' section.
1. Click **Choose File** in the 'Upload Import Files' section.
1. Locate and select the [custom-object](../b2c/sfcc/sites/site_template/meta/custom-objecttype-definitions.xml) definition file in the sfcc [meta](../b2c/sfcc/sites/site_template/meta) directory.
1. With the import file selected, click **Upload** to upload the import file.

##### Import the Custom Object Definition
The following instructions describe how to import the system object definitions containing the service-cloud-connector's site preferences.

> The [custom-object](../b2c/sfcc/sites/site_template/meta/custom-objecttype-definitions.xml) import file must be uploaded to Business Manager as a pre-requisite to the import process.

1. In Business Manager, select **Administration > Site Development > Import & Export.**
1. Click **Import** in the 'Meta Data' section.
1. From the import display, select the file named [custom-objecttype-definitions.xml](../b2c/sfcc/sites/site_template/meta/custom-objecttype-definitions.xml) and click **Next**.
1. The import process attempts to validate the [custom-objecttype-definitions.xml](../b2c/sfcc/sites/site_template/meta/custom-objecttype-definitions.xml) file.  Confirm that the file is validated without any errors, and click **Next** to continue.
1. Allow the import file to complete its processing, and confirm that the import is successful.

	> The 'Status' section shows the import progress of the file being processed.  A successful import has a process status of __Success__.

Once the import is complete, move forward with the importing of the Service Cloud service definition.

#### Import the Services Definition
The service-cloud-connector requires a service that must be configured to interact with the Service Cloud instance.

##### Upload the Service Definition
The following instructions provide guidance on how to import and configure the service definition.

1. In Business Manager, select **Administration > Operations > Import & Export**.
1. Click **Upload** in the 'Import & Export' section.
1. Click **Choose File** displayed in the 'Upload Import Files' section.
1. Locate and select the [services](../b2c/sfcc/sites/site_template/services.xml) definition file in the sfcc [site_template](../b2c/sfcc/sites/site_template) directory.
1. With the import file selected, click **Upload** to upload the import file.

##### Import the Service Definition
The following instructions describe how to import the service definition leveraged by the service-cloud-connector.

> The [services](../b2c/sfcc/sites/site_template/services.xml) import file must be uploaded to Business Manager as a pre-requisite to the import process.

1. In Business Manager, select **Administration > Operations > Import & Export**.
1. Click **Import** in the 'Services' section.
1. Select the file named [services.xml](../b2c/sfcc/sites/site_template/services.xml) and click **Next**.
1. The import process attempts to validate the [services.xml](../b2c/sfcc/sites/site_template/services.xml) file.  Confirm that the file is validated without any errors, and click **Next** to continue.
1. From the 'Import' step, select **Merge** as the import mode, and click **Import** to process the uploaded file.
1. Allow the import file to complete its processing, and confirm that the import is successful.

	> The 'Status' section shows the import progress of the file being processed.  A successful import has a process status of __Success__.

Once the import is complete, move forward with the configuration of the Service Cloud service definition.

#### Configure the Service Definition
The following instructions describe how to configure the service definition leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Services**.
1. Click the **Credentials** tab.
1. Edit the _servicecloud.auth-SiteId_ service credentials by clicking the credential name.
1. Update the suffix of the credential name and replace *SiteId* with your site's siteId.
1. Configure the credentials for the service-cloud-connector service using the following properties:

	| Property Name | Value | Description |
	| ---------------- | -------------------------- | --------------------- |
	| URL | https://__auth-domain__/services/oauth2/token | Enter the Salesforce authentication url; use __login.salesforce.com__ for production and __test.salesforce.com__ for development sandboxes.|
	| User | Service Cloud SCC Integration User ID | Specify the username of the SCC Integration User for the Service Cloud environment. |
	| Password| Service Cloud SCC Integration User Password + SecurityToken | Specify the password and securityToken of the SCC Integration User for the Service Cloud environment. |
	| Consumer Key | ex. 9srSPoex34FV0eWhMyij4PEG4Cddk | Specify the Consumer Key associated to the Service Cloud CommerceCloudConnector Connected App. |
	| Consumer Secret| ex. 2928758854267628764 | Specify the Consumer Secret associated to the Service Cloud CommerceCloudConnector Connected App. |

	> Remember that the URL should use the *login* or *test* domain prefix depending on the Salesforce environment being connected via the service-cloud-connector.  You can review the guidance provided in the [Setup Service Cloud](./setupServiceCloud.md#determine-which-callback-url-to-use-in-the-connected-app) documentation to determine which URL to use.

	> The Consumer Key and Consumer Secret used for [OAuth2 authentication](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1) and can be retrieved by logging into Service Cloud and [viewing](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1) the CommerceCloudConnected Connected App via 'Setup'.

1. Once the service credentials and Service Cloud Connector consumer properties have been updated, click **Apply** to save your changes.

Among the imported service-cloud-connector services is a service named *servicecloud.rest*.  Do not modify the configuration of this service, as its profile and credentials are generated programmatically at runtime.

### Configuring the OCAPI Settings for the Service Cloud Connector
This section of the document describes how to configure the OCAPI Shop and Data API settings to support access to the resources required by the service-cloud-connector.

#### Configure the Shop API
The Open Commerce Shop API configuration settings should be extended to allow access form the connected Salesforce environment and include the resources required by the service-cloud-connector.

##### Configure the Allowed Origins for the Shop API
The OCAPI Shop API configuration should be extended to enable access from the connected Service Cloud environment.

1. In Business Manager, select **Administration > Site Development > Open Commerce API Settings**.
1. Select the API type Shop, and the storefront site that will be connected via the service-cloud-connector.
1. Specify version-format 18.8 for the OCAPI API resource-configuration document.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	  
	  ]
	}
	```

1. For this new version, specify the 'client_id' that will be used to access the Data API
1. In the 'allowed_origins' property of the JSON configuration file, include the Service Cloud instance's lightning and classic domain name that will be connected.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	    {
	        "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
	        "allowed_origins":["https://mydomain.lightning.force.com", "https://mydomain.my.salesforce.com"]
	    }
	    
	  ]
	}
	```

	> Remember to append the new Service Cloud origins to the existing collection of origins.

	> At a minimum, the origin should include the instance_url returned by the [Salesforce remote authorization request](./validateEnvironments.md#supported-test-scenarios).  We recommend including the lightning and classic urls for your instance.

	> View this article to learn more about setting up a [my domain](https://help.salesforce.com/articleView?id=domain_name_overview.htm&type=5) for your instance.

##### Configure the Resource Definitions for the Shop API
The OCAPI Shop API resource definitions should be extended to include the resources required by the service-cloud-connector.

1. Add the following resource definitions to the existing 'resources' property of the JSON configuration file.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	    {
	        "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
	        "allowed_origins":["https://mydomain.lightning.force.com", "https://mydomain.my.salesforce.com"],
	        "resources":[
	            {
	              "resource_id":"/customers/*",
	              "methods":["get"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            },
	            {
	              "resource_id":"/customers/*/auth",
	              "methods":["post"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            },
	            {
	              "resource_id":"/orders/*",
	              "methods":["get","patch"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            },    
	            {
	              "resource_id": "/sessions",
	              "methods":["post"],
	              "read_attributes": "(**)",
	              "write_attributes": "(**)"
	            }   
	        ]
	    }
	    
	  ]
	}
	```

	> When adding the service-cloud-connector resources,  review your existing resources and ensure that you do not overwrite them wholesale. Only add the resources that are not already included in the existing OCAPI resources.

1. With both changes applied to the Shop API configuration .json file, click **Save** to save your changes.

#### Configure the Data API
Similar to the Shop API, the Open Commerce Data API configuration settings should be extended to allow access form the connected Salesforce environment and include the resources required by the service-cloud-connector.

##### Configure the Allowed Origins for the Data API
The OCAPI Data API configuration should be extended to enable access from the connected Service Cloud environment.

1. In Business Manager, select **Administration > Site Development > Open Commerce API Settings**.
1. Select the API type Data, and Global (organization wide) from the OCAPI toolbar.
1. Specify version-format 18.8 for the OCAPI API resource-configuration document.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	  
	  ]
	}
	```

1. For this new version, specify the 'client_id' that will be used to access the Data API
1. In the 'allowed_origins' property of the JSON configuration file, include the Service Cloud instance domains added as part of the Shop API configuration.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	    {
	        "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
	        "allowed_origins":["https://mydomain.lightning.force.com", "https://mydomain.my.salesforce.com"]
	    }
	    
	  ]
	}
	```

	> Remember to append the new Service Cloud origin to the existing collection of origins.

	> At a minimum, the origin should include the instance_url returned by the [Salesforce remote authorization request](./validateEnvironments.md#supported-test-scenarios).  We recommend including the lightning and classic urls for your instance.

	> View this article to learn more about setting up a [my domain](https://help.salesforce.com/articleView?id=domain_name_overview.htm&type=5) for your instance.

##### Configure the Resource Definitions for the Data API
The OCAPI Data API resource definitions should be extended to include the resources required by the service-cloud-connector.

1. Add the following resource definitions to the existing 'resources' property of the JSON configuration file.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	    {
	        "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
	        "allowed_origins":["https://mydomain.lightning.force.com", "https://mydomain.my.salesforce.com"],
	        "resources":[
	            {
	              "resource_id":"/customer_lists/*",
	              "methods":["get"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            },
	            {
	              "resource_id":"/customer_lists/*/customer_search",
	              "methods":["post"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            },
	            {
	              "resource_id":"/customer_lists/*/customers/*",
	              "methods":["get","put","patch","delete"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            },
	            {
	              "resource_id":"/customer_lists/*/customers/*/addresses",
	              "methods":["get","post"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            },
	            {
	              "resource_id":"/customer_lists/*/customers/*/addresses/*",
	              "methods":["get","patch","delete"],
	              "read_attributes":"(**)",
	              "write_attributes":"(**)"
	            }
	        ]      
	    }
	    
	  ]
	}
	```

	> When adding the service-cloud-connector resources, review your existing resources and ensure that you do not overwrite them wholesale.  Only add the resources that are not already included in the existing OCAPI resources.

1. With both changes applied to the Data API configuration .json file, click **Save** to save your changes.

### Continue with the Setup of the Salesforce B2C Commerce Scheduled Jobs
With the initial configuration of Salesforce B2C Commerce environment completed, continue with the [setup of the Salesforce B2C Commerce Jobs](setupB2CCommerceJobs.md) to incorporate the [int_service_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge and set up the scheduled jobs used to synchronize customer and order data across Salesforce B2C Commerce and Service Clouds.
