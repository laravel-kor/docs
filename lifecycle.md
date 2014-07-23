# 요청 사이클

- [소개](#overview)
- [시작 파일](#start-files)
- [어플리케이션 이벤트](#application-events)

<a name="overview"></a>
## 소개

Laravel 요청 사이클은 아주 간단합니다. 어플리케이션에 요청이 들어오면 적절한 라우트나 컨트롤러로 전달됩니다. 그 라우트에 대한 응답은 브라우져로 다시 전송되고 화면에 표시됩니다. 가끔 실제로 라우트가 호출되기 전이나 호출 된 후에 어떤 처리를 하고 싶을 때가 있습니다. 그러기 위해서는 몇가지 방법이 있는데 그중 2가지 방법은 "시작" 파일과 어플리케이션 이벤트 입니다. 

<a name="start-files"></a>
##시작 파일

어플리케이션의 스타트 파일은 `app/start`에 위치해 있습니다. 기본적으로 `global.php`, `local.php`, 그리고 `artisan.php` 3가지 파일이 포합되어 있습니다. `artisan.php`에 대한 더 많은 정보를 원하면 [Artisan 커맨드 라인](/docs/commands#registering-commands)를 참고하세요.

`global.php` 시작 파일은 [Logger](/docs/errors)의 등록과 `app/filters.php` 파일의 포함과 같은 몇 가지 기본적인 항목을 포함하고 있습니다. 이 파일에 원하는 어떠한 것이라도 추가 해도 좋습니다. 추가된 것들은 환경에 관계없이 어플리케이션의 _모든_ 요청에 포함됩니다. 반면에, `local.php`파일은 어플리케이션이 `local` 환경에서 실행될때만 호출됩니다. 환경에 대한 더 많은 정보를 원하면 [설정](/docs/configuration)를 참고하세요.

물론 `local` 환경 말고 다른 환경이 있을 경우 그 환경에 대한 시작 파일을 만들수 있습니다. 어플리케이션이 해당 환경에서 실행될때 자동으로 포함됩니다.

<a name="application-events"></a>
## 어플리케이션 이벤트

또한, `before`, `after`, `close`, `finish`, 그리고 `shutdown` 어플리케이션 이벤트를 등록하여 전후 요청처리를 할수있습니다.:

**어플리케이션 이벤트 등록**

	App::before(function()
	{
		//
	});

	App::after(function($request, $response)
	{
		//
	});

이 이벤트의 리스너들은 각 요청의 `전`과 `후`에 실행됩니다.
