# 세션

- [설정](#configuration)
- [세션 사용법](#session-usage)
- [플래시 데이터](#flash-data)
- [데이터베이스 세션](#database-sessions)
- [세션 드라이버](#session-drivers)

<a name="configuration"></a>
## 설정

HTTP 기반 어플리케이션은 비보존형이므로, 세션은 사용자 크로스 요청에 대한 정보를 저장하는 방법을 제공합니다. Laravel은 다양한 백엔드 세션을 깔끔하고, 톱합된 AIP를 통해 사용할 수 있도록해줍니다. [Memcached](http://memcached.org), [Redis](http://redis.io) 같은 유명한 백엔드와 데이터베이스가 기본적으로 포함되어 있습니다.

세션 설정은  `app/config/session.php`에 저장되어 있습니다. 이 파일안에 있는 잘 정리된 옵셥들을 꼭 읽어 보시길 바랍니다. 기본적으로 laravel은 대부분의 어플리케이션에서 잘 작동하는 `native` 세션 드라이버를 사용하도록 설정되어 있습니다.

<a name="session-usage"></a>
## 세션 사용법

**세션에 아이템 저장**

    Session::put('key', 'value');

**배열 세션에 값 푸쉬**

	Session::push('user.teams', 'developers');

**세션에서 아이템 조회**

    $value = Session::get('key');

**아이템을 조회하거나 기본 값을 반환**

	$value = Session::get('key', 'default');

	$value = Session::get('key', function() { return 'default'; });

**세션의 모든 데이터 조회**

	$data = Session::all();

**세션에 아이템이 존재하는지 확인**

	if (Session::has('users'))
	{
		//
	}

**세션에서 아이템 제거**

	Session::forget('key');

**세션의 모든 아이템 제거**

	Session::flush();

**세션 아이디를 재생성**

	Session::regenerate();

<a name="flash-data"></a>
## 플래시 데이터

다음 요청에만 사용할 일회용 아이템을 세션에 저장할 수도 있습니다. `Session::flash` 메소드를 사용하면 됩니다.:

	Session::flash('key', 'value');

**현재 플래시 데이터를 다음 요청때 사용하기위해 다시 저장**

	Session::reflash();

**플래시 데이터의 일부만 다시 저장**

	Session::keep(array('username', 'email'));

<a name="database-sessions"></a>
## 데이터베이스 세션

`database` 세션 드라이버를 사용할 경우, 세션 아이템을 포함할 테이블을 만들어야 합니다. 아래는 테이블 `Schema`의 예입니다.:

	Schema::create('sessions', function($table)
	{
		$table->string('id')->unique();
		$table->text('payload');
		$table->integer('last_activity');
	});

물론 `session:table` 아티즌 커맨드를 사용하여 위의 마이그레이션을 생성할 수도 있습니다!

	php artisan session:table

	composer dump-autoload

	php artisan migrate

<a name="session-drivers"></a>
## 세션 드라이버

세션 "드라이버"는 각각의 요청에 대한 세션 데이터가 어디에 저장될지 정의합니다. 라라벨은 기본적으로 여러개의 드라이버를 포함하고 있습니다.:

- `native` - 세션이 PHP의 내장 세션 기능에 의해 처리됩니다.
- `cookie` - 세션이 안전하고, 암호화된 쿠키에 저장됩니다.
- `database` - 세션이 어플리케이션에서 사용하는 데이터베이스에 저장됩니다.
- `memcached` / `redis` - 세션이 빠르고, 캐싱을 기본으로 저장하는 이 중 하나에 저장됩니다.
- `array` - 세션이 간단한 PHP 배열에 저장되며, 이는 요청간에 지속되지 않습니다.

> **노트:** 배열 드라이버는 [유닛 테스팅](/docs/testing)을 할때 주로 사용되므로, 세션 데이터가 지속되지 않습니다.