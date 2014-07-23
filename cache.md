# 캐시

- [설정](#configuration)
- [캐시 사용법](#cache-usage)
- [증가 & 감소](#increments-and-decrements)
- [캐시 섹션](#cache-sections)
- [데이터베이스 캐시](#database-cache)

<a name="configuration"></a>
## 설정

Laravel은 다양한 캐시 시스템에 대한 통합 된 API를 제공합니다. 캐시 설정은 `app/config/cache.php` 파일에 있습니다. 이 파일에서 어플리케이션이 어떤 캐시 드라이버를 자동으로 사용할 지 지정할 수 있습니다. Laravel은 [Memcached](http://memcached.org)나 [Redis](http://redis.io)같이 인기있는 백엔드 캐시를 즉시 사용할 수 있도록 지원합니다.

또한 캐시 설정 파일은 문서화 되어있는 다양한 옵션을 포함하고 있으므로 이 옵션들을 꼼꼼히 읽어보시길 바랍니다. 기본적으로, Laravel은 시리얼라이즈된 캐시 오브젝트를 파일시스템에 저장하는 `file` 캐시 드라이버를 사용 하도록 설정되어 있습니다. 규모가 큰 어플리케이션의 경우, Memcached나 APC등의 메모리 캐시를 사용 할 것을 권장합니다.

<a name="cache-usage"></a>
## 캐시 사용법

**캐시에 아이템 저장**

	Cache::put('key', 'value', $minutes);

**아이템이 캐시에 존재하지 않을 경우 저장**

	Cache::add('key', 'value', $minutes);

**캐시에 존재하는지 확인**

	if (Cache::has('key'))
	{
		//
	}

**캐시에서 아이템 조회**

	$value = Cache::get('key');

**아이템을 조회하거나 기본 값을 반환**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**캐시에 아이템을 영구히 저장**

	Cache::forever('key', 'value');

때때로 캐시에서 아이템을 조회할 수도 있지만 만약 요청한 아이템이 없을 경우 기본 값을 저장할 수도 있습니다. `Cache::remember` 메소드를 사용 하여 그렇게 할 수 있습니다.:

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

또한 `remember` 메소드와 `forever` 메소드를 혼합 할 수도 있습니다.:

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

캐시에 저장되는 모든 아이템은 시리얼라이즈 되므로 어떠한 종류의 데이터를 저장해도 상관없습니다.

**캐시에서 아이템 제거**

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## 증가 & 감소

`file`과 `database`를 제외한 모든 드라이버는 `증가`와 `감소` 연산을 제공합니다.:

**값을 증가**

	Cache::increment('key');

	Cache::increment('key', $amount);

**값을 감소**

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-sections"></a>
## 캐시 섹션

> **노트:** `file` 또는 `database` 캐시 드라이버에서는 캐시 세션이 제공되지 않습니다.

캐시 섹션은 관련된 아이템들을 캐시에 그룹화 해주며 섹션 전체를 지울 수 있게 해줍니다. `section` 메소드를 사용하여 섹션을 액세스 할 수 있습니다.:

**캐시 섹션 액세스**

	Cache::section('people')->put('John', $john);

	Cache::section('people')->put('Anne', $anne);

섹션에 캐시된 아이템을 액세스 하는 것도 가능하며, `increment` 와 `decrement` 같은 다른 캐시 메소드 또한 사용할 수 있습니다.:

** 캐시 섹션에 있는 아이템 액세스**

	$anne = Cache::section('people')->get('Anne');

그리고 섹션에 있는 모든 아이템 제거:

	Cache::section('people')->flush();

<a name="database-cache"></a>
## 데이터베이스 캐시

`database` 캐시 드라이버로 사용 할 경우, 캐쉬 아이템을 저장 할 테이블을 셋업해야 합니다. 아래 코드는 테이블 `Schema` 선언문의 예제 입니다.:

	Schema::create('cache', function($t)
	{
		$t->string('key')->unique();
		$t->text('value');
		$t->integer('expiration');
	});
