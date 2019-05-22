### Install the plugin\_service\_cloud and configure Cartridge path

To enable the features of the service-cloud-connector, the [plugin\_service\_cloud](../b2c/sfcc/cartridges/plugin_service_cloud) and [int\_service\_cloud](../b2c/sfcc/cartridges/int_service_cloud) cartridges must be installed in your storefront and setup in the storefront's cartridge path.

1. Download the contents of the [b2c/sfcc](../b2c/sfcc) directory.
1. Add the [plugin\_service\_cloud](../b2c/sfcc/cartridges/plugin_service_cloud) cartridge directory to your storefront codebase.
-1.Upload the cartridge folder to your current code-version within your environment.
1. Log into Business Manager, using an account that has Administrative rights to the site being integrated with Service Cloud.
1. In Business Manager, select **Administration > Site > Manage Sites**.
1. Select the site being integrated to Service Cloud.
1. Select the Settings tab.
1. Update the cartridge path to include the 'plugin\_service\_cloud' and 'int\_service\_cloud' cartridges.  Be sure to include the cartridges near the front of the cartridge-path before any core cartridges.

	```
	plugin_service_cloud:int_service_cloud:app_storefront_base
	```

	> In this example, the SFRA cartridge path is represented. Remember to place the *plugin\_service\_cloud:int\_service\_cloud* reference before the main controller cartridge for your storefront.

1. Click **Apply** to save your changes to the cartridge path.
