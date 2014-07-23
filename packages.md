# 패키지 개발

- [소개](#introduction)
- [패키지 생성](#creating-a-package)
- [패키지 구조](#package-structure)
- [서비스 제공자](#service-providers)
- [패키지 규칙](#package-conventions)
- [개발 워크플로우](#development-workflow)
- [패키지 라우팅](#package-routing)
- [패키지 설정](#package-configuration)
- [패키지 마이그레이션](#package-migrations)
- [패키지 에셋](#package-assets)
- [패키지 발행](#publishing-packages)

<a name="introduction"></a>
## 소개

패키지는 Laravel에 기능을 추가하는 주된 방법입니다. 패키지는 날짜와 관련된 훌륭한 기능을 하는 [Carbon](https://github.com/briannesbitt/Carbon)이나, [Behat](https://github.com/Behat/Behat) 같은 BDD 테스팅 프레임워크 전체가 될 수 있습니다.

물론 저마다 다른 유형의 패키지가 있습니다. 어떤 패키지들은 Laravel뿐만 아니라 어떠한 프레임워크에서도 작동하는 독립형 패키지입니다. Carbon과 Behat 모두 독립형 패키지의 예입니다. 이러한 독립형 패키지들도 간단하게 `composer.json` 파일에 요청하여 Laravel과 함께 사용할 수 있습니다.

반면, 다른 패키지들은 특별히 Laravel과 함께 사용되기 위한 패키지입니다. 이전 버전의 Laravel에서는 이런 패키지들을 "번들"이라고 불렀습니다. 이러한 패키지들은 Laravel 어플리케이션을 향상시킬 라우트, 컨트롤러, 뷰, 설정, 마이그레이션을 포함하고 있습니다. 독립형 패키지 개발에는 특별한 과정이 필요하지 않으므로 이 가이드는 Laravel에 국한된 패키지 개발을 다룹니다.

모든 Laravel 패키지들은 [Packagist](http://packagist.org)와 [Composer](http://getcomposer.org)를 통해 배포됩니다. 그러므로 이러한 훌륭한 PHP 패키지 배포 도구를 배우는 것 또한 중요합니다.

<a name="creating-a-package"></a>
## 패키지 생성

Laravel 패키지를 생성하는 가장 쉬운 방법은 `workbench` Artisan 커맨드입니다. 먼저, `app/config/workbench.php` 파일에 몇가지 옵션을 설정해야 합니다. 이 파일에서, `name`과 `email` 옵션을 볼 수 있을 겁니다. 이 값들은 새로운 패키지의 `composer.json` 파일을 생성하는데 사용됩니다. 이 값들이 제공되고 나면 이제 워크벤치 패키지를 만들 준비가 끝났습니다!

**워크벤치 Artisan 커맨드 실행**

	php artisan workbench vendor/package --resources

벤더 이름은 다른 개발자가 만든 같은 이름의 패키지로부터 자신의 패키지를 구별할 수 있는 방법입니다. 예를들어 만약 제가(Taylor Otwell) "Zapper"라는 이름의 새로운 패키지를 생성했다면, 패키지 이름은 `Zapper`인 반면 벤더 이름은 `Taylor`이 될 수 있습니다. 기본적으로, 워크벤치는 프레임워크 종류에 구애받지않는 패키지를 생성합니다. 그러나, `resources` 커맨드를 사용하면 워크벤치가 Laravel에서 쓰일 수 있는 `migrations`, `views`, `config` 등과 같은 디렉토리들도 만들도록 해줍니다.

`workbench` 커맨드가 실행되고 나면 패키지는 Laravel의 `workbench` 디렉토리에 생성됩니다. 그다음 패키지에 생성된 `ServiceProvider`를 등록해야 합니다. `app/config/app.php` 파일의 `providers` 배열에 추가하여 서비스 제공자를 등록할 수 있습니다. 이렇게 하면 어플리케이션이 시작할때 Laravel에게 패키지를 로드하도록 지시합니다. 서비스 제공자는 `[Package]ServiceProvider`과 같은 네이밍 규칙을 사용합니다. 그러므로 위의 예제를 사용해보면 `providers` 배열에 `Taylor\Zapper\ZapperServiceProvider`를 추가해야 합니다.

서비스 제공자가 등록되면 이제 패키지를 개발할 준비가 끝났습니다! 하지만, 개발을 시작하기 전, 아래의 섹션을 참조하여 패키지 구조나 개발 워크플로우에 좀 더 익숙해지길 바랍니다.

> **노트:** 만약 서비스 프로바이더를 찾을 수 없다면, 어플리케이션의 루트 디렉토리에서 `php artisan dump-autoload` 커맨드를 실행하세요.

<a name="package-structure"></a>
## 패키지 구조

`workbench` 커맨드를 실행하면, 패키지는 Laravel 프레임워크의 다른 부분과도 잘 통합 되도록 규칙에 의해 설치됩니다.:

**기본 패키지 디렉토리 구조**

	/src
		/Vendor
			/Package
				PackageServiceProvider.php
		/config
		/lang
		/migrations
		/views
	/tests
	/public

디렉토리 구조를 좀더 둘러 보겠습니다. `src/Vendor/Package` 디렉토리는 `ServiceProvider` 패키지의 를 포함한 모든 클래스의 홈 디렉토리 입니다. `config`, `lang`, `migrations`, 그리고 `views` 디렉토리는 예상한것과 같이 패키지에 해당하는 리소스를 포함하고 있습니다. 패키지들은 "보통" 어플리케이션처럼 이러한 리소스들 갖고 있을 수 있습니다.

<a name="service-providers"></a>
## 서비스 제공자

서비스 제공자는 간단하게 패키지의 부트스트랩 클래스입니다. 기본으로 서비스 제공자는 `boot`과 `register` 2개의 메소드를 포함하고 있습니다. 이 메소드들 안에서 라우트 파일을 포함하거나, IoC 컨테이너 연결을 등록하거나, 이벤트를 부여하는 것처럼 원하는 모든걸 할 수 있습니다.

`boot` 커맨드는 요청이 라우팅 되기 바로 전에 호출되는 반면, `register` 메소드는 서비스 제공자가 등록될때 바로 호출됩니다. 그러므로 서비스 제공자의 액션이 이미 등록된 다른 서비스 제공자에 의존되어 있거나, 다른 제공자에 얽매어 있는 제공자를 치환할 경우 `boot` 메소드를 사용해야 합니다.

`workbench`를 사용하여 패키지를 생성할 경우 `boot` 커맨드는 이미 한개의 액션을 포함하고 있습니다.:

	$this->package('vendor/package');

이 메소드는 Laravel이 어떻게 뷰, 설정 그리고 다른 리소스들을 어플리케이션에서 제대로 로드할지 알려줍니다. 보통 워크벤치 규칙을 사용하여 패키지를 설정하므로 이 코드를 변경할 필요는 없습니다.

<a name="package-conventions"></a>
## 패키지 규칙

패키지의 환경 변수나, 뷰같은 리소스를 활용해야 할 경우, 이중 콜론 구문을 사용합니다.:

**패키지에서 뷰 로드**

	return View::make('package::view.name');

**패키지의 설정 아이템 조회**

	return Config::get('package::group.option');

> **노트:** 만약 패키지가 마이그레이션을 포함하고 있다면, 다른 패키지의 클래스명과 충돌을 피하기 위해 마이그레이션 이름 앞에 패키지 이름으로 접두어를 붙이는 것이 좋습니다.

<a name="development-workflow"></a>
## 개발 워크플로우

패키지를 개발할 때, 쉽게 템플릿을 확인하고 실험할수 있도록 어플리케이션의 컨텍스트 안에서 개발하는 것이 유용합니다. 그렇게 하기 위해선, Laravel 프레임워크의 새로운 복사본을 설치한다음, `workbench` 커맨드를 사용하여 패키지 구조를 생성합니다.

`workbench` 커맨드를 사용하여 패키지를 생성한 다음, `workbench/[vendor]/[package]` 디렉토리에서 `git init`를 한다음 워크벤치에서 바로 패키지를 `git push`하면 됩니다. 이렇게 하면 어플리케이션의 컨텍스트 안에서 `composer update` 커맨드에 의한 지연없이 편리하게 개발을 할 수 있습니다.

개발중인 패키지가 `workbench` 디렉토리에 있기 때문에 어떻게 컴포저가 패키지의 파일들을 오토로드 하는지 궁금해 할 수 있습니다. `workbench` 디렉토리가 존재한다면, Laravel이 재치있게 패키지를 스캔하여 어플리케이션이 시작할때 패키지의 컴포저 오토로드 파일들을 로딩합니다!

만약 패키지의 오토로드 파일들을 재생성 해야한다면, `php artisan dump-autoload` 커맨드를 사용합니다. 이 커맨드는 루트 프로젝트와 더불어 생성한 모든 워크벤치의 오토로드 파일들까지 재생성 합니다.

**아티즌 오토로드 커맨드 실행**

	php artisan dump-autoload

<a name="package-routing"></a>
## 패키지 라우팅

이전 버전의 Laravel에서는 `handles` 절이 어떤 URI를 패키지가 응답할 수 있는지 지정하는데 사용되었습니다. 하지만 Laravel 4 에서 패키지는 어떠한 URI에도 응답할 수 있습니다. 패키지에 라우트 파일을 로드하려면 간단하게 서비스 제공자의 `boot` 메소드에 라우트 파일을 `include`하면 됩니다.

**서비스 제공자에 라우트 파일 포함**

	public function boot()
	{
		$this->package('vendor/package');

		include __DIR__.'/../../routes.php';
	}

> **노트:** 만약 패키지가 컨트롤러를 사용하고 있다면, `composer.json` 파일의 오토로드 섹션에 해당 컨트롤러가 제대로 설정되어 있어야만 합니다.

<a name="package-configuration"></a>
## 패키지 설정

어떤 패키지들은 설정 파일을 필요로 할 수도 있습니다. 이러한 파일들은 일반적인 어플리케이션 설정 파일과 같은 방법으로 정의되어야 합니다. 그리고 리소스 등록을 서비스 제공자의 기본 `$this->package` 메소드로 사용했을 경우, 평상시처럼 이중 콜론 구문을 사용하여 액세스 할 수 있습니다.:

**패키지의 설정 파일 액세스**

	Config::get('package::file.option');

만약 패키지가 오직 하나의 설정 파일을 갖고 있다면, 간단히 `config.php`로 파일 이름을 정할 수 있습니다. 이렇게 하면, 파일 이름을 지정할 필요없이 바로 옵션을 액세스 할수 있습니다.:

**한개의 패키지 설정 파일 액세스**

	Config::get('package::option');

가끔, 일반적인 `$this->package` 메소드의 외부에 있는 뷰같은 패키지 리소스들을 등록하길 원할 때도 있습니다. 일반적으로 이는 리소스가 형식적인 위치에 있지 않을 경우에만 해당 됩니다. 리소스를 수동으로 등록하려면, `View`, `Lang`, 그리고  `Config` 클래스의 `addNamespace` 메소드를 사용합니다.:

**수동으로 리소스 네임스페이스 등록**

	View::addNamespace('package', __DIR__.'/path/to/views');

네임스페이스가 등록되고 나면, 네임스페이스명과 "더블 콜론" 신택스를 사용하여 해당 리소스를 액세스 할수 있습니다.:

	return View::make('packge::view.name');

`addNamespace` 메소드의 사용법은 `View`, `Lang`, 그리고 `Config` 클래스 모두 동일합니다.

### 설정 파일 캐스케이딩

만약 다른 개발자가 당신의 패키지를 설치하였을 경우, 몇몇의 설정 옵션을 변경하고 싶을 지도 모릅니다. 하지만 만약 패키지의 소스코드에서 옵션 값을 변경하였을 경우, 다음 번에 컴포저가 패키지를 업데이트 할때, 옵션 값들은 덮어씌여 질 겁니다. 그럴경우를 방지하기 위해 `config:publish` artisan 커맨드를 사용해야 합니다.:

**Config Publish 커맨드를 실행**

	php artisan config:publish vendor/package

이 커맨드가 실행되면, 패키지의 설정 파일들이 개발자가 안전하게 수정할 수 있는 `app/config/packages/vendor/package` 디렉토리로 복사 됩니다!

> **노트:** 또한 개발자는 특정 환경에 따른 패키지의 설정 파일들을 `app/config/packages/vendor/package/environment`에 지정할 수 있습니다..

<a name="package-migrations"></a>
## 패키지 마이그레이션

어떠한 패키지에도 마이그레이션을 쉽게 생성하고 실행할 수 있습니다. 워크벤치에 있는 패키지를 위한 마이그레이션을 생성하려면, `--bench` 옵션을 사용합니다.:

**워크벤치 패키지의 마이그레이션 생성**

	php artisan migrate:make create_users_table --bench="vendor/package"

**워크벤치 패키지의 마이그레이션 실행**

	php artisan migrate --bench="vendor/package"

컴포저를 통해 설치된 `vendor` 디렉토리에 있는 완성된 패키지의 마이그레이션을 실행할때는 `--package` 명령을 사용합니다.:

**설치된 패키지의 마이그레이션 실행**

	php artisan migrate --package="vendor/package"

<a name="package-assets"></a>
## 패키지 에셋

몇몇의 패키지는 JavaScript, CSS, images 같은 에셋을 갖고 있습니다. 하지만 `vendor`나 `workbench` 디렉토리로 연결할 수 없기 때문에, 이러한 에셋들을 어플리케이션의 `public`  디렉토리로 옮길 필요가 있습니다. `asset:publish` 커맨드가 이러한 문제를 해결해줍니다.:

**패키지 에셋을 Public으로 이동**

	php artisan asset:publish

	php artisan asset:publish vendor/package

패키지가 여전히 `workbench`에 있다면 `--bench` 명령어를 사용합니다.:

	php artisan asset:publish --bench="vendor/package"

이 커맨드는 에셋을 벤더와 패키지 이름에 따라 `public/packages` 디렉토리로 이동 시킵니다. 그러므로 `userscape/kudos`의 이름을 가진 패키지는 `public/packages/userscape/kudos`로 에셋을 이동 시킵니다. 이 에셋 규칙을 사용하면 패키지의 뷰에서 안전하게 에셋 경로를 코드화 할수 있습니다.

<a name="publishing-packages"></a>
## 패키지 발행

패키지를 발행할 준비가 되면 패키지를 [Packagist](http://packagist.org) 저장소에 제출해야 합니다. 만약 패키지가 Laravel 특유의 패키지라면, 패키지의 `composer.json` 파일에  `laravel` 태그를 추가하는것을 고려해 보길 바랍니다.

또한, 발행할 패키지에 태그를 붙이는 것이, 다른 개발자들이 그들의 `composer.json` 파일에서 당신의 패키지를 요청할때 안전한 버전인지 확인하는데 도움이 됩니다. 만약 안정적인 패키지가 준비되지 않았다면, `branch-alias` 컴포저 명령어 사용을 고려해 보길 바랍니다.

패키지가 발행되고 나서도, `workbench`에 의해 만들어진 어플리케이션 컨텍스트에서 마음껏 개발을 할 수 있습니다. 이 방법이 패키지가 발행되고 나서도 지속적으로 개발을 할 수 있는 좋은 방법입니다.

일부 단체는 그들의 개발자를 위해 그들만의 비공개 저장소를 갖고 있기도 합니다. 이렇게 하기위해선 컴포저 팀에 의해 제공되는 [Satis](http://github.com/composer/satis) 프로젝트를 살펴보시길 바랍니다.
