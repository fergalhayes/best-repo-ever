# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**    
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Supported Use Cases
###### This section details the supported use-cases for the service-cloud-connector; including caveats, considerations, and cloud-specific behavior for each use-case.

##### About Use Case Support in Salesforce B2C Commerce

Several of the content creation and modification use cases (ex. creating and modifying customers or customer addresses) support changes being made from the Salesforce B2C Commerce storefront as well as the administration tool known as [Business Manager](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/ActiveMerchandising/Merchandisingyoursite.html?cp=0_3).  In order for Business Manager updates to trigger record synchronization with Service Cloud, the __Salesforce Service Cloud SyncStatus__ property on the system object being synchronized must be changed to reflect the type of sync-operation being performed (ex. created vs. updated).  The [customer and order synchronization jobs](setupB2CCommerceJobs.md) must be running in order for Business Manager driven updates to synchronize.

> Customer and order updates triggered from within Business Manager are dependent on the complete setup of the service-cloud-connector for both Salesforce B2C Commerce and Service Cloud. Complete the installation instructions outlined in the [ReadMe](../README.md) file before exercising any of these use cases.

### Case Creation
##### Creating customer service cases that are automatically associated to Service Cloud Person Accounts.

__Salesforce B2C Commerce__

Customers can create cases in Service Cloud via the Salesforce B2C Commerce storefront.

##### Cases Submitted by Anonymous Storefront Users

- Cases created by anonymous storefront users without a Service Cloud Person Account spawn the creation of a new Service Cloud Person Account using the anonymous users' email address.
- Cases created by anonymous storefront users with a Service Cloud Person Accounts mapped to the case-submission email automatically have their cases associated to their existing Person Account.

##### Cases Submitted by Registered Storefront Users

- Cases created by registered storefront users without a Service Cloud Person Account spawn the creation of a new Service Cloud Person Account using the registered users' customer information.
- Cases created by registered storefront users with a Service Cloud Person Accounts mapped to the case-submission email automatically have their cases associated to their existing Person Account.

__Service Cloud__

CSRs can create cases and manually associate them to Person Accounts representing Salesforce B2C Customers.

> This is a native feature of Service Cloud, and not a feature provided by the service-cloud-connector.

### Case Retrieval

##### Retrieving the details of a case of all the cases associated to a Person Account.

__Service Cloud__

Service Cloud exposes the ability to retrieve the details of a specific case -- or the details of all cases associated to a specific Person Account.

> Salesforce B2C Commerce does not support any native case-specific functionality.  Exposing case data to Salesforce B2C Commerce customers require customizations enabling these features.

### Customer Registration

##### Creating storefront customer records and their corresponding Service Cloud Person Accounts.

__Salesforce B2C Commerce__

Customers that register via the storefront, or that are created via Business Manager have a corresponding Person Account record created in Service Cloud.

> When [creating customers via Business Manager](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Customers/AddingaNewCustomer.html?cp=0_3_6_1), assign a __Salesforce Service Cloud SyncStatus__ of 'created' to the customer record, as this flags the customer for Service Cloud Person Account creation.

__Service Cloud__

Person Accounts created in Service Cloud do not result in the creation of a corresponding Salesforce B2C Commerce customer record.

> The sevice-cloud-connector does not support the creation of new Salesforce B2C Commerce customer records initiated via Service Cloud.

### Customer Modification

##### Modifying existing storefront customer records and their corresponding Service Cloud Person Accounts.

__Salesforce B2C Commerce__

Modifying a customers' profile via the storefront or through Business Manager triggers the synchronization of these changes for the corresponding Service Cloud Person Account record.

> This use-case is specific to customer profiles, and does not include addresses.

> When [modifying customers via Business Manager](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Customers/AddingaNewCustomer.html?cp=0_3_6_1), assign a __Salesforce Service Cloud SyncStatus__ of 'updated' to the parent customer record, as this flags the customer for Service Cloud synchronization.

__Service Cloud__

Modifying a Person Account's profile properties automatically triggers the synchronization of those modifications for the corresponding Salesforce B2C Commerce customer record.

> In order for the synchronization to occur, a corresponding Salesforce B2C Commerce customer record must exist.  Modifying a Service Cloud Person Account without the corresponding B2C record does not produce any changes to Salesforce B2C Commerce.

### Customer Address Creation

##### Creating addresses for storefront customer records and their corresponding Service Cloud Person Accounts.

__Salesforce B2C Commerce__

