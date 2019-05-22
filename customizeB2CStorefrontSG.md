# service-cloud-connector: configuration guide
###### - Done? Go Back to the [readMe.md](../README.md)
**What is the service-cloud-connector?**    
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Customize the Salesforce B2C Storefront
###### This guide captures the configuration and installation activities necessary to extend the storefront with the custom hooks used to trigger just-in-time synchronization for cases, customers, and orders.

### What You Need to Get Started

Before extending the storefront, review and complete the installation instructions found in these pages:

- [Setup Service Cloud](setupServiceCloud.md) provides guidance on how to configure Service Cloud to support the creation and synchronization of B2C Commerce objects (ex. customers, addresses, orders) in Service Cloud.
- [Setup Salesforce B2C Commerce](setupB2CCommerce.md) provides guidance on how to configure Salesforce B2C Commerce to announce changes to B2C Commerce objects and ingest changes to these objects initiated from Service Cloud.
- [Setup Salesforce B2C Commerce Jobs](setupB2CCommerceJobs.md) provides guidance on how to configure Salesforce B2C Commerce jobs used to synchronize customer and order data between the B2C Commerce and Service Clouds.

> The setup of the connector cartridge's jobs and storefront extensions depends on these three steps being completed. 

### Update the Cartridge path

To enable the features of the service-cloud-connector, the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge must be installed in your storefront and setup in the storefront's cartridge path.

1. Log into Business Manager using an account that has Administrative rights to the site being integrated with Service Cloud.

1. In Business Manager, select **Administration > Sites > Manage Sites**.
1. Select the site being integrated to Service Cloud.
1. Click the Settings tab.
1. Update the cartridge path to include the int\_service\_cloud cartridge. Remember to include the cartridge near the front of the cartridge-path before any core cartridges.
	
	```
	int_service_cloud:app_storefront_controllers:app_storefront_pipelines:app_storefront_core
	```

	> In this example, the siteGenesis cartridge path is represented, with the *int\_service\_cloud* reference before the main controller cartridge.
	
1. Click **Apply** to save your changes to the cartridge path.

### Extending the Storefront Codebase

