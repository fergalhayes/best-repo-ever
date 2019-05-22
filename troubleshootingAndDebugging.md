# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**    
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Troubleshooting and Debugging
###### This guide provides guidance on how to troubleshoot the execution of the service-cloud-connector's use cases across Salesforce B2C Commerce and Service Clouds.

### Getting Started with Salesforce

The following [Trailhead](https://trailhead.salesforce.com/c) trails are recommended for all Salesforce B2C Commerce Cloud developers that do not have experience with Salesforce, or are working with Salesforce Service Cloud for the first time.

- [Developer Console Basics](https://trailhead.salesforce.com/content/learn/modules/developer_console)--Get to know the Salesforce web-based integrated development environment (IDE) -- from which you can inspect log files, monitor Service Cloud REST requests in RealTime, and execute anonymous Apex and SOQL queries.
- [API Basics](https://trailhead.salesforce.com/en/content/learn/modules/api_basics)--Learn about the four APIs available within Salesforce (REST being one of them) and get introduced to the Developer Workbench.
- [Event Monitoring](https://trailhead.salesforce.com/en/content/learn/modules/event_monitoring)--Learn about Salesforce's powerful event monitoring features, and use them to monitor usage of the service-cloud-connector.
- [Apex Basics & Database](https://trailhead.salesforce.com/en/content/learn/modules/apex_database)--The service-cloud-connector is written primarily in Apex and SOQL.  Learn the fundamentals of these two Salesforce languages.
- [Apex Testing](https://trailhead.salesforce.com/en/content/learn/modules/apex_testing)--As you begin to consider extending the service-cloud-connector, learn how to write unit-tests for your Apex customizations.

Collectively, these trails provide the foundation for how to monitor Service Cloud activity and performance, query data created via the service-cloud-connector, and ultimately extend the connector via customizations.  We recommend that Salesforce B2C Commerce developers complete these trails before releasing the service-cloud-connector to production.

### Log Files

#### Service Cloud

All activity incurred by the service-cloud-connector can be inspected through its log files via the [Developer Console](https://help.salesforce.com/articleView?id=code_dev_console.htm&type=5).  To view service-cloud-connector events in real-time, do the following:

1. Log into your Salesforce instance using the SCC Integration User credentials created during the [setup of Service Cloud](./setupServiceCloud.md).
1. Click the Gear icon, and select **Developer Console**.

	> As all service-cloud-connector REST interactions are performed through the SCC Integration User account, you must be logged in as that user to view the connector's events in its log-files.

1. With the console open, invoke the service-cloud-connector from either the [Postman use-case tests](./validateEnvironments.md) or Salesforce B2C Commerce customizations.

	Log entries in the Developer Console appear -- auditing the activities performed by the service-cloud-connector in Service Cloud.

1. To search Service Cloud log-files for errors, use the 'filter' option from within the log-viewer and filter on the following keywords:

	- fault
	- exception
	- error
	- Error
	- Exception

	The 'filter' option for the log-file viewer is case-sensitive. You may need to search for both lower-case and Camel-Case versions of your keywords to find corresponding log entries.

#### Salesforce B2C Commerce

Salesforce B2C Commerce writes service-cloud-connector events to two distinct log-files, capturing service interactions with Salesforce and the connected Service Cloud instance.

##### __service-servicecloud-auth__

This log file is used to capture Service Cloud Authorization Token requests and responses. It logs and audits the auth-service request, the payload passed to the auth-service, the response provided by the Salesforce authentication URL, and the status of the service-call provided by the Salesforce B2C Commerce Service Framework.

> This log audits the username and password of the SCC Integration User in Service Cloud, as well as the ClientId and ClientSecret of the CommerceCloudConnector Connected App in Service Cloud.

> Disable this log in production environments.  It should only be used to troubleshoot cross-cloud authentication issues.

The following log entries provide an example of how a single authentication request is audited.  It can be used to inspect authentication requests, verify the formatting and structure of credentials, and examine the responses provided by Salesforce.

__2018-11-14 18:31:41.599__ - audit that the service has been invoked
```log
DEBUG CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.auth.LOG []  Created local HTTP service servicecloud.auth from int_service_cloud
```
__2018-11-14 18:31:42.508__ - audit the request properties and contents
```log
INFO CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.auth.COMM []  Webservice Communication: servicecloud.auth (POST https://test.salesforce.com/services/oauth2/token?grant_type=password&username=sccintegrationuser%40mydomain.salesforce.com&password=myPasswordtl9S0SWtKnjfaasdf3205TBY8z6jt&client_id=3MVG9srSPoex34FV0eWhM345345344PEG4CddkAPVK4a23423Tx_Y2SYudsACmC5thiQZEKzs41FeSNXOqQYQEQlNuYJ&client_secret=2928703738267628764), 853 ms
```
__2018-11-14 18:31:42.509__ - audit the response object generated by the request
```log
INFO CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.auth.COMM []  Response:
{"access_token":"00D0j000000D0ZG!AQQAQKHzFIRsFyZK4jo2Fx8eVVW2E44ymyh6OrMfXLjoTBRNYJt_SkQDFDUs6PxOjktaxt7Ow3z2s8m5kgQmcNriSz_FJ.I4","instance_url":"https://mydomain.my.salesforce.com","id":"https://test.salesforce.com/id/00D0j000111D0ZGFB0/0050j000001JcTxAAK","token_type":"Bearer","issued_at":"1542220302420","signature":"QpdoNXqR0uDnZ/WtSDC+PdUnE/qHq3ZzmMO7rmChjKg="}
```
__2018-11-14 18:31:42.512__ - audit the service response returned by the Service Framework
```log
INFO CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.auth.HEAD []  service=servicecloud.auth status=OK
```
##### __service-servicecloud-rest__

This log file is used to capture Service Cloud REST requests and responses. The REST requests are used to send case, customer, and order updates to Service Cloud for processing.  It logs and audits the rest-service request, the payload passed to the rest-service, the response provided by the Salesforce instance, and the status of the service-call provided by the Salesforce B2C Commerce Service Framework.

> This log captures the object definitions sent to Service Cloud for cases, customers, and orders processed across each of the service requests processed by the service-cloud-connector.

The following log entries provide an example of how REST requests are audited.  In this example, an order is being sent to Service Cloud for processing.  This log can be used to inspect the object definition and response provided by Service Cloud for any REST service employed by the service-cloud-connector.

__2018-11-14 18:31:41.488__ - audit that the service was invoked
```log
DEBUG CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.rest.LOG []  Created local HTTP service servicecloud.rest from int_service_cloud
```
__2018-11-14 18:31:43.447__ - audit the REST service endpoint to be called
```log
INFO CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.rest.COMM []  Webservice Communication: servicecloud.rest (POST https://mydomain.my.salesforce.com/services/apexrest/Order/Placement/Batch), 927 ms
```
__2018-11-14 18:31:43.448__ - audit the request properties and contents
```log
INFO CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.rest.COMM []  Request:
[{"order_no":"00000601","status":"Draft","order_total":2027.5,"scc_sync_status":"Created"}]
```
__2018-11-14 18:31:43.448__ - audit the response object generated by the request
```log
INFO CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.rest.COMM []  Response:
[{"SFCCOrderNo":"00000601","recordId":"8010j0000003IBLAA2","objectType":"Order","errors":null}]
```
__2018-11-14 18:31:43.450__ - audit the service response returned by the Service Framework
```log
INFO CustomJobThread|3166798|scc-order-sync|custom.SCC-SyncOrders custom.service.servicecloud.rest.HEAD []  service=servicecloud.rest status=OK
```

### Useful SOQL Queries

#### Identify the Default Account
This query can be used to identify the 'Default' Account created within Service Cloud.  The 'Default' account is used to associate orphaned customer data.

```sql
// Identify and Retrieve the Default Account
select  Id,
        Name
from    Account
where   Name = 'Default'
```
#### Sample Case Query
This query can be used to retrieve the details for a case created from a Salesforce B2C Commerce Cloud storefront via the service-cloud-connector.  You can use this query to inspect the case details.

> Salesforce B2C Commerce-specific properties are prefixed with SFCC.

```sql
select  Id,
        IsDeleted,
        CaseNumber,
        AccountId,
        SuppliedName,
        SuppliedEmail,
        SuppliedPhone,
        SuppliedCompany,
        Type,
        RecordTypeId,
        Status,
        Origin,
        Subject,
        Priority,
        Description,
        CreatedDate,
        CreatedById,
        LastModifiedDate,
        LastModifiedById
from    Case
where   AccountId = '[Insert AccountId]'
```
#### Sample Account Query
This query can be used to retrieve the details for a PersonAccount created from a Salesforce B2C Commerce Cloud storefront via the service-cloud-connector.  You can use this query to inspect the Account details.

> Salesforce B2C Commerce-specific properties are prefixed with SFCC.

```sql
select  Id,
        IsDeleted,
        Name,
        FirstName,
        LastName,
        RecordTypeId,
        OwnerId,
        PersonContactId,
        IsPersonAccount,
        PersonEmail,
        SCCActive__c,
        SFCC_update__c,
        Contact_Status__pc,
        From_SFCC__pc,
        SFCC_Customer_Id__pc,
        SFCC_Customer_Number__pc,
        CreatedDate,
        CreatedById,
        LastModifiedDate,
        LastModifiedById
from    Account
where   Id = '[Insert AccountId]'
```

#### Sample Account Address Query
This query can be used to retrieve the details for addresses associated to a PersonAccount imported from a Salesforce B2C Commerce Cloud storefront via the service-cloud-connector.  You can use this query to inspect the address details.

> Salesforce B2C Commerce-specific properties are prefixed with SFCC.

```sql
select  Id,
        Account__c,
        IsDeleted,
        Name,
        First_Name__c,
        Last_Name__c,
        Full_Name__c,
        Second_Name__c,
        Address_Line_1__c,
        Address_Line_2__c,
        Address_Line_3__c,
        City__c,
        State__c,
        Country__c,
        Post_Box__c,
        Postal_Code__c,
        OwnerId,
        CreatedDate,
        CreatedById,
        LastModifiedDate,
        LastModifiedById
from    Address__c
where   Account__c = '[Insert AccountId]'
```

#### Sample Order Query
This query can be used to retrieve the details for orders associated to a PersonAccount imported from a Salesforce B2C Commerce Cloud storefront via the service-cloud-connector.  You can use this query to inspect the order details.

> Salesforce B2C Commerce-specific properties are prefixed with SFCC.

```sql
select  Id,
        OwnerId,
        AccountId,
        EffectiveDate,
        IsReductionOrder,
        Status,
        BillingAddress,
        ShippingAddress,
        StatusCode,
        OrderNumber,
        TotalAmount,
        IsDeleted,
        LastViewedDate,
        LastReferencedDate,
        Order_Contact__c,
        Order_SCCSync_Status__c,
        SFCC_Order_Number__c,
        SFCC_Order_Total__c,
        from_SFCC__c,
        isContactIdExist__c,
        CreatedDate,
        CreatedById,
        LastModifiedDate,
        LastModifiedById        
from    Order
where   AccountId = '[Insert AccountId]'
```

#### Sample Order Line Item
This query can be used to retrieve the details for orders line items associated to an Order imported from a Salesforce B2C Commerce Cloud storefront via the service-cloud-connector.  You can use this query to inspect the order line-item details.

> Salesforce B2C Commerce-specific properties are prefixed with SFCC.

```sql
select  Id,
        Detailview_Image_Link__c,
        Listview_Image_Link__c,
        Order__c,
        Order_Line_Item_Id__c,
        Name,
        OwnerId,
        Order_Line_Item_Price__c,
        Product_Id__c,
        Quantity__c,
        RecordTypeId,
        Unit_Price__c,
        CreatedById        
from    Order_Line_Item__c
where   Order__c = '[Insert OrderId]'
```
### Useful Apex Scripts

#### Purge Service Cloud Connector Anonymous Apex
This snippet of Apex code can be used to purge test data from a service-cloud-environment.  It attempts to purge dependent data first -- so that orders and accounts can ultimately be removed as well.

> This is not designed for use in a production environment, so limit the use of this script to development or test environments.

This script attempts to delete all Cases, Addresses, and Order Line Items as a precursor to deleting all Orders and Accounts that are not the 'Default' Account.

##### Anonymous Apex to Purge Service Cloud Connector Data

Execute this Apex via an [Anonymous Apex Window](https://help.salesforce.com/articleView?id=code_dev_console_execute_anonymous.htm&type=5) to remove the service-cloud-connector related data from your Salesforce Org.  For more information on Apex, visit the [Introduction to Apex](https://trailhead.salesforce.com/en/content/learn/modules/apex_database/apex_database_intro) trail on [Trailhead](https://trailhead.salesforce.com/).

> Remember, this is not designed for use in a production environment, as it attempts to purge all records for the objects outlined in the script.

```apex
// Purge all addresses stored in the custom address object
List<Address__c> addressToDel = new List<Address__c>();
addressToDel = [select id from Address__c];

delete addressToDel;

// Purge all cases
List<Case> caseToDel = new List<Case>();
caseToDel = [select id from Case];

delete caseToDel;

// Purge all order line-items
List<Order_Line_Item__c> orderLineItemsToDel = new List<Order_Line_Item__c>();
orderLineItemsToDel = [select id from Order_Line_Item__c];

delete orderLineItemsToDel;

// Purge all orders
List<Order> ordersToDel = new List<Order>();
ordersToDel = [select id from Order];

delete ordersToDel;

// Purge all Accounts
List<Account> accountsToDel = new List<Account>();
accountsToDel = [select Id from Account where Name != 'Default'];

delete accountsToDel;
```
