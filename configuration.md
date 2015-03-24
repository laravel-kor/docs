# 구성

- [소개](#introduction)
- [설치 후](#after-installation)
- [구성 값 조회](#accessing-configuration-values)
- [구성 환경](#environment-configuration)
- [캐싱 구성](#configuration-caching)
- [점검 모드](#maintenance-mode)
- [Pretty URLs](#pretty-urls)

<a name="introduction"></a>
## 소개

Laravel 프레임워크의 모든 구성 파일은 `config` 디렉토리에 있습니다. 모든 구성 파일들은 문서화 되어있으니 파일들을 쭉 읽어보면서 당신에게 필요한 옵션들에 익숙해지길 바랍니다.

<a name="after-installation"></a>
## 설치 후

### 여러분의 어플리케이션에 이름을 부여

라라벨을 설치하고난 뒤, 여러분은 아마 여러분의 어플리케이션에 "이름"을 부여 하고 싶을 지도 모릅니다. 기본적으로 `app` 디렉토리가 `App` 네임스페이스로 설정되어있고 [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/)을 사용하여 컴포저에 의해 오토로드 됩니다. 하지만, 여러분은 어플리케이션의 이름과 일치하는 네임스페이스로 변경 하고 싶을지도 모릅니다. 그럴땐 `app:name` 아티즌 커맨드를 통해 쉽게 변경 할 수 있습니다.

예를 들어, 만약 여러분의 어플리케이션의 이름이 "Horsefly"라면, 설치된 루트에서 다음의 커맨드를 실행할 수 있습니다.

    php artisan app:name Horsefly

여러분의 어플리케이션 이름을 변경하는것은 옵션이며, 원한다면 `App` 네임스페이스를 유지해도 좋습니다:

### 다른 설정

라라벨은 기본적으로 아주 약간의 설정을 필요로 합니다. 여러분은 바로 개발을 시작해도 좋습니다! 하지만, `config/app.php` 파일과 문서를 한번 훑어보세요. 저 파일은 여러분의 위치를 기반으로 변경해야할 `timezone`과 `locale` 같은 몇몇개의 옵션을 포함하고 있습니다..

라라벨이 설치되고 나면, 여러분은 [로컬 환경을 구성]configure your local environment](/docs/5.0/configuration#environment-configuration)해야 합니다.

> **주의:** 여러분은 프로덕션 어플리케이션에서 절대로 `app.debug` 구성 옵션을 `true`로 설정 해서는 안됩니다.

<a name="permissions"></a>
### 퍼미션

라라벨은 한 세트의 퍼미션이 구성되도록 요구합니다: `storage`과 `vendor` 내에 있는 폴더들은 웹 서버에 의해 쓰기 액세스가 요구 됩니다.

<a name="accessing-configuration-values"></a>
## 구성 값 액세스

여러분은 `Config` 파사드를 사용하여 쉽게 여러분의 구성 값을 액세스 할 수 있습니다.

    $value = Config::get('app.timezone');

    Config::set('app.timezone', 'America/Chicago');

여러분은 또한 `config` 헬퍼 함수를 사용 할 수도 있습니다:

    $value = config('app.timezone');

<a name="environment-configuration"></a>
## 구성 환경

종종 어플리케이션이 실행되는 환경에 따라 다른 구성 값을 가지고 있는게 도움이 될 때도 있습니다. 예를 들면, 프로덕션 서버와 로컬 개발 컴퓨터에 각각 다른 캐쉬 드라이버를 사용하길 원할 수도 있습니다. 구성 환경을 통해 이를 쉽게 해결 할 수 있습니다.

이같은 일을 쉽게 만들기위해, 라라벨은 Vance Lucas의 [DotEnv](https://github.com/vlucas/phpdotenv] PHP 라이브러리를 사용합니다. 막 설치된 깨끗한 상태의 라라벨에서, 여러분의 어플리케이션은 루트 디렉토리에 `.env.example` 파일을 포함하고 있습니다. 만약 여러분이 컴포저를 통하여 라라벨을 설치 했다면, 이 파일은 자동으로 `.env`로 이름이 변경됩니다. 그렇지 않으면 여러분은 파일 명을 수동으로 변경해야 합니다.

이 파일에 나열된 모든 변수는 여러분의 어플리케이션이 요청을 받을 때, PHP 슈퍼-글로벌 `$_ENV`로 로드 됩니다. 여러분은 `env` 헬퍼를 사용하여 이 변수들을 조회 할 수 있습니다. 사실, 여러분이 라라벨의 구성 파일을 리뷰한다면, 이미 몇개의 옵션들이 이 헬퍼를 사용했다는것을 알게 될겁니다!

여러분의 로컬 서버와 프로덕션 환경 모두 구성 변수를 필요한만큼 수정해도 좋습니다. 그렇지만, 각각의 개발자 / 서버가 다른 구성 환경을 요구하므로 여러분의 `.env` 파일은 어플리케이션의 소스 컨트롤에 커밋되지 말아야 합니다.

만약 여러분이 팀 단위로 개발을 하고 있다면, 어플리케이션에 `.env.example` 파일을 계속 포함 하는것이 좋을 수도 있습니다. 이 예제 구성 파일에 필요한 변수명 들을 저장해 놓는다면, 팀의 다른 개발자들이 여러분의 어플리케이션에 어떤 구성 환경이 필요한지 명확하게 알 수 있습니다.

#### 어플리케이션의 현재 환경 액세스

`Application` 인스턴스의 `environment` 메서드를 통해 현재 어플리케이션의 환경을 엑세스 할 수 있습니다.

    $environment = $app->environment();

여러분은 또한 `environment` 메서드에 인수를 전달하여 현재 환경이 주어진 값과 일치 하는지 확일 할 수 있습니다:

    if ($app->environment('local'))
    {
        // local 환경
    }

    if ($app->environment('local', 'staging'))
    {
        // local 또는 staging 환경
    }

어플리케이션의 인스턴스를 얻으려면, [서비스 컨테이너](/docs/5.0/container)를 통해 `Illuminate\Contracts\Foundation\Application` 계약(Contract)을 해결(resolve) 합니다. 물론, 여러분이 [서비스 프로바이더](/docs/5.0/providers)내에 있다면, 어플리케이션 인스턴스는 `$this->app` 인스턴스 변수를 통해 사용 가능 합니다.

어플리케이션 인스턴스는 `app` 헬퍼 또는 `App` 파사드를 통해서도 엑세스 될 수 있습니다:

    $environment = app()->environment();

    $environment = App::environment();

<a name="configuration-caching"></a>
## 캐시 구성

어플리케이션에 약간의 스피드 상승을 부여하려면, `config:cache` 아티즌 커맨드를 사용하여 여러분의 모든 구성 파일들을 하나의 파일로 캐시 할 수 있습니다. 이는 프레임워크에 의해 빠르게 로드 될 수 있도록 어플리케이션의 모든 구성 옵션들을 하나의 파일로 병합합니다.

여러분은 개발 루틴의 한 단계로 `config:cache` 커맨드를 실행하는게 좋습니다.

<a name="maintenance-mode"></a>
## 점검 모드

어플리케이션이 점검 모드일 경우, 어플리케이션의 모든 라우트에 사용자 정의 점검 뷰가 표시됩니다. 점검 모드는 어플리케이션이 업데이트 되는 동안 어플리케이션을 쉽게 "사용 중지" 상태로 만들어 줍니다. 점검 모드 확인은 어플리케이션의 기본 미들웨어 스택에 포함되어 있습니다. 만약 어플리케이션이 점검 모드일 경우, 503 상태 코드와 함께 'HttpException` 예외가 던져집니다.

간단히 아티즌 커맨드의 `down` 메서드를 실행하여 점검모드를 활성화 할 수 있습니다:

    php artisan down

점검 모드를 비활성화 하려면 `up` 커맨드를 사용합니다:

    php artisan up

### 점검 모드 응답 템플릿

점검 모드 응답의 기본 템플릿은 `resources/views/errors/503.blade.php`에 위치해 있습니다.

### 점검 모드 & 큐

어플리케이션이 점검 모드 일 때는, [큐 작업](/docs/5.0/queues)도 마찬가지로 처리되지 않습니다. 큐 작업들은 어플리케이션이 점검 모드에서 빠져 나올 경우 평소처럼 계속해서 처리 됩니다.

<a name="pretty-urls"></a>
## Pretty URLs

### Apache

프레임워크는 `index.php`가 없는 URL을 허용하는데 사용되는 `public/.htaccess` 파일을 포함하고 있습니다. 만약 여러분의 라라벨 어플리케이션이 아파치에서 구동 되고 있다면, `mod_rewrite` 모듈이 활성화 되어있는지 확인하세요.

만약 포함되어 있는 `.htaccess` 파일이 여러분의 아파치에서 작동하지 않는다면 아래것을 시도해보세요:

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

만약 여러분의 웹 호스트가 `FollowSymlinks` 옵션을 허용하지 않는다면, `Options +SymLinksIfOwnerMatch` 옵션을 대신 사용해보세요.

### Nginx

Nginx에서는, 여러분의 사이트 구성에서 아래의 디렉티브가 "pretty" URL이 가능하도록 해줍니다.

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

물론, [Homestead](/docs/5.0/homestead)를 사용 할땐, pretty URL이 자동으로 설정됩니다.
