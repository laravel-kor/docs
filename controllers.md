# 컨트롤러

- [기본 컨트롤러](#basic-controllers)
- [컨트롤러 필터](#controller-filters)
- [RESTful 컨트롤러](#restful-controllers)
- [리소스 컨트롤러](#resource-controllers)
- [누락된 메소드 처리](#handling-missing-methods)

<a name="basic-controllers"></a>
## 기본 컨트롤러

하나의 `routes.php` 파일에 모든 라우팅 로직을 정의하는 것 대신, 컨트롤러 클래스를 사용하여 구조화 하길 원할 수도 있습니다. 컨트롤러는 관련된 로직들을 하나의 클래스로 그룹화 할수 있고, 자동 [의존성 주입(dependency injection)](/docs/ioc)같이 좀 더 고급된 프레임워크 기능을 활용 할 수 있습니다.

컨트롤러는 일반적으로 `app/controllers` 디렉토리에 저장되며, 이 디렉토리는 `composer.json` 파일의 `classmap` 옵션에 자동으로 등록되어 있습니다.

기본 컨트롤러 클래스의 예:

	class UserController extends BaseController {

		/**
		 * 특정 유저의 프로필 표시.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

모든 컨트롤러는 `BaseController` 클래스를 확장 해야 합니다. `BaseController` 또한 `app/controllers` 디렉토리에 저장되어 있으며, 공유되는 컨트롤러 로직을 보관하는 장소로 사용 됩니다. `BaseController`는 프레임워크의 `Controller` 클래스를 확장합니다. 이제, 아래와 같이 컨트롤러 액션으로 라우팅 할수 있습니다.:

	Route::get('user/{id}', 'UserController@showProfile');

PHP 네임스페이스를 사용하여 컨트롤러를 구조화 하거나 네스팅 해야 한다면, 라우트을 정의할 때 간단히 클래스의 전체주소를 사용하면 됩니다.:

	Route::get('foo', 'Namespace\FooController@method');

또한 컨트롤러 라우트에 이름을 지정 할 수도 있습니다.:

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

`URL::action` 메소드르 사용하여 컨트롤러 액션에 해당하는 URL을 생성할 수 잇습니다.:

	$url = URL::action('FooController@method');

`currentRouteAction` 메소드를 사용하여 실행되고 있는 컨트롤러 액션 명을 액세스 할 수 있습니다.:

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## 컨트롤러 필터

[필터](/docs/routing#route-filters)는 "일반" 라우팅과 비슷하게 컨트롤러 라우트로 지정될 수 있습니다.:

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

그러나, 컨트롤러 안에서도 필터를 지정 할수 있습니다.:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth');

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

클로저를 사용하여 인라인으로 컨트롤러 필터를 지정 할 수도 있습니다.:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

<a name="restful-controllers"></a>
## RESTful 컨트롤러

Laravel은 간단한 REST 네이밍 규칙을 사용하여 쉽게 하나의 라우트로 컨트롤러의 모든 액션을 처리 할 수있도록 해줍니다. 첫번째로, `Route::controller` 메소드를 사용하여 라우트를 정의 합니다.:

**RESTful 컨트롤러 정의**

	Route::controller('users', 'UserController');

The `controller` 메소드는 두개의 인수를 받습니다. 첫번째는 컨트롤러가 처리하는 기본 URI고 두번째는 컨트롤러의 이름입니다. 그런 다음, 컨트롤러에 HTTP 동사(get, post, update 등)에 응답하는 HTTP 동사로 시작하는 메소드를 추가하면 됩니다.:

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

`index` 메소드는 컨트롤에 의해 처리되는 루트 URI (이 경우 `users`) 에 응답합니다.

만약 컨트롤러 액션이 여러개의 단어를 포함하고 있다면, URI에 "대시"를 사용하여 액세스 할 수 있습니다. 예를 들어, `UserController` 컨트롤러에 있는 아래의 액션은 `users/admin-profile` URI에 응답 합니다.:

	public function getAdminProfile() {}

<a name="resource-controllers"></a>
## 리소스 컨트롤러

리소스 컨트롤러는 리소스에 맞춰 쉽게 RESTful 컨트롤러를 구축할수 있게 합니다. 예를 들어, 어플리케이션에 저장되어 있는 "사진"을 관리하는 컨트롤러를 만든다고 합니다. Artisan CLI를 통한 `controller:make` 커맨드와 `Route::resource` 메소드를 사용하여 빠르게 컨트롤러를 생성할 수 있습니다.

커맨드 라인을 통해 컨트롤러를 생성하려면 아래와 같은 커맨드를 실행합니다:

	php artisan controller:make PhotoController

이제 리소스 라우트를 컨트롤러에 등록합니다:

	Route::resource('photo', 'PhotoController');

이 한 줄의 라우트 선언이 사진 리소스에 다양한 RESTful 액션을 처리하기 위해 여러개의 라우트를 생성합니다. 마찬가지로, 생성된 컨트롤러는 이미 각각의 URI와 동사(get, post, update 등)를 처리하는 뼈대 메소드들을 갖고 있습니다.

**리소스 컨트롤러에 의해 처리되는 액션**

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | /resource             | index        | resource.index
GET       | /resource/create      | create       | resource.create
POST      | /resource             | store        | resource.store
GET       | /resource/{id}        | show         | resource.show
GET       | /resource/{id}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{id}        | update       | resource.update
DELETE    | /resource/{id}        | destroy      | resource.destroy

때때로 리소스 액션의 일부분만 필요할 때가 있습니다.:

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

그리고, 라우트에서 액션의 일부분만 처리하게 지정 할 수도 있습니다.:

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));

<a name="handling-missing-methods"></a>
## 누락된 메소드 처리

주어진 컨트롤러에 해당하는 메소드가 없을 경우 호출되는 포괄적인 메소드를 정의 할 수 있습니다. 메소드 이름은 `missingMethod`이며 인수로는 매개 변수 배열 한개만 받습니다.:

**포괄적인 메소드 정의**

	public function missingMethod($parameters)
	{
		//
	}