# 지역화

- [소개](#introduction)
- [언어 파일](#language-files)
- [기본적인 사용법](#basic-usage)
- [복수화](#pluralization)

<a name="introduction"></a>
## 소개

Laravel의 `Lang` 클래스는 어플리케이션에서 여러 언어를 쉽게 제공할수 있도록, 다양한 언어에서 문자열을 조회할 수 있는 편리한 방법을 제공합니다.

<a name="language-files"></a>
## 언어 파일

언어 문자열은 `app/lang` 디렉토리 안에 저장되어 있습니다. 이 디렉토리 안에는 어플리케이션에 제공되는 각각의 언어가 서브디렉토리로 저장되어야 합니다.

    /app
  		/lang
  			/en
  				messages.php
  			/es
  				messages.php

언어 파일은 단순히 키 문자열의 배열을 반환합니다. 예를 들면:

**언어파일 예제**

	<?php

	return array(
		'welcome' => 'Welcome to our application'
	);

어플리케이션의 디폴트 언어는 `app/config/app.php` 설정 파일에 설정되어 있습니다. `App::setLocale` 메소드를 사용하여 언제든지 유효한 언어를 바꿀 수 있습니다.:

**런타임에서 디폴트 언어 변경**

	App::setLocale('es');

<a name="basic-usage"></a>
## 기본적인 사용법

**언어 파일에서 라인 조회**

	echo Lang::get('messages.welcome');

`get` 메소드에 전달된 첫번째 세그먼트는 언어 파일의 파일명이며, 두번째는 조회 할 라인 이름입니다.

> **Note**: 언어 라인이 존재하지 않는다면, 키값이 `get` 메소드로부터 반환됩니다.

또한 `Lang::get` 메소드의 별칭인 `trans` 헬퍼 함수를 사용 할 수도 있습니다.

	echo trans('messages.welcome');

**대체라인 생성**

또한 언어 라인에서 플레이스 홀더를 정의할 수 있습니다.:

	'welcome' => 'Welcome, :name',

그런다음, `Lang::get` 메소드의 두번째 인수에 대체할 값을 절달하면 됩니다.:

	echo Lang::get('messages.welcome', array('name' => 'Dayle'));

**언어 파일에 해당 라인을 포함되어 있는지 확인**

	if (Lang::has('messages.welcome'))
	{
		//
	}

<a name="pluralization"></a>
## 복수화

복수화는 언어들마다 다양하고 복잡한 규칙을 가지고 있으므로 복잡합니다. 언어파일에서는 쉽게 이 문제를 다룰 수 있습니다. "파이프" 문자를 사용하여 문자열의 단수와 복수를 구분할 수 있습니다.:

	'apples' => 'There is one apple|There are many apples',

그런 다음 `Lang::choice` 메소드를 사용하여 라인을 조회할 수 있습니다.:

	echo Lang::choice('messages.apples', 10);

Laravel translator는 Symfony Translation component에 의해 제공되기 때문에, 구체적인 복수화 규칙을 쉽게 만들 수 있습니다.:

	'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',
