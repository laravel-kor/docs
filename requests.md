# 요청 & 입력

- [기본 입력](#basic-input)
- [쿠기](#cookies)
- [이전 입력](#old-input)
- [파일](#files)
- [요청 정보](#request-information)

<a name="basic-input"></a>
## 기본 입력

간단한 몇가지 방법으로 사용자의 모든 입력을 액세스 할 수 있습니다. 모든 HTTP 동사(get, post, put, 등..)들이 같은 방법으로 액세스 되므로 요청에 사용되는 HTTP 동사에 신경 쓸 필요가 없습니다.

**입력 값 조회**

	$name = Input::get('name');

**입력 값이 없을 경우 기본값 조회**

	$name = Input::get('name', 'Sally');

**입력 값이 있는지 확인하기**

	if (Input::has('name'))
	{
		//
	}

**모든 요청 입력 값 가져오기**

	$input = Input::all();

**모든 요청 입력의 몇가지만 가져오기**

	$input = Input::only('username', 'password');

	$input = Input::except('credit_card');

Backbone 같은 몇몇의 자바스크립트 라이브러리는 JSON 형태로 어플리케이션에 입력을 보낼지도 모릅니다. 이 데이터들은 평소처럼 `Input::get`을 통해 액세스 할 수 있습니다.

<a name="cookies"></a>
## 쿠키

Laravel 프레임워크에 의해 생성된 쿠키들은 암호화 되고 인증코드와 함께 서명되므로 클라이언트에 의해 변경되었을 경우 유효하지 않은 것으로 간주 됩니다.

**쿠키 값 조회**

	$value = Cookie::get('name');

**응답에 새로운 쿠키 부여하기**

	$response = Response::make('Hello World');

	$response->withCookie(Cookie::make('name', 'value', $minutes));

**영원한 쿠키 만들기**

	$cookie = Cookie::forever('name', 'value');

<a name="old-input"></a>
## 이전 입력

한 요청에서 다른 요청까지 입력 값들을 유지해야 할수도 있습니다. 예를 들어, 양식에 대한 유효성 검사을 한 후 그 양식을 다시 제출 해야 할수도 있습니다.

**세션으로 입력 값 보내기**

	Input::flash();

**입력 값의 몇가지만 세션으로 보내기**

	Input::flashOnly('username', 'email');

	Input::flashExcept('password');

입력 값과 함께 이전 페이지로 리디렉트해야 할 때는 간단하게 입력 값들을 리디렉트에 연결 할수 있습니다.

	return Redirect::to('form')->withInput();

	return Redirect::to('form')->withInput(Input::except('password'));

> **노트:** 세션 클래스를 사용하여 요청을 통해 다른 데이터들을 보낼수 있습니다..

**이전 값 조회**

	Input::old('username');

<a name="files"></a>
## 파일

**업로드된 파일 조회**

	$file = Input::file('photo');

**화일이 업로드 됬는지 확인하기**

	if (Input::hasFile('photo'))
	{
		//
	}

`file` 메소드를 통해 리턴된 오브젝트는 파일과 상호작용 하는데 많은 메소드를 제공하는 PHP의 `SplFileInfo`를 확장한 `Symfony\Component\HttpFoundation\File\UploadedFile` 클래스의 인스턴스 입니다.

**업로드된 파일 이동하기**

	Input::file('photo')->move($destinationPath);

	Input::file('photo')->move($destinationPath, $fileName);

**업로드된 파일의 실제경로 조회**

	$path = Input::file('photo')->getRealPath();

**업로드된 파일의 실제 이름 조회**

	$name = Input::file('photo')->getClientOriginalName();

**업로드된 파일의 확장자 조회**

	$extension = Input::file('photo')->getClientOriginalExtension();

**업로드된 파일의 사이즈 조회**

	$size = Input::file('photo')->getSize();

**업로드된 파일의 MIME 타입 조회**

	$mime = Input::file('photo')->getMimeType();

<a name="request-information"></a>
## 요청 정보

`Symfony\Component\HttpFoundation\Request` 클래스를 확장한 `Request` 클래스는  어플리케이션에 대한 HTTP 요청을 검토하는 많은 메소드를 제공합니다.

**요청 URI 조회**

	$uri = Request::path();

**요청 경로가 패턴과 일치하는지 알아내기**

	if (Request::is('admin/*'))
	{
		//
	}

**요청 URL 조회**

	$url = Request::url();

**요청 URI의 일부분 조회**

	$segment = Request::segment(1);

**요청 헤더 조회**

	$value = Request::header('Content-Type');

**$_SERVER 값 조회**

	$value = Request::server('PATH_INFO');

**AJAX에 의한 요청인지 알아내기**

	if (Request::ajax())
	{
		//
	}

**HTTPS에서 이루어진 요청인지 알아하기**

	if (Request::secure())
	{
		//
	}
