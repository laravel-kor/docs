# 에러 & 로그

- [에러 세부정보](#error-detail)
- [에러 핸들링](#handling-errors)
- [HTTP 예외](#http-exceptions)
- [404 에러 핸들링](#handling-404-errors)
- [로그](#logging)

<a name="error-detail"></a>
## 에러 세부정보

어플리케이션의 에러 세부정보는 자동적으로 켜져있습니다. 에러가 발생했을 경우, 자세한 스택 추적과 에러 메시지를 함께 에러 페이지에 표시 해줍니다. `app/config/app.php` 파일의 `debug` 옵션을 `false` 로 설정하면 세부정보를 사용하지 않을 수 있습니다. **production 환경에서는 에러 세부정보를 사용하지 않기를 강력히 권합니다.**

<a name="handling-errors"></a>
## 에러 핸들링

기본적으로 `app/start/global.php` 파일은 모든 예외에 대한 에러 핸들링을 포함하고 있습니다.:

  App::error(function(Exception $exception)
	{
		Log::error($exception);
	});

위 코드는 가장 기본적인 에러 핸들링입니다. 필요한 경우, 더 많은 에러 핸들링을 지정 할 수 있습니다. 핸들러들은 예외가 처리하는 타입-힌트에 따라 호출됩니다. 예를 들어, `RuntimeException` 인스턴스만 처리하는 핸들러를 만들 수 있습니다.:

	App::error(function(RuntimeException $exception)
	{
		// Handle the exception...
	});

예외 처리가 응답을 반환한다면, 그 응답이 브라우저로 전송되고 다른 에러 핸들러는 호출되지 않습니다.:

	App::error(function(InvalidUserException $exception)
	{
		Log::error($exception);

		return 'Sorry! Something is wrong with this account!';
	});

PHP fatal 에러를 수신 하려면 `App::fatal` 메소드를 사용합니다.:

	App::fatal(function($exception)
	{
		//
	});

<a name="http-exceptions"></a>
## HTTP 예외

HTTP에 관련된 예외는 클라이언트 요청시 일어 날 수 있는 에러에 속합니다. 페이지를 찾을 수 없는 에러(404), 권한 에러(401), 또는 500에러 일수도 있습니다. 아래와 같은 코드를 사용하여 이러한 응답을 반환합니다.:

	App::abort(404, 'Page not found');

첫번째 인수는 HTTP 상태코드(status code)이고, 두번째 인수는 에러와 함께 표시할 사용자 지정 메세지 입니다.

401 권한 예외를 발생시키려면 아래와 같이 하면됩니다.:

	App::abort(401, 'You are not authorized.');

이러한 예외들은 요청 사이클이 일어나는 동안 언제든지 실행될 수 있습니다.

<a name="handling-404-errors"></a>
## 404 에러 핸들링

어플리케이션에서 모든 404 에러에 대해 사용자 지정 에러 페이지를 표시하게 해주는 에러 핸들러를 등록할 수 있습니다.:

	App::missing(function($exception)
	{
		return View::make('errors.missing');
	});

<a name="logging"></a>
## 로그

Laravel의 로그는 강력한 [Monolog](http://github.com/seldaek/monolog)의 위에서 간편한 레이어를 제공합니다. 기본적으로 Laravel는 어플리케이션의 일일 로그 파일을 생성하도록 구성되어 있으며, 이 파일들은 `app/storage/logs`에 저장되어 있습니다 . 아래와 같이 로그를 기록할 수도 있습니다.:

	Log::info('This is some useful information.');

	Log::warning('Something could be going wrong.');

	Log::error('Something is really going wrong.');

로그는 [RFC 5424](http://tools.ietf.org/html/rfc5424)에 정의된 7가지의 로깅 레벨을 제공합니다.: **debug**, **info**, **notice**, **warning**, **error**, **critical**, 그리고 **alert**.

문맥에 맞는 데이터 배열 또한 로그 메소드에 전달 될 수 있습니다.:

	Log::info('Log message', array('context' => '다른 도움이 되는 정보'));

Monolog는 로그에 사용할 다양한 추가적인 로그 핸들러를 포함하고 있습니다. 필요한 경우, Laravel에서 사용되는 기본 Monolog 인스턴스에 액세스 할 수 있습니다.:

	$monolog = Log::getMonolog();

또한 로그에 전달되는 모든 메시지를 받는 이벤트를 등록할 수도 있습니다.:

**로그 수신자 등록**

	Log::listen(function($level, $message, $context)
	{
		//
	});
