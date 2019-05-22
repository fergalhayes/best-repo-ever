# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**  
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Validating the Cloud Environments
###### This guide explains how to test the cross-cloud requests between Salesforce B2C Commerce and Service Cloud enabled via the service-cloud-connector.

### Environment Prerequisites
Before moving forward with testing, ensure that you have completed the Service Cloud and Salesforce B2C Commerce setup instructions documented in this guide.  This should include:

- [Setup Service Cloud](setupServiceCloud.md)
- [Configure Service Cloud Data Objects](configureDataObjects.md)
- [Setup Salesforce B2C Commerce](setupB2CCommerce.md)
- [Setup Salesforce B2C Commerce Scheduled Jobs](setupB2CCommerceJobs.md)
- [Customize the B2C Storefront(SiteGenesis)](customizeB2CStorefrontSG.md) or 
- [Customize the B2C Storefront(SFRA)](customizeB2CStorefrontSFRA.md)

> If you have not completed these setup instructions, we recommend doing so before continuing with the content on this page. The configuration values in the Postman environment data depends on proper configuration of both Service Cloud and Salesforce B2C Commerce.

We also recommend reviewing the [Supported Use Cases](supportedUseCases.md) documentation, as this page captures the use-cases supported by the service-cloud-connector with its default installation.

### Set Up and Configure the Postman Test Suite

###### This section details how to set up and configure the Postman Test Suite so that it can be used to test multiple environments.

