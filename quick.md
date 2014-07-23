# 라라벨 빠른 시작

- [설치](#installation)
- [라우팅](#routing)
- [뷰 생성](#creating-a-view)
- [마이그레이션 생성](#creating-a-migration)
- [엘로퀀트 ORM](#eloquent-orm)
- [데이터 디스플레이](#displaying-data)

<a name="installation"></a>
## 설치

라라벨 프레임워크를 설치하려면, 터미널에서 다음의 커맨드를 실행하면 됩니다.:

	composer create-project laravel/laravel your-project-name --prefer-dist

또는 [Github 저장소](https://github.com/laravel/laravel/archive/master.zip)의 복사본을 다운로드합니다. 다음, [컴포저를 설치](http://getcomposer.org)한 다음, 자신의 프로젝트 디렉토리에서 `composer install` 커맨드를 실행합니다. 이 커맨드는 프레임워크의 의존성들을 다운로드하고 설치합니다.

프레임워크를 설치하고 난 다음, 디렉토리 구조와 친숙해 질수 있도록 한번 훑어 보시기 바랍니다. `app` 디렉토리는 `views`, `controllers`, `models`과 같은 폴더들을 포함하고 있습니다. 어플리케이션 대부분의 코드가 이 디렉토리에 속해 있습니다. 또한 `app/config` 디렉토리와 이용가능한 설정 옵션들을 살펴보십시오.

<a name="routing"></a>
## 라우팅

시작하기에 앞서, 우리의 첫번째 라우트를 만들어 봅시다. 라라벨에서, 가장 간단한 라우트는 클로저로의 라우트 입니다. `app/routes.php` 파일을 열어 하단에 다음의 라우트를 추가합니다.:

	Route::get('users', function()
	{
		return 'Users!';
	});

이제 웹 브라우저에서 `/users`를 치면 `Users!`가 응답으로 디스플레이된것을 볼수 있을 겁니다. 좋습니다! 방금 첫번째 라우트를 만들었습니다.

라우트는 또한 컨트롤러 클래스로 부여 될 수도 있습니다. 예를 들면:

	Route::get('users', 'UserController@getIndex');

이 라우트는 `/users` 라우트가 `UserController` 클래스의 `getIndex` 메소드를 호출해야 한다고 프레임워크에게 알려줍니다. 컨트롤러 라우팅에 대한 더 많은 정보는 [컨트롤러 매뉴얼](/docs/controllers)에서 확인하세요.

<a name="creating-a-view"></a>
## 뷰 생성

다음, 사용자 데이터를 디스플레이하는 간단한 뷰를 만들겁니다. 뷰는 `app/views` 디렉토리에 있으며 어플리케이션의 HTML을 포함하고 있습니다. 우리는 이 디렉토리에 2개의 새로운 뷰 `layout.blade.php`와 `users.blade.php`를 만들겁니다. 첫번째로, `layout.blade.php` 파일을 만들어봅시다.:

	<html>
		<body>
			<h1>Laravel Quickstart</h1>

			@yield('content')
		</body>
	</html>

다음 `users.blade.php` 뷰를 만듭니다.:

	@extends('layout')

	@section('content')
		Users!
	@stop

이 구문 중 몇몇은 아마도 낯설게 보일겁니다. 여기서 우리는 라라벨의 템플릿 시스템 "블레이드"를 사용하고 있기 때문입니다. 블레이드는 템플릿을 순수한 PHP로 컴파일 해주는 몇개의 정규표현식 집합이므로 굉장히 빠릅니다. 블레이드는 `if`나 `for`같은 전형적인 PHP 컨트롤 구조는 물론, 템플릿 상속과 같은 강력한 기능도 제공합니다. 좀 더 자세히 알려면 [블레이드 매뉴얼](/docs/templates)을 확인하세요.

이제 우리는 뷰를 갖고 있습니다. 뷰를 우리의 `/users` 라우트에서 반환해 봅시다. 라우트에서 `Users!`를 반환하는 것 대신, 뷰를 반환하세요:

	Route::get('users', function()
	{
		return View::make('users');
	});

멋집니다! 이제 레이아웃을 확장하는 간단한 뷰를 설정했습니다. 다음으로, 우리의 데이터베이스 레이어에 대해 시작해 봅시다.

<a name="creating-a-migration"></a>
## 마이그레이션 생성

우리의 데이터를 저장할 테이블을 생성하기 위해, 우리는 라라벨의 마이그레이션 시스템을 사용할겁니다. 마이그레이션은 데이터베이스에 표현력있는 수식을 정의할 수 있도록 해주며, 당신의 나머지 팀원들이 쉽게 공유 할 수 있도록 해줍니다.

첫번째로, 데이터베이스 커넥션을 설정합시다. `app/config/database.php` 파일에서 당신의 모든 데이터베이스 커넥션을 설정 할 수 있습니다. 기본적으로 라라벨은 MySQL을 사용하도록 설정되어있으며, 데이터베이스 설정 파일에서 커넥션 자격을 제공해야 합니다. 만약 원한다면, `driver` 옵션을 `sqlite`로 변경하면 `app/database` 디렉토리에 포함되어있는 SQLite 데이터베이스를 사용하게 됩니다.

다음, 마이그레이션을 생성하기위해 [아티즌 CLI](/docs/artisan)를 사용합니다. 터미널의 프로젝트 루트에서 다음을 실행하세요:

	php artisan migrate:make create_users_table

그런 다음, `app/database/migrations` 폴더에서 생성된 마이그레이션 파일을 찾으세요. 이 파일은 `up`과 `down` 2가지 메소드를 갖고 있는 클래스를 포함하고 있습니다. `up` 메소드에서는 데이터베이스 테이블의 원하는 변경을 만들고, `down` 메소드에는 그것들을 거꾸로 실행합니다.

아래와 같이 생긴 마이그레이션을 정의해봅시다.:

	public function up()
	{
		Schema::create('users', function($table)
		{
			$table->increments('id');
			$table->string('email')->unique();
			$table->string('name');
			$table->timestamps();
		});
	}

	public function down()
	{
		Schema::drop('users');
	}

그런 다음, 터미널에서 `migrate` 커맨드를 사용하여 마이그레이션을 실행할 수 있습니다. 자신의 프로젝트 루트에서 간단히 다음의 커맨드를 실행하세요.:

	php artisan migrate

만약 마이그레이션을 롤백하길 원한다면, `migrate:rollback` 커맨드를 사용합니다. 이제 우리는 데이터베이스 테이블을 생성했습니다. 데이터를 입력해봅시다!

<a name="eloquent-orm"></a>
## 엘로퀀트 ORM

라라벨은 훌륭한 ORM 엘로퀀트를 제공하고 있습니다. 만약 Ruby on Rails 프레임워크를 사용해본적이 있다면, 엘로퀀트가 데이터베이스 상호작용에 액티브레코드 ORM을 따르므로 다루는데 친숙할겁니다.

먼저, 모델을 정의해 봅시다. 엘로퀀트 모델은 관련된 데이터베이스 테이블을 쿼리하는데 사용될뿐만 아니라, 테이블에 속하는 지정된 행까지 표시 할 수 있습니다. 걱정마세요, 이 모든것들이 곧 이해가 될겁니다! 모델들은 일반적으로 `app/models` 디렉토리에 저장됩니다. 이 디렉토리안에 다음의 `User.php` 모델을 정의해 봅시다.:

	class User extends Eloquent {}

엘로퀀트에게 모델이 어떤 테이블을 사용해야 하는지 알려줄 필요가 없다는걸 알아두세요. 엘로퀀트는 다양한 컨벤션을 갖고 있으며, 그 중 하나는 모델명의 복수형을 데이터베이스 테이블로 사용하는겁니다. 편리하지요!

즐겨쓰는 데이터베이스 관리 도구를 사용하여 `users` 테이블에 몇개의 레코드를 추가하면, 엘로퀀트를 사용하여 그것들을 조회하고 우리의 뷰에 전달 해볼겁니다.

이제 우리의 `/users` 라우트를 아래와 같이 수정합니다.:

	Route::get('users', function()
	{
		$users = User::all();

		return View::make('users')->with('users', $users);
	});

이 라우트를 하나씩 살펴봅시다. 먼저, `User` 모델의 `all` 메소드는 `users` 테이블의 모든 레코드를 조회합니다. 다음, `with` 메소드를 통해 레코드를 뷰에 전달합니다. `with` 메소드는 키와 값을 매개 변수로 받으며, 뷰에서 사용가능한 데이터로 만드는데 사용됩니다.

쩌네요. 이제 우리의 뷰에서 사용자(users)를 디스플레이할 준비가 되었습니다!

<a name="displaying-data"></a>
## 데이터 디스플레이

이제 우리의 뷰에서 사용할 수 있는 `users`를 만들었고, 다음과 같이 디스플레이 할 수 있습니다.:

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

아마 `echo` 구문이 어디에 있는지 궁금해 할지도 모릅니다. 블레이드를 사용하면, 중괄호 2개를 데이터에 둘러싸서 출력할 수 있습니다. 식은죽 먹기죠. 이제, 브라우저에 `/users` 라우트를 치면 사용자의 이름들을 응답으로 볼수 있습니다.

이것은 단지 시작일 뿐 입니다. 이 튜토리얼에서, 당신은 라라벨의 가장 기본적인 것들만 보았습니다. 그리고 아주 흥미로운 것들이 더 많이 있습니다. 매뉴얼을 계속해서 읽어보면서, [엘로퀀트](/docs/eloquent)와 [블레이드](/docs/templates)의 강력한 기능들을 더 깊게 파헤쳐 보세요. 또는, [큐](/docs/queues)와 [유닛 테스팅](/docs/testing)을 좀더 흥미로워 할수도 있습니다. 한편으로는, [IoC 컨테이너](/docs/ioc)를 사용하여 아키텍쳐를 좀더 부드럽게 만들 수도 있습니다. 선택은 당신의 것입니다!