Creating a customer address via the storefront or through Business Manager triggers the creation of a new Service Cloud address associated to the corresponding Person Account.

> When [creating addresses in Business Manager](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Customers/AddingaNewCustomer.html?cp=0_3_6_1), assign a __Salesforce Service Cloud SyncStatus__ of 'updated' to the parent customer record, as this flags the customer for Service Cloud synchronization.

__Service Cloud__

Creating a customer address for a Person Account that has a corresponding Salesforce B2C Commerce customer record does not create an address for that customer.

> The service-cloud-connector does not support creating Salesforce B2C Commerce customer addresses initiated from Service Cloud.

### Customer Address Modification

##### Modifying the properties of storefront customer addresses and the addresses associated to their corresponding Service Cloud Person Accounts.

__Salesforce B2C Commerce__

Modifying the address name property of an existing address via the storefront or through Business Manager creates an address in Service Cloud associated to the corresponding Person Account.

> The address name is used to uniquely identify addresses in Service Cloud.  As a result, changing the name of an address breaks the cross-cloud link -- and creates an address in Service Cloud leveraging the modified name.

Modifying the properties of an existing address without changing the address name via the storefront or through Business Manager updates that address in Service Cloud for the corresponding Person Account.

> When [modifying addresses in Business Manager](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Customers/AddingaNewCustomer.html?cp=0_3_6_1), assign a __Salesforce Service Cloud SyncStatus__ of 'updated' to the parent customer record, as this flags the customer for Service Cloud synchronization.

### Order Placement

##### Replicating the characteristics of a placed order in Salesforce B2C Commerce to Service Cloud to associate the order with a new or existing Person Account.

__Salesforce B2C Commerce__

Placing an order via the storefront creates a representation of that order in Service Cloud.  The order is associated to a Person Account representing the customer that placed the order.  If the customer is a registered Salesforce B2C Commerce customer, the customer's identification properties are included in the Person Account details.

__Service Cloud__

Orders can be placed via Service Cloud on behalf of registered Salesforce B2C Commerce customer via the 'Launch Shopping Cart' button.  This launches the storefront via a new browser with the selected customer logged in -- enabling a CSR to make purchases on behalf of that customer.

> Orders created in Service Cloud via the Order system object do not result in corresponding Salesforce B2C Commerce orders being created.  Orders __must__ be placed via the Salesforce B2C Commerce storefront.

### Updating Order Status

##### Modifying the status property of an order to reflect its processing, fulfillment, or delivery status.

__Salesforce B2C Commerce__

Order status changes made either [manually via Business Manager](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Ordering/UpdatingOrdersAfterPlacement.html) or through an external status update is pushed to Service Cloud so that the order status changes are reflected.

> When [modifying order status in Business Manager](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Ordering/UpdatingOrdersAfterPlacement.html), assign a __Salesforce Service Cloud SyncStatus__ of 'updated' to the order, as this flags the order for Service Cloud synchronization.

## Unsupported Use Cases

###### This section details the unsupported use-cases for the service-cloud-connector.

### Modifying Customer Addresses

##### Modifying the properties of a customer address within Service Cloud.

__Service Cloud__

By default, modifying the properties of an address in Service Cloud associated to a Person Account with a corresponding Salesforce B2C Commerce customer record yields no changes to the customer's addresses.

> The field-level synchronization behavior for customer addresses can be modified by adjusting the 'Patch' and 'Sync' properties of the [AddressFieldMappings](./configureDataObjects.md) custom setting in Service Cloud.

### Deleting Customer Addresses

##### Deleting a customer address associated to a cross-cloud registered user from either Salesforce B2C Commerce or Service Cloud.

__Salesforce B2C Commerce__

When a customer address is deleted from Salesforce B2C Commerce, the corresponding address record is not removed from Service Cloud.

> To remove customer addresses stored in Service Cloud,  delete them manually via the Service Cloud UI using an Administrative Account.

__Service Cloud__

When deleting a customer address associated to a Person Account with a corresponding Salesforce B2C Commerce customer record, no changes are made to the customer records' addresses.

> The service-cloud-connector does not support the deletion of Salesforce B2C Commerce customer addresses initiated from Service Cloud.

### Modifying Order Status

##### Changing the Status of an Order from Service Cloud.

#### Service Cloud

By default, Service Cloud does not have the ability to push order status changes to Salesforce B2C Commerce.

> The field-level synchronization behavior for orders can be modified by adjusting the 'Patch' and 'Sync' properties of the [OrderFieldMappings](./configureDataObjects.md) custom setting in Service Cloud.