#### Using Postman to Test the Cross-Cloud Connectivity
To simplify the validation process for the service-cloud-connector, configurable test-scripts have been provided for [Postman](https://www.getpostman.com/apps) that include request source-code.  Postman is a free, best-of-breed [ADE (API Development Environment)](https://www.getpostman.com/) that makes it simple to design, customize, and test web-service calls.  If you don't have Postman, we recommend that you [download](https://www.getpostman.com/apps) and install it before moving forward with the validation guidance.  
Alternatively, you can also use another REST client.

> Using another REST client requires that you develop your own tests from scratch.  You can base your tests on the construction of our Postman test-files to work with your chosen client.

#### About the Postman Test Suite Import Files

The Postman test suite contains configurable and repeatable use-case-driven collections of tests that can be used to exercise the round-trip requests leveraged by the service-cloud-connector. The test suite consists of two files to import into Postman:

- The [environment template](../postman/serviceCloudConnector.environmentTemplate.json) (which manages all variables leveraged by the test suite).
- The [test suite](../postman/serviceCloudConnector.testSuite.json) (which contains all of the Salesforce B2C Commerce and Service Cloud REST-based use-case tests and individual test requests for each cloud).

#### Salesforce B2C Commerce Test Suite Prerequisites
The test scripts included in the Postman suite are dependent on the existence of customer and order data in Salesforce B2C Commerce.  To that point, create the following records in Salesforce B2C Commerce.

> We use this date to populate our test scripts with their identifiers and key information so that our test scripts can interact with, and modify, data that exists in both clouds.

> The [test-data](../b2c/sfcc/sites/site_template/test-data) directory contains importable .xml containing a registered user, orders for the registered user, and order for an anonymous order.  You can import this data into your Salesforce B2C environment to establish the test data to be used in these tests.

- Register a new customer in your Salesforce B2C Commerce storefront.
- Create three (3) addresses for this customer.
- Submit an order for this customer as a registered user (the customer must be logged into the storefront).
- Create an two anonymous orders leveraging a different unique customer email address.

With these records created, you can modify the [Postman Environment Template](../postman/serviceCloudConnector.environmentTemplate.json) to include this customer and order data in the tests available, via the [Postman Test Suite](../postman/serviceCloudConnector.testSuite.json).

#### Import the Service Cloud Connector Environment Template
To help abstract the configuration of the test suite, and enable support for multiple environments, a [Postman environment](https://www.getpostman.com/docs/v6/postman/environments_and_globals/manage_environments) template has been authored. This template contains the global variables leveraged by the individual test requests that exist within the Service Cloud Connector Test Suite.

> The environment template can be used to create environment profiles mirroring individual development, QA, staging, and production environment configurations that include key configuration properties for both Service Cloud and Salesforce B2C Commerce.

##### Import the Environment Template

To [import and manage the environment template](https://www.getpostman.com/docs/v6/postman/environments_and_globals/manage_environments) into Postman:

1. Download [serviceCloudConnector.environmentTemplate.json](../postman/serviceCloudConnector.environmentTemplate.json) to a folder on your workstation.
1. Open Postman.
1. Click **File > Import**.
1. Click **Choose Files**.
1. Find and select the [serviceCloudConnector.environmentTemplate.json](../postman/serviceCloudConnector.environmentTemplate.json) file that was recently downloaded to your workstation.
1. Verify that Postman successfully imported the environment template.

	> Postman should provide a message stating that the environment file was imported successfully.

1. From the environment dropdown in the upper-right corner of the Postman display, verify that an environment definition named 'SCC: Environment Template' exists.

With the environment template imported, you can now duplicate the template and customize instances for each of your specific Service Cloud and Salesforce B2C Commerce Cloud environments.

> We recommend configuring environment templates for each of your development and QA environments.  This enables you to use the test suite to validate each of these environments by simply selecting a new environment template from within Postman.

##### Customize Configure the Environment Profile Variables
Prior to executing the Service Cloud connector's Service Cloud services, perform the following activities to configure the environment variables on which they depend.

1. Open the [manage environments](https://www.getpostman.com/docs/v6/postman/environments_and_globals/manage_environments) view in Postman.
1. Verify that the __SCC: Environment Template__ exists in the environment list.

	> If the template is not present, [reimport it](#import-the-environment-template) using the linked instructions.

1. To duplicate the environment template, click the **Duplicate Environment** icon displayed to the right.
1. Rename the duplicate template to uniquely identify it based on your Service / B2C Commerce environments.

	```
	// Here's an example of what the environment profile name can look like:
		SCC: Service Cloud Domain / Salesforce B2C Commerce Domain Prefix
		SCC: mydomain.my.salesforce.com / myB2CEnvironment
	```
1. Edit the renamed environment template.
1. Customize the variables used to drive the test-suite using the configuration properties of your Salesforce Clouds.

##### Environment Configuration Variables

| Variable Name | Example Value | Description |
|------------ | ---------- | ------------------------------------ |
| serviceCloudUrl | myorgname.my.salesforce.com | Specify the base domain name for the Salesforce org instance that will be tested.  This is returned as part of the authentication / token response. |
| authTokenUrl | login.salesforce.com | For production environments, use the __login__ domain prefix; for sandbox environments, use the __test__ domain prefix. |
| sfscConsumerKey | 3MVG9dZJodJWITSsomWoSCSR.uds | Specify the Consumer Key generated for the [CommerceCloudConnector Connected App](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1). |
| sfscConsumerSecret | 9543255638122220244 | Specify the Consumer Secret generated for the [CommerceCloudConnector Connected App](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1). |
| sfscUsername | sccintegrationuser@myorgdomain.com | Specify the full user-name of the SCC Integration User. |
| sfscPassword | password | Specify the password of the SCC Integration user. |
| sfscSecurityToken | ZEn1pdcAtPr6HMxPent4DqBNy1 | Specify the [Security Token](https://success.salesforce.com/answers?id=90630000000glADAAY) associated to the SCC Integration user. |
| sfccHostServer | mydomain-dw.demandware.net |  The customer-facing url representing the Salesforce B2C Commerce environment. |
| sfccUsername | mySampleUserName | Specify the full user-name of the Salesforce B2C Commerce Business Manager user. |
| sfccPassword | myAccontPassword | Specify the password associated to the Salesforce B2C Commerce account. |
| sfccClientId | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa | The Salesforce B2C Commerce OCAPI clientId used to identify resources available to Service Cloud. |
| sfccClientSecret | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa | The clientSecret associated to the Salesforce B2C Commerce OCAPI clientId representing the resources available to Service Cloud. |
| sfccOCAPIVersion | v17_3 | Represents the OCAPI version that is used in a given test |
| sfccSiteId | SiteGenesis | Represents the site that is interacted with hosted on the Salesforce B2C Commerce instance |
| sfccCustomerListId | SiteGenesis | Represents the Salesforce B2C Commerce customer-list from which customer information is retrieved |
| sfccCustomerListId | SiteGenesis | Represents the Salesforce B2C Commerce customer-list from which customer information is retrieved |
| postRequestDelay | 10000 | Describes in milliseconds the amount of time to wait before executing a validation request to Service Cloud as part of tests executed to validate REST requests to Service Cloud |

##### Programmatically Generated Variables

The following variables are read-only, and are generated by the test-suite as tests or requests are executed.  Do not manually modify these variables.

| Variable Name | Description |
|------------ | ------------------------------------ |
| sfccShopUrl | Represents the url template used to generate OCAPI Shop API urls used in test-requests. |
| sfccOUath2Url | Represents the OCAPI authentication url used to authenticate against prior to working with the Data API. |
| sfccDataUrl | Represents the url template used to generate OCAPI Data API urls used in test-requests.  |
| sfccUserCredential | Placeholder variable used to store user-credential generated by Postman prior to requesting an authToken from Salesforce B2C Commerce. |
| sfccAuthToken | Placeholder variable used to store the authToken generated by Salesforce B2C Commerce via OCAPI. |
| sfccCustomerAuthToken | Placeholder variable used to store the customer-specific authToken generated by Salesforce B2C Commerce via OCAPI. |



##### Test Salesforce B2C Commerce Variables

These variables represent test-customer and test-order values available from the [test-data](../b2c/sfcc/sites/site_template/test-data) directory.  These values should be derived from the registered and anonymous customer and order records in Salesforce B2C Commerce that is used during testing.  The test suite uses these values to validate the data retrieved by Salesforce B2C Commerce via OCAPI.

> If you need test-data, import the [test orders and customer data](../b2c/sfcc/sites/site_template/test-data) included in the repository.

| Variable Name | Example Value | Description |
|------------ | --------------| ---------------------- |
| sfccCustomerNo | 00004001 | The customer number of the test registered Salesforce B2C customer |
| sfccCustomerId | ab9AnKj2kHTCu3PCSOZ3MJam3b | The customerId for the test registered Salesforce B2C customer |
| sfccCustomerFirstName | Jerry | The firstname of the test registered Salesforce B2C customer |
| sfccCustomerLastName | Seinfeld | The lastname of the test registered Salesforce B2C customer |
| sfccCustomerEmail | jerry.seinfeld@salesforce.com | The email address of the test registered Salesforce B2C customer |
| sfccCustomerUsername | jerry.seinfeld@salesforce.com | The storefront login / username of the test registered Salesforce B2C customer |
| sfccCustomerPassword | P@ssw0rd! | The storefront password of the test registered Salesforce B2C Customer |
| sfccAnonymousOrderNo | 00000701 | The order number of the Salesforce B2C Commerce anonymous order |
| sfccAnonymousOrderTotal | 562.74 | The order total of the Salesforce B2C Commerce anonymous order |
| sfccRegisteredOrderNo | 00000601 | The order number of the Salesforce B2C Commerce registered order |
| sfccRegisteredOrderTotal | 2,027.50 | The order total of the Salesforce B2C Commerce registered order |
| sfccCustomerAddressName | George's House | The address name used to test customer address for the test Salesforce B2C customer |
| sfccAnonymousCustomerEmail | george.costanza@salesforce.com | The email address of the test anonymous Salesforce B2C customer |

##### Temporary Variables

The following temporary variables are read-only, and are generated by the test-suite as tests or requests are executed.  These variables are used to store temporary values that are referenced across multiple REST requests executed while validating specific use-cases. Do not modify these values.

| Variable Name | Description |
|------------ | ------------------------------------ |
| tmpCaseIds | Represents the collection of CaseIds created in Service Cloud |
| tmpAccountId | Represents the AccountId for a given registered customer in Service Cloud  |
| tmpCustomerFirstName | Represents the customer firstName for a given customer registered via Service Cloud |
| tmpCustomerLastName | Represents the customer lastName for a given customer registered via Service Cloud |
| tmpOrderId | Represents the OrderId for a given order synchronized in Service Cloud |
| tmpOrderStatus | Represents the Order status for a given order in Service Cloud  |
| tmpRegisteredOrderEmail | Represents the email address associated to a given order in Service Cloud |
| tmpRegisteredOrderCustomerId | Represents the customerId for a given order in Service Cloud |

When customizing the variables, remember to set the initial and current values in the Variables tab.  Once all of your updates have been made, click **Update**.  With the variables defined, you should now be able to execute the Service Cloud test scripts.

#### Importing the Service Cloud Connector Test Suite
Before the test suite scripts can be executed, you must import the [serviceCloudConnector.testSuite.json](../postman/serviceCloudConnector.testSuite.json) into Postman.

1. Launch Postman.
1. Click **Import**.
1. From the Import Dialog, select the 'Import' tab, and click **Choose Files**.
1. Locate and select the [serviceCloudConnector.testSuite.json](../postman/serviceCloudConnector.testSuite.json) test suite file.
1. Confirm that the 'Service Cloud Connector: Test Scripts Suite' appears in the Postman client.

If the test suite has been successfully imported and your environment template has been configured, you are now ready to execute the use-case tests and individual requests available in the test suite.

### Supported Test Scenarios

###### This section details the test and request scenarios supported by the Postman Test Suite.

#### Test Suite Folder Structure

The Postman test suite contains three top-level folders used to organize the tests and individual requests that can be used to exercise both Salesforce B2C Commerce and Service Cloud.

- __Cross Cloud Use Cases__ contains [Postman collections](https://www.getpostman.com/docs/v6/postman/collection_runs/using_environments_in_collection_runs) representing each of the test-cases available to test.  These tests are set up as [Postman collections](https://learning.getpostman.com/docs/postman/collections/intro_to_collections/), made up of the individual Service Cloud and Salesforce B2C Commerce requests that collectively represent the use-case test.

- __Service Cloud Requests__ contains individual Postman requests that can be used to exercise the individual Service Cloud REST services enabled by the service-cloud-connector.  The use-case tests leverage these individual requests to build out the Service Cloud components of the test cases.

- __Salesforce B2C Commerce Requests__ contains the individual Postman requests that are used by the service-cloud-connector to retrieve Salesforce B2C Commerce data via OCAPI.  The use-case tests leverage these individual requests to build out the Salesforce B2C Commerce components of the test cases.

>These tests and individual requests are dependent on the [correct configuration](#import-the-environment-template) of the included [environment template](../postman/serviceCloudConnector.environmentTemplate.json).  Configure the [environment variables](#customize-configure-the-environment-profile-variables) leveraged by these tests prior to executing them.

#### Supported Use Case Tests

The Postman test suite ships with the following use-case tests that exercise full round-trips between Salesforce B2C Commerce and Service Cloud for each specific test case.

##### Case Management

The following case-management use-cases are supported by the Postman test suite.  Each test case posts a new case to Service Cloud via REST.  The case is created in Service Cloud, and the customer associated to the test case is created as a Person Account.  The case is then associated to the Person Account.

- __Create Cases for Anonymous Customers__.  This test creates a case for an anonymous Salesforce B2C Commerce customer.

- __Create Cases for Registered Customers__.  This test creates a case for a registered Salesforce B2C Commerce customer.  When the Person Account is created, Service Cloud retrieves the details for that customer from Salesforce B2C Commerce via OCAPI and decorates the Person Account with the B2C Commerce customerId and customerNo.

- __Create Cases for an Anonymous Customer that Registers__.  This test creates a case for an anonymous Salesforce B2C Commerce customer and then creates another case for that same customer as a registered user.

##### Customer Management

The following customer-management use-cases are supported by the Postman test suite.  Each test case posts a new customer to Service Cloud via REST.  Service Cloud takes the customer information and creates a Person Account.  Service Cloud then requests the details of this customer from Salesforce B2C Commerce and updates the Person Account with these details.

- __Create a Registered Customer Account__.  This test creates a Service Cloud account for a registered Salesforce B2C Commerce customer.

- __Update a Registered Customer Account__.  This test updates the properties of an existing Service Cloud account.

- __Add an Address to a Customer Account__.  This test adds a new address to an existing Service Cloud account.

- __Update an Address on a Customer Account__.  This test updates the properties of an existing address associated to a Service Cloud account.

- __Process Customers via Batch Processing__.  This test exercises the batch processing of customer data from Salesforce B2C Commerce to Service Cloud.

##### Order Management

The following order-management use-cases are supported by the Postman test suite.  Each test case posts a placed order to Service Cloud via REST.  Service Cloud then creates the stub-order representation, creates an order in Service Cloud, and creates a Person Account for the customer that placed that order.  Service Cloud then requests the details of this order and customer from Salesforce B2C Commerce and updates the Person Account and order details accordingly.

- __Create an Order for an Anonymous Customer__.  This test creates a Salesforce B2C Commerce order in Service Cloud for an anonymous customer and creates a Person Account for the anonymous customer.

- __Update the Status of an Anonymous Order__.  This test updates the status of an order in Service Cloud associated to an anonymous customer.

- __Create an Order for a Registered Customer__.  This test creates a Salesforce B2C Commerce order placed in Service Cloud for a registered customer and create a Person Account for the registered customer.

- __Update the Status of a Registered Order__.  This test updates the status of an order in Service Cloud associated to a registered customer.

- __Process Orders via Batch Processing__.  This test exercises the batch processing of placed orders from Salesforce B2C Commerce to Service Cloud.

#### Executing Test Cases as Postman Collections

Each Postman test case can be executed via the [Postman Collection Test Runner](https://learning.getpostman.com/docs/postman/collection_runs/intro_to_collection_runs).  Collections can be executed for a given use-case, leveraging the configuration variables for a given environment template.
> These tests do not delete the data they create.  To repeat these tests, purge your Service Cloud environment of the data generated by these tests prior to re-executing them.  A sample purge-script is provided in the [Troubleshooting and Debugging](./troubleshootingAndDebugging.md#anonymous-apex-to-purge-service-cloud-connector-data) documentation.

To execute a test case:

1. Open the Postman Collection Runner by clicking **Runner** in the Postman IDE.
1. Open the 'SCC Test Suite' folder to view the test suite subfolders.
1. Open the 'Cross Cloud Use Cases' folder to view the individual use-case tests.
1. Select the top-level use-case folder containing the test you'd like to execute.
1. In the 'Environment' field, select the [environment](https://learning.getpostman.com/docs/postman/collection_runs/using_environments_in_collection_runs) that should be used to exercise the selected use-case.
1. Click **Run** to [start the collection run](https://learning.getpostman.com/docs/postman/collection_runs/starting_a_collection_run) and execute the use-case test.

The Postman Collection Test Runner displays each request invoked as part of the use-case test.  For each request, the results of the individual request tests are displayed using Postman's green (passed) and red (failed) indicators.  For each use-case, all tests should pass; there should never be a test-failure.

#### Authenticate Before Executing Individual Requests

If you'd like to execute the individual requests present in the Service Cloud and Salesforce B2C Commerce request folders, the authorization requests for the requests' respective cloud must be executed.

> As a reminder, ensure that your environment template is configured accurately -- as the authentication requests are dependent on the template values accurately reflecting the environment and accounts configured in your Salesforce Service and B2C Commerce environments.

> These requests are placed at the top of individual use-case tests, ensuring that the subsequent requests that make-up the test have access to the authorization tokens produced by each Salesforce cloud.

##### Requesting the Service Cloud Authorization Token

Before executing any Service Cloud request, execute the __Retrieve Service Cloud AuthToken__ request.  This request retrieves the Service Cloud authorization token required by each of the individual Service Cloud requests.  A successful request produces the following response from Salesforce:

```
{
    "access_token": "00D0j111111D0ZG!AQQAQCOr4zrA48a5XYLQ5OVkB1n7Lw8WB2o9UzLpN7HrvnzLyY5d4RqReIxFIQBUXo0edmHsjeqMGW44UsE7oITdRiG73qN0",
    "instance_url": "https://mydomain.my.salesforce.com",
    "id": "https://login.salesforce.com/id/00D0j000000D0ZGEA0/0050j000001JcTxAAK",
    "token_type": "Bearer",
    "issued_at": "1542458817601",
    "signature": "sZGvXHccFSGLZnpfC3Rhl4PeEvC76Stkx/u6LkyVhSc="
}
```

When this request is completed, the *access_token* is saved in the *sfscAuthToken* environment variable so that it can be used by subsequent requests in the request's authorization header.

##### Requesting the Salesforce B2C Commerce Authorization Token

Before executing any Salesforce B2C Commerce request, execute the __Retrieve Commerce Cloud AuthToken__ request.  This request retrieves the Salesforce B2C Commerce authorization token required by each of the Commerce Cloud requests.  A successful request produces the following response from Salesforce:

```
{
    "access_token": "1f067c93-9b27-4ecf-998b-23fdc52bb9f8",
    "expires_in": 899,
    "token_type": "Bearer"
}
```

When this request is completed, the *access_token* is saved in the *sfccAuthToken* environment variable so that it can be used by subsequent requests in the request's authorization header.


