# Contracts

- [소개하기](#introduction)
    - [Contracts Vs. Facades](#contracts-vs-facades)
- [Contracts 사용 시기](#when-to-use-contracts)
    - [느슨한 결합](#loose-coupling)
    - [단순함](#simplicity)
- [Contracts 사용법](#how-to-use-contracts)
- [Contract 참고](#contract-reference)


<a name="introduction"></a>
## 소개하기

라라벨의 Contract는 프레임워크에서 제공하는 코어 서비스들을 정의한 인터페이스들의 모음입니다. 예를 들어, `Illuminate\Contracts\Queue\Queue` Contract에는 어떤 작업들을 큐에서 다룰때 필요한 메소드들이 정의되어 있고, `Illuminate\Contracts\Mail\Mailer` Contract에는 이메일을 보내기 위해 필요한 메소드들을 정의되어 있습니다.

라라벨 프레임워크에는 각각의 Contract에 상응하는 구현체(구현 클래스)가 있습니다. 예를 들어, 라라벨은 다양한 드라이버로 구현된 queue의 구현체를 가지고 있고, [SwiftMailer](http://swiftmailer.org/)를 mailer의 구현체로 가지고 있습니다.

라라벨의 모든 Contract는 [각각의 Github 저장소](https://github.com/illuminate/contracts)를 가지고 있습니다. 이것은 별도의 패키지에 의존하지 않는 각각의 단일 패키지로, 개발자들이 사용할 수 있도록 하는 contract를 위한 하나의 레퍼런스를 제공합니다.

<a name="contracts-vs-facades"></a>
### Contracts VS Facades

라라벨의 [파사드](/docs/{{version}}/facades)는 서비스 컨테이너 외부에서 타입 힌트나, contsract 의 의존성 없이도 라라벨의 서비스를 시용할 수 있는 쉬운 방법을 제공합니다.

클래스 생성자에서 요구하지 않아도 되는 facade와 달리 contsract를 통해 클래스에 대한 명시적 종속성을 정의 할 수 있습니다. 일부 개발자는 이러한 방식으로 종속성을 명시적으로 정의하는 contsract를 선호하지만 대다수의 개발자는 facades의 편리함을 누리고 있습니다.

> {tip} Most applications will be fine regardless of whether you prefer facades or contracts. However, if you are building a package, you should strongly consider using contracts since they will be easier to test in a package context.

> {tip} 대부분의 애플리케이션은 facades나 contract중 선호하는 어느것을 사용해도 무방합니다. 그러나 패키지를 빌드하는 경우 패키지 컨텍스트에서 테스트하기 쉽기 때문에 contract 사용을 강력하게 고려해야합니다.

<a name="when-to-use-contracts"></a>
## Contracts 사용 시기

다른 곳에서 논의 된 것처럼, contract나 facade를 사용하기로 한 결정의 대부분은 개인적인 취향과 개발 팀의 취향에 달려 있습니다. contract와 facades 모두 강력하고 잘 테스트 된 Laravel 응용 프로그램을 작성하는 데 사용할 수 있습니다. 클래스가 제 역할을 하는데에 contract와 facades를 사용하는 데는 실제적인 차이점이 거의 없습니다.

그러나 계약과 관련하여 몇 가지 질문이 있을 수 있습니다. 예를 들어, 인터페이스를 사용하는 이유는 무엇입니까? 인터페이스를 더 복잡하게 사용하도 있지는 않습니까? 다음 표제에 대한 인터페이스를 사용하는 이유를 설명합니다. 느슨한 결합 및 단순성.

<a name="loose-coupling"></a>
### 느슨한 결합

우선, 한 캐시 구현체에 강하게 결합돼 있는 코드를 살펴봅시다.

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

이 클래스의 코드는 주어진 캐시 구현체와 밀접하게 결합돼 있습니다. 특정 패키지 벤더의 캐시 구상클래스에 의존하기 때문에 이 코드는 캐시 클래스와 밀접하게 결합돼 있는 것입니다. 만약 이 패키지의 API가 변경되면 예시로든 이 코드 또한 변경되어야 합니다.

또한, 코드가 사용하는 캐시(Memcached)를 다른 것(Redia)으로 변경하고자 하는 경우, 역시나 Repository 클래스를 다시 수정해야만 할 것입니다. 저장소 클래스는 누가 어떻게 데이터를 제공하는지에 대한 정보를 너무 많이 가지고 있어서는 안 됩니다.

**이렇게 접근하는 대신, 특정 벤더에 구속되지 않고 단순한 인터페이스에 의존하도록 하여 코드를 개선할 수 있습니다:**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

이제 코드는 어떤 특정 벤더, 심지어 라라벨과도 결합되지 않습니다. Contract는 구현체를 가지지 않고, 의존성도 없기 때문에, 주어진  Contract의 다른 구현체를 쉽게 작성할 수 있습니다. 캐시를 사용하는 코드를 수정하지 않고도 캐시 구현체를 대체할 수 있습니다.

<a name="simplicity"></a>
### 단순함

라라벨의 모든 서비스들이 단순한 인터페이스로 보기좋게 정의돼 있기 때문에, 그 서비스들에 의해 제공되는 기능을 알아내는 것이 매우 쉽습니다. **Contract들이 프레임워크의 기능들에 대한 간결한 도큐먼트의 역할을 하는 것입니다.**

또한, 여러분이 간단한 인터페이스에 의존하게 되면, 여러분의 코드는 이해하거나 유지보수하기가 더 쉬워집니다. 크고 복잡한 클래스에서 사용할 수 있는 메소드들을 훑어보는 대신, 단순하고 깨끗한 인터페이스를 참고할 수 있습니다.

<a name="how-to-use-contracts"></a>
## Contract 레퍼런스

그럼 어떻게 Contract의 구현체를 얻을 수 있을까요? 사실 매우 간단합니다.

라라벨에 있는 여러 종류의 클래스들은 컨트롤러, 이벤트리스너, 미들웨어, 큐 작업, 라우트 클로저들을 관리하는 [서비스 컨테이너](/docs/{{version}}/container)를 통해 의존성 해결(resolve) 되고 있습니다. 따라서 의존성이 해결되는 어떤 클래스가 특정 Contract의 구현체를 얻으려면 그 클래스의 생성자에 그 인터페이스를 "type-hint"로 지정해놓으면 됩니다

그 예로 아래의 이벤트 리스너를 보겠습니다.

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Database;

    class CacheOrderInformation
    {
        /**
         * The Redis database implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

이벤트리스너가 의존성 해결될 때, 서비스 컨테이너는 클래스의 생성자에 있는 타입힌트를 읽고, 그에 적합한 값을 주입해 줍니다. 서비스 컨테이너에 무언가를 등록하는 것에 대하여 더 알고싶다면, [이 문서](/docs/{{version}}/container)를 보시기 바랍니다.

<a name="contract-reference"></a>
## Contract 참조

아래는 대부분의 라라벨 Contract와 그에 대응되는 파사드들의 레퍼런스입니다.

Contract  |  References Facade
------------- | -------------
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/{{version}}/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php) | &nbsp;
