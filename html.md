# 폼 & HTML

- [폼 오픈](#opening-a-form)
- [CSRF 방지](#csrf-protection)
- [폼 모델 바인딩](#form-model-binding)
- [Labels](#labels)
- [Text, Text Area, Password & Hidden 필드](#text)
- [Checkbox와 Radio 버튼](#checkboxes-and-radio-buttons)
- [파일 입력](#file-input)
- [Drop-Down Lists](#drop-down-lists)
- [버튼](#buttons)
- [사용자 정의 매크로](#custom-macros)

<a name="opening-a-form"></a>
## 폼 오픈

**폼 오픈**

	{{ Form::open(array('url' => 'foo/bar')) }}
		//
	{{ Form::close() }}

기본적인 메소드는 `POST`로 간주 됩니다. 그러나, 다른 메소드도 지정 할 수 있습니다.:

	echo Form::open(array('url' => 'foo/bar', 'method' => 'put'))

> **노트:** HTML 폼이 오직 `POST`와 `GET`만 지원하므로, `PUT`과 `DELETE` 메소드는 폼에 자동으로 `_method` 히든 필드를 추가함으로써 그 기능을 패러디 합니다.

또한 명칭이 붙여진 라우트나 컨트롤러 액션을 향하도록 폼을 오픈 할 수도 있습니다.:

	echo Form::open(array('route' => 'route.name'))

	echo Form::open(array('action' => 'Controller@method'))

매개변수 또한 라우트에 전달 할 수 있습니다.:

	echo Form::open(array('route' => array('route.name', $user->id)))

	echo Form::open(array('action' => array('Controller@method', $user->id)))

만약 폼이 파일 업로드를 허가 한다면 배열에 `files` 옵션을 추가합니다.:

	echo Form::open(array('url' => 'foo/bar', 'files' => true))

<a name="csrf-protection"></a>
## CSRF 방지

Laravel은 크로스 사이트 요청 위조로부터 어플리케이션을 보호 할 수 있는 쉬운 방법을 제공 합니다. 먼저, 랜덤 토큰이 사용자의 세션에 저장됩니다. 자동으로 저장되니 걱정하지 않아도 됩니다. CSRF 토큰은 open 메소드를 사용할 경우 히든 필드로 자동으로 폼에 추가됩니다. 하지만, `token` 메소드를 사용하여 직접 HTML을 생성할수도 있습니다.:

**폼에 SCRF 토큰 추가**

	echo Form::token();

**라우트에 CSRF 필터 부여**

	Route::post('profile', array('before' => 'csrf', function()
	{
		//
	}));

<a name="form-model-binding"></a>
## 폼 모델 바인딩

종종, 모델의 컨텐츠를 기반으로 폼을 채우길 원할 때도 있습니다. 이렇게 하려면, `Form::model` 메소드를 사용합니다.:

**모델 폼 오픈**

	echo Form::model($user, array('route' => array('user.update', $user->id)))

이제, 텍스트 입력 같은 폼 엘레먼트를 생성할 때, 필드 명과 일치하는 모델의 값이 자동으로 필드 값으로 설정됩니다. 예를 들어, `email` 이란 필드 명을 가진 텍스트 입력은 유저 모델의 `email` 속성 값이 필드 값으로 설정됩니다. 그러나, 그것 말고 또 있습니다! 만약 세션의 플래시 데이터에 입력 필드 명과 일치하는 아이템이 있다면, 모델의 값보다 우선 순위가 높아집니다. 그래서, 우선 순위는 아래와 같습니다.:

1. Session Flash Data (Old Input)
2. Explicitly Passed Value
3. Model Attribute Data

이것은 모델 값을 바인딩 해줄 뿐만 아니라 서버에서 검증 에러가 발생 할 경우, 쉽게 폼 값을 다시 채워줄 수 있는 폼을 빠르게 작성 할수 있게 해줍니다!

> **노트:** `Form::model`을 사용 할 경우, `Form::close`를 사용하여 폼을 닫는 것을 잊지마십시오!

<a name="labels"></a>
## Labels

**Label 엘레먼트 생성**

	echo Form::label('email', 'E-Mail Address');

**추가 HTML 속성 부여**

	echo Form::label('email', 'E-Mail Address', array('class' => 'awesome'));

> **노트:** label을 생성하고 난 후, label 명과 일치하는 폼 엘레먼트를 생성 하면 자동으로 label 명과 일치하는 ID를 갖게 됩니다.

<a name="text"></a>
## Text, Text Area, Password & Hidden Fields

**텍스트 입력 생성**

	echo Form::text('username');

**기본값 부여**

	echo Form::text('email', 'example@gmail.com');

> **노트:** *hidden*과 *textarea* 메소드는 *text*와 같은 특징을 갖고 있습니다.

**패스워드 입력 생성**

	echo Form::password('password');

**다른 종류의 입력 생성**

	echo Form::email($name, $value = null, $attributes = array());
	echo Form::file($name, $attributes = array());

<a name="checkboxes-and-radio-buttons"></a>
## Checkbox와 Radio 버튼

**Checkbox 또는 Radio 입력 생성**

	echo Form::checkbox('name', 'value');
	
	echo Form::radio('name', 'value');

**체크된 Checkbox 또는 Radio 입력 생성**

	echo Form::checkbox('name', 'value', true);
	
	echo Form::radio('name', 'value', true);

<a name="file-input"></a>
## 파일 입력

**파일 입력 생성**

	echo Form::file('image');

<a name="drop-down-lists"></a>
## Drop-Down 리스트

**A Drop-Down 리스트 생성**

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'));

**기본으로 선택되어 있는 Drop-Down 리스트 생성**

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'), 'S');

**그룹 리스트 생성**

	echo Form::select('animal', array(
		'Cats' => array('leopard' => 'Leopard'),
		'Dogs' => array('spaniel' => 'Spaniel'),
	));

<a name="buttons"></a>
## 버튼

**Submit 버튼 생성**

	echo Form::submit('Click Me!');

> **노트:** 버튼 엘레먼트를 생성해야 합니까? *button* 메소드를 사용하십시오. *submit*과 같은 특징을 갖고 있습니다.

<a name="custom-macros"></a>
## 사용자 정의 매크로

"매크로"를 호출하여 사용자 정의 폼 클래스 헬퍼를 쉽게 정의 할 수 있습니다. 이렇게 하면 됩니다. 먼저 간단히, 주어질 이름과 클로저를 사용하여 매크로를 등록합니다.:

**폼 매크로 등록**

	Form::macro('myField', function()
	{
		return '<input type="awesome">';
	});

이제 그 이름을 사용해 매크로를 호출 할 수 있습니다.:

**사용자 정의 폼 매크로 호출**

	echo Form::myField();
