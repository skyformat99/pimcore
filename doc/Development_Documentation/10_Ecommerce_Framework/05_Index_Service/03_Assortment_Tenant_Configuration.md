# Assortment Tenant Configuration

The E-Commerce Framework provides a two level Assortment Tenant system for the Product Index: 

   1. Assortment Tenant: The first level of tenants are heavy-weight tenants. They allow multiple shop instances within one 
   system. The shop instances are completely independent from each other and can contain complete different products, 
   index attributes and even use different product index implementations.

   2. Assortment SubTenant: The second level of tenants are light-weight tenants, which exist within a shop instance. 
   Light-weight tenants use the same Product Index with the same attributes as their parent shop, but can contain a subset 
   of the products. So they are meant to be used for implementing different product assortments within one shop. 

One system can have multiple tenants (heavy- and light-weight). But too many tenants can have bad effects on the performance 
of saving products, since all *Product Indices* need to be updated on each save. 

By default the system always uses one heavy-weight tenant (= `DefaultMysql`), but the default tenant can be disabled. 


### Configuration of Assortment Tenants
For setting up a Assortment Tenant, following steps are necessary: 
- **Implementation of a Tenant Config**
The Tenant Config class is the central configuration of an assortment tenant, defines which products are available for 
the tenant and provides the connection to the used *Product Index* implementation. It needs to implement 
[`Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\Config\IConfig`](https://github.com/pimcore/pimcore/blob/master/pimcore/lib/Pimcore/Bundle/EcommerceFrameworkBundle/IndexService/Config/IConfig.php). 
For detailed information see in-source documentation of the interface. Following implementations are provided by the framework 
and may be extended:
   - `\Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\Config\DefaultMysql`: Provides a simple mysql implementation of 
   the product index.
  - `\Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\ConfigOptimizedMysql`: Provides an optimized mysql implementation 
  of the product index.
  - `Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\Config\ElasticSearch`: Provides a default [elastic search](https://www.elastic.co/) 
  implementation of the product index.
  - `Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\Config\DefaultFactFinder`: Provides a default [fact finder](http://www.fact-finder.de/) 
  implementation of the product index.
  - `Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\Config\DefaultFindologic`: Provides a default [findologic](https://www.findologic.com/) 
  implementation of the product index.

- **Configuring Assortment Tenants within `EcommerceFrameworkConfig.php`:** 
Each tenant has to be configured within `EcommerceFrameworkConfig.php` by defining the tenant config class and index 
attributes. Depending on the *Product Index* implementation, additional configuration may be necessary. The configuration 
also can be outsourced in to an additional configuration file. For more information and samples see the 
[Sample EcommerceFrameworkConfig.php](https://github.com/pimcore/pimcore/blob/master/pimcore/lib/Pimcore/Bundle/EcommerceFrameworkBundle/install/EcommerceFrameworkConfig_sample.php#L472). 


### Setting current Assortment Tenant for Frontend
The [E-Commerce Framework Environment](https://github.com/pimcore/pimcore/blob/master/pimcore/lib/Pimcore/Bundle/EcommerceFrameworkBundle/IEnvironment.php#L22-L22) 
provides following methods to set the current Assortment Tenant: 
```php
<?php
    /**
     * sets current assortment tenant which is used for indexing and product lists
     *
     * @param $tenant string
     * @return mixed
     */
    public function setCurrentAssortmentTenant($tenant);

    /**
     * gets current assortment tenant which is used for indexing and product lists
     *
     * @return string
     */
    public function getCurrentAssortmentTenant();

    /**
     * sets current assortment sub tenant which is used for indexing and product lists
     *
     * @param $subTenant string
     * @return mixed
     */
    public function setCurrentAssortmentSubTenant($subTenant);

    /**
     * gets current assortment sub tenant which is used for indexing and product lists
     *
     * @return string
     */
    public function getCurrentAssortmentSubTenant();
```

The current Assortment Tenant have to be set in the application controllers, e.g. after the login of a specific customer. 
The Index Service provides the corresponding Product List implementation based on the current tenant.


##### Example:
```php
<?php
  
  $factory = \Pimcore\Bundle\EcommerceFrameworkBundle\Factory::getInstance();
  
  //setting assortment tenant
  $environment = $factory->getEnvironment();
  $environment->setCurrentAssortmentTenant("elasticsearch");
  
  //getting suitable product list
  $productlist = $factory->getIndexService()->getProductListForCurrentTenant();
  //doing stuff with product list
```