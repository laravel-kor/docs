# 계약

- [소개](#introduction)
- [왜 계약인가?](#why-contracts)
- [계약 참조](#contract-reference)
- [어떻게 계약을 사용하는가](#how-to-use-contracts)

<a name="introduction"></a>
## 소개

라라벨의 계약들은 프레임워크에 의해 제공되는 코어 서비스를 정의한 인터페이스 세트 입니다. 예를 들어, `Mailer` 계약은 이메일 전송에 필요한 메서드를 정의하고 있는 반면, `Queue` 계약은 할일들을 대기시키는데 필요한 메소드를 정의합니다.

각각의 계약은 프레임워크가 제공하는 해당 구현 클래스를 갖고 있습니다. 예를 들어, 라라벨은 다양한 드라이버를 포함하고있는 `Queue` 구현 클래스와 [SwiftMailer](http://swiftmailer.org/)에 의해 작동되는 `Mailer` 구현 클래스를 제공합니다.

라라벨의 모든 계약들은 [그들 만의 깃헙](https://github.com/illuminate/contracts)에 있습니다. 이는 모든 계약들에 대한 빠른 참조 포인트를 제공하며, 다른 패키지 개발자에 의해 사용 될 수도록 한개의 패키지로 분리합니다.

<a name="why-contracts"></a>
## 왜 계약인가?

여러분은 아마 계약에 관한 몇가지 질문이 있을 수도 있습니다. 왜 모든 곳에 인터페이스를 사용하나요? 인터페이스를 사용하는것이 더 복잡하지 않은가요?

다음과 같은 제목으로 인터페이스 사용에 대한 이유를 제거해봅시다: 느슨한 결합과 단순성

### 느슨한 결합

첫번째로, 캐쉬 구현 클래스와 단단하게 결합되어있는 코드를 리뷰해봅시다. 다음을 고려해보세요:

    <?php namespace App\Orders;

    class Repository {

        /**
         * 캐쉬
         */
        protected $cache;

        /**
         * 새로운 리포지토리 인스턴스 생성
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * ID 기반의 주문 조회
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))
            {
                //
            }
        }

    }

이 클래스에서, 코드는 주어진 캐쉬 구현 클래스와 단단하게 묶여 있습니다. 우리가 패키지 벤더의 구체적 클래스에 의존하고 있으므로 단단하게 묶여 있습니다. 만약 해당 패키지의 API가 변경되었다면, 우리의 코드 또한 수정되어야 합니다.

마찬가지로, 우리가 만약 기본 캐쉬 기술 (Memcached)를 다른 기술 (Redis)로 교체하고자 할때, 우리는 또 다시 우리의 리포지토리를 수정해야만 합니다. 우리의 리포지토리는 누가 데이터를 제공하는지 또는 어떻게 제공하는지에 관한 많은 지식을 알 필요가 없습니다.

**우리는 이러한 접근 대신, 간단하고 벤더에 얽매이지 않는 인터페이스로 우리의 코드를 향상 시킬 수 있습니다:**

    <?php namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository {

        /**
         * 새로운 인스턴스 리포지토리 생성
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }

    }

이제 해당 코드는 명시된 어떠한 벤더와도 심지어 라라벨과도 묶여 있지 않습니다. 계약 패키지는 구현 클래스와 의존성을 포함하고 있지 않으므로, 여러분은 캐시를 사용하는 코드를 수정하지 않으면서 캐시 구현 클래스를 대체 할 수 있도록 쉽게 특정 계약의 대체 구현 클래스를 작성 할 수 있습니다.

### 단순성

라라벨의 모든 서비스가 깔끔한 인터페이스 내에서 정의될 때, 주어진 서비스에 의해 제공되는 기능을 판단하는 일은 매우 쉽습니다. **계약은 프레임워크의 기능에 대한 간결한 문서로 제공됩니다.**

추가적으로, 여러분이 간단한 인터페이스에 의존하고 있다면, 여러분의 코드는 쉽게 이해되고 유지 보수 됩니다. 크고 복잡한 클래스 내에서 어떤 메소드를 사용 할 수 있는지 추적하는 것보다, 여러분은 심플하고, 깨끗한 인터페이스를 참조 할 수 있습니다.

<a name="contract-reference"></a>
## 계약 참조

아래는 대부분의 라라벨 계약 레퍼런스와 그것들의 라라벨 "파사드" 상대 입니다:

Contract  |  Laravel 4.x Facade
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## 어떻게 계약을 사용하는가

그러면, 여러분은 어떻게 계약의 구현 클래스를 사용할까요? 이는 사실 매우 간단합니다. 라라벨의 컨트롤러, 이벤트 리스너, 필터, 잡 대기, 그리고 라우트 클로저까지 많은 타입의 클래스들은 [서비스 컨테이너](/docs/{{version}}/container)를 통해 해결 됩니다. 그러므로, 계약 구현 클래스를 사용하려면, 여러분은 해결되어야 하는 클래스의 생성자에 해당 인터페이스를 "타입-힌트" 하기만 하면 됩니다:

    <?php namespace App\Handlers\Events;

    use App\User;
    use App\Events\NewUserRegistered;
    use Illuminate\Contracts\Redis\Database;

    class CacheUserInformation {

        /**
         * Redis 데이터페이스 구현
         */
        protected $redis;

        /**
         * 새로운 이벤트 처리 인스턴스 생성
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * 이벤트 처리
         *
         * @param  NewUserRegistered  $event
         * @return void
         */
        public function handle(NewUserRegistered $event)
        {
            //
        }

    }

이벤트 리스너가 해결 될 때, 서비스 컨테이너가 클래스 생성자에서 타입-힌트를 읽고 적합한 값을 주입합니다. 서비스 컨테이너에서 등록하는 방법에 대하여 더 알려면, [해당 문서](/docs/{{version}}/container)를 확인하세요.
