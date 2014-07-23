# 템플릿

- [컨트롤러 레이아웃](#controller-layouts)
- [블레이드 템플릿](#blade-templating)
- [다른 블레이드 컨트롤 구조들](#other-blade-control-structures)

<a name="controller-layouts"></a>
## 컨트롤러 레이아웃

Laravel에서 팀플릿을 사용하는 방법중 하나는 컨트롤러 레이아웃을 통해서 사용하는 방법입니다. 컨트롤러에서 `layout` 프로퍼티를 지정하면, 지정된 뷰가 생성되며 액션에서 반환될 가상적인 응답이 됩니다??.(By specifying the `layout` property on the controller, the view specified will be created for you and will be the assumed response that should be returned from actions.)

**컨트롤러에 레이아웃 정의**

	class UserController extends BaseController {

		/**
		 * The layout that should be used for responses.
		 */
		protected $layout = 'layouts.master';

		/**
		 * Show the user profile.
		 */
		public function showProfile()
		{
			$this->layout->content = View::make('user.profile');
		}

	}

<a name="blade-templating"></a>
## 블레이드 템플릿

블레이드는 Laravel에서 제공하는 간단하면서도 강력한 템플릿 엔진 입니다. 컨트롤러 레이아웃과 달리 블레이드는  _템플릿 상속_ 과 _섹션_ 으로 작동됩니다. 모든 블레이드 템플릿은 `.blade.php` 확장자를 사용해야 합니다.

**블레이드 레이아웃 정의**

	<!-- Stored in app/views/layouts/master.blade.php -->

	<html>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

**블레이드 레이아웃 사용**

	@extends('layouts.master')

	@section('sidebar')
		@parent

		<p>이 섹션은 master sidebar에 덧붙혀집니다.</p>
	@stop

	@section('content')
		<p>이 섹션은 body content 입니다.</p>
	@stop

블레이드 레이아웃을 `extend` 하는 뷰는 간단하게 레이아웃의 섹션들을 치환한다는 것에 주목하십시오. 섹션에서 sidebar나 footer 같은 레이아웃 섹션에 컨텐츠를 추가할 수 있게 해주는 `@parent` 명령어를 사용하여, 자식 뷰에 포함할 수 있습니다.

가끔, 섹션이 정의됐는지 확실하지 않을 경우처럼, `@yield` 명령에 디폴트 값을 전달 할 수 있습니다. 디폴트 값을 두번째 인수에 전달합니다.:

	@yield('section', 'Default Content');

<a name="other-blade-control-structures"></a>
## 다른 블레이드 컨트롤 구조들

**데이터 출력**

	Hello, {{ $name }}.

	The current UNIX timestamp is {{ time() }}.

물론, 모든 사용자 제공 데이터는 이스케이프 또는 정죄 될 수 있습니다. 중괄호 3개로 이루어진 구문를 사용하여 이스케이프한 데이터를 출력할 수 있습니다.:

	Hello, {{{ $name }}}.

> **노트:** 사용자에 의해 제공된 컨텐츠를 프린트 할때 굉장히 주의하길 바랍니다. 항상 중괄호 3개로 이루어진 구문을 사용하여 컨텐츠의 모든 HTML 태그를 이스케이프하세요.

**If 문**

	@if (count($records) === 1)
		I have one record!
	@elseif (count($records) > 1)
		I have multiple records!
	@else
		I don't have any records!
	@endif

	@unless (Auth::check())
		You are not signed in.
	@endunless

**반복문**

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>This is user {{ $user->id }}</p>
	@endforeach

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

**하위 뷰 인클루드**

	@include('view.name')

또한 인클루드된 뷰에 데이터 배열을 전달 할수도 있습니다.:

	@include('view.name', array('some'=>'data'))

**섹션 오버라이팅**

기본적으로, 섹션들은 섹션에 존재하는 이전 컨텐츠에 첨부가 됩니다. 첨부 말고 섹션 전체를 오버라이팅 하려면, `overwrite` 구문을 사용합니다.:

	@extends('list.item.container')

	@section('list.item.content')
		<p>This is an item of type {{ $item->type }}</p>
	@overwrite

**언어 라인 표시**

	@lang('language.line')

	@choice('language.line', 1);

**주석**

	{{-- 이 주석은 렌더링된 HTML에 포함되지 않습니다. --}}
