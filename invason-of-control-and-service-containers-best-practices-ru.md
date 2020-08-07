# Вторжение лучших практик в области управления и сервисных контейнеров.
## Введение:
Разработка масштабируемых, многократно используемых и расширяемых корпоративных приложений требует больших инженерных знаний, многолетнего опыта и принятия хорошо проверенных шаблонов проектирования.
По этой причине современные фреймворки, такие как symphony, laravel, angular, reactjs и spring, стали полагаться на такие методы и инструменты, как сервисные контейнеры, эмиттеры событий и шаблоны репозитория для решения часто встречающихся проблем.

Эти инструменты при правильном использовании могут ускорить разработку, одновременно увеличивая производительность труда разработчиков и делая процесс кодирования приятным.

Однако при неправильном использовании производительность приложения может значительно снизиться, потребление памяти может увеличиться без необходимости, а в крайних случаях система может остановиться.
Это может происходить как для серверных, так и для клиентских приложений, таких как одностраничные приложения (SPA) и прогрессивные веб-приложения (PWA).

Эта статья посвящена передовому опыту использования контейнеров служб Magento 2, также известных как диспетчер объектов. Однако знания могут быть распространены на другие инфраструктуры, которые используют контейнер службы или аналогичный механизм для построения зависимостей объектов.

## Сервисные контейнеры:
Инверсия управления (IoC) в программной инженерии - это принцип, используемый для перехвата общего потока программы. Это помогает повысить модульность и расширяемость.
Методы реализации включают шаблоны проектирования, такие как:

1. Шаблон внедрения зависимости
2. Шаблон поиска сервисов
3. Контекстный поиск
4. Шаблон проектирования метода шаблона
5. Шаблон разработки стратегии

## Шаблон внедрения зависимостей в Magneto 2:
При использовании шаблона внедрения зависимостей вторжение в управление достигается путем разделения компонентов и функциональных возможностей модуля на интерфейсы. Затем с помощью сервисного контейнера эти интерфейсы привязываются к своим реализациям во время начальной загрузки приложения.

### Пример 1: Регистрация услуг.
В Magneto 2 службы регистрируются в файле di.xml модуля приложения.

<?xml version="1.0" encoding="UTF-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="Shop\Api\ShoppingCartInterface" type="Shop\Model\Carts\ShoppingCart" />
</config>

В этом примере интерфейс корзины покупок Magento под названием ShoppingCartInterface привязан к его реализации под названием ShoppingCart.

## Внимание!
Атрибут XML «for» должен ссылаться на интерфейс, тогда как атрибут «type» должен ссылаться на фактическую реализацию или создаваемый класс.

### Пример 2: Разрешающие услуги.
Зависимости могут быть разрешены вне контейнера службы с помощью метода get.

### Пример: использование Magento 2 ObjectManager (сервисный контейнер)

$ioc = \Magento\Framework\App\ObjectManager::getInstance();

$logger = $ioc->get(\Psr\Log\LoggerInterface::class);

В этом примере мы получаем экземпляр контейнера службы Magento и вызываем метод get для разрешения нашего Logger. Следует отметить, что реализация Psr \ Log \ LoggerInterface будет возвращена. Если объект не зарегистрирован как синглтон, мы всегда получаем новый экземпляр каждый раз, когда вызываем метод get. Таким образом, сервис-контейнер можно использовать как фабрику для создания объектов приложения.
 
## Внедрение конструктора:
Контейнеры служб используют подсказку типов для разрешения зависимостей.

### Пример: метод построения класса PHP


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
    
Внедрение конструктора - обычная практика, но ее следует использовать с осторожностью.
В примере с классами, связанными как синглтоны, такими как LoggerInterface и ShoppingCartInterface, всегда будет правильно разрешаться один экземпляр.
Однако для таких зависимостей, как ShoppingCart и WatchingCart, всегда создается новый экземпляр. Это может привести к высокому потреблению памяти и снижению производительности.
Кроме того, мы создаем экземпляры объектов, которые могут не использоваться во время выполнения.
Например, регистратор нужен только в том случае, если мы действительно перехватываем исключение.
Чтобы исправить эти проблемы, рекомендуется решать классы лениво.

### Пример: приведенный выше пример можно улучшить следующим образом


public function __construct(\Magento\Framework\ObjectManagerInterface $ioc) {
        $this->ioc = $ioc;
}

public function getFilesystem() {
        return $this->ioc->get(\Magento\Framework\Filesystem ::class);
}

public function getLogger() {
        return $this->ioc->get(\Psr\Log\LoggerInterface::class);
}

Таким образом мы не только освобождаем конструктор, но и разрешаем зависимости по мере необходимости во время выполнения. Это освободит память и повысит производительность приложения.

