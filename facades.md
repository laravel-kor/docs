# 파사드

- [소개](#introduction)
- [의미](#explanation)
- [실질적인 사용법](#practical-usage)
- [파사드 생성](#creating-facades)
- [모의 파사드](#mocking-facades)
- [파사드 클래스 참조](#facade-class-reference)

<a name="introduction"></a>
## 소개

파사드는 클래스들에게 어플리케이션의 [서비스 컨테이너](/docs/{{version}}/container)에서 사용 할 수 있는 "static" 인터페이스를 제공합니다. 라라벨은 많은 파사드들을 포함하고 있고, 여러분은 아마 여러분도 모르게 파사드를 사용하고 있었을겁니다! 라라벨 "파사드"는 서비스 컨테이너에 내재되어있는 클래스들을 간결하고, 기존의 static 메서드들보다 더 테스트에 용이하고 유연성을 유지하는 표현 구문의 이점을 제공하는 "static proxies" 형태로 제공합니다.

이따금, 어플리케이션과 패키지들에 여러분만의 파사드를 생성해야 할 수도 있습니다. 그러므로 이러한 클래스들의 컨셉과 개발 그리고 사용법들을 알아봅시다.

> **주의:** 파사드를 파헤치기 전에, 라라벨 [서비스 컨테이너](/docs/{{version}}/container)와 먼저 친숙해지는것을 강력하게 추천합니다.

<a name="explanation"></a>
## 의미

라라벨 어플리케이션의 컨텍스트에서, 파사드는 컨테이너에서 객체의 액세스를 제공해주는 클래스입니다. 이를 가능하게 해주는 장치는 `Facade` 클래스에 있습니다. 라라벨의 파사드들과 여러분이 만든 파사드들 모두 `Facade` 클래스를 기본으로 확장합니다.

여러분의 파사드는 오직 하나의 `getFacadeAccessor` 메서드만 구현하면 됩니다. 컨테이너에서 무엇을 해결해야 할지 정의 하는것이 `getFacadeAccessor`의 역할입니다. `Facade` 기본 클래스는 파사드로부터 해결 된 객체로 호출을 미루기 위해 `__callStatic()` 매직-메서드를 사용합니다.

그러므로, `Cache::get`같은 파사드를 호출 할 때, 라라벨은 서비스 컨테이너로부터 캐시 매니저 클래스를 해결하고 해당 클래스의 `get` 메서드를 호출 합니다. 기술 적인 용어로, 라라벨 파사드는 라라벨 서비스 컨테이너를 서비스 로케이터로 사용하는 편리한 구문입니다.

<a name="practical-usage"></a>
## 실질적인 사용법

아래의 예제에서, 호출은 라라벨 캐시 시스템으로 이루어 집니다. 이 코드를 살펴보면, `Cache` 클래스의 static `get` 메서드가 호출된다고 가정을 할 수도 있습니다.

    $value = Cache::get('key');

하지만, `Illuminate\Support\Facades\Cache` 클래스를 살펴보면, 여러분은 static `get` 메서드가 없다는 것을 보게 됩니다:

    class Cache extends Facade {

        /**
         * 컴포넌트의 등록된 이름 조회
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }

    }

캐시 클래스는 `Facade` 기초 클래스를 확장하고 `getFacadeAccessor()` 메서드를 정의합니다. 기억하세요, 이 메서드의 역할은 바인딩 된 서비스 컨테이너 이름을 반환하는 것입니다

사용자가 `Cache` 파사드에 어떠한 static 메서드를 참조 할 때, 라라벨은 서비스 컨테이너로부터 `cache` 바인딩을 해결하고 해당 객체에 해당하는 요청된 메서드(이 경우, `get`)를 실행합니다.

그러므로, 우리의 `Cache::get` 호출은 다음처럼 재작성 될 수 있습니다:

    $value = $app->make('cache')->get('key');

#### 파사드 가져오기

기억하세요, 네임스페이스된 컨트롤러에서 파사드를 사용한다면, 파사드 클래스를 해당 네임스페이스로 가져와야 합니다. 모든 파사드는 글로벌 네임스페이스에 있습니다:

    <?php namespace App\Http\Controllers;

    use Cache;

    class PhotosController extends Controller {

        /**
         * 모든 어플리케이션 사진을 조회
         *
         * @return Response
         */
        public function index()
        {
            $photos = Cache::get('photos');

            //
        }

    }

<a name="creating-facades"></a>
## 파사드 생성

여러분의 어플리케이션이나 패키지에 파사드를 생성하는 일은 간단합니다. 여러분은 3가지만 필요합니다:

- 서비스 컨테이너 바인딩
- 파사드 클래스
- 파사드 별칭 구성

예제를 봅시다. 여기에서, 우리는 `PaymentGateway\Payment`로 정의된 클래스가 있습니다.

    namespace PaymentGateway;

    class Payment {

        public function process()
        {
            //
        }

    }

우리는 이 클래스를 서비스 컨테이너로부터 해결 될 수 있도록 해야 합니다. 그러므로, 서비스 공급자에게 바인딩을 추가합니다:

    App::bind('payment', function()
    {
        return new \PaymentGateway\Payment;
    });

이 바인딩을 등록하기 좋은 장소는 `PaymentServiceProvider` 이름의 새로운 [서비스 공급자](/docs/{{version}}/container#service-providers)를 생성하는 것이며, 이 바인딩을 `register` 메서드에 추가합니다. 여러분은 그런 다음 `config/app.php` 구성 파일에서 라라벨이 여러분의 서비스 공급자를 로드하도록 구성 할 수 있습니다.

다음, 우리는 우리만의 파사드 클래스를 생성 할 수 있습니다:

    use Illuminate\Support\Facades\Facade;

    class Payment extends Facade {

        protected static function getFacadeAccessor() { return 'payment'; }

    }

마지막으로, 원한다면, 우리는 `config/app.php` 구성 파일의 `aliases` 배열에 파사드의 별칭을 추가 할 수 있습니다. 이제, `Payment` 클래스의 인스턴스에서 `process` 메서드를 호출 할 수 있습니다.

    Payment::process();

### 오토-로딩 별칭에 대한 노트

`aliases` 배열에 있는 클래스들은 [PHP가 정의되지않은 타입-힌트된 클래스의 오토로드를 시도하지 않기](https://bugs.php.net/bug.php?id=39003) 때문에 몇몇의 인스턴스에서 사용할 수 없을 수도 있습니다. 만약 `\ServiceWrapper\ApiTimeoutException`이 `ApiTimeoutException`로 별칭되어 있다면, `\ServiceWrapper` 네임스페이스 밖에서 `catch(ApiTimeoutException $e)` 예외가 던져 진다고 해도 해당 예외를 절대로 캐치하지 않습니다. 유일한 해결 방법은 별칭을 포기하고, 해당 클래스를 요구하는 각각의 파일의 상단에 타입 힌트 하길 원하는 클래스들을 `use`를 통하여 사용하는 것입니다.

<a name="mocking-facades"></a>
## 모의 파사드

유닛 테스팅은 파사드가 이런 방식으로 작동하는 이유의 중요한 측면입니다. 사실, 테스트의 용이성이 파사드가 존재하는 주된 이유입니다. 더 많은 정보는, 문서의 [모의 파사드](/docs/{{version}}/testing#mocking-facades) 섹션을 확인하세요.

<a name="facade-class-reference"></a>
## 파사드 클래스 참조

아래에서 여러분은 모든 파사드와 내부적인 클래스를 찾을 수 있습니다. 이는 주어진 파사드 뿌리의 API 문서를 빠르게 파헤치는데 유용한 도구입니다. [서비스 컨테이너 바인딩](/docs/{{version}}/container) 키가 있을 경우 같이 표시되어 있습니다.

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/{{version}}/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/{{version}}/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |
Storage  |  [Illuminate\Contracts\Filesystem\Factory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/{{version}}/Illuminate/View/View.html)  |
