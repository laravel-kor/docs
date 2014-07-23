# IoC Container

- [소개](#introduction)
- [기본 사용법](#basic-usage)
- [자동 레졸루션](#automatic-resolution)
- [실용적인 사용법](#practical-usage)
- [서비스 프로바이더](#service-providers)
- [컨테이너 이벤트](#container-events)

<a name="introduction"></a>
## 소개

Laravel의 Ioc 컨테이너(inversion of control container: 제어 컨테이너의 도치[순서바꿈])는 클래스 의존성을 관리하는 강력한 도구 입니다. 의존성 삽입(Dependency injection)은 하드코딩된 클래스간 의존성을 제거하는 하나의 방법입니다. 대신에 클래스의 의존성은 런타임시에 삽입되어 강력한 유연성을 제공하게 되고, 이로 인해 의존성 구현이 쉽게 변경될 수 있게 됨을 이야기 합니다.

Laravel 코어 시스템이 그러하듯, IoC 컨테이너를 이해하는 것은 강력하고 더 큰 규모의 Laravel 어플리케이션을 만드는데 핵심입니다.

<a name="basic-usage"></a>
## 기본적인 사용

IoC 컨테이너를 사용하여 의존성을 해결하는데는 두가지 방법이 있습니다. 하나는 클로저 콜백을 사용하는 방법이고, 다른 하나는 자동 레졸루션을 사용하는 방법입니다. 그중 먼저 클로저 콜백을 살펴보겠습니다. 다음은 클로저 콜백으로 컨테이너에 바인딩 하는 방법 입니다.

**클로저 콜백 컨테이너 바인딩 **

	App::bind('foo', function($app)
	{
		return new FooBar;
	});

**사용하는 부분**

	$value = App::make('foo');

`App::make` 메소드가 실행될 때, 클로저 콜백에 의해서 실행된 결과가 리턴됩니다.

위의 예제는 매번 새로운 FooBar 인스턴스를 리턴하는데, 이와는 달리 때로는 컨테이너에 바인딩 된후 서브 시퀀스가 호출 할 때 동일한 인스턴스가 리턴되는 것을 원할 수도 있습니다.

**컨테이너에 공유된 형태로 바인딩**

	App::singleton('foo', function()
	{
		return new FooBar;
	});

또는 이미 존재하는 객체의 인스턴스를 `instance` 메소드를 사용하여 컨테이너에 바인딩 할 수도 있습니다.

**이미 존재하는 인스턴스를 컨테이너에 바인딩 **

	$foo = new Foo;

	App::instance('foo', $foo);

<a name="automatic-resolution"></a>
## 자동 레졸루션

IoC 컨테이너는 특별한 설정없이 클래스 의존성을 알아서 해결 해 줍니다. 예를 들면 :

**클래스의 의존성 해결**

	class FooBar {

		public function __construct(Baz $baz)
		{
			$this->baz = $baz;
		}

	}

	$fooBar = App::make('FooBar');

`FooBar` 클래스를 컨테이너에 등록 하지 않더라도, 자동으로 알아서 의존성을 해줍니다. 심지어 내부의 `Baz` 클래스까지도 자동으로 삽입해주기 때문에 컨테이너는 여전히 클래스 의존성을 고려하지 않아도 됩니다.

이러한 동작은 컨테이너에 바인딩 되어 있지 않더라도, PHP 고유의 리플렉션 기능을 사용하여 클래스를 점검하고 생성자의 타입을 읽어들입니다. 이 정보를 통해서 컨테이너는 자동으로 클래스의 인스턴스를 빌드할 수 있습니다.

때때로 클래스는 인터페이스의 형태로 실제적이지 않을 수도 있습니다. 이경우 `App::bind` 메소드는 어떠한 인터페이스 구현이 삽입되는지 알려줍니다. 다음의 예를 보면

**인터페이스에 실제 구현된 클래스를 바인딩**

	App::bind('UserRepositoryInterface', 'DbUserRepository');

컨트롤러에서 사용된 형태:

	class UserController extends BaseController {

		public function __construct(UserRepositoryInterface $users)
		{
			$this->users = $users;
		}

	}

`UserRepositoryInterface`가 `DbUserRepository` 의 형태로 바인딩 되었기 때문에 `UserController`가 생성될때 자동으로 `DbUserRepository`가 삽입됩니다.

<a name="practical-usage"></a>
## 실제 사용

Laravel은 IoC 컨테이너를 사용하여 어플리케이션을 유연해지고 또한 테스트가 용이하도록 만듭니다. 하나의 주요한 예는 컨트롤로들의 경우 입니다. 모든 컨트롤러들는 IoC 컨테이너를 통해서 의존성을 해결하는데, 이것은 컨트롤러 생성자 안에서 의존성 유형을 알아 내고 이러한 의존성이 자동으로 삽입됨을 의미합니다.

**의존성 유형을 알아내는 컨트롤러 **

	class OrderController extends BaseController {

		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		public function getIndex()
		{
			$all = $this->orders->all();

			return View::make('orders', compact('all'));
		}

	}

이 예제에서 `OrderRepository` 클래스는 자동으로 컨트롤러 안에서 삽입됩니다. 이것은 [유닛 테스트](/docs/testing)가 수행될때 모의 `OrderRepository`가 컨테이너에 바인딩될 수 있고, 컨트롤러에 삽입되어 어렵지 않게 데이타베이스 레이어 상호 작용의 스터빙을 가능하게 합니다.

(번역주 : 스터빙 - 하위 모듈의 상세한 구조에도 불구하고 그 입출력 조건을 모의하기 위한 모듈로, 톱 다운으로 행하는 설계나 테스트에 있어서 어느 추상화 레벨에서의 기술(記述)을 위해 불필요한 하위 모듈 내부의 기술을 없애기 위해 사용하는 것)

[필터](/docs/routing#route-filters), [컴포저](/docs/responses#view-composers), 그리고 [이벤트 핸들러](/docs/events#using-classes-as-listeners) 또한 IoC 컨트롤에 의해서 의존성 해결을 할 수 있습니다. 그것들을 등록할 때. 쉽게 사용되어져야 하는 클래스의 이름을 알려 줄 수 있습니다.

**Ioc 를 사용하는 다른 방법들**

	Route::filter('foo', 'FooFilter');

	View::composer('foo', 'FooComposer');

	Event::listen('foo', 'FooHandler');

<a name="service-providers"></a>
## 서비스 프로바이더

서비스 프로바이더는 관계된 IoC 등록을 한곳에서 모을 수 있는 좋은 방법입니다. 그것들을 어플리케이션의 부트스트랩 컴포넌트들을 구성하는 방법으로 생각해보세요. 서비스 프로바이더 내에서, 사용자 정의 인증 드라이버 등록, 어플리케이션의 IoC 컨테이너 저장소 클래스들을 등록, 또는 사용자 정의 아티즌 커맨드를 등록 할 수도 있습니다.

실제로 대부분은 Laravel 코어 컴포넌트들은 서비스 프로바이더를 포함하고 있습니다. 어플리케이션에 등록된 서비스 프로바이더들은 `app/config/app.php` 설정 파일의 'Providers' 배열에서 확인할 수 있습니다.

간단한 서비스 프로바이더를 새롭게 생성하기 위해서는 `Illuminate\Support\ServiceProvider` 클래스를 상속하고 'register' 메소드를 정의 하면 됩니다.

**서비스 프로바이더 정의 하기 **

	use Illuminate\Support\ServiceProvider;

	class FooServiceProvider extends ServiceProvider {

		public function register()
		{
			$this->app->bind('foo', function()
			{
				return new Foo;
			});
		}

	}

`register` 메소드 안에서 어플리케이션 IoC 컨테이너는 `$this->app`을 통해서 사용이 가능합니다. 일단 당신이 프로바이더를 생성하고 응용 프로그램에 등록 할 준비가 되었다면 간단하게 `app` 설정 파일의 '프로바이더' 배열에 추가하면 됩니다.

`App::register` 메소드를 사용하여 런타임때 서비스 프로바이더를 등록 할 수도 있습니다.:

**런타임때 서비스 프로바이더 등록**

	App::register('FooServiceProvider');

<a name="container-events"></a>
## 컨테이너 이벤트

컨테이너는 오브젝트를 해결 할때마다, 이벤트를 발생시킵니다. `resolving` 메소드를 사용하여 이 이벤트를 listen 할 수 있습니다.:

**해결 리스너 등록**

	App::resolving(function($object)
	{
		//
	});

의존성이 해결된 오브젝트가 콜백으로 전달된다는 것을 참고하시기 바랍니다.