# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**   
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Setting Up Service Cloud
###### This guide captures the configuration and installation activities necessary to enable the service-cloud-connector's integration with Salesforce B2C Commerce.

### What You Need to Get Started

Before continuing with the installation instructions, verify that your cross-cloud environment meets all installation prerequisites.  This includes:

#### Installation Prerequisites

- A Salesforce B2C Commerce Cloud sandbox, running a storefront based on either [SiteGenesis](https://github.com/SalesforceCommerceCloud/sitegenesis) *OR*  the [Storefront Reference Architecture](https://github.com/SalesforceCommerceCloud/storefront-reference-architecture).
- A Service Cloud sandbox mirroring your production org with [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5) and [Lightning](https://www.salesforce.com/campaign/lightning/) enabled.

#### Disqualifying Configurations

- A Salesforce B2C Storefront running on the Pipelines.
- A Service Cloud environment that runs on Salesforce Classic.
- A Service Cloud environment that does not support [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5).

> All installation prerequisites must be met in order for the service-cloud-connector to successfully deliver its complete feature-set.

### Start with a Salesforce Sandbox

> Since the project has several configuration points, we recommend that this collection of configuration instructions be first validated in a [Salesforce sandbox](https://help.salesforce.com/articleView?id=data_sandbox_create.htm&type=5) environment. 

#### Salesforce Sandbox Prerequisites
Before continuing with the setup of the service-cloud-connector in your sandbox, ensure that your sandbox:

- Mirrors the schema and customizations of your production org.
- Has PersonAccounts enabled and configured.
- Has Lightning enabled.

> Verify that your environment meets all installation pre-requisites outlined in this page before proceeding with the Service Cloud configuration instructions.

#### Why Do I Need PersonAccounts?
The service-cloud-connector requires that Service Cloud use [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5)  to manage customer profiles.

- PersonAccounts are a top-level requirement for the service-cloud-connector.
- Salesforce Orgs that use Accounts and Contacts aren't supported by the service-cloud-connector.

For more information on enabling [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5), see the Salesforce [Setting Up PersonAccounts Guide](https://resources.docs.salesforce.com/210/latest/en-us/sfdc/pdf/salesforce_B2C_implementation_guide.pdf).  You can also review this [Salesforce Help](https://help.salesforce.com/articleView?id=account_person_enable.htm&type=5) article for additional information.

> Moving an existing org to PersonAccounts is a non-trivial activity that has ramifications beyond this integration.  If you are unsure of this change, we recommend that you connect with your Success Manager to discuss its impact to your organization.

#### Why Do I Need Lightning?
The service-cloud-connector provides consolidated customer information through Lightning UI.  In this UI, customer, case, and order data can be interacted with via unified displays.  These views are not available to Salesforce Classic environments.

### Pre-Installation Configuration

###### The following configuration steps must be performed before deploying it to your Salesforce instance.

#### Create an SCC Integration User Profile
Before deploying the service-cloud-connector, create a custom profile by cloning the System Administrator profile.

1. From Setup, enter Profiles in the quick find box, then select **Profiles**.
1. From the profiles list, select **Clone** from the System Administrator profile.
1. In the clone form, enter the following values:

	| Property | Value |
	|------------ | ---------- |
	| Profile Name | SCC Integration User |
	| Existing Profile| System Administrator |

	> The Profile Name *must* be *SCC Integration User* as the deployment package depends on this profile existing before installation.

1. To save your changes, lick **Save**.
1. Verify that the Edit view for the SCC Integration User is displayed after saving your changes.

	> If not, find the 'SCC Integration' user in the 	profile listing and click **Edit** next to 	the profile to open this view.

1. Click **System Permissions** to view system-level permissions for the profile.
1. Ensure that 'API Enabled' is selected for this profile.

#### Create an SCC Integration User
With the SCC Integration User Profile created, create the SCC Integration User.  The SCC Integration user
is used by Service Cloud to connect to Salesforce B2C Commerce.  This user must have a
User License set to Salesforce, and leverage the newly created SCC Integration User custom profile.

1. From Setup, enter Users in the quick find box, then select **Users**.
1. Click **New User** at the top of the users listing.
1. Create a user using the following properties:

	| Property | Value |
	|------------ | ---------- |
	| First Name | User |
	| Last Name | SCCIntegration |
	| Alias | uscci |
	| Email | system email associated to the account |
	| Username | sccintegrationuser@domainname.com |
	| Nickname | sccintegrationuser |
	| Role | <None Specified> |
	| User License | Salesforce |
	| Profile | SCC Integration user |
	| Active | Checked |

	> Remember to use a valid email address for the 	account's email, as the account requires verification. The email 	and username for this user-account should both be 	personalized to the organization.

1. Verify that the account details are correct.
1. Click **Save** button to create the account.
1. Verify that you have received the new-user email.
1. Complete the new-user registration.
1. Log in as the SCC Integration User.

### Deploying the service-cloud-connector Codebase to Service Cloud
The service-cloud-connector's Service Cloud codebase can be deployed to any Salesforce environment leveraging the [Apache Ant](https://ant.apache.org/) installer, located in the [sfsc](../b2c/sfsc/) directory.

> If you don't have Ant installed, visit the [Apache Ant](https://ant.apache.org/) homepage for details on how to do so.  If you are a Mac user, you can use [Homebrew to install Ant](https://stackoverflow.com/questions/3222804/how-can-i-install-apache-ant-on-mac-os-x), to set up Apache Ant.  Additionally, the [Apache Ant FAQ](https://ant.apache.org/faq.html) is a great reference for troubleshooting and configuration details.

In a terminal session, enter the following command to verify that Apache Ant is correctly set up:

```
ant -version
```

Which should return the following result:

```
Apache Ant(TM) version 1.10.5 compiled on July 10 2018
```

With Ant properly installed, create the deployment properties configuration file.

#### Setup the Deployment Properties Configuration File
The service-cloud-connector uses a [build.properties](../b2c/sfsc/salesforce\_ant\_43.0/build.properties.sample) file to define the configuration settings necessary to enable the deployment of the service-cloud-connector to a Salesforce instance.  Follow these instructions to configure the build.properties file to map to your Salesforce environment.

1. Duplicate the [build.properties.sample](../b2c/sfsc/salesforce\_ant\_43.0/build.properties.sample) file found in the [salesforce_ant_43.0](../b2c/sfsc/salesforce\_ant\_43.0) directory.
1. Rename the copy of the build.properties.sample file to build.properties.
1. Open the newly created build.properties file.
1. Uncomment line 11, and replace it with the Administrator Account username for the Salesforce instance where the Service Cloud Connector is deployed.
	
	```
	# Manual Deployment: Configuration Step 1
	# - Uncomment the sf.username and sf.password lines
	# -------------------------------------------
	# Specify the login credentials for the desired Salesforce organization
	# The sf.password field must be comprised of the account-password + the account securityToken
	# sf.username = administrator-level-username
	# sf.password = administrator-password
	```

	> This should be deployed leveraging a user account that is different from the SCC Integration User.

1. Uncomment line 12, and replace it with the Administrator Account's password and [Security Token](https://help.salesforce.com/articleView?id=user_security_token.htm&type=5).

	> For guidance on how to reset your Security Token, visit the section of this document titled [Reset Your Security Token](#reset-your-security-token).

1. Uncomment line 27, and replace it with the authentication URL for your Salesforce instance. If you are unsure of what authentication URL to use,  visit [What are Salesforce instances?](https://success.salesforce.com/answers?id=90630000000gugYAAQ) and [How to identify the type of Salesforce Org](https://salesforce.stackexchange.com/questions/50/can-we-determine-if-the-salesforce-instance-is-production-org-or-a-sandbox-org).  These pages explain how to determine your instance-type and the authentication URL to use.

	```
	# Manual Deployment: Configuration Step 2
	# - Uncomment the sf.serverurl line
	# -------------------------------------------
	# Use 'https://login.salesforce.com' for production or developer edition (the default if not specified).
	# Use 'https://test.salesforce.com' for all sandbox environments.
	# sf.serverurl = https://test.salesforce.com
	```

1. Save the changes that were made to the build.properties file.

Before the execution of the deployment script, validate that the three edits included in this section are accurate and contain the correct Administrative Account username and password, Security Token, and authentication URL.

#### Execute the Deployment Script
The deployment is performed from a command-line or terminal session, depending on the platform of your workstation.

> When executed, the script deploys the full-contents of the package included in this repository to the configured Salesforce instance.  It overwrites any classes, objects, and configurations that exist for the items in the package. Remember that if you redeploy to an environment, you are forced to reconfigure that environment.

1. Open a command prompt or terminal session and navigate to the [salesforce_ant_43.0](../b2c/sfsc/salesforce_ant_43.0) directory.
2. From the command prompt, enter the following command:

	```
	ant deployCodeNoTestLevelSpecified
	```

The deployment script typically takes a handful of minutes to execute -- and providea a message stating the deployment was successful if no errors are encountered during the deployment. Once the success message has been generated, move forward with the execution of the post installation script.

### Execute the Post Installation Script
The first post-installation activity that must be completed is the manual execution of the post-installation script.  The post-install script performs
several configuration activities that must be completed before configuring Service Cloud. This includes:

1. Create Setting leveraged by the service-cloud-connector to manage its operation.
1. Seed the Custom Field Mappings used to map Salesforce B2C Commerce objects to Service Cloud objects.
1. Create the Default PersonAccount leveraged by the service-cloud-connector as a default customer mapping.

These instructions explain how to execute this script from the [Salesforce Developer Console](https://help.salesforce.com/articleView?id=code_dev_console.htm&type=5). Visit the [Trailhead Module Developer Console Basics](https://trailhead.salesforce.com/en/content/learn/modules/developer_console) for a tutorial of the various capabilities present within the [Developer Console](https://help.salesforce.com/articleView?id=code_dev_console.htm&type=5).

1. Log out as the SCC Integration user.
1. Click the Setup gear icon, and select **Developer Console**.
1. From the Developer Console, open the debug menu and select **Open Execute Anonymous Window**.
1. Paste the following snippet of code in the modal-dialog labeled 'Enter Apex Code':

	```apex
	// Execute the post-installation script to insert custom-settings data
	SCCConnectorPostInstallScript.insertCustomSettingsData();
	```

1. Click **Execute**.

	> Executing the insertCustomSettingsData() function seeds the Service Cloud instance with constants required by the service-cloud-connector.  You can view the generated log-file for execution details, making sure that the logs do not contain any references to 'error', 'exception', or 'fatal' keywords.

1. To test that the custom settings data was inserted successfully, execute the following [SOQL database query](https://trailhead.salesforce.com/content/learn/modules/apex_database/apex_database_dml).

	```sql
	/* Retrieve the Account field-mapping configuration values
	   Inserted as Custom Settings to Service Cloud */
	select  Id,
	        Field_Api_Name__c,
	        CC_Attribute__c,
	        Enable_Patch__c,
	        Enable_Sync__c,
	        CreatedById,
	        CreatedDate
	from    AccountFieldMapping__c
	```
	
With the post-installation script created, you can now begin to configure Service Cloud.

### Determine Which Callback URL to Use in the Connected App
As a precursor to creating the Connected App used by Service Cloud, you need to identify the type of Salesforce Org you're currently running.  This is necessary as each org-type (Development or Production) has a unique callback URL.  The two callback URLs are outlined in the following table.

> The deployment package should automatically set this value. This section has been included in the documentation supplementarily, describing how callback URLs are environment driven.

| Callback URL Type | URL |
|-------------------|-----|
| Production | https://login.salesforce.com/services/oauth2/success |
| Development | https://test.salesforce.com/services/oauth2/success |

If you are unsure what type of environment you currently have, review the following articles:

- [What are Salesforce instances?](https://success.salesforce.com/answers?id=90630000000gugYAAQ) provides an explanation about the different Salesforce instances that exist and how to tell what instance you're running by inspecting your instance URL.
- [How to identify the type of Salesforce Org](https://salesforce.stackexchange.com/questions/50/can-we-determine-if-the-salesforce-instance-is-production-org-or-a-sandbox-org) explains how to execute an SOQL query to identify your org's instance type.  If this query returns true, your Org is a sandbox.  Otherwise, your Org is a production instance.

	```sql
	/* Execute this ad hoc query to determine if you're running on a sandbox */
	SELECT IsSandbox FROM Organization LIMIT 1
	```

Once you have identified the instance-type of your Salesforce Org, use the corresponding Callback URL when modifying the CommerceCloudConnector Connected App as part of the next set of installation instructions.

### Modify the CommerceCloudConnector Connected App
With the SCC Integration user created, log in as that user.  The SCC Integration user must be logged in to modify the [Connected App](https://developer.salesforce.com/page/Connected_Apps) that is  used to facilitate connections to Salesforce B2C Commerce.

1. Click the Setup gear icon, and click **Setup**.
1. From the 'App' menu, select **App Manager**.
1. In the listing of applications, find the Connected App named CommerceCloudConnector.
1. Select **Edit** option from the dropdown menu on the right to edit the properties of the Connected App.
1. Verify and update the properties of the CommerceCloudConnector Connected App using the following table.

	> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be pre-set by the deployment package and post-installation script.

	| Property | Modify? |Value |
	|------------ |:-----:|---------- |
	| Connected App Name | No | CommerceCloudConnector |
	| API Name | No | CommerceCloudConnector |
	| Contact Email | Yes | system email associated to the account |
	| Enable OAuth Settings | Yes | checked |
	| Enable for Device Flow | Yes | checked |
	| Callback URL | Yes | Specify the Production or Development [Callback URL](#determine-which-callback-url-to-use-in-the-connected-app) |
	| Select OAuth Scopes | No | Full access (full) |
	| Require Secret for Web Server Flow | No | checked |
	
	> Specify a valid email address for the 'Contact Email' field.

	> The callback URL used here is automatically seeded by the deployment package.  If the URL does not align with your Salesforce Org type, update it appropriately using the guidance provided via the section titled [Determine Which Callback URL to Use in the Connected App](#determine-which-callback-url-to-use-in-the-connected-app).

1. Confirm that the form fields are completed and accurate.
1. Click **Save**.

### Configuring the Connected App
With the CommerceCloudConnector [Connected App](https://developer.salesforce.com/page/Connected_Apps) updated, it must be configured and associated to the internal org-user that is used to facilitate authenticated communications with Salesforce B2C Commerce.

#### Relax the CommerceCloudConnector's IP Restrictions
The first change we must make to the Connected App is to [relax IP restrictions](https://help.salesforce.com/articleView?id=connected_app_continuous_ip.htm&type=5).  This change allows the integration user to log in to the configured Salesforce B2C Commerce instance regardless of any [IP restrictions enabled on the user profile](https://salesforce.stackexchange.com/questions/83193/relax-ip-restrictions-in-connected-app).

To enable this as part of the Connected App's policy:

1. Click the Setup gear icon, and click **Setup**.
1. From the 'App' menu, select **App Manager**.
1. Find the CommerceCloudConnector connected app, in the application listing.
1. Select the 'manage' option for CommerceCloudConnector connected app.
1. From the properties display, click **Edit Policies** option.
1. Set the IP Relaxation Oauth policy property to 'Relax IP restrictions'.

	| Property | Value |
	|------------ | ---------- |
	| IP Relaxation | Relax IP restrictions |

1. Click **Save** to save your changes.

### Assign the Connected App to the SCC Integration User Profile
With the CommerceCloudConnector Connected App updated and configured, you can now assign it to the SCC Integration User profile.

1. From Setup, enter Profiles in the quick find box, then select **Profiles**.
1. From the profiles listing, open the SCC Integration User profile.
1. From the apps menu click the **Assigned Connected Apps**.
1. Find the 'Assigned Connected Apps' heading and click **Edit**.
1. From the edit view of the SCC Integration user profile, select the CommerceCloudConnector connected app.
1. Click **Add** to move the CommerceCloudConnector connected app to the 'Enabled Connected Apps' field.
1. Click **Save**.

The CommerceCloudConnector now appears as a connected app associated to this profile.

### Configure the Remote Site Settings
The [Remote Site Settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_callouts_remote_site_settings.htm) are used to configure the location and configuration properties of the Salesforce B2C Commerce endpoint to which Service Cloud connects to.

1. From Setup, enter Remote Site Settings in to the quick find box, then select **Remote Site Settings**.
1. From the listing, locate the remote site named 'SCCSG'.
1. Click **Edit** for this remote site, and update the following properties:

	> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be pre-set by the deployment package and post-installation script.

	| Property | Modify? | Value |
	|------------ |:----:| ---------- |
	| Remote Site Name | No | SCCSG |
	| Remote Site URL | Yes | https://xxxx-dw.demandware.net |
	| Description | Yes |  |
	| Active |  No | checked |
	| Disable Protocol Security |  No | unchecked |

	> Replace the default remote-site URL (https://xxxx-dw.demandware.net) with the URL representing the Salesforce B2C Commerce environment being integrated with Service Cloud.

1. Click **Save**.

The SCCSG record should now appear with the updated remote site URL.

### Configure the Named Credentials
Service Cloud leverages [named credentials](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) attached to the SCC Integration User's profile, which are now available to customize.

1. From Setup, enter Named Credentials in the quick find box, then select **Named Credentials**.
1. From the listing, verify that the *SFCCClientCreds*, *SFCCClientCredsBearer*, and *SFCCUserCred* credentials are present in the credential listing.

	> The __SFCCClientCreds__ credential should not be modified. It is used internally by the service-cloud-connector, and is already configured.

#### Update the SFCCClientCredsBearer Named Credential
The SFCCClientCredsBearer [named credential](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) interacts with the Salesforce B2C Open Commerce API (OCAPI).  Update the URL property to represent the Salesforce B2C Commerce environment being integrated with Service Cloud.

> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be pre-set by the deployment package and post-installation script.

| Property | Modify? | Value |
|------------ |:-------:|---------- |
| Label | No | SFCCClientCredsBearer |
| Name | No | SFCCClientCredsBearer |
| URL | *Yes* | https://xxxx-dw.demandware.net |
| Identity Type | No | Named Principal |
| Authentication Protocol | No | No Authentication |
| Generate Authorization Header | No | false |
| Allow Merge Fields in HTTP Header | No | checked |
| Allow Merge Fields in HTTP Body | No | checked |

> Replace the default remote-site URL (https://xxxx-dw.demandware.net) with the URL representing the Salesforce B2C Commerce environment being integrated with Service Cloud.

1. Click **Save**.



#### Update the SFCCUserCreds Named Credential
The SFCCClientCredsBearer [named credential](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) is used to enable the order-on-behalf of (OOBO) functionality within Service Cloud.  Update the properties of the named credential using the following values.

| Property | Modify? | Value |
|------------ |:---:| ---------- |
| Label | No | SFCCUserCreds |
| Name | No | SFCCUserCreds |
| URL | *Yes* | https://xxxx-dw.demandware.net |
| Identity Type | No | Per User |
| Authentication Protocol | No | No Authentication |
| Generate Authorization Header | No | checked |
| Allow Merge Fields in HTTP Header | No | unchecked |
| Allow Merge Fields in HTTP Body | No | unchecked |

> Replace the default remote-site URL (https://xxxx-dw.demandware.net) with the URL representing the Salesforce B2C Commerce environment being integrated with Service Cloud.

1. Click **Save**.


#### Configure the Authentication Settings for External Systems
To enable the OOBO functionality, the SFCCUserCreds [named credential](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) must be mapped to a specific Salesforce B2C Commerce environment Business Manager user account.  
To create this account mapping:

1. Log in as the SCC Integration User.
1. In the header, click the user profile icon, and click **Settings**.
1. From the 'My Personal Information' menu, click [Authentication Settings for External Applications](https://help.salesforce.com/articleView?id=external_authentication.htm&type=0).
- From the list of settings, click **New**.
- Create a set of credentials using the following property values:

	> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be configured by the deployment package and post-installation script.

	| Property | Modify? | Value |
	|------------ |:---:| ---------- |
	| External System Definition | No | Named Credential |
	| Named Credential | No | SFCCUserCreds |
	| User | *Yes* | Select the SCC Integration User |
	| Authentication Protocol | *Yes* | Password Authentication |
	| Username | *Yes* | Specify the Salesforce B2C Commerce Account Username |
	| Password | *Yes* | Specify the Salesforce B2C Commerce Account Password:OCAPI Client Secret |

	> For the user, click the magnifying glass to view the list of Service Cloud users and select the SCCIntegration user.

	> The credentials entered here must represent an adequately permissioned Salesforce B2C Commerce environment Business Manager account.

	> The password MUST be in the format of *Account Password:OCAPI Client Secret*.  These two values must be delimited by a colon, and are used by Service Cloud to request an [authorization grant via OCAPI](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/OCAPI/18.8/usage/OAuth.html?cp=0_12_2_21) to interface with the [Data API](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/OCAPI/18.8/usage/DataAPIResources.html?cp=0_12_4).

1. Validate that the form values match these properties.
1. Confirm that the Salesforce B2C Commerce environment account exists.
1. Click **Save**.


### Update SFCC Site Information via the SCC Connector Custom Settings
To facilitate the communication between clouds, [custom settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_customsettings.htm) must be configured to point to the Salesforce B2C Commerce environment.

#### Configure the SFCC Configuration Custom Setting

1. From Setup, enter Custom Settings in the quick find box, then select **Custom Settings**.
1. Click **Manage** for the 'SFCC Configuration' setting.
1. Click **New**.
1. Create a configuration using the following values:

	> Only the fields marked as required should be completed.

	| Required | Property | Value |
	|:--------:| ------------------- | ---------- |
	| X        | Name     | [your-storefront-site-id](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Admin/ManagingSitesInBM.html?cp=0_4_5) |
	| X        | Customer List Id | [your-storefront-customerList-id](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Customers/CustomerLists.html?cp=0_3_6_3) |
	|          | Master Catalog Id |  |
	|          | PriceBook Id |  |
	|          | Product Detailview |  |
	|          | Product Listview |  |
	| X        | SFCC Site URL | https://your-b2c-commerce-domain-dw.demandware.net  |
	|          | Sales Catalog Id |  |
	| X        | Site Id | your-site-id |
	| X        | Time | 1 |
	
	

	> Check your Salesforce B2C Commerce site configuration for the [siteId](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Admin/ManagingSitesInBM.html?cp=0_4_5) and [customerListID](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Customers/CustomerLists.html?cp=0_3_6_3) mapping properties.
	
	> Specify your Salesforce B2C Commerce environment URL for the SFCC Site URL property.

1. Click **Save** to save your changes.

	Once these changes have been saved, you are returned back to the listing of [custom settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_customsettings.htm) in the Service Cloud environment.

#### Configure SFCC Integration Creds Custom Setting
The SFCC Integration Creds custom setting is used to store the clientId and secret used to interact with Salesforce B2C Commerce Open Commerce API (OCAPI).

1. Log out as the SCC Integration User.
1. Log in as the Service Cloud System Administrator.
1. From Setup, enter Custom Settings in the quick find box, then select **Custom Settings**.
1. Locate the 'SFCC Integration Creds' custom setting from the displayed list, and click **Manage**.
1. Click the top-most **New** to create the default credentials.
1. Update the credential properties with the following values:
	
	| Property      | Value        |
	| ------------- | ------------ |
	| ClientId      | your-site-clientid|
	| Client Secret | your-site-clientsecret |
	
	> For sandbox environments, the default values of __aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa__ and __aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa__ can be used.
	
	> The [clientId and clientSecret](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/OCAPI/18.8/usage/ClientApplicationIdentification.html?cp=0_12_2_3) must be created in Account Manager before being added to the storefront configuration.  For details on managing clientIds, read [Add a ClientID for OCAPI](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/AccountManager/AccountManagerAddAPIClientID.html?cp=0_7_11) and [Enable and Disable a ClientID for OCAPI](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/AccountManager/AccountManagerEnableDisableClientID.html?cp=0_7_12) via the platform documentation.
	
	> For production environments, these values are used to request an OAuth2 authorization token. Review the [OCAPI OAuth 2.0 platform documentation](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/OCAPI/18.8/usage/OAuth.html?cp=0_12_2_21) for details on how this is performed.

1. Click **Save** to save your changes.

	Once these changes have been saved, the default organization level values should display the clientId and client secret that were entered.

### CORS and Content Security
Lastly, the Service Cloud environment must be configured to allow communication with the Salesforce B2C Commerce environment.

#### Setup CORS
Update the [CORS settings](https://help.salesforce.com/articleView?id=extend_code_cors.htm&type=0) for the Salesforce environment to whitelist the Salesforce B2C Commerce URL.

1. From Setup, enter CORS in the quick find box, then select **CORS**.
1. From the list of whitelisted origins, click **New**.
1. Create a whitelist entry using the following properties:
	
	| Property      | Value        |
	| ------------- | ------------ |
	| Original URL Pattern   | https://xxxx-dw.demandware.net |
	
	> Enter the full domain name of the Storefront URL that is connecting to Service Cloud.  Be sure to include the https:// domain prefix.

1. Click **Save** to save your changes.

	Saving these changes should return you to the listing of whitelisted origins.  Confirm that the URL entered appears in this listing.

#### Content Security
Similar to the CORS setting, a [trusted site](https://help.salesforce.com/articleView?id=csp_trusted_sites.htm&type=0) must be defined within Service Cloud that maps to the Salesforce B2C Commerce Environment.    
To create a trusted site:

1. From Setup, enter CSP Trusted Sites in the Quick Find box, then select **CSP Trusted Sites**.
1. From the list of trusted sites, click **New Trusted Site**.
1. Create a trusted site entry using the following properties:

	| Property      | Value        |
	| ------------- | ------------ |
	| Trusted Site Name | CC_Endpoint |
	| Trusted Site URL  | https://xxxx-dw.demandware.net |
	| Active | checked |
	| Context | All |
	
	> For the Trusted Site URL, enter the full domain name of the Storefront URL that is connecting to Service Cloud.  Be sure to include the https:// domain prefix.

1. Click **Save** to save your changes.

	Saving these changes returns you to the listing of trusted sites.  Confirm that the entered site definition entered appears in this listing.

### Reset Your Security Token

>To successfully configure Salesforce B2C Commerce to interact with Service Cloud REST services, a security token must be generated and saved.

> Complete these activities before starting the Salesforce B2C Commerce configuration instructions.

The SCC Integration User account is used to facilitate service interactions between Service Cloud and Salesforce B2C Commerce.  To enable Salesforce B2C Commerce to connect to Service Cloud's REST Services, a Salesforce [Security Token](https://help.salesforce.com/articleView?id=user_security_token.htm&type=5) must be generated for the SCC Integration User account and passed into Service Cloud authentication requests.

To generate a new security token, execute the following set of instructions:

1. Log in as the SCC Integration User.
1. In a new browser window, go to the following URL:

	###### https://*your-salesforce-environment-domain*/_ui/system/security/ResetApiTokenEdit

	> Replace 'your-salesforce-environment-domain' with your actual Salesforce environment's domain.

1. Click **Reset Security Token**.

	This forces the existing Salesforce environment to generate a new security token for the logged in user (in this case, the SCC Integration User).

1. Check the email address associated to the SCC Integration User for the security token email.
1. When the token is received, store it in a secure place.

The generated security token is used when configuring the Salesforce B2C Commerce environment's site preferences that store the authentication settings for the Service Cloud environment. 

Save the generated security token, as it is used in future configuration, test-suite, and validation steps.

### Continue with the Configuration of Service Cloud Data Objects
With the initial configuration of Service Cloud completed, continue the setup of the service-cloud-connector by [configuring the Service Cloud Data Objects](./configureDataObjects.md) leveraged by the connector to type cases, maintain order status, and define the object / field-mappings used to enable data synchronization across both clouds.
