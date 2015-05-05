# 인증

- [소개](#introduction)
- [사용자 인증](#authenticating-users)
- [인증된 사용자 조회](#retrieving-the-authenticated-user)
- [라우트 보호](#protecting-routes)
- [HTTP 기본 인증](#http-basic-authentication)
- [비밀번호 알림 및 초기화](#password-reminders-and-reset)
- [소셜 인증](#social-authentication)

<a name="introduction"></a>
## 소개

라라벨은 아주 쉬운 인증을 구현해줍니다. 사실, 거의 모든것들이 여러분을 위해 기본적으로 구성되어 있습니다. 인증에 필요한 구성 파일은 `config/auth.php`에 위치해 있으며, 인증 서비스의 기능을 조절 할 수 있도록 잘 문서화된 옵션들이 포함되어있습니다.

기본적으로 라라벨은 `app` 디렉토리에 `App\User` 모델을 포함하고 있습니다. 이 모델은 디폴트인 엘로퀀트 인증 드라이버로 사용 될 수 있습니다.

기억하세요: 이 모델에 대한 데이터베이스 스키마를 작성 할때, 비밀번호 컬럼이 최소한 60 자리가 되도록 만드세요. 또, 시작하기 전에, `users` (또는 동급의) 테이블이 100 자리수이며 `nullable`인 `remember_token` 컬럼을 포함하는지 확인 하세요. 이 컬럼은 어플리케이션에서 관리하는 "기억하기" 세션의 토큰을 저장하는데 사용됩니다. 이는 마이그레이션에서 `$table->rememberToken();`를 사용하여 추가 할수 있습니다. 물론 라라벨 5는 이러한 컬럼들을 기본적으로 포함하고 있습니다!

만약 여러분의 어플리케이션이 엘로퀀트를 사용하지 않는다면, 라라벨 쿼리 빌더를 사용하는 `database` 인증 드라이버를 사용 할 수도 있습니다.

<a name="authenticating-users"></a>
## 사용자 인증

라라벨은 기본적으로 컨트롤러와 연동된 다음의 두가지 인증을 포함하고 있습니다. `AuthController`는 새로운 사용자 등록과 "로그인"을, `PasswordController`는 이미 존재하는 회원이 비밀번호를 분실 했을 경우 초기화 할 수 있도록 도와주는 로직을 포함하고 있습니다.

각각의 컨트롤러는 필요한 메서드를 포함하기 위해 `trait`를 사용합니다. 대부분의 어플리케이션에서, 여러분은 이 컨트롤러들을 수정 할 필요가 없을 것입니다. 이 컨트롤러들이 렌더링 하는 뷰 파일들은 `resources/views/auth` 디렉토리에 위치해 있습니다. 원하는 대로 이 뷰 파일들을 커스터마이징하여 사용하세요.

### 사용자 등록

새로운 사용자 폼의 필수 입력 항목들을 변경 하려면, `App\Services\Registrar` 클래스를 수정합니다. 이 클래스는 어플리케이션에서 새로운 사용자를 검증하고 생성하는 역할을 합니다.

`Registrar` 클래스의 `validator` 메서드는 어플리케이션에서 새로운 사용자에 대한 폼검증 규칙을 포함하고 있으며, `create` 메서드는 데이터베이스에 새로운 `User` 레코드를 생성하는 역할을 합니다. 원하는 대로 이 메서드들을 수정하여도 됩니다. `Registrar`는 `AuthenticatesAndRegistersUsers` `trait`에 포함되어있는 메서드를 통하여 `AuthController`에 의해 호출됩니다.

#### 수동 인증

만약 제공된 `AuthController` 구현 클래스를 사용하지 않는 다면, 라라벨의 인증 클래스를 직접 사용하여 사용자 인증을 관리해야 합니다. 걱정마세요, 그것 또한 식은 죽 먹기 입니다! 첫번째로, `attempt` 메서드를 봅시다:

    <?php namespace App\Http\Controllers;

    use Auth;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller {

        /**
         * 인증 시도 핸들링
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password]))
            {
                return redirect()->intended('dashboard');
            }
        }

    }

`attempt` 메서드는 첫번째 인수로 키 / 값의 쌍으로 이루어진 배열을 받습니다. `password` 값은 [해시화](/docs/{{version}}/hashing)됩니다. 배열의 다른 값들은 데이터베이스 테이블에서 사용자를 찾는데 사용됩니다. 그러므로 위의 예제에서, 사용자는 `email` 컬럼의 값에 의해 조회가 됩니다. 만약 유저가 검색되었다면, 데이터베이스에 저장되어있는 해시화된 비밀번호가 배열을 통해 메서드로 전달된 해시화된 `password` 값가 비교 됩니다. 만약 두개의 해시화된 비밀번호가 일치 한다면, 사용자에게 인증된 세션이 새롭게 시작됩니다.

`attempt` 메서드는 만약 인증에 성공 했을 경우 `true`를 리턴하고 그렇지 않을 경우 `false`를 리턴합니다.

> **주의:** 이 예제에서, `email`은 필수 옵션이 아니며 단지 예제로써 쓰였을 뿐입니다. 여러분은 데이터베이스에서 "username"에 해당하는 어떠한 컬럼을 사용해도 좋습니다.

`intended` 리디렉트 함수는 사용자가 인증 필터에 걸리기 전에 원래 엑세스 하려고 시도했던 URL로 사용자를 리디렉트 시킵니다. 리디렉트 목적지가 없을 경우 주어진 특정 URI로 리디렉트 할수 있도록 대체 URI를 제공 할 수 있습니다.

#### 조건과 함께 사용자 인증

또한 인증 쿼리에 추가 조건을 더할수도 있습니다:

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1]))
    {
        // 해당 유저는 활성화 되어있고, 보류상태가 아니며, 존재합니다.
    }

#### 사용자가 인증이 되어있는지 확인

사용자가 이미 여러분의 어플리케이션에 로그인이 되어있는 지 확인 하려면, `check` 메서드를 사용합니다:

    if (Auth::check())
    {
        // 이 사용자는 로그인 되어있습니다.
    }

#### 사용자를 인증하고 "기억 하기"

여러분의 어플리케이션에 "기억 하기" 기능을 제공 하길 원한다면, 유저의 인증을 무한이나, 사용자가 수동으로 로그아웃 할때까지 유지시켜주는 논리(boolean) 값을 두번째 인수로 전달 하면 됩니다. 물론, 여러분의 `users` 테이블은 "기억 하기" 토큰을 저장할 때 쓰이는 `remember_token` 스트링 컬럼을 포함하고 있어야 합니다.

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember))
    {
        // 사용자의 로그인이 무한 유지..
    }

만약 여러분이 사용자들을 "기억" 하고 있다면, `viaRemember` 메서드를 사용하여 해당 사용자가 "기억 하기" 쿠키를 사용하여 인증이 되었는지 확인 할 수 있습니다:

    if (Auth::viaRemember())
    {
        //
    }

#### ID를 통한 사용자 인증

사용자 ID를 통하여 해당 유저를 어플리케이션에 로그인 시키려면, `loginUsingId` 메서드를 사용합니다:

    Auth::loginUsingId(1);

#### 로그인없이 사용자의 자격 증명을 검증

`validate` 메서드는 실제로 사용자를 어플리케이션에 로그인 시키지 않고 사용자의 자격 증명을 검증할 수 있도록 해줍니다:

    if (Auth::validate($credentials))
    {
        //
    }

#### 단일 요청을 위한 사용자 로그인

단일 요청에 대한 로그인을 할 수 있도록 `once` 메서드를 사용하여 사용자를 어플리케이션에 로그인 시킬 수 있습니다. 이때 세션이나 쿠키는 사용되지 않습니다:

    if (Auth::once($credentials))
    {
        //
    }

#### 사용자를 수동으로 로그인

이미 인스턴스가 존재하는 사용자를 여러분의 어플리케이션에 로그인 시키려면, 사용자의 인스턴스와 함께 `login` 메서드를 호출 합니다:

    Auth::login($user);

이는 자격 증명을 사용하여 사용자를 로그인 시키는 `attempt` 메서드와 같습니다.

#### 사용자를 어플리케이션에서 로그아웃

    Auth::logout();

물론 여러분이 빌트인 되어있는 라라벨의 인증 컨트롤러를 사용하고 있다면, 사용자를 어플리케이션에서 로그아웃 시키하는 메서드가 기본으로 제공 됩니다.

#### 인증 이벤트

`attempt` 메서드가 호출되면, `auth.attempt` [이벤트](/docs/{{version}}/events)가 트리거됩니다. 만약 인증 시도가 성공적이고 사용자가 로그인 되었다면, `auth.login` 이벤트 또한 트리거 됩니다.

<a name="retrieving-the-authenticated-user"></a>
## 인증된 사용자 조회

사용자가 안증이 되고 나면, 사용자의 인스턴스를 얻는 방법에는 여러가지가 있습니다.

첫번째로, `Auth` 파사드로부터 사용자를 액세스 할 수 있습니다.:

    <?php namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;

    class ProfileController extends Controller {

        /**
         * Update the user's profile.
         *
         * @return Response
         */
        public function updateProfile()
        {
            if (Auth::user())
            {
                // Auth::user()는 인증된 사용자의 인스턴스를 리턴합니다.
            }
        }

    }

두번째로, `Illuminate\Http\Request` 인스턴스를 통하여 인증된 사용자를 액세스 할 수 있습니다:

    <?php namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class ProfileController extends Controller {

        /**
         * Update the user's profile.
         *
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user())
            {
                // $request->user()는 인증된 사용자의 인스턴스를 리턴합니다.
            }
        }

    }

새번째로, `Illuminate\Contracts\Auth\Authenticatable` contract를 타입-힌트로 할 수 있습니다. 이 타입-힌트는 컨트롤러 생성자, 컨트롤러 메서드, 또는 [서비스 컨테이너](/docs/{{version}}/container)에 의해 결정 되는 다른 클래스의 생성자에 추가 될 수 있습니다:

    <?php namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use Illuminate\Contracts\Auth\Authenticatable;

    class ProfileController extends Controller {

        /**
         * Update the user's profile.
         *
         * @return Response
         */
        public function updateProfile(Authenticatable $user)
        {
            // $user는 인증된 사용자의 인스턴스 입니다.
        }

    }

<a name="protecting-routes"></a>
## 라우트 보호

[라우트 미들웨어](/docs/{{version}}/middleware)는 주어진 라우트에 인증된 사용자들만 액세스가 가능하도록 하는데 사용될 수 있습니다. 라라벨은 `auth` 미들웨어를 기본으로 제공하며, `app\Http\Middleware\Authenticate.php`에 정의되어 있습니다. 여러분이 해야할 일은 이 미들웨어를 라우트의 정의에 포함시키기만 하면 됩니다.

    // 라우트 클로저에 포함

    Route::get('profile', ['middleware' => 'auth', function()
    {
        // 인증된 사용자만 접근 가능...
    }]);

    // 컨트롤러에 포함

    Route::get('profile', ['middleware' => 'auth', 'uses' => 'ProfileController@show']);

<a name="http-basic-authentication"></a>
## HTTP 기본 인증

HTTP 기본 인증은 "로그인" 전용 페이지 설정없이 사용자를 인증 할 수 있는 빠른 방법을 제공합니다. 시작하려면, 라우트에 `auth.basic` 미들 웨어를 포함시키세요:

#### HTTP 기본 인증을 사용 하여 라우트를 보호

    Route::get('profile', ['middleware' => 'auth.basic', function()
    {
        // 인증된 사용자만 접근 가능...
    }]);

기본적으로, `auth.basic` 미들웨어는 사용자 레코드의 `email` 컬럼을 "username"으로 사용합니다.

#### 무상태 기본 인증 필터 설정

또한, 특히 API 인증에 유용한 방법인, 세션에 사용자의 식별 쿠키 설정 없이 HTTP 기본 인증을 사용하길 원할 수도 있습니다. 이렇게 하려면, `onceBasic` 메서드를 호출하는 [미들웨어](/docs/{{version}}/middleware)를 정의 하세요:

    public function handle($request, Closure $next)
    {
        return Auth::onceBasic() ?: $next($request);
    }

PHP FastCGI 사용한다면, HTTP 기본 인증이 기본적으로 정상 동작 하지 않을 수도 있습니다. 이럴땐 `.htaccess` 파일에 아래의 내용을 추가하세요:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## 비밀번호 알림 및 초기화

### 모델 & 테이블

대부분의 웹어플리케이션은 사용자가 잃어버린 비밀번호를 초기화 하는 방법을 제공합니다. 라라벨은 각각의 어플리케이션에 이 기능을 재구현 하도록 강요하는것 보다, 비밀번호 알림을 보내고 초기화 시켜주는 편리한 메서드들을 제공합니다.

시작하려면, 여러분의 `User` 모델이 `Illuminate\Contracts\Auth\CanResetPassword` contract를 구현하고 있는지 확인하세요. 물론 프레임워크에 포함되어있는 `User` 모델은 이미 이 인터페이스를 구현하고 있으며, 필요한 메서드들을 포함해주는 `Illuminate\Auth\Passwords\CanResetPassword` trait를 사용 하고 있습니다.

#### 알림 테이블 마이그레이션 생성

다음으로, 패스워드 초기화 토큰을 저장하는 테이블이 생성되어야 합니다. 이 테이블의 마이그레이션은 이미 라라벨에 포함되어 있으며, `database/migrations` 디렉토리에 존재합니다. 그러니 여러분은 아래처럼 마이그레이션만 실행하면 됩니다:

    php artisan migrate

### 비밀번호 알림 컨트롤러

라라벨은 또한 사용자의 비밀번호를 초기화 시키는데 필요한 로직들을 담고 있는 `Auth\PasswordController` 컨트롤러를 포함하고 있습니다. 우리는 여러분이 시작 할 수 있도록 뷰파일 또한 제공 하고 있습니다! 뷰 파일들은 `resources/views/auth` 디렉토리에 있습니다. 여러분의 어플리케이션 디자인에 맞도록 이 뷰 파일들을 수정해도 좋습니다.

사용자는 `PasswordController` 컨트롤러의 `getReset` 메서드로 향하는 링크가 들어있는 이메일을 받게 됩니다. 이 메서드는 비밀번호 초기화 폼을 렌더링 하고 사용자가 패스워드를 초기화 할수있도록 해줍니다. 비밀번호가 초기화 되고 나면, 사용자는 자동으로 어플리케이션에 로그인 되고 `/home`으로 리디렉트 됩니다. `PasswordController` 컨트롤러의 `redirectTo` 속성을 정의하여 초기화 후 리디렉트 할 경로를 지정 할 수 있습니다:

    protected $redirectTo = '/dashboard';

> **주의:** 기본적으로, 비밀번호 초기화 토큰은 한시간 후에 리셋됩니다. `config/auth.php` 파일의 `reminder.expire` 옵션을 통해 시간을 변경 할 수 있습니다.

<a name="social-authentication"></a>
## 소셜 인증

전형적인 폼 기반의 인증 외에도 라라벨은 [라라벨 소셜라이트](https://github.com/laravel/socialite)를 사용하여 OAuth 제공자들을 인증하는 간단하고, 편리한 방법을 제공합니다. **소셜라이트는 현재 Facebook, Twitter, Google, GitHub, Bitbucket을 제공합니다.**

소셜라이트를 시작하려면, `composer.json` 파일에 패키지를 추가하세요:

    "laravel/socialite": "~2.0"

다음으로 `config/app.php` 설정 파일에 `Laravel\Socialite\SocialiteServiceProvider` 서비스 제공자를 등록하세요. [파사드](/docs/{{version}}/facades) 또한 등록 할 수 있습니다:

    'Socialize' => 'Laravel\Socialite\Facades\Socialite',

여러분의 어플리케이션에서 사용할 OAuth 서비스들에 대한 자격 증명을 추가 해야합니다. 이 자격 증명들은 `config/services.php` 설정 파일에 위치해 있으며, 어플리케이션이 필요로 하는 서비스에 맞는 `facebook`, `twitter`, `google`, 또는 `github` 키가 사용되어야 합니다. 예를 들면:

    'github' => [
        'client_id' => 'your-github-app-id',
        'client_secret' => 'your-github-app-secret',
        'redirect' => 'http://your-callback-url',
    ],

이제, 여러분은 사용자들을 인증 할 준비가 되었습니다! 여러분은 2개의 라우트가 필요합니다: 사용자를 OAuth 제공자로 리디렉트 시켜주는 라우트와, 인증 후 제공자로부터 콜백을 받을 라우트입니다. 아래에 `Socialize` 파사드를 사용한 예제가 있습니다:

    public function redirectToProvider()
    {
        return Socialize::with('github')->redirect();
    }

    public function handleProviderCallback()
    {
        $user = Socialize::with('github')->user();

        // $user->token;
    }

`redirect` 메서드는 사용자를 OAuth 제공자로 전송 하는 일을 처리하고 `user` 메서드는 제공자로부터 전송 받은 요청을 읽고 사용자 정보를 조회하는 일을 합니다. 사용자를 리디렉트 하기전에, "스코프"를 설정 할 수도 있습니다:

    return Socialize::with('github')->scopes(['scope1', 'scope2'])->redirect();

사용자 인스턴스가 생성되고 나면, 사용자에 대한 몇가지 추가 정보를 조회 할 수도 있습니다:

#### 사용자 정보 조회

    $user = Socialize::with('github')->user();

    // OAuth 2 제공자
    $token = $user->token;

    // OAuth 1 제공자
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // 모든 제공자
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();
