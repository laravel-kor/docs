# 설정

- [소개](#introduction)
- [구성환경 설정](#environment-configuration)
- [점검 모드](#maintenance-mode)

<a name="introduction"></a>
## 소개

Laravel 프레임워크의 모든 설정 파일은 `app/config` 디렉토리에 있습니다. 모든 설정 파일들은 문서화 되어있으니 파일들을 쭉 읽어보면서 당신에게 필요한 옵션들에 익숙해지길 바랍니다.

때때로 런타임에서 설정 값을 액세스 해야 할 때가 있습니다. 이럴 땐 `Config` 클래스를 사용합니다.:

**설정 값 액세스**

	Config::get('app.timezone');

또한 설정 옵션이 존재 하지 않을 경우 리턴 할 기본 값을 지정 할 수도 있습니다.

	$timezone = Config::get('app.timezone', 'UTC');

"점" 스타일의 구문은 여러파일 중에서 값을 액세스 할 때 사용됩니다. 또한, 런타임에서 설정 값을 설정 할 수도 있습니다.:

**설정 값 설정**

	Config::set('database.default', 'sqlite');

런타임에 셋팅되는 설정 값들은 오직 현재 요청에만 설정이 적용되며, 다음 요청까지 이월되지는 않습니다.

<a name="environment-configuration"></a>
## 구성환경 설정

종종 어플리케이션이 실행되는 환경에 따라 다른 설정 값을 가지고 있는게 도움이 될 때도 있습니다. 예를 들면,  프로덕션 서버와 로컬개발 컴퓨터에 각각 다른 캐쉬 드라이버를 사용하길 원할 수도 있습니다. 구성 환경을 통해 이를 쉽게 해결 할 수 있습니다.

간단히, `local` 같은 환경 이름과 일치하는 폴더를 `config` 폴더 안에 만듭니다. 다음, 덮어쓰길 원하는 파일을 만들고 필요한 옵션을 설정합니다. 예를 들어, local 환경에서 쓸 캐시 드라이버를 설정 하려면 `app/config/local` 에 다음 내용으로 `cache.php`파일을 생성합니다.:

    <?php

	return array(

		'driver' => 'file',

	);

> **노트:** 'testing'은 유닛테스팅 예약어 이므로 환경 이름으로 사용하지 마십시오.

덮어 쓸 옵션 말고는 기본 설정 파일에 있는 _모든_ 옵션을 지정 할 필요는 없습니다. 환경 설정 파일은 기본 파일에 종속됩니다.

다음, 우리는 프레임워크가 어떤 환경에서 실행 되고 있는지 프레임워크에게 알려 줄 필요가 있습니다. 기본 환경은 항상 `production` 입니다. 하지만, 루트에 있는 `start.php` 파일 안에 다른 환경을 설정 할 수도 있습니다. 이 파일에서 `$app->detectEnvironment`를 찾을수 있을 겁니다. 이 메소드에 전달되는 배열이 현재 환경을 결정 하는데 사용됩니다. 필요에 따라 배열에 다른 환경과 컴퓨터 이름을 추가 해도 좋습니다.

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

또한 `detectEnvironment` 메소드에 자신만의 구성환경 탐지하는 `클로저`를 전달 할 수도 있습니다.:

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

`environment` 메소드를 통해 현재의 어플리케이션 구성환경을 액세스 할 수 있습니다.:

**현재의 어플리케이션 구성환경 액세스**

	$environment = App::environment();

<a name="maintenance-mode"></a>
## 점검 모드

어플리케이션이 점검 모드일 경우, 어플리케이션의 모든 라우트에 사용자 정의 점검 뷰가 표시됩니다. 점검 모드는 어플리케이션이 업데이트 되는 동안 어플리케이션을 쉽게 "사용 중지" 상태로 만들어 줍니다. `App:down` 메소드는 이미 `app/start/global.php` 파일에 구현되어 있습니다. 어플리케이션이 점검 모드 일 때 이 메소드의 응답이 사용자에게 보내지게 됩니다.

간단히 아티즌 커맨드에 `down` 메소드를 실행하여 점검모드를 활성화 할 수 있습니다.:

	php artisan down

점검 모드를 비활성화 하려면 `up` 커맨드를 사용합니다.:

	php artisan up

어플리케이션의 `app/start/global.php` 파일에 다음과 같은 구문을 추가해 어플리케이션이 점검 모드일때 사용자 정의 뷰를 디스플레이 할 수 있습니다.:

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});
