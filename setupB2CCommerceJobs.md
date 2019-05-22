# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**    
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Set up the Salesforce B2C Commerce service-cloud-connector Jobs
###### This guide captures the configuration and installation activities necessary to set up the int\_service\_cloud cartridge and configure the storefront jobs used to synchronize data across both clouds.

### What You Need to Get Started

Before extending the storefront, review and complete the installation instructions found in these two pages:

- [setupServiceCloud](./setupServiceCloud.md) provides guidance on how to configure Service Cloud to support the creation and synchronization of B2C Commerce objects (ex. customers, addresses, orders) in Service Cloud.
- [setupSalesforceB2CCommerce](./setupB2CCommerce.md) provides guidance on how to configure Salesforce B2C Commerce to announce changes to B2C Commerce objects and ingest changes to these objects initiated from Service Cloud.

> The setup of the connector cartridge's jobs and storefront extensions depend on these two steps being completed.  That said, complete the installation instructions found in these two pages before proceeding with the cartridge setup.

### Install the int\_service\_cloud Cartridge

To enable the features of the service-cloud-connector, the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge must be installed in your storefront and set up in the storefront's cartridge path.

1. Download the contents of the [b2c/sfcc](../b2c/sfcc) directory.
1. Add the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge directory to your storefront codebase.
1. Upload the cartridge folder to your current code-version within your environment.
1. Log into Business Manager with an account that has Administrative rights to the site being integrated with Service Cloud.

#### Trigger the Recognition of the Service Cloud Connector's Job-Steps

The two jobs used to synchronize customer and order data with Service Cloud depend on two job-step definitions. The job-step definitions must be parsed by the Salesforce B2C Commerce Cloud instance where the service-cloud-connector is being deployed. Follow these instructions to trigger the parsing of these job-steps from the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge.

1. In Business Manager, select **Administration > Site Development > Code Deployment**.
1. Set another code-version to be the active code version.  
This code-version should be different from the version that currently contains the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge uploaded to your Salesforce B2C Commerce instance.
1. Set the code-version where the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge was deployed as the Active code version by clicking **Activate**.

	> Toggling the code-versions force Salesforce B2C Commerce to scan the cartridge path for the site and parse any job-steps configured within the cartridges identified in the cartridge-path.
1. Confirm that the selected code-version is now flagged as Active in the code-versions listing (this should be the version with the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge).

1. Open a new browser and load the homepage of your site to confirm that your site is able to successfully parse the cartridge path.  If the homepage for the site you edited successfully renders, continue to the next section of this document.

### Create the Customer Synchronization Job

The service-cloud-connector uses the customer-synchronization job to synchronize customer profiles updated in Commerce Cloud with Service Cloud. 

#### Create the Stub Customer-Job Definition

Complete these instructions to create the customer-synchronization job leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Jobs**, and click **New Job**.
1. Enter the following properties:

	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-customer-sync|
	| Description   | This job is used to synchronize customer data changed in Salesforce B2C Commerce Cloud with Service Cloud. |
	
1. Click **Create** to create the job stub.

#### Configure the scc-customer-sync Job's Properties

Complete these instructions to create the customer-synchronization job leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Jobs**, and click **scc-customer-sync**.
1. From the 'scc-customer-sync' job schedules display, click the **Job Steps** tab.
1. On the Job Steps tab, click the Configure a Step icon centered on the display.
1. In the Select and Configure Step sidebar, select the job-step labeled Custom-SCC-SyncCustomers, and enter the foollowing configuration properties:

	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-customer-sync-jobstep|
	| Description   | This job-step is used to synchronize customer data changed in Salesforce B2C Commerce with Service Cloud. |
	| Restart Enforced | unchecked |
	
	> Do not configure any error handling behavior.

1. Click **Assign**.

	> The job-step should appear in the 'scc-customer-sync' display highlighted in red.  The job step has an error, as its scope has not been configured.

1. From the 'scc-customer-sync' Job Steps tab, click the **Scope** button.
1. Select the site being integrated with Service Cloud (ex. SiteGenesis/RefArch).
1. Click **Assign** to associate this site to the job-step.

	Assigning the site being integrated with Service Cloud completes the configuration of scc-customer-sync job.

	> The scc-customer-sync job leverages the [customerSync.js](../b2c/sfcc/cartridges/int_service_cloud/cartridge/scripts/jobs/customerSync.js) script to perform the Salesforce B2C Commerce to Service Cloud customer synchronization.  View the [steptypes.json](../b2c/sfcc/cartridges/int_service_cloud/steptypes.json) file for custom job-step configuration details.

With this completed, now create the order synchronization job.

### Create the Order Synchronization Job

Similar to the customer synchronization job, the service-cloud-connector makes use of a job to synchronize orders placed via Commerce Cloud with Service Cloud.  

#### Create the Stub Order-Job Definition

Complete these instructions to create the order-synchronization job leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Jobs**, and click **New Job**.
1. Enter the following properties:
	
	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-order-sync|
	| Description   | This job is used to synchronize order placed via Salesforce B2C Commerce with Service Cloud. |

- Click **Create**.

#### Configure the scc-order-sync Job's Properties

Complete these instructions to create the order-synchronization job leveraged by the service-cloud-connector.


