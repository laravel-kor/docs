# 뷰 & 응답

- [기본 응답](#basic-responses)
- [리디렉트](#redirects)
- [뷰](#views)
- [뷰 컴포저](#view-composers)
- [특별한 응답](#special-responses)

<a name="basic-responses"></a>
## 기본 응답

**라우트로부터 문자열 리턴**

	Route::get('/', function()
	{
		return 'Hello World';
	});

**맞춤 응답 만들기**

`Response` 클래스는 HTTPS 응답에 대한 많은 메소드를 제공하는 `Symfony\Component\HttpFoundation\Response` 클래스로부터 상속 됩니다.

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

**응답에 쿠키 부여하기**

	$cookie = Cookie::make('name', 'value');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## 리디렉트

**리디렉트**

	return Redirect::to('user/login');

**플래쉬 데이터와 함께 리디렉트**

	return Redirect::to('user/login')->with('message', 'Login Failed');

> **노트:** `with` 메소드가 세션에 데이터를 플래쉬하므로, `Session::get` 메소드를 사용하여 데이터를 조회 할 수 있습니다.

**명칭이 부여된 라우트로 리디렉트**

	return Redirect::route('login');

**매개 변수와 함께 명칭이 부여된 라우트로 리디렉트**

	return Redirect::route('profile', array(1));

**명칭이 부여된 매개 변수와 함께 명칭이 부여된 라우트로 리디렉트**

	return Redirect::route('profile', array('user' => 1));

**컨트롤러 액션으로 리디렉팅**

	return Redirect::action('HomeController@index');

**매개 변수와 함께 컨트롤러 액션으로 리디렉팅**

	return Redirect::action('UserController@profile', array(1));

**명칭이 부여된 매개 변수와 함께 컨트롤러 액션으로 리디렉팅**

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## 뷰

뷰는 일반적으로 어플리케이션의 HTML을 포함하고 프리젠테이션 로직에서 컨트롤러와 도메인 로직을 분리하는 편리한 방법을 제공합니다. 뷰는 `app/views` 디렉토리에 있습니다.

간단한 뷰는 다음과 같습니다.:

    <!-- app/views/greeting.php 뷰파일 -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

이 뷰는 아래와 같이 브라우저로 반환될 수 있습니다.:

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Taylor'));
	});

`View::make`로 전달되는 두번째 매개 변수는 뷰에 제공하는 데이터 배열입니다.

**뷰에 데이터 전달**

	$view = View::make('greeting')->with('name', 'Steve');

위 예제에서 `$name` 변수는 뷰에서 액세스 할수있고 `Steve` 라는 값을 갖고 있습니다.

원한다면, `make` 메소드의 2번째 매개 변수에 데이터 배열을 전달 할 수도 있습니다.:

	$view = View::make('greetings', $data);

또한 모든 뷰에 데이터를 쉐어할 수도 있습니다.:

	View::share('name', 'Steve');

**뷰에 하위 뷰(sub-view) 전달**

가끔 뷰 안에 또 다른 뷰를 전달 할때도 있을 겁니다. 예를 들어 `app/views/child/view.php` 에 저장되어 있는 하위 뷰(sub-view)를 아래처럼 전달 할수 있습니다.:

	$view = View::make('greeting')->nest('child', 'child.view');

	$view = View::make('greeting')->nest('child', 'child.view', $data);

하위 뷰(sub-view)는 상위 뷰에서 렌더링 할 수 있습니다.:

	<html>
		<body>
			<h1>Hello!</h1>
			<?php echo $child; ?>
		</body>
	</html>

<a name="view-composers"></a>
## 뷰 컴포저

뷰 컴포저는 뷰가 렌더링 될때 호출되는 콜백이나 클래스 메소드 입니다. 어플리케이션을 통해 특정한 뷰가 렌더링 될때마다 바인딩 할 데이터가 있을 경우, 뷰 컴포저가 그 코드를 한곳에서 편성할수 있습니다. 따라서, 뷰 컴포저는 뷰 모델이나 프레젠터처럼 작동할수 있습니다.

**뷰 컴포저 정의**

	View::composer('profile', function($event)
	{
		$event->view->with('count', User::count());
	});

이제 `profile` 뷰가 렌더링 될때마다 `count` 데이터가 뷰에 바운딩 될 겁니다.

또한 한번에 여러개의 뷰에 컴포저를 부여 할수 있습니다.:

    View::composer(array('profile','dashboard'), function($event)
    {
        $event->view->with('count', User::count());
    });

어플리케이션 [IoC Container](/docs/ioc)를 통해 해결되는 혜택을 제공하는 클래스 기반의 컴포저를 사용하기 원한다면, 아래와 같이 해야 하며:

	View::composer('profile', 'ProfileComposer');

뷰 컴포저 클래스는 아래와 같이 정의 되야 합니다.:

	class ProfileComposer {

		public function compose($event)
		{
			$event->view->with('count', User::count());
		}

	}

컴포저 클래스가 꼭 어디에 저장되어야 한다는 규칙은 없습니다. `composer.json` 파일의 지시에 따라 오토로딩만 된다면 어느곳이든 저장해도 좋습니다.

### 뷰 생성자

뷰 **생성자**는 뷰 컴포저와 거의 비슷하게 작동합니다. 하지만, 뷰 생성자는 뷰가 인스턴트화 되는 순간 실행 됩니다. `creator` 메소드를 사용하여 뷰 생성자를 등록할 수 있습니다.:

	View::creator('profile', function($view)
	{
		$view->with('count', User::count());
	});

<a name="special-responses"></a>
## 특별한 응답

**JSON 응답 생성**

	return Response::json(array('name' => 'Steve', 'state' => 'CA'));

**JSONP 응답 생성**

	return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));

**파일 다운로드 응답 생성**

	return Response::download($pathToFile);

	return Response::download($pathToFile, $name, $headers);
