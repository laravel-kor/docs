# 설치

- [Composer 설치](#install-composer)
- [Laravel 설치](#install-laravel)
- [서버 요구사항](#server-requirements)
- [설정](#configuration)
- [Pretty URLs](#pretty-urls)

<a name="install-composer"></a>
## Composer 설치

Laravel은 의존성을 관리하기 위해 [Composer](http://getcomposer.org)을 사용합니다. 먼저, `composer.phar`를 다운받습니다. PHAR 아카이브를 갖고 있다면, 로컬 프로젝트의 디렉토리에 저장하거나 `usr/local/bin` 으로 옮겨 시스템에서 전역적으로 사용할 수 있습니다. Windows에서는 Composer [Windows installer](https://getcomposer.org/Composer-Setup.exe)를 사용할 수 있습니다.

<a name="install-laravel"></a>
## Laravel 설치

### 컴포저의 Create-Project를 통해 설치

터미널에서 컴포저의 `create-project` 커맨드를 사용하여 Laravel을 설치 할 수 있습니다.:

	composer create-project laravel/laravel --prefer-dist

### 다운로드를 통해 설치

Composer가 설치되면 [최신버전](https://github.com/laravel/laravel/archive/master.zip)의 Laravel 프레임워크를 다운 받아 서버 디렉토리에 압축을 풉니다. 다음으로, 프레임워크가 의존하는 코드들의 설치를 위해 Laravel 프레임워크의 루트에서 `php composer.phar install` (또는 `composer install`) 커맨드를 실행합니다. 이 과정이 성공적으로 완료되려면 서버에 Git이 설치되어 있어야 합니다.

`php composer.phar update` 커맨드를 사용하여, Laravel 프레임워크를 업데이트를 할 수 있습니다.

<a name="server-requirements"></a>
## 서버 요구사항

Laravel 프레임워크는 몇가지의 시스템 요구사항이 있습니다:

- PHP >= 5.3.7
- MCrypt PHP Extension

<a name="configuration"></a>
## 설정

Laravel은 설정이 거의 필요하지 않으므로 바로 개발을 시작해도 좋습니다. 그렇지만 `timezone`과 `locale` 같이 변경하고 싶은 몇가지의 설정이 포함되어 있으므로 `app/config/app.php` 파일과 각각의 매뉴얼을 살펴보는게 좋습니다.  

> **노트:** 꼭 설정해야 하는 한개의 설정 옵션은 `app/config/app.php`에 있는 `key` 값 입니다. 이 값은 32자리 랜덤 문자열로 설정되어야 합니다. 이 키 문자열은 값을 암호화 할 때 사용되며, 문자열이 올바르게 설정 될 때까지 암호화한 값은 안전하지 못할 것입니다. `php artisan key:generate`과 같이 Artisan 커맨드를 사용하여 이 문자열을 설정할수 있습니다.

<a name="permissions"></a>
### 퍼미션

Laravel은 app/storage에 안에 있는 폴더에 쓰기 퍼미션을 요구합니다.

<a name="paths"></a>
### Paths

몇 개의 프레임워크 디렉토리 경로는 설정이 가능합니다. 이러한 디렉토리들의 위치를 변경하려면, `bootstrap/paths.php` 파일을 확인하기 바랍니다.

> **노트:** Laravel은 공개가 필요한 파일들만 public 폴더에 위치시킴으로서 어플리케이션 코드와 로컬 스토리지를 보호합니다. public 폴더를 사이트의 documentRoot (웹 루트) 로 설정하거나, public 폴더의 내용을 사이트의 documentRoot 디렉토리에 위치시키고, Laravel의 나머지 모든 파일들은 웹 루트 밖으로 위치시키는 걸 권장합니다.

<a name="pretty-urls"></a>
## Pretty URLs

Laravel 프레임워크는 index.php가 없는 URL 사용을 가능하게 해주는 `public/.htaccess` 파일을 포함하고 있습니다. 아파치를 사용한다면 반드시 `mod_rewrite` 모듈을 사용하도록 설정하길 바랍니다.

만약 포함되어 있는 `.htaccess` 파일이 작동하지 않는다면 아래걸로 시도해 보시길 바랍니다.:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]