With the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge installed, the final step is to extend the specific storefront controllers to include the [custom hooks](https://documentation.demandware.com/DOC2/topic/com.demandware.dochelp/SGJC/SiteGenesisDevelopmentOverview.html#SiteGenesisDevelopmentOverview__hooks) used to trigger data synchronization with Service Cloud.

#### Extend the Customer Service Controller

##### Create Service Cloud cases from Salesforce B2C Commerce

The [CustomerService.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/controllers/CustomerService.js) controller is used to send 'Contact Us' messages submitted by storefront customers to customer service.  This controller can be extended to trigger the creation of a case in Service Cloud via the [custom hooks](../b2c/sfcc/cartridges/int_service_cloud/hooks.json) defined in the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge.

> You can compare the service-cloud-connector's [CustomerService.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/controllers/CustomerService.js) controller to the base SiteGenesis [CustomerService.js](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/controllers/CustomerService.js) controller to view the changes made support the creation of cases in Service Cloud.

##### Implement the app.case.created Custom Hook

To implement case-creation in Service Cloud, add the following code snippet to the [CustomerService.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/controllers/CustomerService.js) controller's [submit()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/controllers/CustomerService.js#L42) function.

```javascript
// Look for this line in the CustomerService.js controller to find
// the location where the snippet should be inserted into the controller

// This snippet should be included after we confirm that the contactUs
// form was successfully processed by Salesforce B2C Commerce
if (contactUsResult && (contactUsResult.getStatus() === Status.OK)) {

    // -----------------------------------------
    // -- BEGIN: service-cloud-connector snippet
    // -----------------------------------------

    /**
     * @type {dw.system.HookMgr}
     */
     
    // Create an instance of the HookMgr
    const HookMgr = require('dw/system/HookMgr');

    // Identify the hook we'll leverage
    var hookID = 'app.case.created';
    
    // Verify that the hook has been defined
    if (HookMgr.hasHook(hookID)) {
    
        // If so, call it -- and pass in the contents of the contactUsForm 
        // to create the corresponding Service Cloud case record
        HookMgr.callHook(
            hookID,
            hookID.slice(hookID.lastIndexOf('.') + 1),
            contactUsForm
        );
        
    } else {
    
        // Otherwise, write to the debug-log that the hook wasn't registered
        require('dw/system/Logger').debug('no hook registered for {0}', hookID);

    }

    // -----------------------------------------
    // --   END: service-cloud-connector snippet
    // -----------------------------------------

    // Render the confirmation message 
    app.getView('CustomerService', {
        ConfirmationMessage: 'edit'
    }).render('content/contactus');

} else {

    // Otherwise, re-render the contact-us form
    app.getView('CustomerService').render('content/contactus');

}
```

With this change made, you should now be able to create cases in Service Cloud by submitting the Contact Us form in SiteGenesis.

#### Extend the Customer Model

##### Synchronize Salesforce B2C Commerce Customers with Service Cloud

The [CustomerModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/CustomerModel.js) model contains the logic to create and modify Salesforce B2C Commerce customer profiles.  This model can be extended to trigger the creation or modification of PersonAccounts in Service Cloud via the [custom hooks](../b2c/sfcc/cartridges/int_service_cloud/hooks.json) defined in the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge.

> You can compare the service-cloud-connector's [CustomerModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/CustomerModel.js) model to the base SiteGenesis [CustomerService.js](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/CustomerModel.js) model to view the changes made to support the synchronization of customer data across both clouds.

##### Implement the app.account.created Custom Hook

To implement PersonAccount creation in Service Cloud, add the following code snippet to the [CustomerModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/CustomerModel.js) model's [createAccount()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/CustomerModel.js#L197) function.

```javascript
// The custom hook should be invoked after the customer-record
// has been created and the transaction used to create the customer has
// been closed via the Transaction.commit() function

// -----------------------------------------
// -- BEGIN: service-cloud-connector snippet
// -----------------------------------------

/**
 * @type {dw.system.HookMgr}
 */

// Initialize an instance of the hook manager
const HookMgr = require('dw/system/HookMgr');

// Define the instance of the hook being leveraged
var hookID = 'app.account.created';

// Verify that the hook has been defined
if (HookMgr.hasHook(hookID)) {

    // If so, call it -- and pass in the contents of the newCustomer 
    // model to create the Service Cloud PersonAccount
    HookMgr.callHook(
        hookID,
        hookID.slice(hookID.lastIndexOf('.') + 1),
        newCustomer.object
    );

} else {

    // Otherwise, write to the debug-log that the hook wasn't registered
    require('dw/system/Logger').debug('no hook registered for {0}', hookID);

}

// -----------------------------------------
// --   END: service-cloud-connector snippet
// -----------------------------------------
```

With this change made, you should now be able to create PersonAccounts in Service Cloud by registering new customers via the SiteGenesis new-customer registration form.

##### Implement the app.account.updated Custom Hook

To implement PersonAccount modifications in Service Cloud, add the following code snippet to the [CustomerModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/CustomerModel.js) model's [editAccount()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/CustomerModel.js#L298) function.

```javascript
// The custom hook should be invoked after the customer-record has been 
// successfully modified and the transaction used to update the customer 
// record has been closed via the Transaction.commit() function.

// -----------------------------------------
// -- BEGIN: service-cloud-connector snippet
// -----------------------------------------

/**
 * @type {dw.system.HookMgr}
 */

// Initialize an instance of the hook manager
const HookMgr = require('dw/system/HookMgr');

// Define the instance of the hook being leveraged
var hookID = 'app.account.updated';

// Verify that the hook has been defined
if (HookMgr.hasHook(hookID)) {

    // If so, call it -- and pass in the contents of the customer object
    // to update the properties of the corresponding Service Cloud PersonAccount
    HookMgr.callHook(
        hookID,
        hookID.slice(hookID.lastIndexOf('.') + 1),
        customer
    );

} else {

    // Otherwise, write to the debug-log that the hook wasn't registered
    require('dw/system/Logger').debug('no hook registered for {0}', hookID);

}

// -----------------------------------------
// --   END: service-cloud-connector snippet
// -----------------------------------------
```

With this change made, you should now be able to update the properties of an existing PersonAccount in Service Cloud by modifying the account's corresponding Salesforce B2C Commerce customer profile via the storefront.

#### Extend the Address Model

##### Synchronize Salesforce B2C Commerce Customer Addresses with Service Cloud

The [AddressModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/AddressModel.js) model contains the logic to create and modify Salesforce B2C Commerce customer addresses.  This model can be extended to trigger the creation or modification addresses associated to a corresponding Service Cloud PersonAccount via the [custom hooks](../b2c/sfcc/cartridges/int_service_cloud/hooks.json) defined in the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge.

> You can compare the service-cloud-connector's [AddressModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/AddressModel.js) model to the base SiteGenesis [AddressModel.js](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/AddressModel.js) model to view the changes made to support the synchronization of customer addresses across both clouds.

> Prior to implementing the customer address synchronization, review the [supported address creation and modification use-cases](supportedUseCases.md).  The use-case documentation includes caveats describing what synchronization between Salesforce B2C Commerce and Service Cloud is supported.

##### Implement the app.account.updated Custom Hook

To enable address synchronization cross-cloud, locate the AddressModel(address) return statement within the [AddressModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/AddressModel.js) [create()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/AddressModel.js#L74) and [update()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/AddressModel.js#L105) functions.

```javascript
// Look for this return statement and insert the snippet just above-it
return new AddressModel(address);
```

Once the return statement has been located, add the following snippet above the identified return statement within the [AddressModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/AddressModel.js) [create()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/AddressModel.js#L74) and [update()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/AddressModel.js#L105) functions.

```javascript
// The custom hook should be invoked after the address has been
// successfully created or updated and attached to the customer profile

// -----------------------------------------
// -- BEGIN: service-cloud-connector snippet
// -----------------------------------------

/**
 * @type {dw.system.HookMgr}
 */

// Initialize an instance of the hook manager
const HookMgr = require('dw/system/HookMgr');

// Define the instance of the hook being leveraged
var hookID = 'app.account.updated';

// Verify that the hook has been defined
if (HookMgr.hasHook(hookID)) {

    // If so, call it -- and pass in the contents of the customer object
    // to update the address details associated to the customer account
    HookMgr.callHook(
        hookID,
        hookID.slice(hookID.lastIndexOf('.') + 1),
        customer
    );

} else {

    // Otherwise, write to the debug-log that the hook wasn't registered
    require('dw/system/Logger').debug('no hook registered for {0}', hookID);

}

// -----------------------------------------
// --   END: service-cloud-connector snippet
// -----------------------------------------
```

With this change made, you should now be able to create or modify addresses for registered Salesforce B2C Commerce users via the storefront and push these updates to the corresponding Service Cloud PersonAccount.

> As a reminder, review the [supported address creation and modification use-cases](supportedUseCases.md) for specifics on the types of address-synchronization supported by the service-cloud-connector.

#### Extend the Order Model

##### Create Orders in Service Cloud

The [Order Model.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/OrderModel.js) library is used to process orders placed by customers within a Salesforce B2C Commerce storefront.  This model can be extended to trigger the creation of an order in Service Cloud via the [custom hooks](../b2c/sfcc/cartridges/int_service_cloud/hooks.json) defined in the [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridge.

> You can compare the service-cloud-connector's [OrderModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/OrderModel.js) library to the base SiteGenesis [OrderModel.js](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/scripts/models/OrderModel.js) library to view the changes made support the creation of orders in Service Cloud.

##### Implement the app.order.created Custom Hook

To implement order-creation in Service Cloud, add the following code snippet to the [OrderModel.js](../b2c/sfcc/cartridges/app_storefront_controllers/cartridge/scripts/models/OrderModel.js) model's [submit()](https://github.com/SalesforceCommerceCloud/sitegenesis/blob/master/app_storefront_controllers/cartridge/scripts/models/OrderModel.js#L64) function.

> When implementing the app.order.created hook, place the custom hook logic *AFTER* the order creation logic has successfully completed.  The order should be placed successfully and written to the Salesforce B2C Commerce platform before sending the order details to Service Cloud.

```javascript
// The custom hook should be invoked after the order has been
// successfully placed and the transaction used to create the order
// record has been closed via the Transaction.commit() function.

// -----------------------------------------
// -- BEGIN: service-cloud-connector snippet
// -----------------------------------------

/**
 * @type {dw.system.HookMgr}
 */

// Create an instance of the hook manager
const HookMgr = require('dw/system/HookMgr');

// Initialize the name of the hook we're going to leverage
var hookID = 'app.order.created';

// Confirm that the hook exists and was registered with the
// service-cloud-connector's int_service_cloud cartridge
if (HookMgr.hasHook(hookID)) {

    // Invoke the hook and include the order details
    HookMgr.callHook(
        hookID,
        hookID.slice(hookID.lastIndexOf('.') + 1),
        order
    );

} else {

    // Otherwise, call out that the order-management hook was not created
    require('dw/system/Logger').debug('no hook registered for {0}', hookID);

}

// -----------------------------------------
// --   END: service-cloud-connector snippet
// -----------------------------------------
```

> As a reminder, review the [supported or use-cases](supportedUseCases.md) for specifics on what order synchronization scenarios are supported by the service-cloud-connector.

##### Updating the Status of Orders in Service Cloud

Communicating order status updates (including shipping, payment processing, and fulfillment) by extending the integration logic used to parse order updates to trigger synchronization of these updates with Service Cloud.  These integration functions are often custom to the storefront and brand.  To trigger order synchronization with Service Cloud, implement the app.order.updated custom hook within the routine(s) used to parse and ingest order changes.

##### Implement the app.order.updated Custom Hook

The app.order.updated customer hook implementation is similar to the app.order.created hook in its structure.  The only difference is the hook name being called.

> When implementing the app.order.updated hook, place the hook logic *AFTER* the order status has been updated successfully.  The order's status should be written to the Salesforce B2C Commerce platform before sending the synchronization request to Service Cloud.

```javascript
// The custom hook should be invoked after the order has been
// successfully updated and the transaction used to modify the order
// record has been closed via the Transaction.commit() function.

// -----------------------------------------
// -- BEGIN: service-cloud-connector snippet
// -----------------------------------------

/**
 * @type {dw.system.HookMgr}
 */

// Create an instance of the hook manager
const HookMgr = require('dw/system/HookMgr');

// Initialize the name of the hook we're going to leverage
var hookID = 'app.order.updated';

// Confirm that the hook exists and was registered with the
// service-cloud-connector's int_service_cloud cartridge
if (HookMgr.hasHook(hookID)) {

    // Invoke the hook and include the order details
    HookMgr.callHook(
        hookID,
        hookID.slice(hookID.lastIndexOf('.') + 1),
        order
    );

} else {

    // Otherwise, call out that the order-management hook was not created
    require('dw/system/Logger').debug('no hook registered for {0}', hookID);

}

// -----------------------------------------
// --   END: service-cloud-connector snippet
// -----------------------------------------
```

> As a reminder, review the [supported or use-cases](supportedUseCases.md) for specifics on what order synchronization scenarios are supported by the service-cloud-connector.