1. In Business Manager, select **Administration > Operations > Jobs**, and click **scc-order-sync**.
1. From the 'scc-order-sync' job schedules display, click the **Job Steps** tab.
1. On the Job Steps tab, click the Configure a Step icon centered on the display.
1. In the Select and Configure Step sidebar, select custom.SCC-SyncOrders, and enter the following configuration properties:

	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-order-sync-jobstep|
	| Description   | This job-step is used to synchronize orders placed via Salesforce B2C Commerce with Service Cloud. |
	| Restart Enforced | unchecked |

	> Do not configure any error handling behavior.

1. Click **Assign**.

	> At this point, the job-step should appear in the 'scc-order-sync' display highlighted in red.  The job step has an error, as its scope has not been configured.

1. From the 'scc-order-sync' Job Steps tab, click the **Scope** button.
1. Select the site being integrated with Service Cloud (ex. SiteGenesis/RefArch).
1. Click **Assign** to associate this site to the job-step.

	Assigning the site being integrated with Service Cloud completes the configuration of scc-order-sync job.

> The scc-order-sync job leverages the [orderSync.js](../b2c/sfcc/cartridges/int_service_cloud/cartridge/scripts/jobs/orderSync.js) script to perform the Salesforce B2C Commerce Cloud to Service Cloud customer synchronization.  View the [steptypes.json](../b2c/sfcc/cartridges/int_service_cloud/steptypes.json) file for custom job-step configuration details.

### Scheduling the Customer and Order Synchronization Jobs

In your PIG, configure the Customer and Order jobs to execute at a high frequency and regular interval (ex. once every five minutes).  In a non-production environment (ex. a development sandbox), remember to execute these jobs manually.

> Development environments cannot schedule jobs.  When testing in your development environment, manually execute these jobs after performing customer or order-related transactions.

### Configure the Synchronization of Customer and Order Information

The custom site preference 'Is async mode for Service Cloud?' can be used to modify the behavior of the synchronization jobs and how they trigger data-synchronization between Salesforce B2C Commerce and Service Clouds.

| Preference Value | Impact on Behavior |
|------------------|--------------------|
| None | Select either 'Yes' or 'No'. This site preference should not be configured with a value on 'None'.   |
| Yes  | This causes all data-synchronizations to be driven by the service-cloud-connector jobs.  No real-time synchronization is performed when interacting with customer or order data. |
| No   | This causes all data-synchronizations in real-time.  The service-cloud-connector jobs only process synchronization failures. |

Synchronization jobs are always running.  What this preference does is slightly modify which records are synchronized when the job executes.  We recommend defaulting this value to No, which enables real-time synchronization, which is ideal for development environments.

> These jobs should be scheduled to run __every five minutes__ ensuring that changes made by customers (ex. customer registration, profile update, order placement, etc.) are synchronized with Service Cloud in a timely fashion.  This interval should consider the creation or update frequency of these storefront objects -- as a shorter interval impacts storefront performance. Adjust the interval accordingly based on your storefront's customer or order frequency, and validate that the setting does not negatively impact storefront performance before deploying this change to production.

### Configure the Customer and Order Synchronization Batch Size

The custom site preference 'Bulk Call Threshold' can be used to manage the size of the customer and order batches sent to Service Cloud for every job-instance executed by Salesforce B2C Commerce Cloud.

| Preference Value | Site Preference Description |
|------------------|--------------------|
| 10 | This site preference describes the value used to determine what portion of customers or orders that have updates and are ready to by synchronized with Service Cloud should be sent via service-cloud-connector's synchronization jobs. |

The business rules used to determine scheduled job batch-size are:

- The job calculates the total number of items (customers and orders) that have updates to share with Service Cloud.
- That total is divided by the 'Bulk Call Threshold' preference value.
- The product of that is then rounded up to the next highest integer.
- The rounded value represents the maximum batch size for the service-cloud-connector's scheduled jobs.

> For example: 100 items found and the preference value is 10; 100 / 10 = the first 10 items of the 100 identified are batched to Service Cloud

> For example: 9 items found and the preference value is 10; 9 / 10 = the first (1) item of the 100 identified are batched to Service Cloud

You can increase the size of batches containing orders or customers by decreasing this preference value.

| Preference Value | Impact on Behavior |
|----------:|--------------------|
| 1 | Sends 100% of all identified items in the batch to Service Cloud |
| 2 | Sends 50% of all identified items in the batch to Service Cloud  |
| 4 | Sends 25% of all identified items in the batch to Service Cloud  |
| 10 | Sends 10% of all identified items in the batch to Service Cloud  |

> If the total number of identified items is less than this preference value, then only one item is included in the batch used to send updates to Service Cloud.

### Continue with the Customization of the Salesforce B2C Commerce Storefront
With the scheduled jobs in place and configured, continue with the [customization of your storefront(SiteGenesis)](customizeB2CStorefrontSG.md) or [customization of your storefront(SFRA)](customizeB2CStorefrontSFRA.md) to include the [custom hooks](https://documentation.demandware.com/DOC2/index.jsp?topic=%2Fcom.demandware.dochelp%2FSGJC%2FSiteGenesisDevelopmentOverview.html&anchor=SiteGenesisDevelopmentOverview__hooks) used to enable the creation of cases as well as the synchronization of customer and order data.
