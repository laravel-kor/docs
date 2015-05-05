# 캐시

- [구성](#configuration)
- [캐시 사용법](#cache-usage)
- [증가 & 감소](#increments-and-decrements)
- [캐시 태그](#cache-tags)
- [캐시 이벤트](#cache-events)
- [데이터베이스 캐시](#database-cache)
- [Memcached 캐시](#memcached-cache)
- [Redis 캐시](#redis-cache)

<a name="configuration"></a>
## 설정

Laravel은 다양한 캐시 시스템에 대한 통합 된 API를 제공합니다. 캐시 구성은 `app/config/cache.php` 파일에 있습니다. 이 파일에서 어플리케이션이 어떤 캐시 드라이버를 기본값으로 사용할 지 지정할 수 있습니다. Laravel은 [Memcached](http://memcached.org)나 [Redis](http://redis.io)같이 인기있는 백 엔드 캐시를 즉시 사용할 수 있도록 지원합니다.

또한 캐시 구성 파일은 문서화 되어있는 다양한 옵션들을 포함하고 있으므로 이 옵션들을 꼼꼼히 읽어보시길 바랍니다. 기본적으로, Laravel은 시리얼라이즈된 캐시 오브젝트를 파일시스템에 저장하는 `file` 캐시 드라이버를 사용 하도록 설정되어 있습니다. 규모가 큰 어플리케이션의 경우, Memcached나 APC등의 메모리 캐시를 사용 할 것을 권장합니다. 또한 동일한 드라이버에 여러개의 캐쉬 구성을 설정 할 수도 있습니다.

Redis 캐쉬를 라라벨과 사용하려면, 컴포저를 통하여 `predis/predis` 패키지(~1.0)를 설치해야합니다.

<a name="cache-usage"></a>
## 캐시 사용법

#### 캐시에 아이템 저장

    Cache::put('key', 'value', $minutes);

#### Carbon 개체를 사용하여 만료 시간 설정

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### 아이템이 캐시에 존재하지 않을 경우 해당 아이템 저장

    Cache::add('key', 'value', $minutes);

아이템이 실제로 캐시에 **추가** 되었을 경우, `add` 메서드는 `true`를 반환합니다. 그렇지 않을 경우, 해당 메서드는 `false`를 반환합니다.

#### 캐시가 존재하는지 확인**

    if (Cache::has('key'))
    {
        //
    }

#### 캐시에서 아이템 조회

    $value = Cache::get('key');

#### 아이템을 조회 하거나 기본값을 반환

    $value = Cache::get('key', 'default');

    $value = Cache::get('key', function() { return 'default'; });

#### 캐시에 아이템을 영구히 저장

    Cache::forever('key', 'value');

때때로 캐시에서 아이템을 조회하고 싶지만, 만약 요청한 아이템이 없을 경우 기본 값을 저장할 수도 있습니다. `Cache::remember` 메서드를 사용 하면 됩니다:

    $value = Cache::remember('users', $minutes, function()
    {
        return DB::table('users')->get();
    });

또한 `remember` 메서드와 `forever` 메서드를 혼합 할 수도 있습니다:

    $value = Cache::rememberForever('users', function()
    {
        return DB::table('users')->get();
    });

캐시에 저장되는 모든 아이템은 시리얼라이즈 되므로 어떠한 종류의 데이터를 저장해도 상관없습니다.

#### 캐시에서 아이템 꺼내기

캐시에서 아이템을 조회하고, 해당 캐시를 삭제 하려면, `pull` 메서드를 사용합니다:

    $value = Cache::pull('key');

#### 캐시에서 아이템 제거

    Cache::forget('key');

#### 지정된 캐쉬 저장소에 엑세스

여러개의 캐쉬 저장소를 사용한다면, `store` 메서드를 통하여 액세스 할 수 있습니다:

    $value = Cache::store('foo')->get('key');

<a name="increments-and-decrements"></a>
## 증가 & 감소

`database`를 제외한 모든 드라이버가 `증가`와 `감소` 연산을 제공합니다:

#### 값을 증가

    Cache::increment('key');

    Cache::increment('key', $amount);

#### 값을 감소

    Cache::decrement('key');

    Cache::decrement('key', $amount);

<a name="cache-tags"></a>
## 캐시 태그

> **주의:** `file` 또는 `database` 캐시 드라이버를 사용할 때에는 캐시 태그가 지원되지 않습니다. 게다가, "forever"에 저장된 캐시에 다양한 태그를 사용할 때에는, 자동으로 오래된 레코드들을 제거하는 `memcached` 같은 드라이버가 가장 좋습니다.

#### 태그된 캐시 액세스

캐시 태그들은 캐시 내에서 연관된 아이템끼리 태그 할 수 있도록 해주며, 주어진 태그명을 가진 모든 캐시를 제거 할 수도 있게 해줍니다. 태그된 캐시를 액세스 하려면 `tags` 메서드를 사용합니다.

정렬된 태그명 목록이나, 배열을 인수로 전달하여 태그된 캐시를 저장 할 수 있습니다:

    Cache::tags('people', 'authors')->put('John', $john, $minutes);

    Cache::tags(['people', 'artists'])->put('Anne', $anne, $minutes);

`remember`, `forever`, 그리고 `rememberForever` 메서드들같이 어떠한 캐쉬 저장 메서드도 tags 메서드와 같이 사용 될 수 있습니다. 또한 `increment`와 `decrement`같이 다른 캐쉬 메서드들을 사용해서도 태그된 캐시 아이템을 액세스 할 수 있습니다.

#### 태그된 캐시 내에서 아이템 액세스

태그된 캐시를 액세스 하려면, 저장할 때 쓰였던 태그 목록을 같은 순서로 전달 합니다.

    $anne = Cache::tags('people', 'artists')->get('Anne');

    $john = Cache::tags(['people', 'authors'])->get('John');

특정 이름 또는 목록으로 태그된 아이템을 모두 제거 할 수도 있습니다. 예를 들어, 아래문은 `people`과 `authors` 또는 두가지 모두 태그된 모든 캐시들을 제거합니다. 그러니, "Anne"과 "John" 모두 캐시에서 제거됩니다:

    Cache::tags('people', 'authors')->flush();

반면에, 아래문은 `authors`로 태그된 캐시들만 제거합니다. 그러므로 "John"은 제거되지만, "Anne"은 제거되지 않습니다.

    Cache::tags('authors')->flush();

<a name="cache-events"></a>
## 캐시 이벤트

캐시가 동작 할 때마다 코드를 실행하려면, 캐쉬에 의해 실행되는 이벤트를 수신 하면 됩니다:

    Event::listen('cache.hit', function($key, $value) {
        //
    });

    Event::listen('cache.missed', function($key) {
        //
    });

    Event::listen('cache.write', function($key, $value, $minutes) {
        //
    });

    Event::listen('cache.delete', function($key) {
        //
    });

<a name="database-cache"></a>
## 데이터베이스 캐시

`database` 캐시 드라이버를 사용 할 경우, 캐쉬 아이템을 저장 할 테이블을 셋업해야 합니다. 아래 코드에서 캐시 테이블의 예제 `Schema` 선언문을 볼 수 있습니다:

    Schema::create('cache', function($t)
    {
        $t->string('key')->unique();
        $t->text('value');
        $t->integer('expiration');
    });

<a name="memcached-cache"></a>
#### Memcached 캐시

Memcached 캐시를 사용하려면 [Memcached PECL 패키지](http://pecl.php.net/package/memcached)가 설치되어 있어야 합니다.

기본 [구성](#configuration)은 [Memcached::addServer](http://php.net/manual/en/memcached.addserver.php)의 TCP/IP를 사용합니다:

    'memcached' => array(
        array('host' => '127.0.0.1', 'port' => 11211, 'weight' => 100),
    ),

여러분은 또한 UNIX 소켓 경로에 `host` 옵션을 설정 할 수도 있습니다. 이렇게 사용 하려면, `port` 옵션은 `0`으로 설정 되어야 합니다:

    'memcached' => array(
        array('host' => '/var/run/memcached/memcached.sock', 'port' => 0, 'weight' => 100),
    ),

<a name="redis-cache"></a>
#### Redis 케시

[Redis 구성](/docs/redis#configuration)을 보세요.
