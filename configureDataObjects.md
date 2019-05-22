# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**   
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Configuring Salesforce Data Objects
###### This section details how to configure the Salesforce data objects leveraged by service-cloud-connector during its operation.

### Case Support

#### Extending the Type and Case Sub-Type Fields

When cases are created via the service-cloud-connector, the JSON document describing the case includes two fields that are validated against picklists on the Case object.

```json
{
	"first_name":"Jerry",
	"last_name":"Seinfeld",
	"email":"jerry.seinfeld@salesforce.com",
	"phone":"555-555-1212",
	"order_no":"012345",
	"case_type":"ContactUs",	
	"case_sub_type":"General Information",
	"subject":"Sample Case-Creation JSON",
	"description":"This JSON case-payload should generate an error in Service Cloud."
}
```

This table highlights the JSON properties that map back to the Case object fields.  The values in these JSON properties must exist in their corresponding Salesforce picklists.

| JSON Field Name| Salesforce Field Name | Is Custom? | Description |
|-----------------|--------------|:-------------:|-----------------------------|
| case_type | Type | No | Represents the top-level category for a given case (ex. 'General Information' or 'Contact Us'). |
| case\_sub\_type | Case Sub Type | Yes | Represents the subcategory for a given case (ex. 'Order Status', 'Technical Support').

If the JSON object being sent to Service Cloud contains types or subtypes that are not present in these picklists, Service Cloud prevents the cases from being created and throws an error.

```json
{
    "errors": {
        "typeOfError": "System.DmlException",
        "status": "500",
        "errorMessage": "Insert failed. First exception on row 0; first error: INVALID_OR_NULL_FOR_RESTRICTED_PICKLIST, Case Sub Type: bad value for restricted picklist field: General Information1: [Case_Sub_Type__c]",
        "errorLine": "56"
    }
}
```

To correct this issue, update the 'Type' and 'Case Sub Type' picklists to align with the category and subcategory values your storefront uses for case creation.  The case-form should be modified to send these values as part of the JSON delivered via the Case Creation Service.

> The storefront case-form *must* populate the JSON properties with values that exist within these picklists.  If an unknown value is encountered during Case Creation, Service Cloud throws the exception highlighted above.

To modify your picklists:

1. Log in to Service Cloud as the System Administrator.
1. From Setup, select **Object Manager**, and select the **Case** object.
1. In the Case properties, select **Fields and Relationships**.

	> Administrators can edit the 'Type' and 'Case Sub Type' picklist values by clicking the individual field names.  This opens the field properties display, where [the field's picklist values can be modified](https://trailhead.salesforce.com/en/content/learn/modules/picklist_admin/picklist_admin_manage).
	
	> Any 'Types' or 'Case Sub Types' created must be mapped back to the 'ContactUs' case [record type](https://webevents.force.com/s/features-record-types.html).

1. Click **Save** to save your picklist changes.

To test case-creation, and validate that the 'Type' and 'Case Sub Type' picklist values have been successfully modified, visit the [Validate the Environments](./validateEnvironments.md) page and use the [Service Cloud Connector - Test Scripts Suite](../postman/serviceCloudConnector.testSuite.json). The test suite contains request definitions for Service Cloud and Commerce Cloud that can be used to exercise common service-cloud-connector use-cases.

### Order Support

#### Extending the Order Status Field

When an order is placed via the storefront and announced to Service Cloud, the order stored in Salesforce B2C Commerce contains a status property that represents the current [workflow or processing status of a given order](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/Ordering/FindinganOrder.html?cp=0_3_9_0_0).  This status property
must contain a value that exists in the Service Cloud Order object's Status field.  If the Salesforce B2C Commerce order contains a status not defined in Service Cloud, the order will not synchronize.

The table below highlights the JSON properties that map back to the Order object fields.  Similar to Cases, The values in the JSON properties must exist in the corresponding Salesforce picklist.

| JSON Field Name| Salesforce Field Name | Is Custom? | Description |
|-----------------|--------------|:-------------:|-----------------------------|
| status | Status | No | Represents the workflow or processing status for a given order. |

If the JSON object being sent to Service Cloud contains status values that are not present in the Status picklist, Service Cloud prevents the order from being synchronized.  The stub order record is created, but its full details and related line-items are not synchronized with Service Cloud.  

The following exception is thrown during synchronization:

```
System.DmlException: Update failed. 
First exception on row 0 with id 8010b000002McIoAAK; 
first error: INVALID_OR_NULL_FOR_RESTRICTED_PICKLIST, 
Status: bad value for restricted picklist field: cancelled: [Status]
```

In this example, the order being synchronized has a status of 'cancelled', which does not exist in the Service Cloud Status picklist.  To correct this issue, update the Status picklist to align with the order status values supported by your storefront.

> For a list of supported Salesforce B2C Commerce order status values, review the [Order Class](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/DWAPI/scriptapi/html/api/class_dw_order_Order.html?cp=0_16_2_19_19) and [OCAPI Shop API Order Resource](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/OCAPI/18.8/shop/Resources/Orders.html?cp=0_12_3_9) pages via the platform documentation.

1. From Setup, select **Object Manager**, and select the **Order** object.

1. In the Order properties, select **Fields and Relationships**.

	> Administrators can edit the 'Status' picklist values by clicking **Status**.  This opens the field properties, from which [the field's picklist values can be modified](https://trailhead.salesforce.com/en/content/learn/modules/picklist_admin/picklist_admin_manage).
	
	> Order status values tend to be customer-specific.  The service-cloud-connector includes two order status categories (Draft and Activated) to which statuses can be mapped.  These status categories may need to be modified to align with your order management workflow.

1. Click **Save** to save your picklist changes.

	To test order-creation, and validate that order synchronization occurs successfully, visit the [Validate the Environments](./validateEnvironments.md) page and use the [Service Cloud Connector - Test Scripts Suite](../postman/serviceCloudConnector.testSuite.json).  The test suite contains request definitions for Service Cloud and Commerce Cloud that can be used to exercise common service-cloud-connector use-cases.

### Custom Setting Field Mappings

#### Managing field-level synchronization for Cases, Accounts, Addresses, and Orders

The service-cloud-connector makes use of [Custom Settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_customsettings.htm) to organize and manage the behavior of field mappings that define how Salesforce B2C Commerce Cloud fields map to their Service Cloud object counterparts.  Four custom setting objects are used to manage data-synchronization behavior.

| Custom Setting | Service Cloud Object | Salesforce B2C Commerce Resource | Description |
|----------|--------------|-------------------|-------------------------------|
|CaseFieldMapping|Case| N/A | This Custom Setting holds field mapping between the case .json object definition and the Service Cloud Case Object. This mapping is used by the Customer Case Management Service (SCCCustomerCaseMgmtService) class. |
|AccountFieldMapping|Account| Customer |This custom setting holds the field mapping between the Salesforce B2C Commerce customer profile and the corresponding PersonAccount record in Service Cloud.  It is leveraged by the Customer Registration Service (SCCCustomerRegistrationService) class. |
|AddressFieldMapping|Address| Customer Address| This custom setting holds attribute mapping between the Salesforce B2C Commerce address attributes and the Service Cloud NSSCCConnector__Address__c attributes. |
|OrderFieldMapping|Order| Order | This Custom Setting holds field mapping between Salesforce Commerce Cloud Order Object and Service Cloud Order Object. This mapping is used by the Order Placement Service (SCCCustomerPlaceOrderService) class to create the order header in Service Cloud. |

##### Field Mapping Object Schema
This schema is used to establish the relationships between Salesforce B2C Commerce Cloud object fields and their corresponding Service Cloud object properties.

| Field Name | Description |
|-------------------------|---------------------------------------|
|Field API Name | This field holds the Service Cloud Field API Name mapped to the corresponding Salesforce B2C Commerce attribute. |
|CC Attribute | This field holds the Salesforce B2C Commerce attribute name which maps to the corresponding Service Cloud Field API Name. |
|Enable Patch| This field controls whether a Salesforce B2C Commerce attribute can be updated from Service Cloud. |
|Enable Sync| This field controls whether a Service Cloud attribute can be updated from Salesforce B2C Commerce. |

> The CC Attribute represents the OCAPI response's object property name that will be mapped.

> The patch and sync values only apply to synchronization activities.  When an object is created in Service Cloud, the request object being sent from Salesforce B2C Commerce leverages the mapping properties -- but does not drive 'write' behavior off on the sync or patch configuration settings.

To access the field-mapping custom settings and field-mapping instances:

1. From Setup, enter Custom Settings in the quick find box, and select **Custom Settings**.
1. Verify that the listing of supported custom settings includes the field-mapping objects described above.
1. Click the name of the field-mapping custom setting (ex. CaseFieldMapping) to view its schema and details.

	> The schema can be extended to support additional properties or controls similar to 'enable patch' or 'enable sync'. Extending the schema requires customization of the service-cloud-connector base classes (see the [Apex Custom Settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_customsettings.htm) guide for details).

1. Click **Manage** in the schema details to view a listing of the individual field mappings configured for a given object.

	The manage view presents a listing of custom setting instances representing mapped fields for the object chosen.  
	
	For example, the AddressFieldMapping custom setting has the following instances:
	
	| Field Instance Name | Field API Name | CC Attribute| Enable Patch | Enable Sync |
	|---------------------|---------------------|------------|:----------:|:----------:|
	| address1 | Address_Line_1__c | address1 | | checked |
	| address2 | Address_Line_2__c | address2 | | checked |
	| address_id | Name | address_id | | checked |
	| city | City__c | city | | checked |
	| first_name | First_Name__c | first_name | | checked |
	| full_name | Full_Name__c | full_name | | checked |
	| last_name | Last_Name__c | last_name | | checked |
	| phone | Phone__c | phone | | checked |
	| postal_code | Postal_Code__c | postal_code | | checked |
	
	Collectively, these field mapping definitions define the cross-cloud field mappings and synchronization behavior for each mapped property.  In this example, the Field API Name and CC Attribute columns describe the Service Cloud and Salesforce B2C Commerce address object fields that correspond to each other.  Enabling 'Sync' for all of these fields enforces that addresses in Service Cloud can only be updated from Salesforce B2C Commerce.  No updates to Salesforce B2C Commerce addresses can be made as the 'Patch' property for each field mapping has not been enabled.

#### Extending the Salesforce Case, Account, Address, and Order Objects

The field mapping custom settings can be used to extend the core objects managed by the service-cloud-connector to include additional custom fields that should be shared between both clouds.  These mappings enable additional data properties not initially configured with the installation of the service-cloud-connector to be added to the object's existing field synchronization.

> Adding new field mappings require __no code or development__ changes to the Service Cloud or Salesforce B2C Commerce code bases.

Before creating field mappings, perform the following pre-requisite tasks:

1. Identify which objects and fields should be mapped between Salesforce B2C Commerce and Service Cloud.
1. Create the fields in Service Cloud prior to defining the field-mapping rules.
1. Verify the desired 'patch' and 'sync' behavior for each object or field mapping being created.

With this information, you can begin defining a new object or field mapping.

1. From Setup, enter Custom Settings in the quick find box, and select **Custom Settings**.
1. Find the name of the object field-mapping for the new the field-mapping definition from the list of displayed custom settings.
	
	> For example, if a new field-mapping is added to the Case object, find the 'CaseFieldMapping' custom setting.

1. Click **Manage** next to the field-mapping object that will extend to view the collection of field-mappings for that object.

1. From the field-mapping listing, click **New** to create a object-specific field-mapping.
1. In the field-mapping form, enter the mapping properties captured during the prerequisite activities.

	| Field Name | Description |
	|------------|----------------------|
	| Name | Describes the external facing label for the field mapping. |
	| CC Attribute | Represents the Salesforce B2C Commerce object attribute being mapped. |
	| Enable Patch | Check this checkbox to allow the Salesforce B2C Commerce object attribute to be modified from Service Cloud. |
	| Enable Sync | Check this checkbox to allow the Service Cloud object attribute to be modified from Salesforce B2C Commerce. |
	| Field API Name | Represents the Service Cloud object attribute being mapped. |

1. Click **Save** to save your changes.
1. Verify that the newly created field-mapping appears in the Custom Settings instance list.

#### Enable Custom Logging

The *SCC LogLevels* custom setting can be used to enable custom logging built into the service-cloud-connector's Service Cloud classes.  This logging renders log messages via the [Developer Console](https://trailhead.salesforce.com/en/content/learn/modules/developer_console).

1. From Setup, enter Custom Settings in the quick find box, and select Custom Settings.
1. Locate the SCC LogLevels custom setting in the settings listing.
1. Click **Manage** displayed to the left of the SCC LogLevels setting name.
1. From the SCC LogLevels detail display, click New to create a setting instance.
1. Complete the new setting form using the values in this table:
	
	| Property Name | Value |
	|---------------|-------|
	| Name | SCCLogLevel |
	| Description | log-level property to enable execution auditing via custom-logging |
	| Loglevel | SYSDEBUG |

1. Click **Save** to save your changes.
1. Verify that the newly created logLevel appears in the Custom Settings instance list.

With the logLevel configured, debug messages now appear in the [Developer Console](https://trailhead.salesforce.com/en/content/learn/modules/developer_console) when triggered via a Service Cloud Apex class that belongs to the service-cloud-connector deployment package.

> For more information on debug logging, and log-levels supported by Salesforce Service Cloud, visit the [Debug Logs](https://help.salesforce.com/articleView?id=code_debug_log.htm&type=5) section of the platform documentation.

### Continue the Setup of Salesforce B2C Commerce
With the initial configuration of Service Cloud completed, continue the setup of the service-cloud-connector by [configuring your Salesforce B2C Commerce](./setupB2CCommerce.md) environment.

