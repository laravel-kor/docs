# HTTP 컨트롤러

- [소개](#introduction)
- [기본 컨트롤러](#basic-controllers)
- [컨트롤러 미들웨어](#controller-middleware)
- [암시적 컨트롤러](#implicit-controllers)
- [RESTful 리소스 컨트롤러](#restful-resource-controllers)
- [의존성 주입 & 컨트롤러dency-injection-and-controllers)
- [라우트 캐싱](#route-caching)

<a name="introduction"></a>
## 소개

모든 요청처리 로직을 하나의 `routes.php` 파일에서 처리 하도록 정의하는것 대신에, 컨트롤러 클래스를 사용하여 이러한 동작을 구성할 수도 있습니다. 컨트롤러는 연관된 HTTP 요청 처리 로직들을 하나의 클래스로 그룹화 시켜줍니다. 컨트롤러는 기본적으로 `app/Http/Controllers` 디렉토리에 위치해 있습니다.

<a name="basic-controllers"></a>
## 기본 컨트롤러

여기 기본 컨트롤러 클래스의 예제가 있습니다:

    <?php namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;

    class UserController extends Controller {

        /**
         * 주어진 사용자의 프로필 표시
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }

    }

우리는 다음과 같이 컨트롤러 액션에 라우트 할 수 있습니다:

    Route::get('user/{id}', 'UserController@showProfile');

> **주의:** 모든 컨트롤러는 기본 컨트롤러 클래스를 확장해야 합니다.

#### 컨트롤러 & 네임스페이스

컨트롤러의 네임스페이스 전체를 지정 할 필요 없이 "루트" 네임스페이스 `App\Http\Controllers` 뒤에 오는 클래스명 일부분만 지정하는 일은 매우 중요합니다. 기본적으로, `RouteServiceProvider`가 `routes.php` 파일을 루트 컨트롤러 네임스페이스를 포함한 라우트 그룹 안에서 로드 됩니다.

만약 여러분이 PHP 네임스페이스를 사용하여 컨트롤러를 `App\Http\Controllers` 디렉토리보다 안쪽에 구성하려면, 간단하게 루트 네임스페이스 `App\Http\Controllers` 다음에 해당 클래스 명을 지정하세요. 그러면, 만약 전체 클래스명이 `App\Http\Controllers\Photos\AdminController`라면, 다음과 같이 라우트를 등록해야 합니다:

    Route::get('foo', 'Photos\AdminController@method');

#### 컨트롤러 라우트 명명

클로저 라우트처럼, 여러분은 컨트롤러 라우트에 이름을 지정 할 수 있습니다:

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

#### 컨트롤러의 액션으로 보내는 URL

컨트롤러의 액션으로 보내는 URL을 생성하려면 `action` 헬퍼 메서드를 사용합니다:

    $url = action('App\Http\Controllers\FooController@method');

만약 컨트롤러 네임스페이스에 연관된 클래스명의 일부분만 사용하여 URL을 생성하길 원한다면, URL을 생성 하기 전에 루트 컨트롤러 네임스페이스를 등록하세요:

    URL::setRootControllerNamespace('App\Http\Controllers');

    $url = action('FooController@method');

실행되고 있는 컨트롤러의 액션 명을 엑세스 하려면 `currentRouteAction` 메서드를 사용합니다:

    $action = Route::currentRouteAction();

<a name="controller-middleware"></a>
## 컨트롤러 미들웨어

[미들웨어](/docs/5.0/middleware)는 아래와 같이 컨트롤러 라우트에 명시 될 수 있습니다:

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

추가적으로, 컨트롤러의 생성자에도 미들웨어를 명시 할 수 있습니다:

    class UserController extends Controller {

        /**
         * Instantiate a new UserController instance.
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

            $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
        }

    }

<a name="implicit-controllers"></a>
## 암시적 컨트롤러

라라벨은 여러분이 컨트롤러의 모든 액션을 처리 할 수 있는 하나의 라우트를 정의 할 수 있도록 해줍니다. 첫번째로 `Route::controller` 메서드를 사용하여 라우트를 정의하세요:

    Route::controller('users', 'UserController');

`controller` 메서드는 두개의 인수를 받습니다. 첫번째는 컨트롤러 처리의 기본 URI이며, 두번째는 컨트롤러 이름입니다. 그런 다음, 여러분의 컨트롤러에 HTTP 동사가 접두사로 시작하는 메서드를 추가하세요:

    class UserController extends BaseController {

        public function getIndex()
        {
            //
        }

        public function postProfile()
        {
            //
        }

        public function anyLogin()
        {
            //
        }

    }

위의 경우에서 `index` 메서드는 컨트롤러에 의해 처리되는 루트 URI `users`에 응답합니다.

만약 컨트롤러 액션이 여러개의 단어를 포함하고 있다면, URI에서 "대시" 구문을 사용하여 해당 액션에 엑세스 할 수 있습니다. 예를 들어, 아래의 `UserController` 컨트롤러 액션은 `users/admin-profile` URI에 응답합니다:

    public function getAdminProfile() {}

#### 라우트명 부여

만약 컨트롤러의 라우트에 "이름"을 부여하고 싶다면, 컨트롤러 메서드의 새번째 인수에 전달하면 됩니다:

    Route::controller('users', 'UserController', [
        'anyLogin' => 'user.login',
    ]);

<a name="restful-resource-controllers"></a>
## RESTful 리소스 컨트롤러

리소스 컨트롤은 리소스를 중심으로한 RESTful 컨트롤러를 구축 할 때 도움이 됩니다. 예를 들어, "사진"을 저장하는 어플리케이션에 대한 HTTP 요청들을 처리하는 컨트롤러를 만들 수 있습니다. `make:controller` 아티즌 커맨드를 사용하여, 해당 컨트롤러를 빠르게 생성 할 수 있습니다:

    php artisan make:controller PhotoController

다음, 해당 컨트롤러에 리소스 라우트를 등록합니다:

    Route::resource('photo', 'PhotoController');

이 하나의 라우트 선언이 사진 리소스에 다양한 RESTful 액션들을 처리하는 여러 라우트들을 생성합니다. 또한, 생성된 컨트롤러는 이미 각각의 액션에 대한 스터브 메서드와 함께 어떤 URI들과 동사를 처리하는지 알려주는 노트를 포함하고 있습니다.

#### 리소스 컨트롤러에 의해 처리되는 액션들

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | /photo                | index        | photo.index
GET       | /photo/create         | create       | photo.create
POST      | /photo                | store        | photo.store
GET       | /photo/{photo}        | show         | photo.show
GET       | /photo/{photo}/edit   | edit         | photo.edit
PUT/PATCH | /photo/{photo}        | update       | photo.update
DELETE    | /photo/{photo}        | destroy      | photo.destroy

#### 리소스 라우트 사용자 지정

추가적으로, 라우트에서 부분적으로만 사용하여 처리할 액션들을 지정 할 수도 있습니다:

    Route::resource('photo', 'PhotoController',
                    ['only' => ['index', 'show']]);

    Route::resource('photo', 'PhotoController',
                    ['except' => ['create', 'store', 'update', 'destroy']]);

기본적으로, 모든 리소스 컨트롤러 액션은 이름을 갖고 있습니다; 하지만, 옵션에 `names` 배열을 전달하여 이러한 이름을 재정의 할 수 있습니다:

    Route::resource('photo', 'PhotoController',
                    ['names' => ['create' => 'photo.build']]);

#### 중첩된 리소스 컨트롤러 처리

"중첩된" 리소스 컨트롤러는, 라우트 선언에 "도트" 표기법을 사용합니다:

    Route::resource('photos.comments', 'PhotoCommentController');

이 라우트는 `photos/{photos}/comments/{comments}` URL처럼 엑세스 될 수 있는 "중첩된" 리소스를 등록합니다.

    class PhotoCommentController extends Controller {

        /**
         * 지정된 사진의 커멘트 표시
         *
         * @param  int  $photoId
         * @param  int  $commentId
         * @return Response
         */
        public function show($photoId, $commentId)
        {
            //
        }

    }

#### 리소스 컨트롤러에 추가적인 라우트 추가

리소스 컨트롤러의 기본 리소스 라우트 이외에 추가적인 라우트가 필요하게 된다면, 여러분은 `Route::resource`를 호출하기 전에 해당 라우트들을 먼저 정의 해야 합니다:

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## 의존성 주입 & 컨트롤러

#### 생성자 주입

라라벨 [서비스 컨테이너](/docs/5.0/container)는 모든 라라벨 컨트롤러들을 해결하는데 사용됩니다. 결과적으로, 여러분은 컨트롤러가 필요로 하는 어떠한 의존성들이라도 생성자에 타입-힌트 할 수 있습니다:

    <?php namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\UserRepository;

    class UserController extends Controller {

        /**
         * 사용자 리포지토리 인스턴스
         */
        protected $users;

        /**
         * 새로운 컨트롤러 인스턴스 생성
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

    }

물론, 여러분은 아무 [라라벨 계약](/docs/5.0/contracts)들도 타입-힌트 할 수 있습니다. 만약 컨테이너가 무엇인가를 해결할 수 있다면, 여러분은 그것을 타입-힌트 할 수 있습니다.

#### 메서드 주입

생성자 주입과 더불어, 여러분은 또한 컨트롤러의 메서드에도 의존성을 주입 할 수 있습니다. 예를 들어, 메서드 중 하나에 `Request` 인스턴스를 타입-힌트 해봅시다:

    <?php namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller {

        /**
         * 새로운 사용자 저장
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }

    }

만약 여러분의 컨트롤러 메서드가 라우트 매개 변수로부터 입력을 전달 받는다고 생각된다면, 간단히 라우트 인수를 다른 의존성들 뒤에 나열하여 받으세요:

    <?php namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller {

        /**
         * 새로운 사용자 저장
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }

    }

> **주의:** 메서드 주입은 [모델 바인딩](/docs/5.0/routing#route-model-binding)과 완벽하게 호환됩니다. 컨테이너는 어떤 인수들이 모델에 바인딩 되어야하고 어떤 인수들이 주입되어야 하는지 지능적으로 결정합니다.

<a name="route-caching"></a>
## 라우트 캐싱

만약 어플리케이션이 오로지 컨트롤러 라우트들만 사용하고 있다면, 여러분은 라라벨의 라우트 캐시를 활용 할 수 있습니다. 라우트 캐시 사용은 어플리케이션의 모든 라우트를 등록하는데 걸리는 시간을 획기적으로 줄여줍니다. 어떠한 경우에는, 라우트 등록이 최대 100배 빨라질 수도 있습니다! 라우트 캐시를 생성하려면, 단순하게 `route:cache` 아티즌 커맨드를 실행하세요:

    php artisan route:cache

그게 전부입니다! 이제 캐시된 라우트 파일이 여러분의 `app/Http/routes.php` 파일 대신 사용됩니다. 기억하세요, 만약 새로운 라우트가 추가된다면 여러분은 새로운 라우트 캐시를 생성해야 합니다. 이 때문에, 여러분은 아마도 프로젝트를 배포 할 때 `route:cache` 커맨드를 실행하는것이 좋을 수도 있습니다.

새로운 캐시 생성없이 캐시된 라우트 파일을 삭제하려면, `route:clear` 커맨드를 사용합니다:

    php artisan route:clear
