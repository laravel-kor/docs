# 프레임워크 확장

- [매니저 & 팩토리](#managers-and-factories)
- [캐시](#cache)
- [세션](#session)
- [인증](#authentication)
- [서비스 컨테이너 기반의 확장](#container-based-extension)

<a name="managers-and-factories"></a>
## 매니저 & 팩토리

라라벨은 드라이버 기반의 컴포넌트 생성을 관리하는 여러개의 `Manager` 클래스들을 갖고 있습니다. 이는 캐시, 세션, 인증, 그리고 대기(queue) 컴포넌트들 입니다. 매니저 클래스는 어플리케이션의 설정을 기반으로 특정 드라이버 구현을 만드는 역할을 합니다. 예를 들어, `CacheManager` 클래스는 APC, Memcached, File, 그리고 다른 많은 캐시 드라이버 구현 클래스를 만들 수 있습니다.

이러한 매니저들은 각각의 매니저에 쉽게 새로운 드라이버를 주입시킬때 사용할 수 있는 `extend` 메서드를 포함하고 있습니다. 우리는 각각의 매니저가 사용자 정의 드라이버 주입을 어떻게 지원하는지 예제와 함께 아래에서 다룰 것입니다.

> **주의:** 라라벨에 포함된 `CacheManager`와 `SessionManager`같은 `Manager` 클래스들을 한번 살펴보세요. 이 클래스들을 한번 읽어보면 라라벨이 내부적으로 어떻게 작동하는지 좀 더 깊이 이해를 할 수 있습니다. 모든 매니저 클래스는 몇가지의 유용하고 각각의 매니저에서 공통으로 사용되는 기능을 제공하는 `Illuminate\Support\Manager` 기본 클래스를 확장합니다.

<a name="cache"></a>
## 캐시

라라벨 캐시 기능을 확장하기 위해 우리는 사용자 정의 드라이버 해결자를 매니저에 바인딩 해주는 `CacheManager`의 `extend` 메소드를 사용하고, 이는 모든 매니저 클래스에게 공통됩니다. 예를 들어, "mongo"라는 이름의 새로운 캐시 드라이버를 등록하려면, 우리는 다음과 같이 합니다:

    Cache::extend('mongo', function($app)
    {
        return Cache::repository(new MongoStore);
    });

`extend` 메서드에 첫번째로 전달되는 인수는 드라이버의 이름입니다. 이는 `config/cache.php` 설정 파일의 `driver` 옵션에 해당됩니다. 두번째 인수는 `Illuminate\Cache\Repository` 인스턴스를 반환해야하는 클로저 입니다. 클로저는 `Illuminate\Foundation\Application` 인스턴스이며 서비스 컨테이너인 `$app` 인스턴스를 전달받습니다.

`Cache::extend`의 호출은 신규 라라벨 어플리케이션에 포함되어있는 기본 `App\Providers\AppServiceProvider`의 `boot` 메서드에서 실행되거나 또는 여러분만의 서비스 공급자를 생성하여 확장을 저장 할 수도 있습니다 - `config/app.php`의 공급자 배열에 해당 공급자를 추가 하는 것을 잊지마세요.

사용자 정의 캐시 드라이버를 생성하려면, 첫번째로 `Illuminate\Contracts\Cache\Store` 컨트렉트를 구현해야 합니다. 그러므로 우리의 MongoDB 캐시 구현 클래스는 다음과 같이 생겼을 겁니다:

    class MongoStore implements Illuminate\Contracts\Cache\Store {

        public function get($key) {}
        public function put($key, $value, $minutes) {}
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}

    }

우리는 그냥 MongoDB 연결을 사용하여 각각의 메서드를 구현 하기만하면 됩니다. 구현이 완성되면, 사용자 정의 드라이버 등록을 마칠 수 있습니다:

    Cache::extend('mongo', function($app)
    {
        return Cache::repository(new MongoStore);
    });

만약 여러분의 사용자 정의 캐시 드라이버 코드를 어디에 위치시켜야 할지 공금하다면, Packagist에서 사용 가능하도록 만들어보세요! 또는, `app` 디렉토리 안에 `Extensions` 네임스페이스를 생성 할 수도 있습니다. 하지만, 라라벨은 엄격한 어플리케이션 구조를 갖고 있지 않으므로 여러분은 입맛에 따라 여러분의 어플리케이션을 구성 할 수 있습니다.

<a name="session"></a>
## 세션

사용자 정의 세션 드라이버와 함께 라라벨을 확장하는 일은 캐시 시스템 확장만큼 쉽습니다. 또 다시, 우리는 사용자 정의 코드를 등록하기 위해 `extend` 메서드를 사용 할 겁니다:

    Session::extend('mongo', function($app)
    {
        // SessionHandlerInterface 구현 클래스를 반환
    });

### 세션 확장을 어디에 위치 시키는가

여러분은 `AppServiceProvider`의 `boot` 메서드에 세션 확장 코드를 위치 시킬 수 있습니다.

### 세션 확장 작성

우리의 사용자 정의 세션 드라이버는 `SessionHandlerInterface`를 구현해야 한다는걸 명심하세요. 이 인터페이스는 우리가 구현해야할 몇 가지의 간단한 메서들을 포함하고 있습니다. 스텁 MongoDB 구현 클래스는 다음과 같이 생겼습니다:

    class MongoHandler implements SessionHandlerInterface {

        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}

    }

이 메서드들은 캐시 `StoreInterface`만큼 쉽사리 이해가 되지 않으므로, 각각의 메서드가 무엇을 담당하는지 알아봅시다:

- `open` 메서드는 보통 파일 기반의 세션 저장 시스템에서 사용됩니다. 라라벨은 `file` 세션 드라이버를 포함하고 있으므로, 여러분은 이 메서드에 거의 아무것도 넣을 필요가 없습니다. 여러분은 이 메서드를 빈 스텁으로 나둘 수 있습니다. 이는 단지 PHP가 우리에게 이 메서드를 구현하도록 요구하는 변변치 못한 인터페이스 디자인(곧 다룰)이라는 사실입니다.
- `close` 메서드 또한 `open` 메서드처럼 보통 무시 될 수 있습니다. 대부분의 드라이버에서 필요로 하지 않습니다.
- `read` 메서드는 주어진 `$sessionId`에 해당하는 세션 데이터의 문자열 버전을 반환해야 합니다. 라라벨이 여러분을 위해 직렬화를 수행하므로, 드라이버에서 세션 데이터를 조회하거나 저장 할때 어떠한 직렬화도 할 필요가 없습니다.
- `write` 메서드는 `$sessionId`에 해당하는 주어진 `$data` 문자열을 MongoDB, Dynamo 등의 영구적인 저장 시스템에 작성해야 합니다.
- `destroy` 메서드는 영구적인 저장소로부터 `$sessionId`에 해당하는 데이터를 제거 해야 합니다.
- `gc` 메서드는 주어진 UNIX 타임스탬프 형식의 `$lifetime`보다 오래된 모든 세션 데이터를 제거해야 합니다. Memcached 또는 Redis같이 스스로 만료되는 시스템들은 이 메서드가 비어 있을 수 있습니다.

`SessionHandlerInterface`가 구현되면, 세션 매니저에 등록 할 준비가 되었습니다:

    Session::extend('mongo', function($app)
    {
        return new MongoHandler;
    });

세션 드라이버가 등록되고 나면, 우리는 `config/session.php` 설정 파일에서 `mongo` 드라이버를 사용 할 수 있습니다.

> **주의:** 기억하세요, 만약 사용자 정의 세션 핸들러를 작성 했다면 Packagist에서 공유하세요!

<a name="authentication"></a>
## 인증

인증은 캐시와 세션 기능과 같은 방법으로 확장 될 수 있습니다. 또, 우리는 익숙해진 `extend` 메서드를 사용 할 겁니다:

    Auth::extend('riak', function($app)
    {
        // Illuminate\Contracts\Auth\UserProvider의 구현클래스를 반환
    });

`UserProvider` 구현 클래스는 오직 MySQL, Riak 등과 같은 영구 저장 시스템에서 `Illuminate\Contracts\Auth\Authenticatable` 구현 클래스를 가져오는 책임을 집니다. 이 두개의 인터페이스는 라라벨 인증 메카니즘이 사용자 데이터가 어떻게 저장 되는지 또는 데이터를 표현하는데 어떤 클래스 형식이 사용되는지에 상관없이 지속적인 역할을 할 수 있도록 해줍니다.

`UserProvider` 컨트렉트를 살펴봅시다:

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById` 함수는 보통 사용자를 나타내는 MySQL 데이터베이스의 자동-증가 ID같은 숫자 키를 받습니다. ID와 일치하는 `Authenticatable` 구현 클래스가 조회되고 메서드에 의해 반환되어야 합니다.

`retrieveByToken` 함수는 유저의 고유한 `$identifier`와 `remember_token` 필드에 저장되어있는 "기억하기" `$token`을 받습니다. 이전 메서드처럼, `Authenticatable` 구현 클래스가 반환되어야 합니다.

`updateRememberToken` 메서드는 `$user`의 `remember_token` 필드를 새로운 `$token`으로 업데이트 합니다. 새로운 토큰은 "기억하기" 로그인 시도가 성공했을 때 부여된 신규 토큰 또는, 사용자가 로그아웃 할때 null 중 하나가 될 수 있습니다.

`retrieveByCredentials` 메서드는 어플리케이션에 로그인을 시도 할때 `Auth::attempt` 메서드로 전달되는 자격 증명 배열을 받습니다. 그 다음, 이 메서드는 영구적인 저장소에서 자격 증명에 일치하는 사용자가 있는지 "쿼리"합니다. 보통, 이 메서드는 `$credentials['username']`을 "where" 조건과 함께 쿼리를 실행합니다. 메서드는 그 다음 `UserInterface` 구현 클래스를 반환합니다. **이 메서드는 어떠한 비밀번호 검증이나 인증도 시도하지 않아야 합니다.**

`validateCredentials` 메서드는 사용자를 인증하기 위해 주어진 `$user`와 `$credentials`을 비교해야 합니다. 예를 들어, 이 메서드는 `$user->getAuthPassword()` 문자열을 `$credentials['password']`의 `Hash::make`와 비교 할 수 있습니다. 이 메서드는 사용자의 자격 증명만을 검증해야하며 참거짓(boolean)을 반환해야 합니다.

이제 우리는 `UserProvider`에 각각의 메서드를 알아 보았습니다, 이제 `Authenticatable`을 살펴봅시다. 기억하세요, 공급자는 `retrieveById`와 `retrieveByCredentials` 메서드로부터 이 인터페이스의 구현 클래스를 반환해야 합니다:

    interface Authenticatable {

        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

이 인터페이스는 간단합니다. `getAuthIdentifier` 메서드는 사용자의 "기본 키"를 반환해야 합니다. MySQL 백엔드에서, 이 키는 자동-증가 기본 키가 될것입니다. `getAuthPassword` 메서드는 사용자의 해시된 비밀번호를 반환해야 합니다. 이 인터페이스는 인증 시스템이 여러분이 사용하고 있는 어떠한 ORM이나 저장 추상화 레이어와 상관없이 모든 사용자 클래스와 동작 할 수 있도록 해줍니다. 기본적으로, 라라벨은 `app` 디렉토리에 이 인터페이스를 구현한 `User` 클래스를 포함하고 있습니다, 그러므로 이 클래스를 구현 예제로 참고 할 수 있습니다.

마지막으로, `UserProvider`를 구현하고 나면, 우리는 `Auth` 파사드와 함께 우리의 확장을 등록할 준비가 끝났습니다:

    Auth::extend('riak', function($app)
    {
        return new RiakUserProvider($app['riak.connection']);
    });

`extend` 메서드로 드라이버를 등록하고 난 뒤, `config/auth.php` 설정 파일에서 새로운 드라이버로 전환 할 수 있습니다.

<a name="container-based-extension"></a>
## 서비스 컨테이너 기반의 확장

라라벨 프레임워크에 포함되어있는 거의 모든 서비스 공급자는 객체들을 서비스 컨테이너로 바인딩 합니다. `config/app.php` 설정 파일에서 어플리케이션의 서비스 공급자 목록을 찾을 수 있습니다. 시간이 있을 때 이 공급자들의 소스 코드를 대강 훑어 보는게 좋습니다. 그러면, 여러분은 각각의 공급자가 프레임워크에 무엇을 추가하는지 좀더 나은 이해를 할 수 있게 되며 서비스 컨테이너에 다양한 서비스들을 바인딩 할 때 어떤 키들이 사용 되었는지도 이해 할 수 있게 됩니다.

예를 들면, `HashServiceProvider`는 서비스 컨테이너에 `Illuminate\Hashing\BcryptHasher` 인스턴스로 해결하는 `hash` 키를 바인딩 합니다. 여러분은 이 바인딩을 재정의 함으로써 여러분만의 어플리케이션에서 이 클래스를 쉽게 확장하고 재정의 할 수 있습니다. 예를 들면:

    <?php namespace App\Providers;

    class SnappyHashProvider extends \Illuminate\Hashing\HashServiceProvider {

        public function boot()
        {
            parent::boot();

            $this->app->bindShared('hash', function()
            {
                return new \Snappy\Hashing\ScryptHasher;
            });
        }

    }

이 클래스는 기본 `ServiceProvider` 클래스가 아닌, `HashServiceProvider` 클래스를 확장 했다는 것을 명심하세요. 서비스 공급자를 확장하고 나면, `config/app.php` 설정 파일에서 `HashServiceProvider`를 여러분이 확장한 공급자로 교체하세요.

이는 컨테이너로 바인딩 되는 아무런 코어 클래스를 확장하는 기본 메서드 입니다. 모든 코어 클래스는 반드시 이러한 방식으로 컨테이너에 바인딩 되며, 재정의 될 수 있습니다. 다시 말하지만, 포함된 프레임워크 서비스 공급자들을 쭉 훑어 보는게 다양한 클래스들이 어디에서 컨테이너로 바인딩 되었는지 그리고 어떤 키들로 바인딩 되었는지 익숙하도록 해줍니다. 이는 라라벨이 어떻게 구성 되었는지 좀 더 배울 수 있는 좋은 방법입니다.
