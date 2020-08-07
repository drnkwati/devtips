# Invasion of control and service containers best practices.
## Introduction: 
Developing enterprise applications that are scalable, reusable and extendable requires a great deal of engineering knowledge, years of experience, and adoption of well-tested design patterns.
For this reason, modern frameworks such as symphony, laravel, angular, reactjs, and spring have come to rely on tools and techniques such as service containers, event emitters, and repository patterns to solve commonly encountered problems.

These tools if used appropriately can speed up development while increasing developer productivity and making the coding process enjoyable.

However, if used inappropriately, application performance can significantly degrade, memory consumption can increase unnecessarily and in extreme cases, the system can come to a halt.
This can happen for both server and client-based applications such as single page applications (SPA’s) and progressive web applications (PWA’s).

This article is focused on best practices when using Magento 2 service container also known as the object manager. However, the knowledge can be extended to other frameworks that make use of a service container or a similar mechanism for building object dependencies.

## Service containers: 
Inversion of control (IoC) in software engineering is a principle used for intercepting the general program flow. This helps to increase modularity and extensibility.
Implementation Techniques include design patterns such as:

1. Dependency injection pattern
2. Service locator pattern
3. Contextualized lookup
4. Template method design pattern
5. Strategy design pattern

## Dependency injection pattern in Magneto 2:
When using the dependency injection pattern, invasion of control is achieved by separating components and module functionality into interfaces. Using the service container, these interfaces are then bound to their implementations during application bootstrapping. 

### Example 1: Registering services.
In Magneto 2, services are registered in the app module di.xml file.

<?xml version="1.0" encoding="UTF-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="Shop\Api\ShoppingCartInterface" type="Shop\Model\Carts\ShoppingCart" />
</config>

In this example, the Magento shopping cart interface called ShoppingCartInterface is bound to its implementation called ShoppingCart.

## Attention!
The XML "for" attribute should reference an interface, while the "type" attribute should reference an actual implementation or instantiable class.

### Example 2: Resolving services.
Dependencies can be resolved out of the service container using the get method.

### Example: Using Magento 2  ObjectManager (service container)

$ioc = \Magento\Framework\App\ObjectManager::getInstance();

$logger = $ioc->get(\Psr\Log\LoggerInterface::class);

In this example we get an instance of Magento service container and call the get method to resolve our Logger. It should be noted that an implimentation of the Psr\Log\LoggerInterface will be returned. If the object is not registered as a singleton, then we always get a new instance each time we call the get method. Therefor the service container can be used like a factory for creating application objects.  
 
## Constructor injection:
Service containers make use of type hinting to resolve dependencies. 

### Example: PHP class construct method

    public function __construct(
        \Magento\Framework\Filesystem $fileSystem,
        \Psr\Log\LoggerInterface $logger,
        \Shop\Api\Carts\ShoppingCartInterface $cart,
        \Shop\Model\Carts\ShoppingCart $shoppingCart,
        \Shop\Model\Carts\WatchingCart $watchingCart
    ) {
        $this->fileSystem = $fileSystem;
        $this->logger = $logger;
        $this->cart = $cart;
        $this->shoppingCart = $shoppingCart;
        $this->watchingCart = $watchingCart;
    }

Constructor injection is a common practice but should be used with coutious.
In the example about, classes bound as singletons like the LoggerInterface and ShoppingCartInterface will always be properly resolved to a single instance. 
However, for dependencies such as  ShoppingCart and  WatchingCart a new instance is always created. This might result in high memmory consumption and degrade performance.
Furthermore we are creating object instances that might not actually be used during runtime.
Forexample, the logger is only needed if we actually catch an exception. 
To fix this issues, it is advisable to resolve classes lazzyly.

### Example: The example above can be improved as follows


public function __construct(\Magento\Framework\ObjectManagerInterface $ioc) {
        $this->ioc = $ioc;
}

public function getFilesystem() {
        return $this->ioc->get(\Magento\Framework\Filesystem ::class);
}

public function getLogger() {
        return $this->ioc->get(\Psr\Log\LoggerInterface::class);
}

This way we not only free up the constructor, but also resolve dependencies as needed durin runtime. This will free up memmory and improve application performance.


