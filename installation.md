# 설치

- [Composer 설치](#install-composer)
- [Laravel 설치](#install-laravel)
- [서버 요구사항](#server-requirements)

<a name="install-composer"></a>
## Composer 설치

Laravel은 의존성을 관리하기위하여 [Composer](http://getcomposer.org)를 사용합니다. 그러므로 Laravel을 사용하기전에, 여러분의 컴퓨터에 Composer가 설치되어 있는지 확인하세요.

<a name="install-laravel"></a>
## Laravel 설치

### Laravel 인스톨러를 통한 설치

첫번째로, Composer를 사용하여 Laravel 인스톨러를 다운로드 합니다.

    composer global require "laravel/installer=~1.1"

`laravel` 실행파일이 여러분의 시스템에서 찾아지도록 PATH에 `~/.composer/vendor/bin` 디렉토리를 추가되어 있는지 확인하세요.

설치가되고 나면, 간단한 `laravel new` 커맨드가 여러분이 명시한 디렉토리 안에 새로운 Laravel 설치를 생성할 것입니다. 예를 들면 `laravel new blog`는 모든 의존성들이 설치된 새로운 Laravel 설치를 포함하고 있는 `blog` 디렉토리를 생성합니다. 이 설치 방법은 Composer를 통한 설치보다 훨씬 빠릅니다:

    laravel new blog

### Composer Create-Project를 통한 설치

터미널에서 Composer의 `create-project` 커맨드를 사용하여 Laravel을 설치할 수도 있습니다:

    composer create-project laravel/laravel --prefer-dist

### 스캐폴딩

Laravel 사용자 등록과 인증의 스캐폴딩을 포함하고 있습니다. 만약 이 스캐폴딩을 제거하고 싶다면, `fresh` Artisan 커맨드를 사용하세요:

    php artisan fresh

<a name="server-requirements"></a>
## 서버 요구사항

Laravel 프레임워크는 몇몇의 시스템 요구사항이 있습니다:

- PHP >= 5.4
- Mcrypt PHP Extension
- OpenSSL PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension

PHP 5.5에서, 몇몇의 OS 배포는 여러분에게 PHP JSON 확장을 수동으로 설치하도록 요구할 수도 있습니다. Ubuntu를 사용할 경우, `apt-get install php5-json`을 통하여 설치 할 수 있습니다.

<a name="configuration"></a>
## 구성

Laravel 설치 이후에 가장 먼저 해야하는 일은, 여러분의 어플리케이션 키를 랜덤 문자열로 설정하는 것 입니다. 만약 Composer을 통하여 Laravel을 설치했다면, 아마 `key:generate` 커맨드를 통해 이미 설정 되어 있을 겁니다.

일반적으로, 이 문자열은 32 자릿수 입니다. 이 키는 `.env` 환경 파일에 설정 될 수 있습니다. **만약 어플리케이션 키가 설정되어 있지 않다면, 여러분의 사용자 세션과 다른 암호화된 데이터들은 안전하지 않게 됩니다!**

Laravel은 설치 이후에 거의 다른 구성을 필요로 하지 않습니다. 여러분은 바로 개발을 시작할 수 있습니다! 하지만, `config/app.php` 파일과 해당 문서를 리뷰할 수도 있습니다. 이 파일은 `timezone`과 `locale`같이 여러분의 어플리케이션에 따라 변경할 수있는 몇몇의 옵션들을 포함하고 있습니다.

Laravel이 설치되면, 여러분은 또한 [로컬 환경을 구성](/docs/{{version}}/configuration#environment-configuration)해야 합니다.

> **주의:** 프로덕션 어플리케이션에서는 `app.debug` 구성을 절대로 `true`로 설정해서는 안됩니다.

<a name="permissions"></a>
### 퍼미션

Laravel은 몇개의 퍼미션이 구성되도록 요구합니다: `storage`과 `vendor` 내에 있는 폴더들은 웹 서버에 의해 쓰기 액세스가 요구 됩니다.

<a name="pretty-urls"></a>
## Pretty URLs

### Apache

프레임워크는 `index.php`가 없는 URL을 허용하는데 사용되는 `public/.htaccess` 파일을 포함하고 있습니다. 만약 Laravel 어플리케이션을 Apache에서 구동된다면, `mod_rewrite` 모듈이 활성화 되어있는지 확인하세요.

만약 포함되어 있는 `.htaccess` 파일이 여러분의 아파치에서 작동하지 않는다면 아래것을 시도해보세요:

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

### Nginx

Nginx에서는, 여러분의 사이트 구성에서 아래의 디렉티브가 "pretty" URL을 가능하도록 해줍니다:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

물론, [Homestead](/docs/{{version}}/homestead)를 사용 할땐, pretty URL이 자동으로 설정됩니다.
